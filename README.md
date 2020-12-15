# 使用 kustomize 管理 Kubernetes 应用

使用说明：[commonLabels | SIG CLI](https://kubectl.docs.kubernetes.io/references/kustomize/commonlabels/)

[Kustomize](https://github.com/kubernetes-sigs/kustomize) 是一个独立的工具，用来通过 [kustomization 文件](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/glossary.md#kustomization) 定制 Kubernetes 对象。

从 1.14 版本开始， `kubectl` 也开始支持使用 kustomization 文件来管理 Kubernetes 对象。 要查看包含 kustomization 文件的目录中的资源，执行下面的命令：

```bash
kubectl kustomize <kustomization_directory>
```

要应用这些资源，使用参数 `--kustomize` 或 `-k` 标志来执行 `kubectl apply` ：

```bash
kubectl apply -k <kustomization_directory>
```

## 什么是 kustomize

`kustomize`（ [Github链接](https://github.com/kubernetes-sigs/kustomize) ） 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件。而其他用户可以完全不受影响的使用任何一个 Base YAML 或者任何一层生成出来的 YAML 。这使得每一个用户都可以通过类似fork/modify/rebase 这样 Git 风格的流程来管理海量的应用描述文件。这种 PATCH 的思想跟 Docker 镜像是非常相似的，它可以规避“字符替换”对应用描述文件的入侵，也不需要用户学习额外的 DSL 语法（比如 Lua）。

而其成为 `kubectl` 子命令则代表这 `kubectl` 本身的插件机制的成熟，未来可能有更多的工具命令集成到 `kubectl` 中。从下图不难看出，在 kubernetes 原生应用管理系统中，应用描述文件在整个应用管理体系中占据核心位置，通过应用描述文件可以组合和编排多种 kubernetes API 资源，kubernetes 通过控制器来保证集群中的资源与应用状态与描述文件完全一致。
![架构图](https://tva1.sinaimg.cn/large/0081Kckwgy1glijyrm5ovj31bq0r8qbv.jpg)
Kustomize 不像 Helm 那样需要一整套独立的体系来完成管理应用，而是完全采用 kubernetes 的设计理念来完成管理应用的目的。同时使用起来也更加的得心应手。

Kustomize 是一个用来定制 Kubernetes 配置的工具。它提供以下功能特性来管理 应用配置文件：

* 从其他来源生成资源
* 为资源设置贯穿性（Cross-Cutting）字段
* 组织和定制资源集合

### 工作原理

kustomize 将对 K8s 应用定义为由 Yaml 表示的资源描述，对 K8s 应用的变更可以映射到对 Yaml 的变更上。而这些变更操作可以利用 git 等版本控制程序来管理，因此用户得以使用 git 风格的流程对 K8s 应用进行管理。

对于一个受 kustomize 管理的 App，都有若干个 Yaml 组成。Github 中描述了一个 Demo 目录结构如下：

```tree
~/someApp
├── deployment.yaml
├── kustomization.yaml
└── service.yaml
```

其中包含用来描述元数据的 `kustomization.yaml` ，以及两个资源文件，其内容如下图所示：
![base](https://github.com/kubernetes-sigs/kustomize/raw/master/docs/images/base.jpg)

该目录下的 Yaml 是不完整的，可以被补充、覆盖，因此该 Yaml 被称为 Base，而补充 Base 的 Yaml 被称作 Overlay，可以补充适用不同环境、场景的 overlay yaml 到该 app 中，从而使得 K8s 应用描述更完整，其目录结构为：

``` tree
~/someApp
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── development
    │   ├── cpu_count.yaml
    │   ├── kustomization.yaml
    │   └── replica_count.yaml
    └── production
        ├── cpu_count.yaml
        ├── kustomization.yaml
        └── replica_count.yaml
```

![overlay](https://github.com/kubernetes-sigs/kustomize/raw/master/docs/images/overlay.jpg)
kustomize 通过描述文件的叠加，来生成完整的部署用 Yaml，可以直接使用 `build` 命令可以生成并部署：

```bash
kustomize build ~/someApp/overlays/development | kubectl apply -f -
```

在 Kubernetes 1.14 之后，也支持直接 `apply` 该应用。

```bash
kubectl apply -k ~/someApp/overlays/development
```

### 工作流

在 Kubernetes 应用管理系统中，应用的描述文件（Yaml）是一个非常核心的组成部分，用户通过描述文件来向集群声明自己应用的资源和服务编排要求，kustomize 也是社区对描述文件管理的一个重要的尝试。

对于 kustomize，用户可以使用 Git 对 Kubernetes 应用进行管理，通过 fork 现有 App，拓展 Base 或者定制 overlay，基本流程如下：

![img](https://blog.cdn.updev.cn/2019-05-14-121434.jpg)
在官方 Github 仓库中，有一篇完整的工作流文档可以参考： [https://github.com/kubernetes-sigs/kustomize/blob/master/docs/zh/workflows.md](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/zh/workflows.md)

### 和 Helm 比较

通过对 kustomize 工作原理的了解，可以看到其和 Helm 的差别最直观的在于 Yaml 渲染上。Helm 通过编写 Yaml 模板，在部署时进行渲染，而 kustomize 是 overlay 叠加的方式，制定不同 patch，在部署时选择使用。

但更重要的，是 Helm 和 kustomize 致力解决的问题不同，Helm 更努力做一个自称体系的生态圈，可以方便管理应用的制品（镜像 + 配置），而对于一个已经发布的制品来说，Chart 相对固定、稳定，是一个静态的管理，更适合对外交付。而 kustomize 则不然，kustomize 管理的是正在变更的应用，可以随时 fork 出一个新版本，也可以创建新的 overlay 将应用推向新的环境，是一个动态的管理，而这种动态，非常适合集成到 DevOps 的流程中。

## 安装

kustomize 的安装非常简单，因为其本身只是一个 cli，只有一个二进制文件，可以在 release 页面下载： [https://github.com/kubernetes-sigs/kustomize/releases](https://github.com/kubernetes-sigs/kustomize/releases)

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

## 使用 kustomize 管理 K8s 应用

目录结构：

```tree

├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   └── service.yaml
└── overlays
    ├── production
    │   ├── config
    │   │   └── application.properties
    │   ├── env_patch.yaml
    │   ├── healthcheck_patch.yaml
    │   ├── image_pull_secrets_patch.yaml
    │   ├── increase_replicas.yaml
    │   ├── kustomization.yaml
    │   ├── kustomize
    │   ├── memorylimit_patch.yaml
    │   ├── service_patch.yaml
    │   └── strategy_patch.yaml
    └── sit
        ├── env_patch.yaml
        ├── healthcheck_patch.yaml
        ├── kustomization.yaml
        └── memorylimit_patch.yaml
```

部署：

```bash
cd  app/overlays/production
kubectl kustomize . | kubectl apply -f -
```

如需要特别指定namespace，如下：

```bash
kubectl kustomize . | kubectl apply -f - -n project-name
```

安装`kustomize`工具后,使用命令即可部署(sit/uat/prod三个环境)

```bash
./create prod | kubectl apply --dry-run -f -
```

patch 的三种形式

```yaml
# pacth
patches:
# 参数配置在 replicas.yaml 文件
- path: replicas.yaml
  target:
    group: apps
    version: v1
    kind: Deployment
    # 支持通配符
    # name: docker-20.*
    name: docker-2048
    # # 通配符时使用
    # labelSelector: "app: docker-2048"
    # annotationSelector: "zone=west"

# 直接配置参数,kubectl kustomize build 无法识别该字段,使用 kustomize build .
- patch: |-
    - op: replace
      path: /spec/replicas
      value: 0
  target:
    group: apps
    version: v1
    kind: Deployment
    name: docker-2048
    # labelSelector: "app: docker-2048"

# patchesJson6902
patchesJson6902:
- path: replicas.yaml
  target:
    group: apps
    version: v1
    kind: Deployment
    name: docker-2048
```
