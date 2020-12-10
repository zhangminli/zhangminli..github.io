---
title: OVS Tunnel Options
date: 2020-12-06 16:08:13
tags: OVS
categories: OVS
---

- **options: remote_ip**

  String类型，远程隧道端口。必须是个单播地址ipv4或者ipv6，或者是一个关键词flow。如果设置成flow，即remote_ip=flow则flow这个动作(set_field)必须指定tun_dst或tun_ipv6_dst为远程隧道端口的IP。

- **options: local_ip**

  String类型，本地隧道端口。必须是个单播地址ipv4或者ipv6，或者是一个关键词flow。如果设置成flow，即local_ip=flow则flow这个动作(set_field)必须指定tun_src或者tun_ipv6_src为本地隧道端口的IP。

  <!-- more -->

- **options: in_key**

  String类型，隧道接收的包，包的key值含以下三者之一：0，包中如果没有key或者这个key是0，则表示没有这个属性；一个确定的24bit(Geneve，VXLAN，LISP)，32bit（GRE），64bit（STT）数字，隧道会接收以上指定key的包；关键字flow，隧道会接收任意key的包，key会被替换成tun_id，并使用tun_id去匹配流表。

- **options: out_key**

  String类型，隧道发送的包，包的key值被设置为：0，包如果被设置为0，则发送的包不会带有该属性，等同于不设置改属性；一个确定的24bit(Geneve，VXLAN，LISP)，32bit（GRE），64bit（STT）数字，隧道发送的包会带上该key；关键字flow，隧道会设置key值。

- **options: key**

  String类型，如果in_key和out_key是同一数值，则可以合并为该属性。

- **options: tos**

  String类型，在封包的时候添加进ToS字段。

- **options: ttl**

  String类型，可以设置为inherit、数字、不设置，TTL字段会被设置在封装包中，该属性表示该包在路由前最大经过的网段数量。inherit表示从inner packet(必须要是ipv4或ipv6)中拷贝TTL字段，不设置则默认为64。

- **options: df_default**

  布尔类型，如果为true表示不分包bit会被设置在封装包头中，去让MTU发现。

- **Tunnel Options: vxlan only**

  - **options: exts**—以逗号为分隔的vxlan扩展功能，目前支持的属性有：gbp，VXLAN-GBP允许传输一个包的组策略上下文在vxlan隧道中。

- **Tunnel Options: gre, ipsec_gre, geneve, vxlan**

  - **options: csum** — String类型，只可能是True、False。在发包中计算封包头的校验，GRE默认支持，vxlan和geneve则需要kernel > 4.0