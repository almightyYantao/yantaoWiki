---
title: node_exporter 连接数指标
date: 2024-01-12
tags: ['node_exporter','prometheus','连接数指标']  
categories: ['prometheus','node_exporter']
---

## node-export中连接数相关指标

### tcp相关指标

|名称|类型|单位|说明|
|-|-|-|-|
|node_netstat_Tcp_InErrs|counter|报文数|TCP 接收的错误报文数|
|node_netstat_Tcp_InSegs|counter|报文数|TCP 接收的目前所有建立连接的错误报文数|
|node_netstat_Tcp_OutSegs|counter|报文数|TCP 发送的报文数（包括当前连接的段但是不包括重传的段)|
|node_netstat_Tcp_RetransSegs|counter|报文数|TCP 重传报文数|
|node_netstat_Tcp_CurrEstab|counter|报文数|当前状态为 ESTABLISHED 或 CLOSE-WAIT 的 TCP 连接数|
|node_netstat_Tcp_ActiveOpens|counter|报文数|已从 CLOSED 状态直接转换到 SYN-SENT 状态的 TCP 连接数|
|node_netstat_Tcp_PassiveOpens|counter|报文数|已从 LISTEN 状态直接转换到 SYN-RCVD 状态的 TCP 平均连接数|
|node_netstat_TcpExt_ListenDrops|counter|报文数|监听队列连接丢弃数|
|node_netstat_TcpExt_ListenOverflows|counter|报文数|监听 socket 的队列溢出|
|node_netstat_TcpExt_SyncookiesFailed|counter|报文数|接收的无效的 SYN cookies 的数量|
|node_netstat_TcpExt_SyncookiesRecv|counter|报文数|接收的 SYN cookies 的数量|
|node_netstat_TcpExt_SyncookiesSent|counter|报文数|发送的 SYN cookies 的数量|
|node_sockstat_TCP_alloc|Graph|报文数|已分配（已建立、已申请到sk_buff）的TCP套接字数量|
|node_sockstat_TCP_inuse|Graph|报文数|正在使用（正在侦听）的TCP套接字数量|
|node_sockstat_TCP_mem|Graph|报文数|TCP 套接字缓冲区使用量|
|node_sockstat_TCP_orphan|Graph|报文数|无主（不属于任何进程）的TCP连接数（无用、待销毁的TCP socket数）|
|node_sockstat_TCP_tw|Graph|报文数|等待关闭的TCP连接数|
|node_sockstat_TCP_mem_bytes|Graph|bytes|TCP 套接字缓冲区比特数|

### udp相关指标

|名称|类型|单位|说明|
|-|-|-|-|
|node_sockstat_UDPLITE_inuse|Graph|报文数|正在使用的 UDP-Lite 套接字数量|
|node_sockstat_UDP_inuse|Graph|报文数|正在使用的 UDP 套接字数量|
|node_sockstat_UDP_mem|Graph|报文数|UDP 套接字缓冲区使用量|
|node_sockstat_UDP_mem_bytes|Graph|bytes|UDP 套接字缓冲区比特数|
|node_netstat_Udp_InDatagrams|Graph|报文数|接收的 UDP 数据包|
|node_netstat_Udp_OutDatagrams|Graph|报文数|发送的 UDP 数据包|
|node_netstat_Udp_InErrors|Graph|报文数|本机端口未监听之外的其他原因引起的 UDP 入包无法送达(应用层)的数量|
|node_netstat_Udp_NoPorts|Graph|报文数|未知端口接收 UDP 数据包的数量|
|node_netstat_UdpLite_InErrors|Graph|报文数|本机端口未监听之外的其他原因引起的 UDP-Lite 入包无法送达(应用层)的数量|

### udp与udp-lite的区别

传统的UDP协议是对其载荷（Payload）进行完整的校验的，如果其中的一些位（哪怕只有一位）发生了变化，那么整个数据包都有可能被丢弃，在某些情况下，丢掉这个包的代价是非常大的，尤其当包比较大的时候。在UDP-Lite协议中，一个数据包到底需不需要对其载荷进行校验，或者是校验多少位都是由用户控制的（leeming注释：这是这种可选择性，其实udp_lite的代码实现是比udp复杂的，though 字面上有个lite），并且UDP-Lite协议就是用UDP协议的Length字段来表示其Checksum Coverage的，所以当UDP-Lite协议的Checksum Coverage字段等于整个UDP数据包（包括UDP头和载荷）的长度时，UDP-Lite产生的包也将和传统的UDP包一模一样

### ICMP

Internet Control Message Protocol，ICMP是网路协议族的核心协议之一。它用于TCP/IP网络中发送控制消息，提供可能发生在通信环境中的各种问题反馈，通过这些信息，令管理者可以对所发生的问题作出诊断，然后采取适当的措施解决。

ICMP通常用于返回的错误信息或是分析路由。ICMP错误消息总是包括了源数据并返回给发送者。 ICMP错误消息的例子之一是TTL值过期。每个路由器在转发数据报的时候都会把ip包头中的TTL值减一。如果TTL值为0，“TTL在传输中过期”的消息将会回报给源地址。

|名称|类型|单位|说明|
|-|-|-|-|
|node_netstat_Icmp_InErrors|Graph|报文数|接收的 ICMP 错误的报文（例如ICMP校验和错误、长度错误等）|
|node_netstat_Icmp_InMsgs|Graph|报文数|接收的报文数|
|node_netstat_Icmp_OutMsgs|Graph|报文数|发送的报文数|

### Sockstat的其他指标

|名称|类型|单位|说明|
|-|-|-|-|
|node_sockstat_sockets_used|Graph|报文数|使用的所有协议套接字总量|
|node_sockstat_FRAG_inuse|Graph|报文数|正在使用的 Frag 套接字数量|
|node_sockstat_FRAG_memory|Graph|报文数|使用的 Frag 缓冲区|
|node_sockstat_RAW_inuse|Graph|报文数|正在使用的 Raw 套接字数量|
