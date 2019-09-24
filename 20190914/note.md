## 1.Promise中await的问题
```javascript
function fn(){
    return new Promise((resolve,reject)=>{
        resolve([1,2,3]);
    })
}
async function getData(){
    await fn(); 
    console.log(1);
}
getData();
Promise.resolve().then(data=>{
    console.log(2);
});
```

## 2.编码问题
- ASCII -> ANSI(GB2312) -> GBK -> GB18030 -> Unicode -> UTF-8
- base64编码  缺点比以前大
- 编码问题
```javascript
var iconv = require('iconv-lite'); // node中的编码转化
function readGBKText(pathname) {
    var bin = fs.readFileSync(pathname);
    return iconv.decode(bin, 'gbk');
}
```

## 3.Buffer方法
- 分配方法 alloc 、from
- slice toString concat isBuffer length(split)


## 4.核心模块 fs 、event、 path、stream
fs:
- readFile writeFile
- (open read write)
- mkdir access *stat* unlink
- createReadStream  createWriteStream (pipe用法)

event:
- on emit off once (newListener)

path:
- join  resolve  extname  dirname  `__dirname` ` __filename`

  流的类型:
- 1)可读(on('data')，on('end'))   req
- 2)可写 write.end    res
- 3)双工  
- 4)转化流 lib createGzip



## 5.HTTP (tcp用法)
### 1).一个页面从输入url到页面加载完成显示，这个过程做了什么？
- 1). 浏览器通过DNS将url地址解析为ip (如果有缓存直接返回缓存,否则递归解析)    
- 2). 通过DNS解析得到了目标服务器的IP地址后，与服务器建立TCP连接。  
	- ip协议： 选择传输路线,负责找到  
	- tcp协议： 三次握手，分片，可靠传输，重新发送的机制  (tcp)  
- 3).浏览器通过http协议发送请求 (增加http的报文信息)  
- 4).服务器接受请求后，查库，读文件，拼接好返回的http响应   
- 5).浏览器收到html，开始渲染    (浏览器的渲染原理)
- 6).解析html为dom，解析css为css-tree,最终生成render-tree 阻塞渲染  
- 7).遍历渲染树开始布局，计算每个节点的位置大小信息   
- 8).将渲染树每个节点绘制到屏幕  
- 9).加载js文件,运行js脚本  
- 10).reflow (样式)与repaint(位置)  

> OSI协议分层: 
  - 应用层 HTTP,FTP,DNS (与其他计算机进行通讯的一个应用服务，向用户提供应用服务时的通信活动)
  - 传输层 TCP（可靠） UDP 数据传输 (HTTP -> TCP  DNS->UDP)
  - 网络层 IP 选择传输路线 (通过ip地址和mac地址)(使用ARP协议凭借mac地址进行通信)
  - 链路层 网络连接的硬件部分 

### 2).Http与Https的区别：
缺陷： 
    - 1).Http本身不具备加密的功能,HTTP 报文使用明文方式发送
    - 2).无法确认你发送到的服务器就是真正的目标服务器
    - 3).无法阻止海量请求下的 DoS 攻击
    - 4).内容被篡改

- HTTPS 是 HTTP 建立在 SSL/TLS 安全协议上的。 （信息加密，完整校验,身份验证）

### 3).URI和URL的区别
- URI(Uniform Resource Identifier)是统一资源标识符,在某个规则下能把这个资源独一无二标示出来
- URL(Uniform Resource Locator) 统一资源定位符，表示资源的地点


### 4).HTTP1.1版本特性 
- 长链接:建立链接后可以发送多次请求
- 管线化:不用等待可以直接发送下一个请求
- 断点续传:206  bytes

### 5).常见的http状态码
- 200、204、206、301、302、304、400、401、403、404、500、503


### 6).常用的http方法有哪些
- GET 获取资源
- POST 向服务器端发送数据，传输实体主体
- PUT 传输文件
- HEAD 获取报文首部
- DELETE 删除文件
- OPTIONS 询问支持的方法

### 7).Jsonp的原理 (不符合规范的)


### 8).页面的性能优化
![img](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562322737439&di=6940a8a49d168d9690f3732794b01a4e&imgtype=0&src=http%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1555757%2F201812%2F1555757-20181210173358459-380857182.png)

- 缓存
    强制缓存 & 对比缓存
    
    > from memory cache 和 from disk cache
    > 查找优先级:
    >
    > - Service Worker (PWA)  cacheApi 离线缓存 
    - Memory Cache 网页关闭则失效 （几乎所有的请求资源）
    - DiskCache
    - 网络请求  
    对于大文件来说，大概率是不存储在内存中的,当前系统内存使用率高的话，文件优先存储进硬盘
    
- gzip压缩 重复率越高 压缩越高

- 本地存储:localStorage,sessionStorage,session,cookie,indexDB,cacheApi

- CDN: 内容分发网络,实现负载均衡，并且就近返回内容

- defer & async ／ preload & prefetch
  
    - defer 和 async 在网络读取的过程中都是异步解析
    - defer是有顺序依赖的，async只要脚本加载完后就会执行
    - preload 可以对当前页面所需的脚本、样式等资源进行预加载  vue 路由懒加载
    - prefetch 加载的资源一般不是用于当前页面的，是未来很可能用到的这样一些资源

### HTTP相关Header内容
- 缓存Header 
    - 强:Cache-Control && Expires
        - private 客户端可以缓存
        - public 客户端和代理服务器都可以缓存
        - max-age=60 缓存内容将在60秒后失效
        - no-cache 需要使用对比缓存验证数据,强制向源服务器再次验证  (没有强制缓存)
        - no-store 所有内容都不会缓存，强制缓存和对比缓存都不会触发 (不缓存)
    - 对比:Last-Modified & If-Modified-Since  /  ETag & *If-None-Match*
- 跨域:Access-Control
- 压缩:Content-Encoding : gzip
- 范围请求:range
- 防盗链:referer  (防止盗用链接)
- 用户内核:user-agent
- 单主机多域名:host
- 多语言:accept-language
- 文件上传:Content-Type:multipart/form-data
- 文件下载:Content-Description


## 6.KOA
- compose应用
```javascript
let fn = this.arr.reduce((a, b, index) => (...args) =>
    Promise.resolve(a(() => b(...args)))
)(() => {}).then(()=>{
    console.log('ok')
})
```

- async + await
```javascript
 const dispatch = async (index) => {
    if(index === this.arr.length) return;
    return this.arr[index](()=>dispatch(index+1));
};
return dispatch(0);
```



下周 1-4 webpack训练营 周六flutter 两个周六

node进程







## HTTP无状态的
不知道每次谁来了
- cookie 是存放到 浏览器上的 ,服务器可以设置，每此请求时会带上cookie
- cookie 不安全 不能存放敏感信息 


- session 服务端（基于cookie） 服务器的内存中，-》 redis 数据库 (get set)

