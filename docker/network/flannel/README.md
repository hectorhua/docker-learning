# flannel

## How it works

Flannel runs a small, single binary agent called `flanneld` on each host, and is responsible for allocating a subnet lease to each host out of a larger, preconfigured address space.
Flannel uses either the Kubernetes API or [etcd][etcd] directly to store the network configuration, the allocated subnets, and any auxiliary data (such as the host's public IP).
Packets are forwarded using one of several [backend mechanisms][backends] including VXLAN and various cloud integrations.

> Flannel可以在主机节点上以一个较小的二进制文件运行，并负责从手动配置的全网络地址中为每个节点分配子网络。Flannel可以使用K8S的API接口或ETCD服务存储网络配置信息，包括已分配的子网和其他附属信息（如节点的对外IP）。后端使用类似VXLAN或其他各种继承方式完成数据包的转发。

### Networking details

Platforms like Kubernetes assume that each container (pod) has a unique, routable IP inside the cluster.
The advantage of this model is that it removes the port mapping complexities that come from sharing a single host IP.

```
类似Kubernetes这样的平台会假定每个容器（pod）在集群内部都有一个唯一的可路由到的IP地址。
这样的设定的消除了多服务共享单个主机IP所造成的端口映射复杂性。（多个服务不能使用同一个IP地址下的同一个端口）
```

Flannel is responsible for providing a layer 3 IPv4 network between multiple nodes in a cluster. Flannel does not control how containers are networked to the host, only how the traffic is transported between hosts. However, flannel does provide a CNI plugin for Kubernetes and a guidance on integrating with Docker.

```
Flannel负责在集群中的多个节点之间提供第3层的IPv4模型网络。 Flannel不控制容器与主机之间的网络传输，只控制主机之间的网络传输方式。 但是flannel为Kubernetes提供了一个CNI插件和与Docker集成的说明。
```

Flannel is focused on networking. For network policy, other projects such as [Calico][calico] can be used.

```
Flannel关注于网络通信，对于网络策略（network policy），可使用其他项目，如Calico。
```

## Getting started on Kubernetes

The easiest way to deploy flannel with Kubernetes is to use one of several deployment tools and distributions that network clusters with flannel by default. For example, CoreOS's [Tectonic][tectonic] sets up flannel in the Kubernetes clusters it creates using the open source [Tectonic Installer][tectonic-installer] to drive the setup process.

```
使用Kubernetes部署flannel的最简单方法是使用一些集群默认的flannel部署工具。 例如，CoreOS的Tectonic，在使用开源[Tectonic Installer]创建的Kubernetes集群中建立flannel并启动。
```

Though not required, it's recommended that flannel uses the Kubernetes API as its backing store which avoids the need to deploy a discrete `etcd` cluster for `flannel`. This `flannel` mode is known as the *kube subnet manager*.

```
虽然不是强制性的，但是建议flannel使用Kubernetes API作为后端存储，这样就不需要为flannel单独部署一个etcd集群。 这种flannel模式被认为是kube子网的管理器。
```

### Deploying flannel manually

Flannel can be added to any existing Kubernetes cluster though it's simplest to add `flannel` before any pods using the pod network have been started.

For Kubernetes v1.7+
`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

See [Kubernetes](Documentation/kubernetes.md) for more details.

## Getting started on Docker

flannel is also widely used outside of kubernetes. When deployed outside of kubernetes, etcd is always used as the datastore. For more details integrating flannel with Docker see [Running](Documentation/running.md)

## Documentation
- [Building (and releasing)](Documentation/building.md)
- [Configuration](Documentation/configuration.md)
- [Backends](Documentation/backends.md)
- [Running](Documentation/running.md)
- [Troubleshooting](Documentation/troubleshooting.md)
- [Projects integrating with flannel](Documentation/integrations.md)
- [Production users](Documentation/production-users.md)

## Contact

* Mailing list: coreos-dev
* IRC: #coreos on freenode.org
* Slack: #flannel on [Calico Users Slack](https://slack.projectcalico.org)
* Planning/Roadmap: [milestones][milestones], [roadmap][roadmap]
* Bugs: [issues][flannel-issues]

## Contributing

See [CONTRIBUTING][contributing] for details on submitting patches and the contribution workflow.

## Reporting bugs

See [reporting bugs][reporting] for details about reporting any issues.

## Licensing

Flannel is under the Apache 2.0 license. See the [LICENSE][license] file for details.

[calico]: http://www.projectcalico.org
[pod-cidr]: https://kubernetes.io/docs/admin/kubelet/
[etcd]: https://github.com/coreos/etcd
[contributing]: CONTRIBUTING.md
[license]: https://github.com/coreos/flannel/blob/master/LICENSE
[milestones]: https://github.com/coreos/flannel/milestones
[flannel-issues]: https://github.com/coreos/flannel/issues
[backends]: Documentation/backends.md
[roadmap]: https://github.com/kubernetes/kubernetes/milestones
[reporting]: Documentation/reporting_bugs.md
[tectonic-installer]: https://github.com/coreos/tectonic-installer
[installing-with-kubeadm]: https://kubernetes.io/docs/getting-started-guides/kubeadm/
[tectonic]: https://coreos.com/tectonic/
