# 知识库：RAG

## 目标

知识库用于解决大模型不了解私有数据、实时数据或长文档的问题。典型方式是 RAG：先检索相关知识，再把检索结果作为上下文交给模型回答。

大模型虽然有大量通用知识，但它的知识来自训练数据：训练之后的新信息、企业内部文档、个人笔记、项目代码、数据库记录，它默认都不知道。直接问模型这些问题，模型可能会猜测甚至编造。

知识库的思路不是重新训练模型，而是在提问时先从外部资料中找出相关片段，再把这些片段作为上下文交给模型。这样模型回答时就有“材料依据”，更适合企业文档问答、客服知识库、代码库问答、个人笔记检索等场景。

RAG 的关键不是单个组件，而是整条链路的质量：文档是否加载完整、切分是否合理、向量模型是否合适、索引维度是否一致、检索结果是否准确、提示词是否约束模型只基于上下文回答。

## 两个阶段

```text
知识准备阶段：
Loader 加载文档
  -> Transformer 切分文档
  -> Embedder 向量化
  -> Indexer 存储索引

检索增强阶段：
用户问题
  -> Retriever 检索相关文档
  -> ChatTemplate 拼接上下文
  -> ChatModel 生成回答
```

知识准备阶段通常离线执行或定时执行；检索增强阶段在用户每次提问时执行。把这两个阶段分清楚，后续排查问题会容易很多。

## 1. 知识准备阶段

### Document Loader

`Loader` 从本地文件、URL 等来源加载文档，并转换为统一的 `schema.Document`。

```go
type Loader interface {
    Load(ctx context.Context, src document.Source, opts ...LoaderOption) ([]*schema.Document, error)
}

type Source struct {
    URI string
}

type Document struct {
    ID       string
    Content  string
    MetaData map[string]any
}
```

本地文件加载：

```go
loader, err := file.NewFileLoader(ctx, &file.FileLoaderConfig{
    UseNameAsID: true,
})

docs, err := loader.Load(ctx, document.Source{
    URI: "./loader/test.md",
})
```

URL 加载：

```go
loader, err := url.NewLoader(ctx, &url.LoaderConfig{})

docs, err := loader.Load(ctx, document.Source{
    URI: "https://www.cloudwego.io/zh/docs/eino/",
})
```

读取 docx 等格式时，需要配置对应 parser，再交给 file loader 使用。

### Embedding

`Embedder` 把文本转为向量，后续才能做相似度检索。

```go
type Embedder interface {
    EmbedStrings(ctx context.Context, texts []string, opts ...Option) ([][]float64, error)
}
```

Ollama 向量化：

```go
embedder, err := ollama.NewEmbedder(ctx, &ollama.EmbeddingConfig{
    Model:   "modelscope.cn/nomic-ai/nomic-embed-text-v1.5-GGUF:latest",
    BaseURL: "http://127.0.0.1:11434",
})

vectors, err := embedder.EmbedStrings(ctx, []string{"hello world"})
```

阿里百炼向量化：

```go
embedder, err := dashscope.NewEmbedder(ctx, &dashscope.EmbeddingConfig{
    APIKey: "your api key",
    Model:  "text-embedding-v4",
})
```

### Transformer

`Transformer` 用于处理文档，最常见的是切分。

```go
type Transformer interface {
    Transform(ctx context.Context, src []*schema.Document, opts ...TransformerOption) ([]*schema.Document, error)
}
```

常见切分器：

- `html.HeaderSplitter`：按 HTML 标题结构切分。
- `markdown.HeaderSplitter`：按 Markdown 标题切分。
- `recursive.Splitter`：按分隔符和 chunk 大小递归切分。
- `semantic.Splitter`：基于语义相似度切分，需要 embedding。

递归切分骨架：

```go
splitter, err := recursive.NewSplitter(ctx, &recursive.Config{
    ChunkSize:   500,
    OverlapSize: 50,
    Separators:  []string{"\n\n", "\n", "。", ".", " "},
})

chunks, err := splitter.Transform(ctx, docs)
```

切分要点：

- chunk 太大：检索结果不精准，占用上下文。
- chunk 太小：语义不完整，回答容易丢信息。
- overlap 用于保留上下文连续性。

### Indexer

`Indexer` 负责把文档和向量写入存储系统。

```go
type Indexer interface {
    Store(ctx context.Context, docs []*schema.Document, opts ...Option) (ids []string, err error)
}
```

Redis 向量索引需要 Redis Stack，因为要使用 RediSearch 的向量检索能力。

```bash
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```

Redis indexer 骨架：

```go
client := redis.NewClient(&redis.Options{
    Addr:          "localhost:6379",
    Protocol:      2,
    UnstableResp3: true,
})

indexer, err := redisindexer.NewIndexer(ctx, &redisindexer.IndexerConfig{
    Client:    client,
    KeyPrefix: "eino_doc:",
    BatchSize: len(chunks),
    Embedding: embedder,
})

ids, err := indexer.Store(ctx, chunks)
```

## 2. 检索增强阶段

### Retriever

`Retriever` 根据用户 query 从知识库中找相关文档。

```go
type Retriever interface {
    Retrieve(ctx context.Context, query string, opts ...Option) ([]*schema.Document, error)
}
```

Redis Retriever 骨架：

```go
retriever, err := redisretriever.NewRetriever(ctx, &redisretriever.RetrieverConfig{
    Client:    client,
    Index:     "doc_index",
    Embedding: embedder,
})

docs, err := retriever.Retrieve(ctx, "Eino 的核心组件有哪些？")
```

### 拼接上下文回答

```go
var contextText strings.Builder
for _, doc := range docs {
    contextText.WriteString(doc.Content)
    contextText.WriteString("\n---\n")
}

tpl := prompt.FromMessages(
    schema.FString,
    schema.SystemMessage("请只根据给定知识库内容回答问题。\n知识库：\n{context}"),
    schema.UserMessage("{question}"),
)

messages, err := tpl.Format(ctx, map[string]any{
    "context":  contextText.String(),
    "question": userQuestion,
})

answer, err := model.Generate(ctx, messages)
```

## 完整 RAG 流程速记

```text
准备：
文件 / URL
  -> Loader
  -> Document
  -> Splitter
  -> Chunks
  -> Embedding
  -> Redis / Milvus / ES / VikingDB

问答：
Question
  -> Embedding
  -> Vector Search
  -> TopK Documents
  -> Prompt(context + question)
  -> LLM Answer
```

## 记忆点

- RAG 不是让模型“学会”知识，而是把相关知识临时放进上下文。
- Loader 解决“从哪里来”，Transformer 解决“怎么切”，Embedder 解决“怎么表示”，Indexer 解决“存哪里”，Retriever 解决“怎么找”。
- 向量维度必须和模型输出维度、Redis 索引维度一致。
- 检索结果需要再经过提示词约束，避免模型脱离知识库乱答。
