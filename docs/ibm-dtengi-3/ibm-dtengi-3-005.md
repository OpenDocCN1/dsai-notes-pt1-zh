# 005：Python数据工程项目课程 📚



## 第5部分：可选内容 - 使用Requests库进行REST API HTTP请求 🌐

在本节课中，我们将学习如何使用Python的Requests库来处理HTTP协议。我们将回顾这个用于处理HTTP协议的流行库，并概述GET请求和POST请求的基本概念和用法。

### 概述Requests模块



首先，我们来回顾Python中的Requests模块。这是几个可以处理HTTP协议的库之一，其他库还包括HTTPLib和URLLib。



Requests是一个Python库，可以让你轻松地发送HTTP/1.1请求。我们可以通过以下方式导入这个库：

```python
import requests
```

你可以使用`get`方法向`www.ibm.com`发起一个GET请求。我们会得到一个响应对象`r`，它包含了关于请求的信息，比如请求的状态。

```python
r = requests.get('https://www.ibm.com')
```

### 检查响应信息

我们可以使用`status_code`属性查看状态码，状态码200表示请求成功。




```python
print(r.status_code)  # 输出: 200
```



你可以查看请求头。由于GET请求没有请求体，所以请求体为`None`。



```python
print(r.request.body)  # 输出: None
```

你可以使用`headers`属性查看HTTP响应头。这会返回一个包含HTTP响应头的Python字典。

```python
print(r.headers)
```


我们可以查看这个字典的值。例如，使用键`date`可以获取请求发送的日期。键`content-type`指示了响应数据的类型。

```python
print(r.headers['date'])
print(r.headers['content-type'])
```



使用响应对象`r`，我们还可以检查编码。


```python
print(r.encoding)
```

### 处理响应内容



由于内容类型是`text/html`，我们可以使用`text`属性来显示响应体中的HTML内容。我们可以查看前100个字符。


```python
print(r.text[:100])
```

你也可以下载其他内容，更多细节请参阅实验部分。



### 使用GET方法进行查询


你可以使用GET方法来修改查询结果，例如从API检索数据。在实验中，我们将使用`httpbin.org`，这是一个简单的HTTP请求和响应服务。

我们像之前一样向服务器发送一个GET请求。我们有基础URL和路由。我们附加`/get`，这表示我们想要执行一个GET请求。



在GET请求之后，我们有查询字符串。这是统一资源定位符（URL）的一部分，它向Web服务器发送其他信息。查询字符串以一个问号开始，后面跟着一系列参数和值对。


以下是参数和值对的示例：

*   第一个参数名是`name`，值是`Joseph`。
*   第二个参数名是`id`，值是`123`。

每个参数和值对之间用等号分隔。一系列参数对之间用`&`符号分隔。



让我们在Python中完成一个示例：


```python
url_get = 'http://httpbin.org/get'
payload = {'name': 'Joseph', 'id': '123'}
r = requests.get(url_get, params=payload)
```


我们有基础URL，并在末尾附加了`/get`。我们使用字典`payload`，其键是参数名，值是查询字符串的值。然后我们将字典`payload`传递给`get`函数的`params`参数。

我们可以打印出URL，查看其中的名称和值。



```python
print(r.url)  # 输出: http://httpbin.org/get?name=Joseph&id=123
```



由于信息是通过URL发送的，请求体的值为`None`。

```python
print(r.request.body)  # 输出: None
```

我们可以打印出状态码。



```python
print(r.status_code)  # 输出: 200
```

我们可以将响应视为文本查看。


```python
print(r.text)
```



我们可以查看键`content-type`来了解内容类型。由于内容类型是JSON，我们使用`json()`方法格式化它，它会返回一个Python字典。

```python
print(r.headers['content-type'])  # 输出: application/json
data = r.json()
print(data['args'])  # 输出: {'name': 'Joseph', 'id': '123'}
```

键`args`包含了查询字符串的名称和值。

### 理解POST请求

与GET请求类似，POST请求也用于向服务器发送数据。但POST请求将数据放在请求体中发送，而不是放在URL里。

为了在URL中发送POST请求，我们将路由改为`/post`。这个端点期望接收数据，这是一种配置HTTP请求以向服务器发送数据的便捷方式。

我们使用`payload`字典来发起POST请求，使用`post`函数。变量`payload`被传递给参数`data`。



```python
url_post = 'http://httpbin.org/post'
payload = {'name': 'Joseph', 'id': '123'}
r_post = requests.post(url_post, data=payload)
```



### 比较GET和POST请求



比较GET和POST请求响应对象的`url`属性，我们看到POST请求的URL中没有名称和值对。




```python
print("GET URL:", r.url)     # 包含查询参数
print("POST URL:", r_post.url) # 不包含查询参数
```


我们可以比较POST和GET请求的请求体，我们看到只有POST请求有请求体。

```python
print("GET request body:", r.request.body)    # 输出: None
print("POST request body:", r_post.request.body) # 输出: name=Joseph&id=123
```

我们可以查看键`form`来获取POST请求的负载数据。



```python
data_post = r_post.json()
print(data_post['form'])  # 输出: {'name': 'Joseph', 'id': '123'}
```



### 总结


![](img/6fce607ac09d6e177cf3638bdf4c9575_68.png)

本节课中，我们一起学习了如何使用Python的Requests库进行HTTP通信。我们介绍了如何发送GET请求和POST请求，如何检查响应状态码、头部信息和内容，以及如何通过查询字符串（GET）或请求体（POST）向服务器发送数据。理解这些基本操作是进行Web API交互和数据工程任务的重要基础。