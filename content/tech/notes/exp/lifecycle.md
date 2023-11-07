---
title: "在 Kubernetes 上控制容器的启动顺序"
slug: ""
date: 2023-02-13T18:32:25+08:00
lastmod: 2023-02-13T18:32:25+08:00
author: ["路非非"]
tags: # 标签
- Kubernetes
series:
- 避坑指南
description: ""
weight:
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "tech/notes/exp/lifecycle/cover.png" #图片路径例如：posts/tech/123/123.png
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

--- 
## 背景

为什么要控制容器的启动顺序？

工作中遇到pod部署中包含类似于 `sidecar` 容器与服务容器一起, 且需要在服务容器启动之前完成 `sidecar` 容器启动和准备工作。若服务容器先启动，而 `sidecar` 还未准备就绪，那么服务容器将无法正常对外提供服务。

**现实**
![sidecar-lifecycle-1](sidecarlifecycle1.gif)

**期望**

![sidecar-lifecycle-2](sidecarlifecycle2.gif)

我们都知道 `pec.containers[]` 是一个数组，数组是有顺序的。Kubernetes 也确实是按照顺序来创建和启动容器，但是 **容器启动成功，并不表示容器可以对外提供服务**。

>在 Kubernetes 1.18 非正式版中曾在 Lifecycle 层面提供了对 `sidecar 类型容器的` 支持，但是最终该功能并[没有落地](https://github.com/kubernetes/enhancements/issues/753#issuecomment-713471597)。

那么该怎么做呢？

## 解决思路

![img](process.jpg)

**Lifecycle hooks**

在 `lifecycle` 中提供了 `postStart` 和 `preStop` 两种处理函数。其中 `postStart` 为 pod 启动后触发函数。

**Kubernetes 源码**

在 kubelet 的源码 `pkg/kubelet/kuberuntime/kuberuntime_manager.go` 中，`#SyncPod` 方法用于创建 Pod，我们直接看第7步

```go
// SyncPod syncs the running pod into the desired pod by executing following steps:
//
//  1. Compute sandbox and container changes.
//  2. Kill pod sandbox if necessary.
//  3. Kill any containers that should not be running.
//  4. Create sandbox if necessary.
//  5. Create ephemeral containers.
//  6. Create init containers.
//  7. Create normal containers.
func (m *kubeGenericRuntimeManager) SyncPod(ctx context.Context, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, backOff *flowcontrol.Backoff) (result kubecontainer.PodSyncResult) {
    ...
    // Step 7: start containers in podContainerChanges.ContainersToStart.
	for _, idx := range podContainerChanges.ContainersToStart {
		start(ctx, "container", metrics.Container, containerStartSpec(&pod.Spec.Containers[idx]))
	}
    ...
}
```

在 `#start` 方法中调用了 `#startContainer` 方法，该方法会启动容器，并返回容器启动的结果。注意，这里的结果还 **包含了容器的 Lifecycle hooks 调用**。

也就是说，假如容器的 `PostStart` hook 没有正确的返回，kubelet 便不会去创建下一个容器。

我们可以利用`postStart`进行阻塞，直到 `sidecar` 准备就绪。

```go
// startContainer starts a container and returns a message indicates why it is failed on error.
// It starts the container through the following steps:
// * pull the image
// * create the container
// * start the container
// * run the post start lifecycle hooks (if applicable)
func (m *kubeGenericRuntimeManager) startContainer(podSandboxID string, podSandboxConfig *runtimeapi.PodSandboxConfig, spec *startSpec, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, podIP string, podIPs []string) (string, error) {
     
     ...
     
	// Step 4: execute the post start hook.
	if container.Lifecycle != nil && container.Lifecycle.PostStart != nil {
		kubeContainerID := kubecontainer.ContainerID{
			Type: m.runtimeName,
			ID:   containerID,
		}
		msg, handlerErr := m.runner.Run(kubeContainerID, pod, container, container.Lifecycle.PostStart)
		if handlerErr != nil {
			m.recordContainerEvent(pod, container, kubeContainerID.ID, v1.EventTypeWarning, events.FailedPostStartHook, msg)
			if err := m.killContainer(pod, kubeContainerID, container.Name, "FailedPostStartHook", reasonFailedPostStartHook, nil); err != nil {
				klog.ErrorS(fmt.Errorf("%s: %v", ErrPostStartHook, handlerErr), "Failed to kill container", "pod", klog.KObj(pod),
					"podUID", pod.UID, "containerName", container.Name, "containerID", kubeContainerID.String())
			}
			return msg, fmt.Errorf("%s: %v", ErrPostStartHook, handlerErr)
		}
	}

	return "", nil
}
```

## 实现方案

**检查脚本**

在 `PostStart` 中持续的去检查 `sidecar` 是否准备就绪，这样可以 hold 住当前容器的创建流程。保证 `sidecar` 就绪后，kubelet 才会去创建下一个容器。

这样就达到了前面截图中演示的效果。

```go
...

timeoutAt := time.Now().Add(time.Duration(timeoutSeconds) * time.Second)
var err error
ticker := time.NewTicker(time.Duration(periodMillis) * time.Millisecond)
defer ticker.Stop()
for {
	<-ticker.C
	err = checkIfReady(param...)
	if err == nil {
		log.Println(fmt.Sprintf("container is ready", port))
		return nil
	}
    log.Println(fmt.Sprintf("container is not ready, err=%v", err))
	if time.Now().After(timeoutAt) {
		break
	}
}

...
```

Kubernetes 配置

```yaml
- image: image_addr
  imagePullPolicy: IfNotPresent
  name: image_name
  lifecycle:
      postStart:
          exec:
              command:
                - wait
```

## 参考
[Sidecar container lifecycle changes in Kubernetes 1.18](https://banzaicloud.com/blog/k8s-sidecars/)

[Delaying application start until sidecar is ready](https://medium.com/@marko.luksa/delaying-application-start-until-sidecar-is-ready-2ec2d21a7b74)