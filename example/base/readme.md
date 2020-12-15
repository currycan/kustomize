# 非常用资源使用说明

## 1. 资源配额

[资源配额 | Kubernetes](https://kubernetes.io/zh/docs/concepts/policy/resource-quotas/)

### 1.1 配额作用域(scopeSelector)

每个配额都有一组相关的 `scope` （作用域），配额只会对作用域内的资源生效。 配额机制仅统计所列举的作用域的交集中的资源用量。

当一个作用域被添加到配额中后，它会对作用域相关的资源数量作限制。 如配额中指定了允许（作用域）集合之外的资源，会导致验证错误。

作用域描述 `Terminating` 匹配所有 `spec.activeDeadlineSeconds` 不小于 0 的 Pod。 `NotTerminating` 匹配所有 `spec.activeDeadlineSeconds` 是 nil 的 Pod。 `BestEffort` 匹配所有 Qos 是 BestEffort 的 Pod。 `NotBestEffort` 匹配所有 Qos 不是 BestEffort 的 Pod。 `PriorityClass` 匹配所有引用了所指定的 [优先级类](https://kubernetes.io/zh/docs/concepts/configuration/pod-priority-preemption) 的 Pods。

`BestEffort` 作用域限制配额跟踪以下资源：

* `pods`

`Terminating`、`NotTerminating`、`NotBestEffort` 和 `PriorityClass` 这些作用域限制配额跟踪以下资源：

* `pods`
* `cpu`
* `memory`
* `requests.cpu`
* `requests.memory`
* `limits.cpu`
* `limits.memory`

需要注意的是，你不可以在同一个配额对象中同时设置 `Terminating` 和 `NotTerminating` 作用域，你也不可以在同一个配额中同时设置 `BestEffort` 和 `NotBestEffort` 作用域。

`scopeSelector` 支持在 `operator` 字段中使用以下值：

* `In`
* `NotIn`
* `Exists`
* `DoesNotExist`

定义 `scopeSelector` 时，如果使用以下值之一作为 `scopeName` 的值，则对应的 `operator` 只能是 `Exists` 。

* `Terminating`
* `NotTerminating`
* `BestEffort`
* `NotBestEffort`

## 2.Pod 安全策略

[Pod 安全策略 | Kubernetes](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/)

## 3. Pod 优先级与抢占

[Pod 优先级与抢占 | Kubernetes](https://kubernetes.io/zh/docs/concepts/configuration/pod-priority-preemption/)

PriorityClass 是一种不属于任何名字空间的对象，定义的是从优先级类名向优先级整数值的映射。 优先级类名称用 PriorityClass 对象的元数据的 `name` 字段指定。 优先级整数值在必须提供的 `value` 字段中指定。 优先级值越大，优先级越高。 PriorityClass 对象的名称必须是合法的 [DNS 子域名](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) 且不可包含 `system-` 前缀。

PriorityClass 对象可以设置数值小于等于 10 亿的 32 位整数。 更大的数值保留给那些通常不可被抢占或逐出的系统 Pod。 集群管理员应该为每个优先级值映射创建一个 PriorityClass 对象。

PriorityClass 对象还有两个可选字段： `globalDefault` 和 `description` 。 前者用来表明此 PriorityClass 的数值应该用于未设置 `priorityClassName` 的 Pod。 系统中只能存在一个 `globalDefault` 设为真的 PriorityClass 对象。 如果没有 PriorityClass 对象的 `globalDefault` 被设置，则未设置 `priorityClassName` 的 Pod 的优先级为 0。

`description` 字段可以设置任意字符串值。其目的是告诉用户何时该使用该 PriorityClass。
