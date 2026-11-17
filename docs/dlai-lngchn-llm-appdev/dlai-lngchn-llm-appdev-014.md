# 014：构建问答聊天机器人 🤖



在本节课中，我们将学习如何利用检索到的文档，结合语言模型来构建一个能够回答问题的聊天机器人。我们将探讨几种不同的实现方法，并了解它们各自的优缺点。

---



## 环境准备与数据加载


上一节我们介绍了文档检索的基本概念，本节中我们来看看如何将检索到的文档用于生成答案。首先，我们需要设置环境并加载数据。



以下是初始化步骤：


1.  导入必要的环境变量。
2.  加载之前持久化保存的向量数据库。
3.  验证数据库是否正确加载了文档。
4.  执行一次快速的相似性搜索，确保检索功能正常工作。










```python
# 示例代码：加载向量数据库并验证
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
# 加载已持久化的向量数据库
persist_directory = ‘your_persist_dir’
embedding = OpenAIEmbeddings()
vectordb = Chroma(persist_directory=persist_directory, embedding_function=embedding)
# 验证文档数量
print(f“数据库中文档数量：{vectordb._collection.count()}”)
# 执行相似性搜索测试
docs = vectordb.similarity_search(“这门课的主要主题是什么？”, k=3)
print(docs[0].page_content)
```



---


## 初始化语言模型与问答链



在准备好数据后，我们需要初始化用于生成答案的语言模型，并创建一个问答链。



以下是核心步骤：


1.  初始化语言模型（例如 GPT-3.5），并将温度设置为0以保证答案的稳定性和准确性。
2.  导入并创建 `RetrievalQA` 链，将语言模型和向量数据库作为检索器传入。









```python
# 示例代码：初始化模型与问答链
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
question = “这门课的主要主题是什么？”
result = qa_chain({“query”: question})
print(result[“result”])
```


---


## 理解提示模板与返回源文档


为了更深入地理解问答链的工作原理，我们可以自定义提示模板，并让链返回它用于生成答案的源文档。



以下是具体操作：


1.  定义一个提示模板，其中包含指导语言模型如何利用上下文（文档）来回答问题的指令。
2.  创建新的问答链时，传入自定义的提示模板，并设置参数以返回源文档。








```python
# 示例代码：自定义提示模板并返回源文档
from langchain.prompts import PromptTemplate
# 定义提示模板
template = “””请根据以下上下文信息回答问题。如果你不知道答案，就说不知道。
上下文：{context}
问题：{question}
答案：“””
QA_CHAIN_PROMPT = PromptTemplate.from_template(template)
# 创建新的问答链，返回源文档
qa_chain_new = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    return_source_documents=True, # 返回源文档
    chain_type_kwargs={“prompt”: QA_CHAIN_PROMPT}
)
# 提问并查看结果与源文档
result = qa_chain_new({“query”: “概率是课堂主题吗？”})
print(“答案：”, result[“result”])
print(“\n参考的源文档：”)
for doc in result[“source_documents”]:
    print(doc.page_content[:200], “...”) # 打印文档前200字符
```


---


## 探索不同的问答策略：MapReduce 与 Refine


默认的“stuff”方法将所有文档塞入一次模型调用，当文档过多时可能超出上下文窗口限制。因此，我们需要了解其他策略。


以下是两种替代方法：



*   **MapReduce**：首先将每个文档单独发送给语言模型得到初步答案，然后将这些答案组合起来，通过最后一次模型调用生成最终答案。这种方法能处理大量文档，但速度较慢，且可能因信息分散在不同文档中而影响答案质量。
    *   **公式**：`Final_Answer = LLM( Combine( LLM(Doc1), LLM(Doc2), … ) )`
*   **Refine**：按顺序处理文档。首先基于第一个文档生成初始答案，然后依次将后续文档和当前答案提供给模型，要求其优化或完善答案。这种方法允许信息在文档间传递，通常能产生比MapReduce更好的结果。
    *   **公式**：`Answer_i = LLM( Answer_{i-1}, Doc_i )`， 其中 `Answer_0` 基于 `Doc_1` 生成。






```python
# 示例代码：尝试不同的链类型
# MapReduce 方法
qa_chain_mapreduce = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    chain_type=“map_reduce” # 指定链类型
)
# Refine 方法
qa_chain_refine = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectordb.as_retriever(),
    chain_type=“refine” # 指定链类型
)
# 比较结果
question = “概率是课堂主题吗？”
print(“MapReduce 答案：”, qa_chain_mapreduce.run(question))
print(“Refine 答案：”, qa_chain_refine.run(question))
```


> **提示**：你可以使用 LangChain 的平台（如 LangSmith）来追踪和可视化这些链的调用过程，观察模型每一步的输入和输出，这对于调试和理解内部机制非常有帮助。


---


## 实现带记忆的对话：处理后续问题

基本的问答链是无状态的，无法记住之前的对话历史。为了构建能处理后续问题的聊天机器人，我们需要引入记忆机制。

以下是实现思路：


1.  使用 `ConversationalRetrievalChain` 代替基础的 `RetrievalQA` 链。
2.  该链内部会管理一个“聊天历史”，将当前问题与历史对话一起提供给模型，使其能理解上下文。






```python
# 示例代码：创建带记忆的对话链
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory

# 创建记忆对象
memory = ConversationBufferMemory(memory_key=“chat_history”, return_messages=True)

# 创建对话式检索链
conversational_qa_chain = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=vectordb.as_retriever(),
    memory=memory
)
# 进行多轮对话
result1 = conversational_qa_chain({“question”: “概率是课堂主题吗？”})
print(“第一轮回答：”, result1[“answer”])
result2 = conversational_qa_chain({“question”: “为什么需要这些先决条件？”})
print(“第二轮回答（基于上下文）：”, result2[“answer”])
```


---


## 总结

![](img/e1413a46a066fb0fe58e11f740158ac6_110.png)

本节课中我们一起学习了构建问答聊天机器人的核心步骤。我们从加载数据和初始化模型开始，创建了基本的问答链，并深入了解了提示模板的作用。接着，我们探讨了处理大量文档的 MapReduce 和 Refine 策略。最后，我们引入了记忆机制，使机器人能够处理连贯的、有上下文的对话。你现在已经掌握了使用 LangChain 构建一个功能完整的问答系统的基础知识。