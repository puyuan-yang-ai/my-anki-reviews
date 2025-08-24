太好了！你已经能进容器了，接下来“配环境”这步说白了就是——让容器里具备你项目运行所需的一切依赖（Python 包 + 必要的系统包 + 模型缓存/配置）。Stable Diffusion 项目常见有两种主流做法：**临时容器里 pip 安装** 和 **自定义镜像（Dockerfile）固化依赖**。我给你梳理清楚并给出可直接用的模板。

---

# 主流做法对比

## A) 直接在容器里 `pip install -r requirements.txt`（上手最快）

**场景**：你在开发迭代、依赖会改动、先跑通再说
**步骤**：

1. 启动容器（已挂载项目根，进入 `/workspace`）
2. 容器里执行：

   ```bash
   python3 -m pip install --upgrade pip
   pip install -r requirements.txt
   ```
3. 运行你的脚本/入口：

   ```bash
   python3 launch.py   # 举例
   ```

**优点**：立刻可用，改 requirements 立刻生效
**缺点**：每次新容器都要重新安装，慢；可复现性弱

> 小技巧：把 **pip 缓存** 和 **HuggingFace 缓存**挂出来，安装和下载模型会快很多
> 在 `docker run` 里加：
>
> ```bash
> -v ~/.cache/pip:/home/$(id -un)/.cache/pip \
> -v ~/.cache/huggingface:/home/$(id -un)/.cache/huggingface \
> -e HF_HOME=/home/$(id -un)/.cache/huggingface
> ```

---

## B) 写一个 Dockerfile，把依赖“烤进镜像”（可复现、启动快）

**场景**：团队协作、要交付/上线、强调可复现
**步骤**：

1. 在项目根新建 `Dockerfile`（基于你现在的 ROCm PyTorch 镜像）
2. 构建镜像 `docker build -t my-sd:rocm .`
3. 以后启动就直接用这个镜像，几乎“秒起”，无需再 pip 装

**优点**：一次安装、处处复用；环境固定、出问题好排查
**缺点**：改依赖要重构镜像

---

# 给你的“稳定扩散”最小清单（AMD/ROCm）

Stable Diffusion 常用到的 Python 依赖（示例，按 Diffusers 路线）：

* `torch`（你基底镜像已经带了 ROCm 版）
* `torchvision`（版本要与 torch 匹配）
* `diffusers`, `transformers`, `accelerate`, `safetensors`
* 可能还会用到：`xformers`（**CUDA 友好，ROCm 下不稳定/常不可用**，一般不装），`opencv-python`, `Pillow`, `scipy`, `einops`, `tqdm`, `pyyaml`, `huggingface_hub`

系统工具（有时需要）：

* `git`, `ffmpeg`, `libgl1`（OpenCV 无头渲染常用）

---

# 方案 A：你现在就能跑的“临时容器安装”步骤

1. 启动容器（你已有类似脚本，略）。建议再挂缓存：

```bash
docker run --rm -it \
  --name sd-dev \
  -u $(id -u):$(id -g) \
  --device=/dev/kfd --device=/dev/dri --group-add video \
  --shm-size=16g \
  -e PYTHONUNBUFFERED=1 \
  -e HF_HOME=/workspace/.cache/huggingface \
  -v /group/.../agent_benchmark_clean:/workspace \
  -v /group/.../agent_benchmark_clean/.cache/pip:/home/$(id -un)/.cache/pip \
  -v /group/.../agent_benchmark_clean/.cache/huggingface:/workspace/.cache/huggingface \
  -w /workspace \
  rocm/pytorch:rocm6.3.2_ubuntu22.04_py3.10_pytorch_release_2.5.1_preview
```

2. 容器内安装依赖：

