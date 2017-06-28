# SDN控制器

控制器是整个SDN网络的核心大脑，负责数据平面资源的编排、维护网络拓扑和状态信息等，并向应用层提供北向API接口。其核心技术包括

- 链路发现和拓扑管理
- 高可用和分布式状态管理
- 自动化部署以及无丢包升级

## 链路发现和拓扑管理

在SDN中通常使用LLDP发现其所控制的交换机并形成控制层面的网络拓扑。

> LLDP（Link Layer Discovery Protocol，链路层发现协议）定义在802.1ab中,它是一个二层协议，提供了一种标准的链路层发现方式。LLDP协议使得接入网络的一台设备可以将其主要的能力，管理地址、设备标识、接口标识等信息发送给接入同一个局域网络的其它设备。当一个设备从网络中接收到其它设备的这些信息时，它就将这些信息以MIB的形式存储起来。这些MIB信息可用于发现设备的物理拓扑结构以及管理配置信息。需要注意的是LLDP仅仅被设计用于进行信息通告，它被用于通告一个设备的信息并可以获得其它设备的信息，进而得到相关的MIB信息。它不是一个配置、控制协议，无法通过该协议对远端设备进行配置，它只是提供了关于网络拓扑以及管理配置的信息。

发现过程为

- 控制器通过`Packet_out`包向所有与之相连的交换机发送LLDP包，同时要求交换机发出广播包（为了发现非OpenFlow交换机）
- 这些交换机收到消息后会向其所有端口发送LLDP数据包
- OpenFlow交换机收到LLDP数据包后通过`Packet_in`消息将链路信息发送给控制器
- 控制器根据收到的`Packet_in`消息创建网络拓扑

## 下发流表

SDN控制器通过南向接口（如OpenFlow）向SDN交换机下发流表，有两种方式

- 主动下发：控制器在数据包到达OpenFlow交换机之前就已经下发流表。这种方式不存在控制器的瓶颈问题，是主流的设计
- 被动下发：OpenFlow交换机收到第一个数据包并且没有发现与之匹配的流表项时发送给控制器处理，这种方式增加了流表设置的时间，也增加了控制器的处理负担，使控制器容易成为瓶颈

## 北向接口

北向接口是直接面向业务应用服务的，其设计密切联系业务应用需求，具有多样化的特征。北向接口目前还没有统一的标准，各家的实现均不相同，但共同点是都提供了一个开放的API，方便业务应用编程。

## 高可用

为了保证逻辑集中控制器的高可用性，需要多台控制器形成分布式集群，避免单点故障。由于控制器掌握着全网的网络设备，通过分布式协作的方式确保网络状态的一致性尤为重要。

## 开源控制器

- OpenDaylight
- ONOS
- NOX & POX
- OpenContrail
- Ryu
- Floodlight
- Beacon