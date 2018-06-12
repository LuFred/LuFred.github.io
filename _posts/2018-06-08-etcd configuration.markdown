---
layout: post
title: "ETCD命令行标志和环境变量配置参数"
subtitle: ""
date: 2018-06-08
author: LuJiangBo
tags: etcd 
keyword: etcd
finished: true
---
<style>

table td:first-of-type {
    white-space: nowrap;
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

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --name        | 这个成员的可读名称|default|ETCD_NAME|
| --data-dir    | 数据目录|${name}.etcd|ETCD_DATA_DIR|
| --wal-dir     | 专用wal目录的路径。如果这个标志被设置，etcd会将WAL文件写入walDir而不是dataDir。这允许使用专用磁盘，并有助于避免记录和其他IO操作之间的竞争|""|ETCD_WAL_DIR|
| --snapshot-count        | 触发快照到磁盘的已提交事务数|100000|ETCD_SNAPSHOT_COUNT|
| --heartbeat-interval        | 心跳间隔的时间（以毫秒为单位）|100|ETCD_HEARTBEAT_INTERVAL|
| --election-timeout        | 选举超时的时间（以毫秒为单位）|1000|ETCD_ELECTION_TIMEOUT|
| --listen-peer-urls        | 要监听对等流量的URL列表。该标志告诉etcd接受来自指定方案的对等方的传入请求scheme://IP:port 组合。Scheme可以是http或https。如果将0.0.0.0指定为IP，则etcd会侦听所有接口上的给定端口。如果给出了一个IP地址和一个端口，etcd将监听给定的端口和接口。可以使用多个URL来指定要侦听的多个地址和端口。etcd将响应来自任何列出的地址和端口的请求。|"http://localhost:2380,http://localhost:7001"|ETCD_LISTEN_PEER_URLS|
| --listen-client-urls        | 要监听客户端流量的URL列表。该标志告诉etcd接受来自指定方案的客户端的传入请求scheme://IP:port组合。Scheme可以是http或https。如果将IP指定为0.0.0.0，则etcd会侦听所有接口上的给定端口。如果给出了一个IP地址和一个端口，etcd将监听给定的端口和接口。可以使用多个URL来指定要侦听的多个地址和端口。etcd将响应来自任何列出的地址和端口的请求。| "http://localhost:2379,http://localhost:4001"|ETCD_LISTEN_CLIENT_URLS|
| --max-snapshots        | 要保留的最大快照文件数（0无限制）[Windows用户的默认值是无限的，建议手动清除至5]|5|ETCD_MAX_SNAPSHOTS|
| --max-wals        | 要保留的最大wal文件数量（0无限制）[Windows用户的默认值是无限的，建议手动清除至5]|5|ETCD_MAX_WALS|
| --cors        | 用逗号分隔的CORS起源白名单（跨源资源共享）|none|ETCD_CORS|

## Clustering flags

> --initial前缀标志用于引导（静态引导，发现服务引导或运行时重新配置）新成员，并在重新启动现有成员时被忽略。  
> --discovery在使用发现服务时需要设置前缀标志

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --initial-advertise-peer-urls        | 此成员的对等URL列表以通告给群集的其余部分。这些地址用于在群集周围传送etcd数据。至少有一个必须可路由到所有集群成员。这些URL可以包含域名。|"http://localhost:2380,http://localhost:7001"|ETCD_INITIAL_ADVERTISE_PEER_URLS|
| --initial-cluster        | 引导的初始群集配置。|"default=http://localhost:2380,<br>default=http://localhost:7001"|ETCD_INITIAL_CLUSTER|
| --initial-cluster-state        | 初始群集状态 ("new" or "existing")。设置new为在初始静态或DNS自举期间存在的所有成员。如果此选项设置为existing，则etcd将尝试加入现有群集。如果设置了错误的值，etcd将尝试启动但安全失败。|"new"|ETCD_INITIAL_CLUSTER_STATE|
| --initial-cluster-token        | 在引导期间，用于etcd集群的初始群集令牌。|"etcd-cluster"|ETCD_INITIAL_CLUSTER_TOKEN|
| --advertise-client-urls        | 此成员的客户端URL列表以通告给群集的其余部分。这些URL可以包含域名。|"http://localhost:2379,http://localhost:4001"|ETCD_ADVERTISE_CLIENT_URLS|
| --discovery        | 用于引导群集的发现URL。|none|ETCD_DISCOVERY|
| --discovery-srv        | 用于引导群集的DNS srv域。|none|ETCD_DISCOVERY_SRV|
| --discovery-fallback        | 发现服务失败时的预期行为 ("exit" or "proxy")。|"proxy"|ETCD_DISCOVERY_FALLBACK|
| --discovery-proxy        | 用于流量发现服务的HTTP代理。|none|ETCD_DISCOVERY_PROXY|
| --strict-reconfig-check        | 拒绝会导致法定人数丢失的重新配置请求|false|ETCD_STRICT_RECONFIG_CHECK|

## Proxy Flags

> --proxy前缀标志配置etcd以代理模式运行。  

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --proxy        | 代理模式设置 ("off", "readonly" or "on").|off|ETCD_PROXY|
| --proxy-failure-wait        | 在重新考虑代理请求之前，时间（以毫秒为单位），端点将保持失败状态。|5000|ETCD_PROXY_FAILURE_WAIT|
| --proxy-refresh-interval        | 端点刷新间隔的时间（以毫秒为单位）。|30000|ETCD_PROXY_REFRESH_INTERVAL|
| --proxy-dial-timeout        | 拨号超时的时间（毫秒）或禁用超时的时间（以毫秒为单位）|1000|ETCD_PROXY_DIAL_TIMEOUT|
| --proxy-write-timeout        | 写入超时的时间（毫秒）或禁用超时的值为0|5000|ETCD_PROXY_WRITE_TIMEOUT|
| --proxy-read-timeout        | 读取超时的时间（以毫秒为单位）或0以禁用超时。|0|ETCD_PROXY_READ_TIMEOUT|


## Security Flags

> 安全标志有助于[构建安全的etcd集群](https://coreos.com/etcd/docs/latest/v2/security.html)。

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --ca-file[弃用] | 客户端服务器TLS CA文件的路径。--ca-file ca.crt可以被替换--trusted-ca-file ca.crt --client-cert-auth和etcd将执行相同的操作|none|ETCD_CA_FILE|
| --cert-file        | 客户端服务器TLS证书文件的路径。|none|ETCD_CERT_FILE|
| --key-file        | 客户端服务器TLS密钥文件的路径。|none|ETCD_KEY_FILE|
| --client-cert-auth        | 启用客户端证书认证。|false|ETCD_CLIENT_CERT_AUTH|
| --trusted-ca-file        | 客户端服务器的路径TLS可信CA密钥文件。|none|ETCD_TRUSTED_CA_FILE|
| --peer-ca-file[弃用]        | 对等服务器TLS CA文件的路径。--peer-ca-file ca.crt可以被替换--peer-trusted-ca-file ca.crt --peer-client-cert-auth和etcd将执行相同的操作|none|ETCD_PEER_CA_FILE|
| --peer-cert-file        | 对等服务器TLS证书文件的路径。|none|ETCD_PEER_CERT_FILE|
| --peer-key-file        | 对等服务器TLS密钥文件的路径|none|ETCD_PEER_KEY_FILE|
| --peer-client-cert-auth        | 启用对等客户端证书认证。|false|ETCD_PEER_CLIENT_CERT_AUTH|
| --peer-trusted-ca-file        | 对等服务器的路径TLS可信CA文件。|none|ETCD_PEER_TRUSTED_CA_FILE|


## Logging Flags

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --debug        | 将所有子包的默认日志级别降为DEBUG。|false (INFO for all packages)|ETCD_DEBUG|
| --log-package-levels     | 将各个etcd子包设置为特定的日志级别。例如etcdserver=WARNING,security=DEBUG|none (INFO for all packages)|ETCD_LOG_PACKAGE_LEVELS|
| --name        | 这个成员的可读名称|none|ETCD_NAME|

## Unsafe Flags

> 使用不安全标志时请小心，因为它会破坏共识协议给出的保证。例如，如果集群中的其他成员仍然活着，它可能会出现panic。按照说明使用这些标志

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --force-new-cluster        | 强制创建一个新的一个成员群集。它提交配置更改强制删除群集中的所有现有成员并添加它自己。它需要设置为[恢复备份](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#restoring-a-backup)。|false|ETCD_FORCE_NEW_CLUSTER|

## Experimental Flags

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --experimental-v3demo        | 启用实验性[v3演示API](https://coreos.com/etcd/docs/latest/v2/rfc/v3api.html)。|false|ETCD_EXPERIMENTAL_V3DEMO|

## Miscellaneous Flags

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --version        | 打印版本并退出。|false|-|


## Profiling flags

| flag        | 说明| 默认值| 环境变量|
| --------      | -----| -----| -----|
| --enable-pprof        | 通过HTTP服务器启用运行时分析数据。地址在客户端URL+ "/debug/pprof/"|false|-|


[原文](https://coreos.com/etcd/docs/latest/v2/configuration.html)