# 013：高级检索技术 🔍


在本节课中，我们将深入学习几种克服基础语义搜索局限性的高级检索技术。我们将探讨最大边际相关性、自我查询检索以及上下文压缩等方法，以提升检索结果的相关性和多样性。



---



## 概述 📋


在上一课中，我们介绍了语义搜索的基础知识，并看到它在大量用例中工作得很好。但我们也看到了一些边缘情况，事情可能会出一点问题。在本节课中，我们将深入探讨检索，并介绍一些克服这些边缘情况的更先进的方法。



检索在查询时很重要。当您有一个查询进来时，您想检索最相关的文档片段。我们在上一课中谈到了语义相似性搜索，但我们将在这里讨论一些不同的、更先进的方法。



---



## 最大边际相关性 (MMR) ⚖️


上一节我们介绍了语义搜索，本节中我们来看看最大边际相关性。这背后的想法是：如果您总是获取与查询在嵌入空间中最相似的文档，您可能会错过不同的信息，就像我们在一个边缘案例中看到的那样。



在本例中，我们有一个查询：“所有的白蘑菇”。如果我们查看最相似的结果，这将是前两份文档，它们有很多类似于子实体查询的信息（“全身都是白的”）。但我们真的想确保我们也能得到其他信息，比如“它真的有毒”。这就是MMR发挥作用的地方，因为它将为一组不同的文档进行选择。



MMR背后的想法是：我们发送一个查询，然后我们最初得到一组响应。参数 `fetch_k` 是我们可以控制的，它决定了我们最初得到多少回应。这完全基于语义相似性。从那里，我们处理这组较小的文档，并优化选择，使其不仅是基于语义相似性的最相关文档，也是多样化的。从这组文档中，我们选择最后的 `k` 个返回给用户。


以下是MMR过程的简化描述：


```
1. 输入查询 Q。
2. 使用语义相似性检索前 fetch_k 个文档 D_fetch。
3. 从 D_fetch 中，根据“相关性”和“与已选文档的差异性”的加权组合，选择最终的 k 个文档 D_final。
4. 返回 D_final。
```



---


## 自我查询检索 🤖



接下来，我们看看自我查询检索。当您遇到不仅要从语义上查找内容，还要根据元数据进行筛选的问题时，这很有用。



让我们来回答这个问题：“1980年拍的关于外星人的电影有哪些？” 这真的有两个组成部分：
1.  它有一个语义部分：“关于外星人”。
2.  它有一个引用元数据的部分：“年份应该是1980年”。



我们能做的就是使用语言模型本身将最初的问题分成两个独立的部分：**筛选器**和**搜索词**。大多数向量存储支持元数据筛选器，因此您可以轻松地基于元数据（如“年份=1980”）筛选记录。



以下是使用自我查询检索器的步骤：





```python
# 伪代码示例
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain.chains.query_constructor.base import AttributeInfo
# 1. 定义元数据字段及其描述
metadata_field_info = [
    AttributeInfo(name="year", description="电影上映的年份", type="integer"),
    AttributeInfo(name="genre", description="电影的类型", type="string"),
]
# 2. 初始化语言模型和检索器
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description="电影的描述",
    metadata_field_info=metadata_field_info,
)
# 3. 进行查询
docs = retriever.get_relevant_documents("1980年拍的关于外星人的电影有哪些？")
```



---


## 上下文压缩 🗜️



最后我们将讨论压缩。这对于只提取检索到的段落中最相关的部分是有用的。例如，在问问题时，您拿回存储的文档，但可能只有前一两句是相关的部分。



然后，您可以通过语言模型运行所有这些文档，并提取最相关的段落，然后只将最相关的段传递到最终的语言模型调用中。这是以对语言模型进行更多调用为代价的，但它也非常有利于将最终答案集中在最重要的事情上。所以这是一种权衡。



以下是上下文压缩的基本流程：



