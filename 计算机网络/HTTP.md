* [HTTP是什么](#http是什么)
* [HTTP特性](#http特性)
* [HTTP请求报文](#http请求报文)
* [HTTP响应报文](#http响应报文)
* [参考文献](#参考文献)

# HTTP是什么
HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

# HTTP特性
- HTTP构建于TCP/IP协议之上，默认端口号是80
- HTTP是一个属于应用层的协议
- HTTP是无连接无状态的

# HTTP请求报文
HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：**状态行、请求头、空行和请求数据**。类似于下面这样：

```
<method> <request-URL> <version>
<headers>

<entity-body>
```

HTTP定义了与服务器交互的不同方法，最基本的方法有4种，分别是`GET`，`POST`，`PUT`，`DELETE`。

HTTP请求方法我这里只讨论GET方法与POST方法。
- GET方法（幂等——幂等的意味着对同一URL的多个请求应该返回同样的结果）：
**GET方法是默认的HTTP请求方法，我们日常用GET方法来提交表单数据**，然而用GET方法提交的表单数据只经过了简单的编码，同时它将作为URL的一部分向Web服务器发送，因此，如果使用GET方法来提交表单数据就存在着安全隐患。例如：
`Http://127.0.0.1/login.jsp?Name=zhangshi&Age=30&Submit=%cc%E+%BD%BB`
从上面的URL请求中，很容易就可以辩认出表单提交的内容。（？之后的内容）另外由于**GET方法提交的数据是作为URL请求的一部分所以提交的数据量不能太大**。

    ```
    GET /books/?sex=man&name=Professional HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
    Gecko/20050225 Firefox/1.0.1
    Connection: Keep-Alive
    ```

- POST方法：
POST方法是GET方法的一个替代方法，它**主要是向Web服务器提交表单数据，尤其是大批量的数据**。POST方法克服了GET方法的一些缺点。**通过POST方法提交表单数据时，数据不是作为URL请求的一部分而是作为标准数据传送给Web服务器**，这就克服了GET方法中的信息无法保密和数据量太小的缺点。因此，出于安全的考虑以及对用户隐私的尊重，通常表单提交时采用POST方法。**POST方法可用于文件上传**。

    ```
    POST / HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
    Gecko/20050225 Firefox/1.0.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 40
    Connection: Keep-Alive
    
    sex=man&name=Professional  
    ```


# HTTP响应报文
HTTP 响应与 HTTP 请求相似，HTTP响应也由3个部分构成，分别是：**状态行、响应头(Response Header)、空行和响应正文**。**状态行由协议版本、数字形式的状态代码、及相应的状态描述**，各元素之间以空格分隔。

状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:
- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

常见的状态码有如下几种：
- 200 OK 客户端请求成功
- 301 Moved Permanently 请求永久重定向
- 302 Moved Temporarily 请求临时重定向
- 400 Bad Request 由于客户端请求有语法错误，不能被服务器所理解
- 404 Not Found 请求的资源不存在，例如，输入了错误的URL
- 500 Internal Server Error 服务器发生不可预期的错误，导致无法完成客户端的请求


```
HTTP/1.1 200 OK

Server:Apache Tomcat/5.0.12
Date:Mon,6Oct2003 13:23:42 GMT
Content-Length:112

<html>...
```

在浏览器地址栏键入URL，按下回车之后会经历以下流程：
1. 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
2. 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;
3. 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次握手的第三个报文的数据发送给服务器;
4. 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
5. 释放 TCP连接;
6. 浏览器将该 html 文本并显示内容;


# 参考文献
[关于HTTP协议，一篇就够了](https://www.jianshu.com/p/80e25cb1d81a)   
[HTTP协议](https://hit-alibaba.github.io/interview/basic/network/HTTP.html)