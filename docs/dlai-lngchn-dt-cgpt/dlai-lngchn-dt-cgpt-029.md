# 029：基于文档的问答 📄❓



在本节课中，我们将学习如何构建一个基于文档的问答系统。这是人们利用大语言模型构建的最常见、最强大的复杂应用之一。我们将学习如何让语言模型理解并回答关于特定文档内容的问题，并深入探讨其背后的核心技术：嵌入模型和向量数据库。

---

## 概述


基于文档的问答系统的核心目标是：给定一段文本（例如PDF、网页或公司内部文档），让语言模型能够回答关于这些文档内容的问题。这非常强大，因为它将语言模型的能力与它未训练过的专有数据结合起来，使其更加灵活和适应特定用例。

上一节我们介绍了链的基本概念，本节中我们来看看如何构建一个实际的问答链。



---


## 环境准备与导入


首先，我们需要导入必要的库并设置环境变量。



```python
# 导入环境变量
import os
# 导入构建链所需的组件
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI
from langchain.document_loaders import CSVLoader
from langchain.vectorstores import DocArrayInMemorySearch
# 导入用于显示的库
from IPython.display import display, Markdown
```

以下是导入组件的说明：
*   `RetrievalQA`：用于构建检索式问答链。
*   `ChatOpenAI`：我们将使用的聊天语言模型。
*   `CSVLoader`：用于加载我们的专有数据（一个CSV文件）。
*   `DocArrayInMemorySearch`：一个内存向量存储，无需连接外部数据库，非常适合快速入门。



---



## 快速构建：一行代码创建问答系统



LangChain 提供了非常便捷的方法来快速构建应用。让我们看看如何用几行代码创建一个功能完整的问答系统。


我们提供了一个关于户外服装的CSV文件。首先，初始化一个加载器来读取这个文件。



```python
# 初始化CSV文档加载器
file = ‘./OutdoorClothingCatalog_1000.csv‘
loader = CSVLoader(file_path=file)
```



接下来，我们将使用 `VectorstoreIndexCreator` 来轻松创建一个向量存储索引。


```python
# 导入索引创建器
from langchain.indexes import VectorstoreIndexCreator
```



创建索引只需指定向量存储的类型和文档加载器。


```python
# 创建向量存储索引
index = VectorstoreIndexCreator(
    vectorstore_cls=DocArrayInMemorySearch
).from_loaders([loader])
```


索引创建完成后，我们就可以直接开始提问了。


```python
# 向索引提问
query = “请列出所有带有防晒（UPF）功能的衬衫，并用表格总结。”
response = index.query(query)
# 显示结果
display(Markdown(response))
```



系统将返回一个包含防晒衬衫名称和描述的Markdown表格，并由语言模型进行了总结。



我们已经快速体验了文档问答的方法，但内部究竟是如何运作的呢？


---


## 核心原理：嵌入与向量数据库



要理解系统如何工作，我们需要先思考一个关键问题：语言模型一次只能处理有限数量的文本（几千个单词）。如果我们有非常大的文档，如何让模型回答关于其中所有内容的问题？


这时，**嵌入**和**向量数据库**就开始发挥作用了。



### 理解嵌入



嵌入是为文本片段创建数值表示的过程。这个数值表示（一个高维向量）捕获了文本的语义含义。


**核心概念**：具有相似内容的文本片段，其向量表示也相似。这让我们可以在向量空间中比较文本片段的相似度。


例如，我们有三个句子：
1.  “我喜欢我的狗。”
2.  “我的宠物很可爱。”
3.  “那辆车的速度很快。”


在向量空间中，前两个关于宠物的句子其向量会非常接近，而与第三个关于汽车的句子向量则相距甚远。这使我们能轻松找出哪些文本在语义上是相似的。


### 理解向量数据库



向量数据库是存储这些文本向量表示的地方。构建它的流程如下：

1.  **分块**：将大文档拆分为较小的文本块。这很重要，因为我们可能无法将整个文档传递给语言模型。
2.  **嵌入**：为每个文本块创建嵌入向量。
3.  **存储**：将这些向量及其对应的原始文本块存储到向量数据库中。



当用户提出查询时：
1.  为查询文本本身创建嵌入向量。
2.  在向量数据库中搜索与该查询向量最相似的N个文本块（通过计算向量间的距离，如余弦相似度）。
3.  将这些最相关的文本块作为上下文，与原始问题一起放入提示中，传递给语言模型以生成最终答案。



---


## 逐步详解：手动构建问答链



上面我们使用一行代码快速构建了链条。现在，让我们更详细地手动实现每一步，以彻底理解底层机制。


第一步与之前类似，创建文档加载器并加载数据。




```python
# 加载文档
docs = loader.load()
# 查看一个文档示例
print(docs[0])
```


