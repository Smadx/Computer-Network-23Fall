# lab:GBN
## 实验目的
实现一个搭建在OpenNetLab上的使用GBN协议的发送端。这个发送端需要将一个字符串中的每个字符封装为分组发送给接收端，并且遵循GBN协议
## 实验环境
- OpenNetLab
- VMware虚拟机Ubuntu20.04
## 实验代码
在``sender.py``中补充的代码如下:
```python
def run(self, env: Environment):
    """
    TODO: 
    （1）检查滑动窗口是否已满，来产生分组并发送（发送滑动窗口内所有可以发送的分组）
    （2）每发送一个分组，保存该分组在缓冲区中，表示已发送但还未被确认
    （3）记得在规定的时机重置定时器: self.timer.restart(self.timeout)
    """

    """
    通过`self.finish_channel.get()`获取状态
    即当`self.finish_channel.put(True)`时发送端模拟结束
    """    
    while self.seqno < self.window_size and self.absno < len(self.message):
        packet = self.new_packet(self.seqno, self.message[self.absno])
        self.send_packet(packet)
        self.outbound.append(packet)
        self.seqno += 1
        self.absno += 1
    # 重置定时器
    self.timer.restart(self.timeout)
    # 等待结束发送过程的信号
    yield self.finish_channel.get()
def put(self, packet: Packet):
    """从接收端收到ACK"""
    ackno = packet.packet_id

    """
    TODO: 
    （1）检查收到的ACK
    （2）采取累积确认，移动滑动窗口，并发送接下来可以发送的分组
    （3）重置定时器: self.timer.restart(self.timeout)
    （4）检查是否发送完message，若发送完毕则告知结束: self.finish_channel.put(True)
    """
    if (ackno - self.seqno_start) % self.seqno_range < self.window_size:
        ran = (ackno - self.seqno_start) % self.seqno_range
        for _ in range(ran + 1):
            self.outbound.popleft()
            self.seqno_start = (ackno + 1) % self.seqno_range
            self.timer.restart(self.timeout)
            if self.absno < len(self.message):
                packet = self.new_packet(self.seqno, self.message[self.absno])
                self.send_packet(packet)
                self.outbound.append(packet)
                self.seqno = (self.seqno + 1) % self.seqno_range
                self.absno += 1
            elif not self.outbound and self.absno == len(self.message):
                self.finish_channel.put(True)
def timeout_callback(self):
    self.dprint("timeout")
    
    """
    TODO: 
    （1）超时重传所有已发送但还未被确认过的分组
    （2）注意这个函数结束会自动重置定时器，不用手动重置
    """
    for packet in self.outbound:
        self.send_packet(packet)
```
## 对补充代码的解释
``run``函数中,首先检查滑动窗口是否已满,如果未满,则产生分组并发送,并将该分组保存在缓冲区中,表示已发送但还未被确认,然后重置定时器. 
``put``函数中,首先检查收到的ACK是否在有效范围内,然后采取累积确认,移动滑动窗口,并发送接下来可以发送的分组,然后重置定时器,最后检查是否发送完message,若发送完毕则告知结束. 
``timeout_callback``函数中,超时重传所有在缓冲区中已发送但还未被确认过的分组.
## 实验结果
![](本地测试.png) 
![](在线测试.png) 
## 实验总结
本次实验主要是实现一个使用GBN协议的发送端.实验中,我首先实现了一个发送端,使其可以将一个字符串中的每个字符封装为分组发送给接收端,并且遵循GBN协议.实验中,我遇到的主要问题是代码效率问题,在优化了循环次数和修改代码后,实验结果得到了改善.通过本次实验,我对GBN协议有了更深入的了解.
