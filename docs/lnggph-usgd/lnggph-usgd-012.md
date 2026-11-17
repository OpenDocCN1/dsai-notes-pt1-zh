#  012：使用 LangGraph 构建自反思 RAG 流程 🧠


在本节课中，我们将学习如何使用 LangGraph 构建一个名为“自反思 RAG”或“主动 RAG”的复杂检索增强生成流程。我们将基于 CRAG 论文的思想，实现一个能够评估检索结果质量、并在必要时进行修正的智能系统。



## 概述：从线性 RAG 到主动 RAG

传统的 RAG 流程是线性的：**问题 -> 检索 -> 生成**。然而，在实践中，我们常常需要回答几个关键问题：我们是否真的需要检索？检索到的文档质量如何？如果质量不佳，我们该如何修正？这些问题催生了“主动 RAG”的概念。


主动 RAG 是一个让大语言模型根据现有检索结果或生成内容，动态决定何时、如何进行检索的流程。



## 核心概念：状态机与流程工程


为了实现主动 RAG，我们需要超越简单的链式调用，引入**状态机**的概念。状态机允许我们定义更复杂、更多样化的逻辑流程，让 LLM 在不同的步骤间进行选择，同时由我们指定所有可能的转换路径。





LangGraph 是构建此类状态机的理想工具。它鼓励我们进行**流程工程**：先仔细设计期望的工作流，再将其实现为图。




## 实现 CRAG 工作流

我们将实现一个简化版的 CRAG 工作流。其核心逻辑如下：
1.  **检索**：根据问题从向量库中获取相关文档。
2.  **评估**：对每个检索到的文档进行相关性评分。
3.  **决策**：
    *   如果**至少有一个文档**是相关的，则直接进入**生成**步骤。
    *   如果所有文档都**不相关或模糊**，则进行**网络搜索**以获取补充信息，然后进入生成步骤。





我们会对流程做一个小调整：即使有相关文档，如果存在不相关文档，我们也会进行网络搜索来补充上下文。


### 第一步：定义状态




状态是一个核心对象，它会在图的各个节点间传递和修改。我们将其定义为一个字典。

```python
from typing import TypedDict, List, Annotated
import operator

class GraphState(TypedDict):
    question: str
    documents: List[str]
    generation: str
    search: bool  # 是否需要进行网络搜索
```


### 第二步：实现节点函数




每个节点都是一个函数，它接收当前状态，执行特定操作，并返回更新后的状态。




以下是关键节点的功能描述：


*   **检索节点**：从向量库中获取文档，并添加到状态中。
*   **评估节点**：使用一个 LLM 链对每个文档进行“是否相关”的二元判断。如果任何文档被判定为不相关，则将状态中的 `search` 标志设为 `True`。
*   **查询转换节点**：如果需要搜索，则对原始问题进行优化重写。
*   **网络搜索节点**：使用转换后的问题进行网络搜索，并将结果添加到文档列表中。
*   **生成节点**：将最终的问题和文档列表（可能包含原始检索结果和网络搜索结果）传递给 LLM 生成答案。




### 第三步：构建图并定义边




我们将各个节点连接起来，并设置条件边来实现决策逻辑。























```python
from langgraph.graph import StateGraph, END
# 初始化图
workflow = StateGraph(GraphState)
# 添加节点
workflow.add_node(“retrieve”, retrieve_node)
workflow.add_node(“grade_documents”, grade_documents_node)
workflow.add_node(“transform_query”, transform_query_node)
workflow.add_node(“web_search”, web_search_node)
workflow.add_node(“generate”, generate_node)
# 设置入口点
workflow.set_entry_point(“retrieve”)
# 添加固定边
workflow.add_edge(“retrieve”, “grade_documents”)
workflow.add_edge(“transform_query”, “web_search”)
workflow.add_edge(“web_search”, “generate”)
workflow.add_edge(“generate”, END)
# 添加条件边（决策点）
def decide_to_search(state):
    if state[“search”]:
        return “transform_query”
    else:
        return “generate”
workflow.add_conditional_edges(
    “grade_documents”,
    decide_to_search,
    {
        “transform_query”: “transform_query”,
        “generate”: “generate”,
    }
)
# 编译图
app = workflow.compile()
```




### 第四步：运行与调试

运行图应用，并观察其执行过程。通过 LangSmith 可以清晰地追踪每个节点的输入、输出和状态变化，这对于调试和理解流程非常有帮助。



```python
# 运行图
inputs = {“question”: “智能体的记忆系统是如何工作的？”}
for output in app.stream(inputs):
    for key, value in output.items():
        print(f”节点 ‘{key}’ 完成:”)
        print(f”    状态: {value}\n”)
```






## 总结



本节课我们一起学习了如何使用 LangGraph 构建一个自反思的 RAG 系统。我们主要掌握了以下内容：






1.  **主动 RAG 的概念**：让 LLM 参与决策，动态管理检索过程。
2.  **状态机与流程工程**：通过设计清晰的状态转换图来构建复杂逻辑。
3.  **LangGraph 核心操作**：定义状态、实现节点函数、构建图并设置条件边。
4.  **CRAG 工作流的实现**：将“检索-评估-决策-生成”的完整流程编码为可执行的图。

![](img/7bbc8872412b78b5d231191522cb3256_125.png)

这种“流程工程”的思维方式，使得构建包含复杂逻辑和条件分支的 RAG 应用变得直观且易于维护。通过 LangGraph 和 LangSmith，我们不仅能实现强大功能，还能获得出色的可观察性和调试体验。