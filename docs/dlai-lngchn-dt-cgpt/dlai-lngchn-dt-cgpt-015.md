# 015：第6集 转换 🛠️

![](img/a32ab69211f39eadd0da37920a2e5c99_0.png)


![](img/a32ab69211f39eadd0da37920a2e5c99_4.png)

在本节课中，我们将要学习大型语言模型（LLM）在**格式转换**方面的强大能力。我们将探索如何利用LLM进行翻译、语气转换、格式转换以及拼写和语法检查，并通过具体的代码示例来演示这些功能。

![](img/a32ab69211f39eadd0da37920a2e5c99_6.png)

---


![](img/a32ab69211f39eadd0da37920a2e5c99_10.png)

## 概述 📋


大型语言模型擅长处理各种格式转换任务。它们可以输入一种语言的文本并将其转换为另一种语言，帮助修正拼写和语法，甚至在不同数据格式（如HTML、JSON）之间进行转换。本节课将通过一系列实例，展示如何利用简单的提示词来实现这些转换功能。

---


## 翻译任务 🌐



上一节我们介绍了LLM的基本转换能力，本节中我们来看看具体的翻译应用。大型语言模型在大量多语言文本上训练，因此具备出色的翻译能力。


以下是一个简单的翻译示例，将英文翻译成西班牙文：



```python
prompt = "翻译以下英语文本到西班牙语：\n\nHello, I would like to order a blender."
response = get_completion(prompt)
print(response)
```
输出：`Hola, me gustaría ordenar una licuadora.`



模型不仅能识别语言，还能进行多语言同时翻译。例如，将同一句话翻译成法语、西班牙语和“海盗英语”：


```python
prompt = """翻译以下文本为法语、西班牙语和英语海盗语：
I want to order a basketball.
"""
response = get_completion(prompt)
print(response)
```

在某些语言中，翻译会根据说话者与听者的关系（正式或非正式）而改变。我们可以通过提示词指导模型进行相应转换：


```python
prompt = """将以下文本翻译成西班牙语，分别使用正式和非正式形式：
Would you like to order a pillow?
"""
response = get_completion(prompt)
print(response)
```


假设我们负责一家跨国电商公司的IT支持，需要处理来自不同语言的用户反馈。我们可以构建一个通用翻译器：



以下是构建通用翻译器的步骤代码：





```python
user_messages = [
    "La performance du système est plus lente que d'habitude.",  # 法语
    "Mi monitor tiene píxeles que no se iluminan.",              # 西班牙语
    "Il mio mouse non funziona",                                 # 意大利语
    "Mój klawisz Ctrl jest zepsuty",                             # 波兰语
    "我的屏幕在闪烁"                                               # 中文
]
for issue in user_messages:
    prompt = f"告诉我这是什么语言，然后将其翻译成英语和韩语：```{issue}```"
    lang = get_completion(prompt)
    print(f"原始消息: {issue}")
    print(f"检测到的语言与翻译: {lang}")
    print()
```



---



## 语气转换 ✍️



上一节我们探讨了语言翻译，本节中我们来看看如何转换文本的语气。写作可以根据预期受众进行调整，例如，给同事的邮件与给朋友的短信语气截然不同。

以下是将俚语转换为正式商务信函的例子：



```python
prompt = """将以下俚语翻译成商务信函：
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```


---


## 格式转换 🔄



ChatGPT非常擅长在不同格式之间进行转换，例如将Python字典转换为HTML表格。



以下是将Python字典数据转换为HTML表格的示例：







```python
data = {
    "employees": [
        {"name": "John Doe", "email": "john@example.com"},
        {"name": "Jane Smith", "email": "jane@example.com"},
        {"name": "Bob Johnson", "email": "bob@example.com"}
    ]
}
prompt = f"""将以下Python字典转换为带列标题和标题的HTML表格：
{data}
"""
response = get_completion(prompt)
print(response)
# 使用IPython的Display函数来渲染HTML
from IPython.display import display, HTML
display(HTML(response))
```



---



## 拼写与语法检查 ✅


拼写和语法检查是LLM非常流行的应用场景，尤其对非母语写作者帮助巨大。



以下是一些常见语法和拼写问题的纠正示例：





```python
sentences = [
    "The girl with the black and white puppies have a ball.",  # 主谓不一致
    "Yolanda has her notebook.",                               # 正确
    "Its going to be a long day. Do you think is going to rain?", # 拼写错误
    "Their goes my freedom. There going to bring they’re suitcases." # 多个错误
]
for sentence in sentences:
    prompt = f"""校对并更正以下文本。如果未发现错误，请说“未发现错误”：
    ```{sentence}```
    """
    response = get_completion(prompt)
    print(f"原始: {sentence}")
    print(f"更正后: {response}\n")






```
我们还可以对更长的文本（如产品评论）进行校对和语气增强：
```python
review = """我需要为我的生日给女儿买一个毛绒熊猫玩具。\
她很喜欢，走到哪都带着。就是有点贵，\
但考虑到它的做工，我认为物有所值。快递提前一天就到了，\
所以在女儿生日前我就拿到了。"""



prompt = f"""校对并修正此评论，使其更具吸引力，遵循APA格式，面向高级读者。
以Markdown格式输出。
文本：```{review}```
"""
response = get_completion(prompt)
print(response)
```



为了更直观地看到修改，我们可以比较原始文本和修正后文本的差异（例如使用Python的`difflib`库）。


---



## 总结 🎯



本节课中我们一起学习了大型语言模型在**转换任务**上的多种应用：
1.  **翻译**：在不同语言间进行准确转换，并能处理正式与非正式语气。
2.  **语气转换**：根据受众调整文本的正式程度。
3.  **格式转换**：在不同数据结构（如字典到HTML）间进行转换。
4.  **拼写与语法检查**：识别并修正文本中的错误，并能按要求重写以增强表达。


![](img/a32ab69211f39eadd0da37920a2e5c99_126.png)

通过精心设计的提示词，我们可以轻松引导模型完成这些复杂的转换任务，极大地提升内容处理的效率和质量。在下一节课中，我们将学习如何使用较短的提示来**扩展**生成更长的文本。