## Overview of kubeadm

Kubeadm is a tool built to provide kubeadm init and kubeadm join as best-practice “fast paths” for creating Kubernetes clusters.

> Kubeadm是用于创建初始环境并向集群添加工作节点的工具，在创建Kubernetes集群方面是最优选的快捷工具。

kubeadm performs the actions necessary to get a minimum viable cluster up and running. By design, it cares only about bootstrapping, not about provisioning machines. Likewise, installing various nice-to-have addons, like the Kubernetes Dashboard, monitoring solutions, and cloud-specific addons, is not in scope.

> kubeadm执行最基本的步骤来完成集群的创建和运行。从设计上来说，kubeadm更加关注工具的自动化，而不是服务器的具体配置。同样的，安装各类实用的插件，如仪表盘显示工具，监控方案工具以及特定的云插件等，不在此管理范围内。

Instead, we expect higher-level and more tailored tooling to be built on top of kubeadm, and ideally, using kubeadm as the basis of all deployments will make it easier to create conformant clusters.

> 相反的，我们希望更高级和定制化的工具是在kubeadm之上按需求构建，理想情况下，kubeadm只负责创建部署最基础的集群环境，这样能够使得集群创建更加简单便捷。


