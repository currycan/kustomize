# 使用模板创建 Kustomize 配置清单

template 中根据应用配置清单中常用的资源.

通过 create 工具可以自动创建配置清单,参数使用 `ini` 格式,具体如下:

```bash
[project-name]
# 一级项目名,用于 namespace 等变量
project_name        = app-demo
# 二级项目名,用于 deployment service 等变量
app_demo_name       = docker-2048
# 镜像1名称和标签
app_demo_image_name = lexwhen/docker-2048
app_demo_image_tag  = latest
# 镜像2名称和标签
app_demo2_image_name = lexwhen/docker-2048
app_demo2_image_tag  = latest
```

project_name 字段: 一级项目名,用于 namespace 等变量
app_demo_name 字段: 二级项目名,用于 deployment service 等变量
image_name 字段: 镜像名称
image_tag 字段: 镜像标签

一个 pod 内有多个镜像时,可配置多个镜像相关参数
