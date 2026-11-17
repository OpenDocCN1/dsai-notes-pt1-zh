# 027：记忆 🧠





在本节课中，我们将要学习LangChain中的“记忆”功能。大型语言模型本身是无状态的，不会记住之前的对话。为了构建能够进行连贯对话的应用程序（如聊天机器人），我们需要一种机制来存储和利用对话历史。本节将介绍LangChain提供的多种记忆管理方法。




## 概述与准备工作



首先，我们需要导入必要的API密钥和工具。这是使用LangChain和OpenAI模型的基础步骤。



```python
import os
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory, ConversationBufferWindowMemory, ConversationTokenBufferMemory, ConversationSummaryBufferMemory
```



上一节我们介绍了LangChain的基本概念，本节中我们来看看如何为对话应用添加记忆功能。



## 对话缓冲记忆





最基本的记忆类型是**对话缓冲记忆**。它会存储完整的对话历史。




以下是初始化一个带有对话缓冲记忆的聊天链的步骤：




1.  设置语言模型为OpenAI的聊天接口，并设定温度参数。
2.  将记忆设置为`ConversationBufferMemory`。
3.  构建对话链。















```python
llm = OpenAI(temperature=0)
memory = ConversationBufferMemory()
conversation = ConversationChain(llm=llm, memory=memory, verbose=True)
# 开始对话
response = conversation.predict(input="嗨，我的名字是安德鲁。")
print(response) # 输出：你好，安德鲁！很高兴见到你。
response = conversation.predict(input="1加1等于几？")
print(response) # 输出：1加1等于2。
response = conversation.predict(input="你知道我的名字吗？")
print(response) # 输出：你的名字是安德鲁，正如你之前提到的。
```




通过将`verbose`设置为`True`，我们可以看到LangChain生成的完整提示。它会将之前的对话历史作为上下文附加到当前问题中，从而使模型能够“记住”信息。




我们可以直接查看记忆对象中存储的内容：
```python
print(memory.buffer) # 打印完整的对话历史
print(memory.load_memory_variables({})) # 以字典形式加载记忆变量
```




## 对话缓冲窗口记忆





随着对话变长，存储完整历史会导致令牌数量激增，增加成本和模型处理的负担。**对话缓冲窗口记忆**只保留最近`k`轮对话。




以下是使用对话缓冲窗口记忆的方法：












```python
memory = ConversationBufferWindowMemory(k=1) # 只记住最近一轮对话（一次用户输入和一次AI回复）
conversation = ConversationChain(llm=llm, memory=memory, verbose=True)
# 模拟对话
memory.save_context({"input": "嗨"}, {"output": "怎么了？"})
memory.save_context({"input": "没什么，就挂着"}, {"output": "酷"})
print(memory.load_memory_variables({}))
# 因为k=1，只输出最近的交换：
# {'history': 'Human: 没什么，就挂着\nAI: 酷'}
```




当`k=1`时，模型会忘记更早的对话。在实践中，通常会将`k`设置为一个较大的数字，在控制记忆长度的同时保留足够的上下文。




## 对话令牌缓冲记忆




另一种控制记忆长度的方法是直接限制存储的令牌数量。**对话令牌缓冲记忆**会根据设定的最大令牌数来保留最近的对话内容。




以下是其应用示例：


```python
memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=50) # 最大令牌限制为50
```



这种方式更直接地与语言模型的计费方式挂钩。你需要指定`llm`参数，因为不同的模型计算令牌的方式不同。




## 对话摘要缓冲记忆



**对话摘要缓冲记忆**是一种更高级的方法。它不会简单地丢弃旧对话，而是使用语言模型为超长的对话历史生成一个摘要。




以下是其工作原理：




1.  它会显式地存储最近的对话，直到达到设定的令牌限制。
2.  对于更早的、超出限制的部分，它会调用语言模型生成一个摘要。
3.  最终，记忆由“最近的对话片段”+“早期对话的摘要”构成。


```python
# 假设有一段很长的日程描述
schedule = “上午与产品团队开会，准备PPT。中午与客户在意大利餐厅共进午餐，需要带上笔记本电脑展示最新的AI演示...”
memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=100)

# 保存包含长文本的对话
memory.save_context({"input": “今天日程上有什么？”}, {"output": schedule})
print(memory.load_memory_variables({}))
# 如果日程文本超过100个令牌，输出将包含一个由模型生成的摘要，而不是完整的文本。
```

这种方式能在有限的令牌预算内，保留对话的核心信息和上下文。



## 其他记忆类型与总结




除了上述类型，LangChain还支持更强大的记忆系统：



*   **向量存储记忆**：利用文本嵌入和向量数据库，存储记忆并检索与当前对话最相关的片段。这适用于处理大量信息。
*   **实体记忆**：专门用于记忆对话中提到的特定实体（如人物、地点）的详细信息。




在实际应用中，开发者可以组合使用多种记忆类型。例如，同时使用对话缓冲窗口记忆来维持对话流，并使用实体记忆来记住关键人物的细节。



此外，将完整的对话历史存储在传统数据库（如SQL或键值存储）中也很常见，以便进行审计或系统分析。



![](img/c86c51310bd0360f0802bd988e1cb8c6_202.png)

本节课中我们一起学习了LangChain中“记忆”的核心概念与几种实现方式。我们了解到，通过**对话缓冲记忆**可以保存完整历史，通过**对话缓冲窗口记忆**和**对话令牌缓冲记忆**可以控制记忆长度，而**对话摘要缓冲记忆**则能用摘要的形式保留长对话的精华。这些工具是构建能够进行连贯、上下文感知对话的AI应用的基础。