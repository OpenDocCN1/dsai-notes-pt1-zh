# 038：6——问答聊天机器人 🤖



在本节课中，我们将学习如何利用检索到的文档，结合语言模型来构建一个能够回答问题的聊天机器人。我们将探讨几种不同的文档处理与答案生成方法，并了解它们的优缺点。



---


上一节我们介绍了如何检索相关文档。本节中，我们来看看如何将这些文档与原始问题一同传递给语言模型，从而得到最终答案。



## 环境与数据准备 🛠️


首先，我们需要加载环境变量和之前持久化的向量数据库。








```python
# 加载环境变量
import os
from dotenv import load_dotenv
load_dotenv()
# 加载向量数据库
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
persist_directory = ‘你的数据库路径’
embedding = OpenAIEmbeddings()
vectordb = Chroma(persist_directory=persist_directory, embedding_function=embedding)
```



检查数据库以确保其包含209个文档，并测试相似性搜索功能。



```python
# 测试检索功能
question = “本课程的主要主题是什么”
docs = vectordb.similarity_search(question, k=3)
print(len(docs))
```


## 初始化语言模型与问答链 ⚙️



接下来，我们初始化用于回答问题的语言模型，并创建一个基础的问答链。










```python
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA
# 初始化语言模型
llm = ChatOpenAI(model_name=“gpt-3.5-turbo”, temperature=0)
# 创建检索问答链
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever()
)
# 进行提问
result = qa_chain.run(“本课程的主要主题是什么”)
print(result)
```


运行上述代码，模型会返回一个答案，例如：“本课主要话题是机器学习。此外，讨论部分可能会复习统计和代数。”


## 理解底层机制：提示模板 🔍



为了更好地控制问答过程，我们需要理解底层使用的提示模板。以下是默认的提示模板结构：







```python
from langchain.prompts import PromptTemplate
template = “”“使用以下上下文片段来回答问题。如果你不知道答案，就说不知道，不要编造答案。
上下文：{context}
问题：{question}
有帮助的答案：”“”
QA_CHAIN_PROMPT = PromptTemplate.from_template(template)
```

我们可以创建一个新的问答链，并指定使用自定义的提示模板和返回源文档。




```python
qa_chain_new = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    return_source_documents=True,
    chain_type_kwargs={“prompt”: QA_CHAIN_PROMPT}
)
result = qa_chain_new({“query”: “概率是课程主题吗？”})
print(result[“result”])
print(result[“source_documents”])
```



## 探索不同的链类型 🔄


默认的“stuff”方法将所有文档塞入同一个提示中，虽然高效，但可能受限于上下文窗口的长度。以下是其他几种处理多文档的方法：


### MapReduce 方法
这种方法先将每个文档单独送至语言模型获取初步答案，然后再将这些答案组合成最终答案。



```python
qa_chain_mr = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    chain_type=“map_reduce”
)
```


**优点**：可以处理任意数量的文档。
**缺点**：速度较慢，且如果答案信息分散在多个文档中，可能无法有效整合。



### Refine 方法
这种方法顺序处理文档，用后续文档的信息不断迭代和改进前一个答案。


```python
qa_chain_refine = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    chain_type=“refine”
)
```


**优点**：比MapReduce更能鼓励信息的跨文档传递，通常能产生更连贯的答案。
**缺点**：仍然涉及多次LLM调用，速度较慢。


你可以使用LangChain平台的可视化工具来观察这些链内部的具体调用步骤，这有助于深入理解其工作原理。


## 实现有记忆的对话 💬

基础的问答链是无状态的，无法处理后续问题。为了实现连贯的对话，需要引入记忆机制。

```python
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(memory_key=“chat_history”, return_messages=True)
conversational_qa_chain = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=vectordb.as_retriever(),
    memory=memory
)

# 第一轮提问
result1 = conversational_qa_chain({“question”: “概率是课程主题吗？”})
print(result1[“answer”])

# 基于记忆的后续提问
result2 = conversational_qa_chain({“question”: “为什么需要这个前提？”})
print(result2[“answer”])
```



这样，聊天机器人就能记住之前的对话上下文，从而回答后续的澄清或深入问题。


---

![](img/8fdba8c04f845e8d78897dbf4b174b32_104.png)

本节课中我们一起学习了构建问答聊天机器人的核心步骤：从准备数据、初始化模型，到使用不同的链类型处理文档，最后引入记忆功能实现多轮对话。你可以尝试不同的问题、提示模板和链类型，观察它们对答案质量的影响，这是掌握RAG应用的关键。