```
1. 使用基础检索器（如向量存储）获取一组初始文档 D_initial。
2. 使用一个“压缩器”（通常是另一个LLM链）处理每个文档，提取或总结与查询最相关的片段。
3. 将压缩后的文档片段 D_compressed 传递给生成答案的LLM。
```



---


## 技术实践演示 💻



现在让我们看看这些不同的技术如何起作用。



我们将从加载环境变量开始，就像我们一直做的那样。导入必要的库（如Chroma和OpenAI），并加载我们之前使用的文档集合。


### MMR 示例



以下是MMR的一个应用示例：





```python
# 伪代码：对比相似性搜索和MMR
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
# 初始化检索器
vectorstore = Chroma(...)
# 1. 标准相似性搜索
standard_docs = vectorstore.similarity_search("所有的白蘑菇", k=2)
# 2. MMR 搜索
mmr_docs = vectorstore.max_marginal_relevance_search("所有的白蘑菇", k=2, fetch_k=5)
# 比较结果，MMR结果可能包含关于“毒性”的多样化信息。
```



### 自我查询示例



对于之前“第三堂课回归”的问题，我们可以使用自我查询自动生成元数据过滤器：


```python
# 运行自我查询检索器
question = "他们在第三堂课上对回归说了什么？"
docs = self_query_retriever.get_relevant_documents(question)
# 检索器内部会将问题解析为：查询词=“回归”， 过滤器=“source == ‘lecture3.pdf’”
```


### 上下文压缩与MMR结合



我们可以结合多种技术以获得最佳结果。例如，在使用上下文压缩时，可以指定基础检索器使用MMR：





```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
# 创建基础检索器（使用MMR）
base_retriever = vectorstore.as_retriever(search_type="mmr", search_kwargs={"k": 5, "fetch_k": 10})
# 创建压缩器
compressor = LLMChainExtractor.from_llm(llm)
# 创建压缩检索器
compression_retriever = ContextualCompressionRetriever(base_compressor=compressor, base_retriever=base_retriever)
# 进行查询
compressed_docs = compression_retriever.get_relevant_documents("他们怎么说Matlab？")
```



---



## 其他检索技术概览 🧠


到目前为止，我们提到的所有附加检索技术都建立在矢量数据库之上。值得一提的是，还有其他类型的检索根本不使用矢量数据库，而是使用更传统的NLP技术。

例如，SVM检索器和TF-IDF检索器。如果您从传统的NLP或机器学习中认识这些术语，那很好；如果不认识，也没关系，这只是其他技术的一个例子。除了这些，还有很多技术，我鼓励你去探索。





```python
from langchain.retrievers import SVMRetriever, TFIDFRetriever
# SVM 检索器（需要嵌入）
svm_retriever = SVMRetriever.from_texts(texts, embeddings)
# TF-IDF 检索器
tfidf_retriever = TFIDFRetriever.from_texts(texts)
```



---



## 总结 🎯


在本节课中，我们一起学习了三种高级检索技术：
1.  **最大边际相关性 (MMR)**：用于在保证相关性的同时，增加检索结果的多样性，避免信息冗余。
2.  **自我查询检索**：利用语言模型自动将自然语言查询分解为语义搜索词和元数据过滤器，特别适用于混合查询。
3.  **上下文压缩**：通过额外的LLM调用，从检索到的大篇幅文档中提取最相关的片段，有助于提升最终答案的聚焦度和效率，但会增加延迟和成本。

![](img/d616ba48c25503a419d35bb2fb15cd3e_162.png)

我们还了解到，这些技术可以组合使用（如MMR+压缩），并且除了基于向量的方法，还存在其他基于传统NLP的检索器（如SVM、TF-IDF）。

我鼓励您在各种问题上尝试这些不同的检索技术。自我查询检索器尤其是我最喜欢的，所以我建议用越来越复杂的元数据过滤器来尝试。既然我们已经深入探讨了检索，在接下来的课程中，我们将讨论这个过程的下一步。

---