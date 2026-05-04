---
title: K8S学习
date: 2021-05-04 15:17:21
tags:
	
	- K8S
---





以下是我个人对学习 k8s 的意见，很多资料都涉及网络（自行解决）和英文（google翻译）：

1. 仅以 k8s 官方文档作为参考手册，尽量不看中文博客（具有局限性和滞后性，多数都有错误）官方文档： https://kubernetes.io/docs/home/

2. （初步了解它能做什么）先了解基本概念，然后动手实践

     建议用官方的 tutorial 做实践内容：https://kubernetes.io/docs/tutorials/ 
     建议用本地环境做实践环境，可以用 Minikube 或者 Kind, 我建议用kind：
https://github.com/kubernetes-sigs/kind
     建议把 https://www.katacoda.com/ 的实验做一遍

3. 进一步了解Kubernetes的架构、理念及组件

     3.1 部署方案（下面都是针对on-premise方案，如果能用 cloud，建议优先考虑cloud）：   只建议用官方文档推荐的方案

https://kubernetes.io/docs/setup/production-environment/

    多数情况下我都建议用 kubeadm 做部署         

  如果有兴趣可以看 https://github.com/kelseyhightower/kubernetes-the-hard-way，可以深入理解 k8s 的组件及组件关系大规模部署交付建议考虑 kubespray

     3.2 较为系统的了解 k8s 的 concepts、glossary，这些在官方文档中都有，建议自行查找（用于练习文档查阅能力，k8s的CKA认证就是开卷考试，所有内容都可以在官方文档中找到参考）

     3.3 继续多做练习，比如官网的 tasks

4. 了解 K8S 的生态，包括 CRI、CNI、CSI、可观测性等，建议通过 CNCF 全景图找感兴趣的部分，具体情况可以找我沟通。 https://landscape.cncf.io/

5. 多看k8s 博客（https://kubernetes.io/blog/）及cncf博客（ https://www.cncf.io/blog/ ），都是大厂分享理念和实践