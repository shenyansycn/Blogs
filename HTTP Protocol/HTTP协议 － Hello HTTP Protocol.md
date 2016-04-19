# HTTP协议 － Hello HTTP Protocol
> 作者 : shenyansycn
> Github : https://github.com/shenyansycn
> CSDN Blog : http://blog.csdn.net/shenyansycn
> 此文同步在[Github](https://github.com/shenyansycn/Blogs/blob/master/HTTP%20Protocol/HTTP%E5%8D%8F%E8%AE%AE%20%EF%BC%8D%20Hello%20HTTP%20Protocol.md)和[CSND Blog](http://blog.csdn.net/shenyansycn/article/details/51191801)上发布

本系列课程主要面向有HTTP基础的开发者，需要了解基础知识的请访问[这里](https://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)。
本章会一个HTTP请求的例子来作为接入点。在讲解例子前，我们需要做些准备工作。包括：

* [Chrome](https://www.google.com/chrome/browser/desktop/index.html?utm_source=google&utm_medium=sem&utm_campaign=1001342|ChromeWin10|GLOBAL|en|Hybrid|Text|BKWS~TopKWDS-Exact)：Google浏览器，支撑Postman插件
* [Chrome Plugin Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?utm_source=chrome-ntp-launcher)：可以模拟HTTP请求，包括：GET、POST等
* [Golang 1.6](https://golang.org/)：服务端开发语言，更好理解HTTP协议
* [IntelliJ IDEA 15 CE](https://www.jetbrains.com/idea/)：服务端开发IDE，用户测试HTTP
* [IntelliJ IDEA 15 CE Plugin GO](https://github.com/go-lang-plugin-org)：服务端用GO语言进行开发，需要这个插件。

> 具体安装以及配置方法并不是本课程涉及的内容，请Google自行解决。

## Postman

![](https://raw.githubusercontent.com/shenyansycn/ImageBackup/master/Hello_Protocol_Postman_normal.png)

[Postman](http://www.getpostman.com/)官方的定位是“Modern software is built on APIs. Postman helps you develop APIs faster.
“。有Chrome App和Mac App，都是免费的。本系列课程主要用的就是测试API，通过测试API来学习HTTP Protocol。Postman还有其他重要的功能，详细信息可以看[官方文档](https://www.getpostman.com/docs/)。更深入的功能不在本系列文章中讨论。

Postman相当于一个发送请求的客户端，在使用之前，我们需要创建一个服务端。本课程使用的是[Golang](https://golang.org/)语言，所用的IDE是[IntelliJ IDEA 15 CE](https://www.jetbrains.com/idea/)，通过[IntelliJ IDEA 15 CE Plugin GO](https://github.com/go-lang-plugin-org)插件来实现Golang语言的开发，具体开发配置请自行Google。测试代码在Golang语言和IDE相关配置完成的情况下可以直接使用。主要代码如下：

``` java
func main() {
	fmt.Println("Start")
	http.HandleFunc("/appFeedback", appFeedback)
	fmt.Println("Linstenering")
	err := http.ListenAndServe(":9898", nil)
	if err != nil {
		fmt.Println("ListenerAndServer: ", err)
	} else {
		fmt.Println("start server success")
	}
	fmt.Println("over")
}
```
`http.HandleFunc("/appFeedback", appFeedback)`
是服务端访问的路径。
`err := http.ListenAndServe(":9898", nil)`
是服务端监听的接口
`http.HandleFunc`第二个参数`appFeedback`是服务端处理请求的函数名。具体内容如下：


``` java
func appFeedback(w http.ResponseWriter, r *http.Request) {
	//go process(w, r)
	process(w, r)
}

func process(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()
	fmt.Println("------------------------------------------------------------------------------------------------------")
	fmt.Println("Method = ", r.Method)
	fmt.Println("Host = ", r.URL.Host)
	fmt.Println("path = ", r.URL.Path)

	fmt.Println("------------------------------------------------------------------------------------------------------")
	fmt.Println("print URL Params")
	for k, v := range r.Form {
		fmt.Println("	 ", k, ": ", v)
	}

	fmt.Println("------------------------------------------------------------------------------------------------------")
	fmt.Println("print Headers")
	for key, value := range r.Header {
		fmt.Println("	", key, ": ", value)
	}

	fmt.Println("------------------------------------------------------------------------------------------------------")
	fmt.Println("scheme = " + r.URL.Scheme)
	fmt.Println("UserAgent = ", r.UserAgent())

	fmt.Println("------------------------------------------------------------------------------------------------------")
	fmt.Println("print POST Body")
	bodyData, _ := ioutil.ReadAll(r.Body)
	fmt.Println("	bodyData: ", string(bodyData))
	//}
	fmt.Println("------------------------------------------------------------------------------------------------------")

	var respone AppFeedbackRespone
	respone.Message = "http Method is " + r.Method
	respone.State = "1"
	b, err := json.Marshal(respone)
	var backString string = string(b)
	fmt.Println("back String = ", backString)
	if (err != nil) {
		fmt.Println("json err = ", err)
	} else {
		w.Write(b)
	}
}
```

调试信息都是用英文标示，方便大家阅读和调试。在服务端调用起来后，我们就可以通过Postman访问服务端来测试HTTP协议了。

在Postman的`Enter request URL`的地方填写`http://localhost:9898/appFeedback`，点击`send`，返回内容如下图：

![](https://github.com/shenyansycn/ImageBackup/blob/master/Hello_Protocol_Postman_request.png?raw=true)

服务端打印的内容如下：

```
Start
Linstenering
------------------------------------------------------------------------------------------------------
Method =  GET
Host =  
path =  /appFeedback
------------------------------------------------------------------------------------------------------
print URL Params
------------------------------------------------------------------------------------------------------
print Headers
	 Connection :  [keep-alive]
	 Cache-Control :  [no-cache]
	 User-Agent :  [Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Postman/4.1.1 Chrome/47.0.2526.73 Electron/0.36.2 Safari/537.36]
	 Postman-Token :  [53487ea8-7319-6d96-3a1e-4b0a0a497603]
	 Accept :  [*/*]
	 Accept-Encoding :  [gzip, deflate]
	 Accept-Language :  [zh-CN]
------------------------------------------------------------------------------------------------------
scheme = 
UserAgent =  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Postman/4.1.1 Chrome/47.0.2526.73 Electron/0.36.2 Safari/537.36
------------------------------------------------------------------------------------------------------
print POST Body
	bodyData:  
------------------------------------------------------------------------------------------------------
back String =  {"state":"1","message":"http Method is GET"}
```

OK，一个简单的HTTP请求的客户端和服务端的Demo就是这些了，上面相关信息具体含义，以后课程逐步来详细讲解。
服务端的代码在[这里](https://github.com/shenyansycn/Golang_Http_Demo)，main.go的文件。

© 2016，[Shenyansycn](https://github.com/shenyansycn)。 版权所有。