```bash
python3 -m pip install --upgrade pip
# 先装与 torch 匹配的 torchvision（举例：2.5 对应 torchvision 0.20）
pip install torchvision==0.20.* --extra-index-url https://download.pytorch.org/whl/rocm6.3

# 再装项目依赖
pip install -r requirements.txt
# 如果你没有 requirements.txt，可先试用 diffusers/accelerate 路线：
# pip install diffusers[torch] transformers accelerate safetensors opencv-python Pillow scipy einops tqdm pyyaml huggingface_hub
```

3. 首次运行建议验证 GPU（ROCm）可见：

```bash
python3 - <<'PY'
import torch
print("torch:", torch.__version__)
print("HIP:", getattr(torch.version, "hip", None))
print("cuda.is_available():", torch.cuda.is_available())
print("device_count:", torch.cuda.device_count())
if torch.cuda.is_available():
    for i in range(torch.cuda.device_count()):
        print(i, torch.cuda.get_device_name(i))
PY
```

4. 跑你的入口脚本（示例）：

```bash
python3 scripts/txt2img.py --prompt "a cute cat" --steps 20
```

> 注意：**xformers** 在 ROCm 下通常不可用或不稳定，建议先不用它；Diffusers 自带的 attention 实现即可。

---

# 方案 B：推荐的 Dockerfile（把依赖烤进镜像）

在项目根写 `Dockerfile`：

```dockerfile
FROM rocm/pytorch:rocm6.3.2_ubuntu22.04_py3.10_pytorch_release_2.5.1_preview

# 基础工具（按需）
RUN apt-get update && apt-get install -y --no-install-recommends \
    git ffmpeg libgl1 ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# 避免缓冲，日志实时
ENV PYTHONUNBUFFERED=1

# 先装与 torch 匹配的 torchvision（示例版本按 torch 而定）
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir torchvision==0.20.* --extra-index-url https://download.pytorch.org/whl/rocm6.3

# 把项目依赖写入镜像（如果有 requirements.txt）
WORKDIR /workspace
COPY requirements.txt /workspace/requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# 也可以把项目代码 COPY 进镜像（看你是否希望镜像内带代码）
# COPY . /workspace

# 默认启动 bash（开发用）；上线可改为 python 入口
CMD ["/bin/bash"]
```

构建与运行：

```bash
docker build -t my-sd:rocm .
docker run --rm -it \
  --name sd-dev \
  -u $(id -u):$(id -g) \
  --device=/dev/kfd --device=/dev/dri --group-add video \
  --shm-size=16g \
  -v /group/.../agent_benchmark_clean:/workspace \
  -w /workspace \
  my-sd:rocm
```

---

# 常见坑 & 建议

* **torch/torchvision 版本要匹配**：用 `pip show torch torchvision` 看版本，尽量按官方对应表来。
* **ROCm 下不要强求 xformers**：很多情况下没有可用的轮子或编译困难，先跑通再优化。
* **只读数据集**：若数据在项目外，建议 `-v /data/dataset:/data/dataset:ro` 防止误改。
* **大共享内存**：`--shm-size=16g`（或更大）能避免 DataLoader 多进程报错。
* **GPU 权限与 -u 的关系**：若加 `-u` 后 GPU 不可见，要按我们前面说的用 **数字 GID** 的 `--group-add` 补齐 `/dev/kfd` 与 `/dev/dri/renderD*` 的组权限。
* **HuggingFace Token**：需要私有模型时设置环境变量 `HF_TOKEN` 或 `huggingface-cli login`（把缓存目录挂出来可复用登录）。

---

# 结论（给你一条路线）

* 想**先跑通**：用 **方案 A**，容器内 `pip install -r requirements.txt`，把缓存挂出来。
* 想**长期稳定/交付**：迁移到 **方案 B**，用 Dockerfile 固化依赖，一次安装到处用。

你把当前项目的 `requirements.txt`（或依赖列表）贴我，我可以帮你给出**适配 ROCm 的精简/固定版本**，并一起检查 `torch/torchvision` 的版本匹配。
