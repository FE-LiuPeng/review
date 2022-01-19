### <center>一更新文--做个简单的复习（标题待定）</center>

### Ajax: 
&nbsp; 次次的重温，不可能只是单纯的做复习吧。
XMLHttpRequest(XHR)； 对象用于对服务器的交互。通过这个玩意可以在不刷新页面的情况下发送请求特定的URL

### Fetch: 


### TCP/IP
- 同源策略 ：  
    &nbsp; &nbsp;是一个重要的安全策略，它用于限制一个origin的文档或者它加载的脚本如何能与另一个源的资源进行交互。它能帮助阻隔恶意文档，减少可能被攻击的媒介。  
    &nbsp; &nbsp;何为同源？  **其最简单的回答就是：协议/域名/端口；**   
准确的话术应该是：协议/<font color=#0099ff >主机</font>/<font color=#0099ff>端口元组</font>；
    - 四元组： 源IP地址、目的IP地址、源端口、目的端口;  
    - 五元组： 源IP地址、目的IP地址、协议号、源端口、目的端口;  
    - 七元组： 源IP地址、目的IP地址、协议号、源端口、目的端口、服务类型、接口索引;

- 同源策略的限制  :
  1. Cookie 、LocalStorage 和 IndexDB无法读取
  2. 无法获取或操作另一个资源的DOM (iframe)
  3. AJAX请求不能发送
   
| URL                                             | 姐果 | 因因                             |
| ----------------------------------------------- | ---- | -------------------------------- |
| http://store.company.com/dir2/other.html        | 同源 | 只有路径不同                     |
| http://store.company.com/dir/inner/another.html | 同源 | 只有路径不同                     |
| https://store.company.com/secure.html           | 失败 | 协议不同                         |
| http://store.company.com:81/dir/etc.html        | 失败 | 端口不同 ( http:// 默认端口是80) |
| http://news.company.com/dir/other.html          | 失败 | 主机不同                         |

- CORS:  
&emsp;跨源资源共享 (CORS)（或通俗地译为跨域资源共享）是一种基于 HTTP 头的机制，该机制通过允许服务器标示除了它自己以外的其它origin（域，协议和端口），这样浏览器可以访问加载这些资源。跨源资源共享还通过一种机制来检查服务器是否会允许要发送的真实请求，该机制通过浏览器发起一个到服务器托管的跨源资源的"预检"请求。在预检中，浏览器发送的头中标示有HTTP方法和真实请求中会用到的头。


    *前端设置origin地址，后端设置是允许被跨域访问通配符或指定地址；*

- **OPTIONS：**  
&emsp;对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME类型 的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨源请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP认证 相关数据）。   

- **简单请求** 

    &emsp;某些请求不会出发Cors预检请求。该术语不属于Fetch（fetch自己里面定义了cors的规范，若请求 **满足所有下述条件**，则该请求可视为“简单请求”；
  1. 使用下列方法之一：
      - GET
      - HEAD
      - POST
  2. 请求的Heder是
      - Accept
      - Accept-Language
      - Content-Language
      - Content-Type 
  3. Content-Type 的值仅限于下列三者之一：
     - text/plain
     - multipart/form-data
     - application/x-www-form-urlencoded

    请求头字段Origin应该标明该请求源于哪里`Origin: http://www.baidu.com`
    > GET /resources/public-data/ HTTP/1.1   
Host: bar.other  
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: en-us,en;q=0.5  
Accept-Encoding: gzip,deflate  
Connection: keep-alive  
Origin: http://www.baidu.com

    &emsp;而响应头部则应该设置`Access-Control-Allow-Origin: *`表明，该资源可以被 任意 外域访问
    >HTTP/1.1 200 OK  
Date: Mon, 01 Dec 2008 00:23:53 GMT  
Server: Apache/2  
Access-Control-Allow-Origin: *  
Keep-Alive: timeout=2, max=100  
Connection: Keep-Alive  
Transfer-Encoding: chunked  
Content-Type: application/xml  

    使用 Origin 和 Access-Control-Allow-Origin 就能完成最简单的访问控制。如果服务端仅允许来自 http://www.baidu.com 的访问，该首部字段的内容如下：
    > Access-Control-Allow-Origin: http://www.baidu.com

- **预检请求**

    &emsp;与上述简单请求不同， 预检请求是要在真实的请求之前发送一个options的请求，已获得服务器返回的是否可以世纪的请求，“预检请求”的使用 可以避免浏览器和服务器产生未预测的影响。  
`现在需要执行一下HTTP请求`
    > const xhr = new XMLHttpRequest();  
xhr.open('POST', 'https://bar.other/resources/post-here/');  
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');  
xhr.setRequestHeader('Content-Type', 'application/xml');  
xhr.onreadystatechange = handler;  
xhr.send('<person><name>Arun</name></person>');

上面的代码使用 POST 请求发送一个 XML 文档，该请求包含了一个自定义的请求首部字段（X-PINGOTHER: pingpong）。另外，该请求的 Content-Type 为 application/xml。因此，该请求需要首先发起“预检请求”。
![1](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/preflight_correct.png)
&emsp;图上所述， 首先发起的OPTIONS带有两个字段：Access-Control-Request-*（Method｜Headers），实际的post请求是不会携带这两个参数的。  
&emsp;检测之后会发送是实际请求；
&emsp;`Access-Control-Max-Age 看文章尾部信息。`

- withCredentials（携带cookie）
    ```js
        // 携带凭证（cookie）
        const invocation = new XMLHttpRequest();
        invocation.withCredentials = true; // 凭证开启
        //响应头部字段： Access-Control-Allow-Credentials: true
    ```
    预检请求和Cookie
    CORS 预检请求不能包含Cookie。预检请求的 响应 必须指定Access-Control-Allow-Credentials: true 来表明可以携带凭据进行实际的请求。
  


#### H5 Q2



### 预检请求字段与其他相关字段：(Response)
|key|value|explain|
|:--------:|:--:|:--:|
Access-Control-Request-Method|POST,GET| 说明服务器允许客户端使用 POST 和 GET 方法发起请求（与 Allow 响应首部类似，但其具有严格的访问控制）。
Access-Control-Request-Headers| X-PINGOTHER, Content-Type|表明服务器允许请求中携带字段 X-PINGOTHER 与 Content-Type
Access-Control-Max-Age|86400|表明该响应的有效时间为86400秒，也就是24小时，有效时间内无需发送第二次预检请求。
Access-Control-Allow-Origin|* \| 单个 \| 列表|可以允许哪些客户端来访问，指可以是*，也可以是某个域名或者用逗号隔开的域名列表。
Access-Control-Expose-Headers|-|浏览器可以访问的一些头部。
Access-Control-Allow-Credentials| boolean|表示是否允许发送Cookie；。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。

### HTTP 请求首部字段：(Request)
|key|value|explain|
|:--------:|:--:|:--:|
Origin| URI |origin 参数的值为源站 URI。它不包含任何路径信息，只是服务器名称; <font color=#ccc>PS:有时候将该字段的值设置为空字符串是有用的，例如，当源站是一个 data URL 时。</font>
Access-Control-Request-Method|method|首部字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器。
Access-Control-Request-Headers|field-name[,field-name]|首部字段用于预检请求。其作用是，将实际请求所携带的首部字段告诉服务器。


Access-Control-Request-Method|

### 参考文献
- [Fetch](https://fetch.spec.whatwg.org/#http-cors-protocol)