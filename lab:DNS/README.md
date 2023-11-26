# lab:DNS
## 实验目的
实现一个搭建在OpenNetLab上的DNS服务器,使其可以拦截特定域名,并返回域名对应的IP地址.
## 实验环境
- OpenNetLab
- VMware虚拟机Ubuntu20.04
## 实验代码
在``server.py``中补充的代码如下:
```python
def recv_callback(self, data: bytes):
        """
        TODO: 处理DNS请求，data参数为DNS请求数据包对应的字节流
        1. 解析data得到构建应答数据包所需要的字段
        2. 根据请求中的domain name进行相应的处理:
            2.1 如果domain name在self.url_ip中，构建对应的应答数据包，发送给客户端
            2.2 如果domain name不再self.url_ip中，将DNS请求发送给public DNS server
        """
        # resolve data to DNS frame recvdp
        recvdp = DNSPacket(data)

        # 判断recvdp是否是查询报文
        if recvdp.QR == 0:
            if recvdp.name in self.url_ip:
                if self.url_ip[recvdp.name] == "0.0.0.0":
                    # 拦截
                    senddp = recvdp.generate_response(self.url_ip[recvdp.name], True)
                else:
                    # 不拦截
                    senddp = recvdp.generate_response(self.url_ip[recvdp.name], False)
            else:
                # send query message to public DNS server
                self.server_socket.sendto(recvdp.data, self.name_server)
                # receive data from public server
                data, addr = self.server_socket.recvfrom(1024)
                senddp = data
            # send data to client
            self.send(senddp)
```
## 对补充代码的解释
1. 首先判断DNS请求是否为查询报文,如果是查询报文,则进行下一步处理,否则不处理.
2. 判断查询报文中的域名是否在``self.url_ip``中,如果在,则判断其对应的IP地址是否为``0.0.0.0``,如果是,则构建拦截报文,否则构建不拦截报文.
3. 如果查询报文中的域名不在``self.url_ip``中,则将查询报文发送给公共DNS服务器,并接收公共DNS服务器返回的数据,构建应答报文,并发送给客户端.
## 实验结果
![](本地测试.png)  
![](在线测试.png)  
## 实验总结
本次实验主要是实现一个DNS服务器,使其可以拦截特定域名,并返回域名对应的IP地址.实验中,我首先实现了一个DNS服务器,使其可以将域名解析为IP地址,然后在此基础上,实现了拦截特定域名的功能.实验中,我遇到的主要问题是不熟悉``socket``库的使用,在查阅了相关资料后,我成功解决了这个问题.通过本次实验,我对DNS服务器的工作原理有了更深入的了解,并且对socket编程有了更深入的了解.
