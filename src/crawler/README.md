# 爬虫模块

`src/crawler` 模块提供了一个完整的网页抓取解决方案，用于从网页中提取干净、可读的内容，并将其转换为适合AI/LLM处理的格式。

## 概述

该模块协调多个组件来完成：
- 使用Jina Reader API获取网页内容
- 使用可读性算法提取干净的文章内容
- 将HTML转换为Markdown格式
- 格式化内容供LLM使用，支持多模态处理

## 组件说明

### Crawler
主入口点，协调整个抓取过程。

**位置**: `crawler.py`

```python
from src.crawler import Crawler

crawler = Crawler()
article = crawler.crawl(url)
```

### Article
表示抓取的文章，具有格式化功能。

**位置**: `article.py`

```python
# 转换为Markdown
markdown = article.to_markdown(including_title=True)

# 转换为LLM消息格式（文本+图片）
messages = article.to_message()
```

### JinaClient
使用Jina Reader API处理网页获取。

**位置**: `jina_client.py`

### ReadabilityExtractor
使用可读性算法从HTML中提取干净的文章内容。

**位置**: `readability_extractor.py`

## 使用示例

### 基本用法

```python
from src.crawler import Crawler

# 创建爬虫实例
crawler = Crawler()

# 抓取网页
url = "https://example.com/article"
article = crawler.crawl(url)

# 获取Markdown内容
markdown_content = article.to_markdown()
print(markdown_content)

# 获取LLM兼容的消息格式
messages = article.to_message()
```

### 完整处理示例

```python
from src.crawler import Crawler

def process_article(url: str):
    crawler = Crawler()
    article = crawler.crawl(url)
    
    return {
        "title": article.title,
        "url": article.url,
        "markdown": article.to_markdown(),
        "messages": article.to_message()
    }

result = process_article("https://example.com/news/article")
```

### 与LangChain工具集成

```python
from src.crawler import Crawler

def crawl_tool(url: str) -> str:
    """抓取URL并获取可读的markdown内容"""
    try:
        crawler = Crawler()
        article = crawler.crawl(url)
        return {"url": url, "crawled_content": article.to_markdown()[:1000]}
    except Exception as e:
        return f"抓取失败: {repr(e)}"
```

### 错误处理

```python
from src.crawler import Crawler
import logging

logger = logging.getLogger(__name__)

def safe_crawl(url: str):
    try:
        crawler = Crawler()
        article = crawler.crawl(url)
        return article.to_markdown()
    except Exception as e:
        logger.error(f"抓取失败 {url}: {e}")
        return None
```

## 配置

### 环境变量

```bash
# 可选：设置Jina API密钥以获得更高的请求限制
export JINA_API_KEY=your_jina_api_key
```

### API密钥设置

Jina API密钥是可选的，但建议使用以获得更高的请求限制：

1. 从[Jina AI](https://jina.ai/reader)获取API密钥
2. 设置环境变量：`JINA_API_KEY=your_key`

## 数据流

```
URL → JinaClient → HTML → ReadabilityExtractor → Article → Markdown/LLM消息
```

## 输出格式

### Markdown格式
```python
# 在markdown中包含标题
markdown_with_title = article.to_markdown(including_title=True)

# 在markdown中排除标题
markdown_only = article.to_markdown(including_title=False)
```

### LLM消息格式
将内容转换为支持文本和图片的消息对象列表：

```python
messages = article.to_message()
# 返回: [{"type": "text", "text": "..."}, {"type": "image_url", "image_url": {"url": "..."}}]
```

## 依赖项

- `requests`: 用于Jina API的HTTP请求
- `readabilipy`: 从HTML中提取内容
- `markdownify`: HTML到Markdown的转换

## 错误处理

模块包含内置的错误处理，涵盖：
- 网络连接问题
- 无效的URL
- 内容提取失败
- API请求限制

## 测试

运行测试以验证功能：

```bash
# 运行所有爬虫测试
make test

# 运行特定测试文件
uv run pytest tests/integration/test_crawler.py

# 运行覆盖率测试
make coverage
```

## 最佳实践

1. **请求限制**: 注意Jina API的请求限制
2. **错误处理**: 始终在try-catch块中包装抓取调用
3. **内容长度**: 考虑截断长内容以供LLM处理
4. **URL验证**: 抓取前验证URL
5. **缓存**: 为频繁访问的页面实施缓存

## 集成示例

爬虫在DeerFlow项目中被广泛使用：

- **研究代理**: 用于收集网络信息
- **工具集成**: 作为代理工作流的LangChain工具
- **RAG系统**: 用于文档预处理
- **内容分析**: 用于处理基于网络的内容