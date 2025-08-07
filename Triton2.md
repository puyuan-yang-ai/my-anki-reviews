
好的，我理解您的需求了。我将用自己的话复述一遍，以便我们达成共识。

您希望扩展 `classify_gaagent_output.py` 这个脚本的功能。

**目前脚本的功能是：**
读取一个大的JSON文件（如 `data/20250709_gaagent_hip_mem_15.json`），分析其中每个文件（以文件名作为key）的执行结果，并将其归类为“编译失败”、“执行失败”、“性能不达标”或“完全成功”等类别，最后生成一个统计摘要。

**您希望增加的新功能是：**
1.  **引入新的数据源**：脚本需要额外读取一个 `.jsonl` 文件（如 `20250709_gaagent_hip_15.jsonl`）。
2.  **数据关联与拆分**：利用 `.jsonl` 文件中每个JSON对象里的 `filename` 字段，与之前从大JSON文件中分析出的文件分类结果进行匹配。
3.  **生成分类后的文件**：根据匹配上的分类结果，将原始 `.jsonl` 文件中的内容拆分到三个新的 `.jsonl` 文件中。文件名将基于原始 `.jsonl` 文件名生成，例如：
    *   `20250709_gaagent_hip_15_compilation_failure.jsonl`（包含所有编译失败的文件所对应的JSON对象）
    *   `20250709_gaagent_hip_15_execution_failure.jsonl`（包含所有执行失败的文件所对应的JSON对象）
    *   `20250709_gaagent_hip_15_success.jsonl`（包含所有完全成功的文件所对应的JSON对象）
4.  **精简内容**：在生成的新 `.jsonl` 文件中，每个JSON对象只保留 `label`、`filename` 和 `test_code` 这三个字段。

我注意到原来的脚本有四个分类，但您只提到了生成三个新的 `.jsonl` 文件。关于“性能不达标”（Performance Failure）这一类别，我的理解是这个类别的数据将被忽略，不会为它生成对应的 `.jsonl` 文件。

请问我的理解是否准确？
