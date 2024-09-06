# CloudAuditTool

## 免责声明

该工具用于测试和验证云平台是否安全，协助甲方安全人员对云环境做安全评估测试，可以评估容器，k8s，云主机安全等新基础设施


## Overview

该工具是一款云下安全审计工具，自动化识别容器和节点以及云平台归属，协助在云环境下的安全评估工作


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
