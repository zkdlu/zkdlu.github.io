---
layout: post
title: "[Trouble shooting] 1.쿠버네티스 설치하기"
description: "Kubernetes 설치하면서 생긴 어려움들"
date: 2020-12-28
tags: [Trouble shooting, Kubernetes]
comments: true
share: true
---

AWS EC2에서 쿠버네티스를 사용하면서 생긴 이슈

# 쿠버네티스

쿠버네티스를 직접 설치하면서 마스터 클러스터를 설정하기 위해 **> sudo kubeadm init**을 실행 했더니 다음과 같은 오류메시지가 출력 되었다.

```bash
user:~$ sudo kubeadm init
[init] Using Kubernetes version: v1.20.1
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.1. Latest validated version: 19.03
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
        [ERROR Mem]: the system RAM (978 MB) is less than the minimum 1700 MB
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

눈에 띄는 불안한 메시지가 보인다.

> [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
> [ERROR Mem]: the system RAM (978 MB) is less than the minimum 1700 MB

프리티어로는 쿠버네티스를 사용할 수 있는 최소 사양이 부족하다고 한다...

> 당분간은 힘들거같다.



## 쿠버네티스 최소 사양

- CPU 2개 이상

- 메모리  2GB 이상

- 노드간 원활한 네트워크

- 사용하는 포트

  | 노드   | 프로토콜 | 규칙    | 포트범위    | 목적                    | 사용자               |
  | ------ | -------- | ------- | ----------- | ----------------------- | -------------------- |
  | Master | TCP      | Inbound | 6443        | Kubernetes API Server   | All                  |
  | Master | TCP      | Inbound | 2379-2380   | etcd server client API  | kube-apiserver, etcd |
  | Master | TCP      | Inbound | 10250       | Kubelet API             | Self, Control plane  |
  | Master | TCP      | Inbound | 10251       | kube-scheduler          | Self                 |
  | Master | TCP      | Inbound | 10252       | kube-controller-manager | Self                 |
  | Worker | TCP      | Inbound | 10250       | Kubelet API             | Self, Control plane  |
  | Worker | TCP      | Inbound | 30000-32767 | NotePort Services       | All                  |
  
  