接下来，我们需要为这些文档创建嵌入。我们将使用OpenAI的嵌入模型。


```python
# 导入并初始化嵌入模型
from langchain.embeddings import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```


我们可以看看为一个简单句子创建的嵌入是什么样子。



```python
# 为示例文本创建嵌入
embed = embeddings.embed_query(“你好，我是Harrison。”)
print(f”嵌入向量长度：{len(embed)}“)
print(f”前5个值：{embed[:5]}“)
```


嵌入向量通常有上千个维度，共同构成文本的数值表示。


现在，为所有加载的文档创建嵌入并存储到向量数据库中。



```python
# 从文档创建向量存储
db = DocArrayInMemorySearch.from_documents(docs, embeddings)
```


创建好向量存储后，我们可以用它来查找与查询相似的文本。

```python
# 进行相似性搜索
query = “请推荐一件带防晒功能的衬衫。”
docs_result = db.similarity_search(query)
print(f”返回了 {len(docs_result)} 个相关文档”)
print(docs_result[0].page_content)
```



返回的文档正是关于防晒衬衫的描述。那么，如何使用这些文档来生成最终答案呢？


首先，我们需要从这个向量存储创建一个**检索器**。检索器是一个通用接口，任何能接受查询并返回文档的方法都可以实现它。


```python
# 创建检索器
retriever = db.as_retriever()
```


如果我们手动操作，需要将检索到的文档合并成一个文本，然后构造提示词发送给语言模型。



```python
# 手动合并文档内容
docs_content = “\n\n”.join([doc.page_content for doc in docs_result])
# 构造提示词
prompt = f”””基于以下产品信息：
{docs_content}
请回答：{query}
请用表格列出并总结。”””
# 调用语言模型（此处为示意）
# llm_response = chat_model.predict(prompt)
```


所有这些步骤都可以被LangChain的链封装起来。我们可以创建一个`RetrievalQA`链。



```python
# 创建RetrievalQA链
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(temperature=0), # 指定语言模型
    chain_type=“stuff”,           # 指定链类型，最简单的一种
    retriever=retriever,          # 传入我们创建的检索器
    verbose=True                   # 显示详细运行日志
)
# 运行链条
response = qa_chain.run(query)
display(Markdown(response))
```

这样，我们就手动构建了一个完整的文档问答系统。请注意，这个详细步骤的结果与之前使用`VectorstoreIndexCreator`单行代码创建的结果是等价的。



---


## 高级配置与链类型



在创建索引时，我们也可以进行自定义，就像手动构建时一样。


```python
# 创建索引时指定嵌入模型
index_custom = VectorstoreIndexCreator(
    vectorstore_cls=DocArrayInMemorySearch,
    embedding=embeddings # 指定自定义嵌入模型
).from_loaders([loader])
```


我们在笔记本中使用了`chain_type=“stuff”`方法。这是最简单的方法，它把所有检索到的文档内容“塞”进一个提示中，只调用一次语言模型。它便宜、高效且易于理解。

但`stuff`方法并不总是有效，尤其是当检索到的文档非常多、总长度超过模型上下文限制时。LangChain提供了其他几种链类型来处理这种情况：



以下是不同链类型的比较：
*   **MapReduce**：将每个文档块与问题一起单独发送给语言模型，得到多个答案，再用一次调用总结所有答案。**优点**：可处理任意数量文档，可并行处理。**缺点**：调用次数多，文档被视为独立个体。
*   **Refine**：迭代处理文档，基于前一个文档的答案和当前文档构建新答案。**优点**：适合组合跨文档信息，答案通常更连贯。**缺点**：调用是串行的，速度较慢。
*   **MapRerank**：对每个文档进行一次调用，要求语言模型返回一个相关性分数，然后选择分数最高的答案。**优点**：调用可并行。**缺点**：严重依赖模型对“分数”的理解，需要精心设计提示词。


最常用的方法是`stuff`，其次是`MapReduce`。这些方法不仅用于问答，也可用于其他任务，例如`MapReduce`链就常用于长文档摘要。


---



## 总结


本节课中，我们一起学习了如何构建基于文档的问答系统。


我们首先了解了这是将语言模型与专有数据结合的强大应用。接着，我们通过`VectorstoreIndexCreator`快速体验了“一行代码”构建的便捷性。然后，我们深入探讨了其核心原理：**嵌入**将文本转化为可比对的向量，**向量数据库**则高效存储和检索这些向量。通过手动分步实现，我们清晰地看到了从文档加载、嵌入、存储到检索和生成答案的完整流程。最后，我们介绍了不同的链类型（Stuff, MapReduce, Refine, MapRerank）及其适用场景，让你能根据实际需求选择最合适的工具。

![](img/a79e11877c93e78d6e0286c3c4597b94_180.png)

在下一节中，我们将探索LangChain中更多不同类型的链和它们的应用。