---
layout: post
title: "ETCD命令行标志和环境变量配置参数"
subtitle: ""
date: 2018-06-08
author: LuJiangBo
tags: etcd 
keyword: etcd
finished: false
---
<style>
table th:first-of-type {
    width: 200px;
}
table th:nth-of-type(2) {
    width: 600px;
}
</style>

etcd可以通过命令行标志和环境变量进行配置。 在命令行上设置的选项优先于来自环境的选项。
flag --my-flag的环境变量格式为ETCD_MY_FLAG。 它适用于所有标志。
官方的etcd端口为2379，客户端请求为2380，对等通信为2380。 etcd端口可以设置为接受TLS流量，非TLS流量或者TLS流量和非TLS流量。
要在Linux中启动时使用自定义设置自动启动etcd，强烈建议使用systemd单元。

## Member flags

| family        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --data-dir    | 数据目录|${name}.etcd|ETCD_DATA_DIR|
| --wal-dir     | 专用wal目录的路径。如果这个标志被设置，etcd会将WAL文件写入walDir而不是dataDir。这允许使用专用磁盘，并有助于避免记录和其他IO操作之间的竞争|""|ETCD_WAL_DIR|
| --snapshot-count        | 触发快照到磁盘的已提交事务数|100000|ETCD_SNAPSHOT_COUNT|
| --heartbeat-interval        | 心跳间隔的时间（以毫秒为单位）|100|ETCD_HEARTBEAT_INTERVAL|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --name        | 这个成员的可读名称|default|ETCD_NAME|

