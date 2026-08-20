---
title: 如何使用人工智能准确提取文档中的所有信息
author: Umair Ali Khan, Ph.D.
url: https://ai.gopubby.com/how-to-accurately-extract-everything-from-documents-using-ai-cf12d0125238
translated: 2026-06-11
excerpt: 要求人工智能模型以特定格式输出会改变一切
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/01.thumb.webp
---

# 如何使用人工智能准确提取文档中的所有信息

要求人工智能模型以特定格式输出会改变一切

从文档中准确提取内容对于人工智能应用至关重要。例如，在使用人工智能处理采购订单以生成销售草案时，人工智能代理或 LLM 首先需要准确理解采购订单的内容及其确切的布局。

表格标题错位会导致 LLM 读取错误的数量，从而导致价格计算错误。同样，由于跨页表格分隔符导致漏掉一个行项目，也可能导致 LLM 完全跳过某个产品。

我们在**生成式人工智能项目**（[**GAIK**](https://gaik.ai/)）中经常看到这种情况，我们使用**开源生成式人工智能工具包**（参见 GAIK 的 GitHub 存储库[**此处**](https://github.com/GAIK-project/gaik-toolkit/tree/main)）构建了使用企业文档的人工智能应用程序。

许多客户文件，例如采购订单或物料清单，都包含杂乱的表格。例如，请参阅以下两页采购订单（出于隐私原因，内容已更改，但布局保持不变）。

表格标题和行数据之间有一段文字。当然，这并非添加这段文字的正确位置。它本可以添加到表格之前或之后的其他位置。但客户文档中出现这种结构是很常见的。此外，这段文字在同一表格的其他地方也出现了不必要的重复。

表格跨越多页，部分行数据跨页显示。例如，第一页包含第三项（030）的部分数据，之后是表格页脚，下一页则是页眉和标题（重复显示）。

诸如 **PyMuPDF、PyMuPDF4LLM** 或 **Docling** 之类的专用解析器无法准确提取此表格结构。

我在以下文章中探讨了这个问题：

即使是这两个解析器也无法正确提取这种混乱的表格布局。请参见以下提取截图。

![Docling+LLM 解析器提取的表格部分视图（作者提供图片）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/01.webp)

最终的解决方法比我想象的要简单得多。

关键在于使用正确的提示语。这只是向人工智能模型询问你需求的一种不同方式。

本月初，[LlamaIndex](https://www.llamaindex.ai/) 发布了他们在[基准研究](https://arxiv.org/abs/2604.08538) 中使用的文档解析提示，这些提示改变了一切。

证据如下。同一份采购订单，我们尝试过的所有解析器都无法解析：

![这是使用 GAIK 的多模态解析器，结合 gemini-3.1-flash-lite-preview 模型，并将推理难度设置为低，提取出的采购订单表的部分视图（已转换为 HTML 以提高可读性）（图片由作者提供）。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/02.webp)

> 在本文中，我们将创建一个多模态解析器，以准确地从布局混乱的文档中提取/解析内容。

多模态解析器允许用户选择提供商和模型（**OpenAI、Anthropic 或 Google**），并提供其他几个选项来评估其用例的解析质量。

多模态解析器的源代码以及详细的指南和文档可在 GAIK 工具包的 GitHub 代码库中找到，链接如下：[**multimodal\_parser**](https://github.com/GAIK-project/gaik-toolkit/tree/main/implementation_layer/src/gaik/software_components/parsers/multimodal_parser)。它需要至少一个提供商（**OpenAI、Azure 或 Google Vertex AI**）的 API 凭据。

## 这一切都与一些特定的提示有关。

本月初，[LlamaIndex](https://www.llamaindex.ai/) 发布了一个开源框架（[ParseBench](https://github.com/run-llama/ParseBench)），用于测试文档解析器将 PDF 转换为 AI 代理实际可用的输出的性能。他们测试了 14 种不同的方法：通用视觉 LLM、专用解析器以及他们自己的 LlamaParse。

他们的基准测试结果发表在以下研究中：

这些结果表明，代理解析（例如他们自己的 [LlamaParse](https://developers.llamaindex.ai/llamaparse/)）和基于 LLM 的解析（例如使用 Gemini 3 Flash）优于专门的文档解析器。

![基于代理和 VLM 的解析器性能优于专用解析器。图片来源：https://www.llamaindex.ai/blog/parsebench](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/03.webp)

我很好奇他们是如何让基于 LLM 的解析方法如此有效，所以我仔细阅读了那篇研究论文。

我原本期待一些巧妙的架构设计，比如一个精细调优的模型，或者一个带有验证循环的多步骤流水线。

相反，我找到了一些具体的提示。一些我们从未想过要使用的提示。

## 这些提示有什么特别之处？

当您要求 LLM 将表格输出为 Markdown 格式（大多数解析器的默认格式）时，它无法正确保留合并单元格和多级标题的布局。例如，一个包含跨多列两行标题的采购订单会变得扁平且难以理解。

LlamaIndex 提出的提示会提取一个带有 HTML `colspan` 和 `rowspan` 属性的表格，这些属性编码了表格的完整结构。这些提示要求 LLM 将表格转换为 HTML 格式（`<table>`、`<tr>`、`<th>`、`<td>`），并使用 `colspan` 和 `rowspan` 属性来保留合并的单元格和分层标题。

同样地，对于图表，LLM 被要求使用扁平的组合列标题将其转换为表格，以便每个数据单元格的行包含其所有标签。

本文建议对 OpenAI 和 Claude 模型使用共享的系统提示。而对于 Google 模型，用户提示则略有不同。

以下是 LlamaParse 为 OpenAI 和 Claude 模型提出的系统和用户提示，我们将使用这些模型来构建一个可重用的多模态解析器包。

```
OPENAI_CLAUDE_SYSTEM_PROMPT = """You are a document parser. Your task is to convert document PDFs into clean, well-structured Markdown.Guidelines:
- Preserve the document structure, including headings, paragraphs, lists, and tables.
- Convert tables to HTML using `<table>`, `<tr>`, `<th>`, and `<td>`.
- For existing tables in the document, use `colspan` and `rowspan` attributes to preserve merged cells and hierarchical headers.
- For charts or graphs converted into tables, use flat combined column headers (for example, "Primary 2015" instead of separate header rows) so that each data cell's row contains all of its labels.
- Describe images and figures briefly in square brackets, for example: `[Figure: description]`.
- Preserve any code blocks with appropriate syntax highlighting.
- Maintain reading order: left to right, top to bottom for Western documents.
- Do not add commentary or explanations. Output only the parsed content.Additionally, wrap each layout element in a `<div>` tag with:
- `data-bbox="[x1, y1, x2, y2]"` for the bounding box in normalized 0-1000 coordinates, where x is horizontal (left edge = 0, right edge = 1000) and y is vertical (top = 0, bottom = 1000). `x1, y1` is the top-left corner and `x2, y2` is the bottom-right corner.
- `data-label="<category>"` where category is one of: `Caption`, `Footnote`, `Formula`, `List-item`, `Page-footer`, `Page-header`, `Picture`, `Section-header`, `Table`, `Text`, `Title`.Place elements in reading order. Every piece of content must be inside exactly one `<div>` wrapper."""OPENAI_CLAUDE_USER_PROMPT = """The attached PDF is read from the input folder next to this script.Parse the full document and output its content as clean markdown, with each layout element wrapped in a <div data-bbox="[x1,y1,x2,y2]" data-label="Category"> tag. Use HTML tables for any tabular data. For charts and graphs, use flat combined column headers. Output ONLY the parsed content with div wrappers and no explanations.
"""
```

Google 的 Gemini 模型使用相同的提示，唯一的区别是 `data-bbox` 格式更改为它们的原生坐标顺序。

对于 Gemini 型号，系统和用户提示如下：

```
GOOGLE_SYSTEM_PROMPT = """You are a document parser. Your task is to convert document PDFs into clean, well-structured Markdown.Guidelines:
- Preserve the document structure, including headings, paragraphs, lists, and tables.
- Convert tables to HTML using `<table>`, `<tr>`, `<th>`, and `<td>`.
- For existing tables in the document, use `colspan` and `rowspan` attributes to preserve merged cells and hierarchical headers.
- For charts or graphs converted into tables, use flat combined column headers (for example, "Primary 2015" instead of separate header rows) so that each data cell's row contains all of its labels.
- Describe images and figures briefly in square brackets, for example: `[Figure: description]`.
- Preserve any code blocks with appropriate syntax highlighting.
- Maintain reading order: left to right, top to bottom for Western documents.
- Do not add commentary or explanations. Output only the parsed content.Additionally, wrap each layout element in a `<div>` tag with:
- `data-bbox="[y_min, x_min, y_max, x_max]"` for the bounding box in normalized 0-1000 coordinates where x is horizontal (left edge = 0, right edge = 1000) and y is vertical (top = 0, bottom = 1000). The order is `[y_min, x_min, y_max, x_max]`.
- `data-label="<category>"` where category is one of: `Caption`, `Footnote`, `Formula`, `List-item`, `Page-footer`, `Page-header`, `Picture`, `Section-header`, `Table`, `Text`, `Title`.Place elements in reading order. Every piece of content must be inside exactly one `<div>` wrapper."""GOOGLE_USER_PROMPT = """Parse this document page and output its content as clean markdown, with each layout element wrapped in a <div data-bbox="[y_min,x_min,y_max,x_max]" data-label="Category"> tag.
Use HTML tables for any tabular data. For charts/graphs, use flat combined column headers. Output ONLY the parsed content with div wrappers, no explanations.
"""
```

> 提示信息要求使用带有 `colspan` 和 `rowspan` 属性的 HTML 表格，而不是无法准确表示合并单元格的 Markdown 表格和杂乱的实际表格。每个布局元素都会获得一个以规范化坐标表示的边界框。这意味着下游代码无需依赖原始 PDF 渲染器即可重建页面的空间布局。

Google 提示使用不同的边界框坐标顺序 (\[`y_min, x_min, y_max, x_max`\])，因为这与 Gemini 模型原生生成的顺序相匹配。

解析提示在 GitHub 的 `prompts.py` 脚本中定义。

## 用于精确解析的多模态解析器

我使用上述提示将解析管道打包成一个 Python 类，该类读取 PDF 文档，并允许用户选择提供商和模型（**OpenAI、Anthropic 或 Google**）以及其他几个选项。

完整的代码结构位于 [**GitHub 仓库**](https://github.com/GAIK-project/gaik-toolkit/tree/main/implementation_layer/src/gaik/software_components/parsers/multimodal_parser)。`MultimodalParser` 类的完整实现位于 `[multimodal_parser.py](https://github.com/GAIK-project/gaik-toolkit/blob/main/implementation_layer/src/gaik/software_components/parsers/multimodal_parser/multimodal_parser.py)` 中。

`MultimodalParser` 只有一个方法，即 `parse()`，它负责协调整个管道。

```
parser = MultimodalParser(
    model_provider="google",
    model="gemini-3.1-flash-lite-preview",
    reasoning_effort="low",
    merge_table=True,
    create_html=True,
)
result = parser.parse("document.pdf")
```

几个值得了解的参数：

- `reasoning_effort` — 模型的思考预算：`"low"`、`"medium"` 或 `"high"`。更高的思考预算可以提高复杂布局的准确率，但速度更慢，成本更高。
- `merge_table` — 当为 `True` 时，向用户提示添加一条指令，告诉模型合并跨页拆分的表格。
- `additional_instructions` — 附加到用户提示的域特定规则的附加说明。
- `create_html` — 当为 `True` 时，将清理后的 markdown 渲染成 HTML 文档。

`parse()` 方法首先将 PDF 读取为原始字节，并将其编码为 base64 字符串，以便将其嵌入到 JSON API 有效负载中，并连同上述提供商特定的提示一起发送到选定的 LLM。

然后，它会根据所选提供商构建正确的消息结构，因为 OpenAI、Claude 和 Google 各自期望其内容的格式不同。

调用 LLM API 后，模型会返回其原始响应，该响应是 markdown 文本和 `<div>` 包装器的组合，这些包装器包含页面上每个布局元素的边界框坐标。清理过程会移除模型可能用于包装输出的任何代码块围栏。另一种方法以简洁的 Markdown 形式从这些 `<div>` 包装器中提取实际内容。

如果选择 `create_html=True` 选项，则会将干净的 Markdown 转换为样式化的 HTML 文档，您可以直接在浏览器中打开该文档以直观地检查提取内容。

**每次调用都会返回一个 `UsageRecord` 对象，其中包含已解析的内容以及 token 消耗的详细信息和相关成本。**

![多模态解析器的流程图（图片由作者提供）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/04.webp)

## 如何使用它

多模态解析器是我们[GAIK 项目开源生成式 AI 工具包](https://github.com/GAIK-project/gaik-toolkit/tree/main)中的一个可重用软件组件。该软件组件可以作为独立的 Python 包进行安装，具体步骤如下：

```
pip install "gaik[multimodal-parser]"
```

以下是一个使用 `gemini-3.1-flash-lite-preview` 解析同一份采购订单的极简示例，解析过程简单易懂。该示例将解析结果保存为原始 Markdown、精简 Markdown 和 HTML 格式。

有关 token 消耗和价格计算的详细示例，请参阅[此链接](https://github.com/GAIK-project/gaik-toolkit/blob/main/implementation_layer/examples/software_components/parsers/demo_multimodal_parser.py)。

```python
from pathlib import Path
from dotenv import load_dotenv
from gaik.software_components.parsers.multimodal_parser import MultimodalParserload_dotenv()OUTPUT_DIR = Path(__file__).parent / "output"def save_result(result, prefix: str) -> None:
    """Save parse results to the output directory."""
    OUTPUT_DIR.mkdir(parents=True, exist_ok=True)    raw_path = OUTPUT_DIR / f"{prefix}_raw.md"
    raw_path.write_text(result.raw_markdown, encoding="utf-8")    clean_path = OUTPUT_DIR / f"{prefix}_clean.md"
    clean_path.write_text(result.clean_markdown, encoding="utf-8")    print(f"Saved: {raw_path}")
    print(f"Saved: {clean_path}")    if result.html is not None:
        html_path = OUTPUT_DIR / f"{prefix}_clean.html"
        html_path.write_text(result.html, encoding="utf-8")
        print(f"Saved: {html_path}")parser = MultimodalParser(
    model_provider="google",
    model="gemini-3.1-flash-lite-preview",
    reasoning_effort="low",
    merge_table=True,
    create_html=True,
)
result = parser.parse("sample_PO.pdf")
save_result(result, "output_google")
```

上面显示的是同一采购订单的输出结果，已正确合并和结构化（HTML 格式）。

以下是一些输入输出示例。

![一张布局复杂的表格（图片由作者提供）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/05.webp)

![由 gemini-3.1-flash-lite-preview 解析的表格，推理难度为“低”（图片由作者提供）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/06.webp)

![示例文档，包含两张图表（图片由作者提供）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/07.webp)

![由 gemini-3.1-flash-lite-preview 解析的两张图表，推理难度为“低”（图片由作者提供）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@f1fc9ed0cb6f99b00a26bd6d8e3cc60928827d2f/ai-insights/2026-06/11/images/how-to-accurately-extract-everything-from-documents-using-ai/08.webp)

## 观察与总结

即使是像 `gemini-3.1-flash-lite-preview` 和 `gpt-5.4-mini` 这样较小的模型，在将 `reasoning_effort` 设置为低的情况下，也能准确地提取出杂乱的采购订单。关键在于使用了正确的提示。

需要注意的是，这种解析方法适用于准确性比速度更重要，且无需快速或实时响应的场景。例如，在采购订单处理中，准确的解析对于精确的价格计算比速度更重要。

提高推理难度会显著增加处理时间和成本。对于大型模型，例如推理难度设置为“高”的 `gpt-5.4`，速度会变得非常慢。因此，对于大型模型而言，“高”推理难度尤其应该仅用于布局极其复杂的文档。

就成本和速度而言，`gemini-3.1-flash-lite-preview` 是最佳选择。

`additional_instructions` 参数可用于提供特定用例的解析指令。例如，对于法律文档，可以将其设置为“*完全保留源文件中出现的脚注编号*”。

*我很想知道您在文档解析方面遇到了哪些问题。这些问题能否用这种解析方法解决？*
