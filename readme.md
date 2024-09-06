# CloudAuditTool

## 免责声明

该工具用于测试和验证云平台是否存在一定的安全隐患，同样也协助甲方安全人员对云环境做安全评估测试，可以评估容器，k8s，云主机安全等新基础设施安全


## Overview

该工具是一款云下安全审计工具，自动化识别容器和节点以及云平台归属，协助在云环境下的安全评估工作
```
1.自动化识别容器环境和主机环境，识别云归属厂商
2.自动化根据云厂商从Metadata读取敏感信息
3.自动化枚举主机和容器凭证权限，标识出高风险项
4.具备扫描关键组件的未授权漏洞(Apiserver,Etcd,kubelet,Dockersock)
5.todo
6.todo
```

## 使用方法
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudPentestSuite
Usage:
  Test [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  info        信息收集模块
  scan        扫描模块

Flags:
  -h, --help   help for Test

```

### 信息收集
```
[节点信息]
+----------------------------+--------------------------+----------+----------------------------+--------+
| 当前环境（容器内/主机节点) |      APISERVER地址       | K8S版本  |           节点名           | 云平台 |
+----------------------------+--------------------------+----------+----------------------------+--------+
| 主机节点                   | https://10.211.55.6:6443 | v1.23.17 | ubuntu-linux-22-04-desktop | 未知   |
+----------------------------+--------------------------+----------+----------------------------+--------+
[凭证类收集]
+------------------------------+--------------------------+----------------------------------------+
|      KUBECONFIG凭证路径      | 凭证指向的APISERVER地址  |                凭证用户                |
+------------------------------+--------------------------+----------------------------------------+
| /etc/kubernetes/admin.conf   | https://10.211.55.6:6443 | kubernetes-admin                       |
| /etc/kubernetes/kubelet.conf | https://10.211.55.6:6443 | system:node:ubuntu-linux-22-04-desktop |
+------------------------------+--------------------------+----------------------------------------+
凭证权限: /etc/kubernetes/admin.conf
Resources                                       Non-Resource URLs   Resource Names   Verbs
*.*                                             []                  []               [*]
get,watch,list.*                                []                  []               [*]
                                                [*]                 []               [*]
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
                                                [*]                 []               [get,watch,list]
                                                [/api/*]            []               [get]
                                                [/api]              []               [get]
                                                [/apis/*]           []               [get]
                                                [/apis]             []               [get]
                                                [/healthz]          []               [get]
                                                [/healthz]          []               [get]
                                                [/livez]            []               [get]
                                                [/livez]            []               [get]
                                                [/openapi/*]        []               [get]
                                                [/openapi]          []               [get]
                                                [/readyz]           []               [get]
                                                [/readyz]           []               [get]
                                                [/version/]         []               [get]
                                                [/version/]         []               [get]
                                                [/version]          []               [get]
                                                [/version]          []               [get]
```

### 扫描
##### Apiserver
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudPentestSuite scan apiserver --cidr=8.130.97.27
┌─────────────────────────────┐
│       存在未授权的节点         |
├───┬─────────────────────────┤
│   │ 未授权地址                │
├───┼─────────────────────────┤
│ 1 │ http://8.130.97.27:8080 │
└───┴─────────────────────────┘
```
