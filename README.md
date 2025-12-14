# 让 LLM 高效支持文件传入

## 介绍

这是一个增强版的AstrBot文件读取插件，基于原先的`astrbot_plugin_file_reader`改进而来。与原版不同，pro版采用了更高效的基于嵌入模型的内容处理方式，替代了直接将文件内容注入prompt的低效方式。

## 核心特性

### 1. 高效的嵌入模型集成
- 使用AstrBot内置的`embedding_provider`生成文本向量
- 替代直接将文件内容注入prompt的低效方式
- 支持重排序（`RerankProvider`）提升检索准确性

### 2. 智能分块与向量存储
- **递归字符分块**：`RecursiveCharacterChunker`按语义分割文本
- **可配置参数**：支持自定义块大小（`chunk_size`）和重叠度（`chunk_overlap`）
- **高效存储**：基于FAISS的`FaissVecDB`实现向量存储与管理

### 3. 多文件共存支持
- 支持同一对话中上传多个文件
- 基于`session_id`和`conversation_id`实现文件隔离存储
- 将在文件有效期内检索当前对话(conversation_id)中上传的文件

### 4. 智能文件生命周期管理
- **时间有效期**：默认60分钟，可配置
- **最大使用轮数**：默认5轮，可配置
- 任一条件满足即自动清理过期文件

### 5. 用户命令
- 支持手动文件清理命令：`/clear_file`、`/clean_file`
- 所有清理命令统一清理当前用户 `session_id` 的所有文件

### 6. 灵活的配置选项
- 通过`_conf_schema.json`提供可配置参数
- 支持自定义分块大小、文件有效期、最大使用轮数等
- 支持配置文件大小限制

## 安装要求

1. 安装依赖库：
```bash
pip install -r requirements.txt
```

2. Linux用户可能需要安装`libmagic`：
```bash
sudo apt-get install libmagic1
```

## 使用方法

### 1. 上传文件
- 在AstrBot中直接上传文件
- 支持多种文件格式（见下方支持列表）
- 上传后文件会自动进行分块和嵌入处理

### 2. 查询文件内容
- 上传文件后，直接提问关于文件内容的问题
- 插件会自动检索相关内容并添加到prompt中

### 3. 清理文件
支持以下清理命令，所有命令都会清理当前用户的所有文件：
- `/clear_file`
- `/clean_file`

## 支持的文件格式

```python
SUPPORTED_EXTENSIONS: Dict[str, str] = {
    # 文档格式
    "pdf": "read_pdf_to_text",
    "docx": "read_docx_to_text",
    "doc": "read_docx_to_text",
    "rtf": "read_txt_to_text",
    "odt": "read_txt_to_text",

    # 电子表格
    "xlsx": "read_excel_to_text",
    "xls": "read_excel_to_text",
    "ods": "read_excel_to_text",
    "csv": "read_csv_to_text",

    # 演示文稿
    "pptx": "read_pptx_to_text",
    "ppt": "read_pptx_to_text",
    "odp": "read_pptx_to_text",

    # 编程语言源代码
    "py": "read_txt_to_text",
    "java": "read_txt_to_text",
    "cpp": "read_txt_to_text",
    "c": "read_txt_to_text",
    "h": "read_txt_to_text",
    "hpp": "read_txt_to_text",
    "cs": "read_txt_to_text",
    "js": "read_txt_to_text",
    "ts": "read_txt_to_text",
    "php": "read_txt_to_text",
    "rb": "read_txt_to_text",
    "go": "read_txt_to_text",
    "rs": "read_txt_to_text",
    "swift": "read_txt_to_text",
    "kt": "read_txt_to_text",
    "scala": "read_txt_to_text",
    "sh": "read_txt_to_text",
    "bash": "read_txt_to_text",
    "ps1": "read_txt_to_text",
    "bat": "read_txt_to_text",
    "cmd": "read_txt_to_text",
    "vbs": "read_txt_to_text",

    # 标记语言
    "html": "read_txt_to_text",
    "htm": "read_txt_to_text",
    "xml": "read_txt_to_text",
    "json": "read_txt_to_text",
    "yaml": "read_txt_to_text",
    "yml": "read_txt_to_text",
    "md": "read_txt_to_text",
    "markdown": "read_txt_to_text",

    # 配置文件
    "ini": "read_txt_to_text",
    "cfg": "read_txt_to_text",
    "conf": "read_txt_to_text",
    "properties": "read_txt_to_text",
    "env": "read_txt_to_text",

    # 数据库/查询
    "sql": "read_txt_to_text",

    # 其他文本格式
    "txt": "read_txt_to_text",
    "log": "read_txt_to_text",
    "": "read_txt_to_text",  # 无扩展名文件

    # 构建/项目文件
    "toml": "read_txt_to_text",
    "lock": "read_txt_to_text",
    "gitignore": "read_txt_to_text",

    # 网络相关
    "url": "read_txt_to_text",
    "webloc": "read_txt_to_text",
}
```

## 配置说明

插件支持通过`_conf_schema.json`配置以下参数：

- `chunk_size`: 文件内容分块大小（默认512）
- `chunk_overlap`: 相邻块之间的重叠大小（默认100）
- `retrieve_top_k`: 最终检索返回的相关块数量（默认5）
- `fetch_k`: 重排序前检索的相关块数量（默认20）
- `enable_rerank`: 是否启用结果重排序（默认true）
- `file_retention_time`: 文件嵌入的有效期（分钟，默认60）
- `file_max_rounds`: 文件嵌入的最大使用轮数（默认5）
- `max_file_size`: 支持的最大文件大小（MB，默认100）
- `embedding_provider_id`: 嵌入服务提供商ID（默认使用AstrBot配置的第一个嵌入提供商）
- `rerank_provider_id`: 重排序模型提供商ID（默认使用AstrBot配置的第一个重排序提供商）

## 版本历史

### v2.0.0
- 新增基于嵌入模型的内容处理方式
- 实现智能分块与向量存储
- 支持多文件共存
- 提供灵活的配置选项
- 实现标准用户命令
- 改进文件生命周期管理

### v1.0.2
- 使用了get_file()

### v1.0.1
- 支持更多文件后缀名

## 技术原理

处理流程：
1. **文件解析**：读取并解析各种格式的文件
2. **智能分块**：使用`RecursiveCharacterChunker`按语义分割文本
3. **生成嵌入**：通过`embedding_provider`生成文本向量
4. **向量存储**：将向量存储到`FaissVecDB`中
5. **高效检索**：基于向量相似度检索相关内容
6. **上下文组装**：将检索结果组装为上下文添加到prompt中

## 注意事项

- 文件处理会消耗一定的计算资源，请合理配置分块大小和文件大小限制
- 过期文件会自动清理，无需手动干预
- 切换对话时文件会暂时失效，但不会立即清理，用户可以随时切换回来使用