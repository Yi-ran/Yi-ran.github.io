---
layout:     post
title:      Timely SIGCOMM 2015
subtitle:   RTT-based Congestion Control
date:       2019-3-27
author:     Yiran
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Congestion Control
---

### 核心思想

第一个RTT-based congestion control在数据中心的应用，认为RTT是个很好的拥塞信号。

### 拥塞信号的信息量

- 传统TCP的ECN   ————> 超过一定阈值才标记，只有0 和 1的变化
- DCTCP的ECN    ————> 更进了一步，将单bit转化为一个RTT内的多bit 但是ECN的方式对多优先级不general：低优先级不被标记不一定没经历拥塞
- 而RTT              ————> 与队列长度很好契合，更general，信息量丰富.  

   **论文通过实验证明：1. RTT directly reflects latency. 2. RTT can be measured accurately in practice.  3. RTT is a rapid, multi-bit signal.**


### 以前的方案为什么不用delay

Datacenter RTTs are difficult to measure at microsecond granularity. This level of precision is easily overwhelmed by host delays such as interrupt processing for acknowledgments.
但是现在的网卡这个功能比较完善了

### 如何利用RTT作为拥塞信号进行拥塞控制
   <img width="500" height="350" src="/img/post-timely-1.jpg"/>


-  ***Rate-based control而不是window-based***: Window-based 的拥塞控制算法容易有burst: TIMELY is rate-based rather than window-based because it gives better control over traffic bursts given the widespread use of NIC offload. The bandwidth-delay product is only a small number of packet bursts in datacenters, e.g., 51us at 10 Gbps is one 64 KB message. In this regime, windows do not provide fine-grained control over packet transmissions

- **Pacing Engine**: The scheduler uses the segment size, flow rate (provided by the rate computation engine), and time of last transmission to compute the send time for the current segment with the appropriate pacing delay

- **Rate Computation**: **采用的是RTT变化的梯度，设置一个high和一个low阈值**
   
   <img width="500" height="350" src="/img/post-timely-2.jpg"/>


   gradient >= 0 : rate超出了链路速度，减小rate

   gradient < 0 : rate小于链路速度，可以增加

   <img width="350" height="850" src="/img/post-timely-3.jpg"/>



- **为什么不直接控制队列长度**：[Kelly et al.](http://www.sigcomm.org/sites/default/files/ccr/papers/2008/July/1384609-1384615.pdf) argue that it is not possible to control the queue size when it is shorter in time than the control loop delay. 数据中心10Gbps，64KB的message， control loop 大约为51us，一个数据包的排队delay为1us

### 参考文献
[TIMELY: RTT-based Congestion Control for the Datacenter](https://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p537.pdf)





