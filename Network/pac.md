# PAC
[TOC]

PAC 是Proxy Auto-Config[^intro] 的简写

PAC，一个自动代理配置脚本，包含了很多使用 JavaScript 编写的规则，
它能够决定网络流量走默认通道还是代理服务器通道，控制的流量类型包括：HTTP、HTTPS 和 FTP。

## 基础知识
PAC文件里面包含一个JavaScript函数`FindProxyForURL(url, host)`，
函数返回一个字符串，这个字符串就是代理的配置。
函数有两个参数：url和host，url是浏览的地址url全路径，host是这个url中的主机名部分。

return 语句有三种指令[^return]：
1. `DIRECT` 就是直接连接，不通过代理
2. `PROXY http://www.example.com:8080` http代理的主机和端口，指定代理服务器的地址和端口号
3. `SOCKS socks5sample.com:1080` socks5代理的主机和端口，主机也可以用IP表示

一个简单的例子：
```javascript
function FindProxyForURL(url, host)
{
    if (host == "www.mydomain.com") {
        return "DIRECT";
    }
    
    return "PROXY myproxy:80; PROXY myotherproxy:8080; DIRECT";
}
```

上面的 `return "PROXY myproxy:80; PROXY myotherproxy:8080; DIRECT";` 
这里面先走 myproxy:80 代理，如果超时那就再走myotherproxy:8080，
如果这个还走不通？就不走代理使用直接连接。

## PAC 几个内置的函数
1. `isPlainHostName(host)`：判断是否是本地主机
2. `dnsDomainIs(host, domain)`, `localHostOrDomainIs(host, "")`：判断访问主机是否属于某个域和某个域名
3. `isInNet(host, subnet1, subnet2)`：访问IP是否在某个子网内
4. `shExpMatch(host, "")`：主机名判断，正则
5. `url.substring(0, n)`：字符串截取
6. `myIpAddress()`：本地IP

还有一些其他的函数，参考这里：https://www.barretlee.com/blog/2016/08/25/pac-file/


## 一些简单的例子
例子里面[^pac_functions]

```javascript
function FindProxyForURL(url, host) {
// If the hostname matches, send direct.
    if (dnsDomainIs(host, "intranet.domain.com") ||
        shExpMatch(host, "(*.abcdomain.com|abcdomain.com)"))
        return "DIRECT";

// If the protocol or URL matches, send direct.
    if (url.substring(0, 4)=="ftp:" ||
        shExpMatch(url, "http://abcdomain.com/folder/*"))
        return "DIRECT";

// If the requested website is hosted within the internal network, send direct.
    if (isPlainHostName(host) ||
        shExpMatch(host, "*.local") ||
        isInNet(dnsResolve(host), "10.0.0.0", "255.0.0.0") ||
        isInNet(dnsResolve(host), "172.16.0.0", "255.240.0.0") ||
        isInNet(dnsResolve(host), "192.168.0.0", "255.255.0.0") ||
        isInNet(dnsResolve(host), "127.0.0.0", "255.255.255.0"))
        return "DIRECT";

// If the IP address of the local machine is within a defined
// subnet, send to a specific proxy.
    if (isInNet(myIpAddress(), "10.10.5.0", "255.255.255.0"))
        return "PROXY 1.2.3.4:8080";

// DEFAULT RULE: All other traffic, use below proxies, in fail-over order. 
    return "PROXY 4.5.6.7:8080; PROXY 7.8.9.10:8080";
}
```


[^intro]: http://findproxyforurl.com/introduction-to-pac-files/
[^return]: 作者:小拓先森 https://juejin.cn/post/6844903567204024328
[^pac_functions]: http://findproxyforurl.com/pac-functions/


