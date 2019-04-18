## kubeadm init 集群初始化

This command initializes a Kubernetes control-plane node.

> kubeadm init用于初始化一个Kubernetes控制节点。

Run this command in order to set up the Kubernetes control plane.

> 运行命令来安装Kubernetes控制机制

### Synopsis 概要

The “init” command executes the following phases:

> init命令将会执行以下步骤：

```
preflight                  Run pre-flight checks
kubelet-start              Writes kubelet settings and (re)starts the kubelet
certs                      Certificate generation
  /ca                        Generates the self-signed Kubernetes CA to provision identities for other Kubernetes components
  /apiserver                 Generates the certificate for serving the Kubernetes API
  /apiserver-kubelet-client  Generates the Client certificate for the API server to connect to kubelet
  /front-proxy-ca            Generates the self-signed CA to provision identities for front proxy
  /front-proxy-client        Generates the client for the front proxy
  /etcd-ca                   Generates the self-signed CA to provision identities for etcd
  /etcd-server               Generates the certificate for serving etcd
  /apiserver-etcd-client     Generates the client apiserver uses to access etcd
  /etcd-peer                 Generates the credentials for etcd nodes to communicate with each other
  /etcd-healthcheck-client   Generates the client certificate for liveness probes to healtcheck etcd
  /sa                        Generates a private key for signing service account tokens along with its public key
kubeconfig                 Generates all kubeconfig files necessary to establish the control plane and the admin kubeconfig file
  /admin                     Generates a kubeconfig file for the admin to use and for kubeadm itself
  /kubelet                   Generates a kubeconfig file for the kubelet to use *only* for cluster bootstrapping purposes
  /controller-manager        Generates a kubeconfig file for the controller manager to use
  /scheduler                 Generates a kubeconfig file for the scheduler to use
control-plane              Generates all static Pod manifest files necessary to establish the control plane
  /apiserver                 Generates the kube-apiserver static Pod manifest
  /controller-manager        Generates the kube-controller-manager static Pod manifest
  /scheduler                 Generates the kube-scheduler static Pod manifest
etcd                       Generates static Pod manifest file for local etcd.
  /local                     Generates the static Pod manifest file for a local, single-node local etcd instance.
upload-config              Uploads the kubeadm and kubelet configuration to a ConfigMap
  /kubeadm                   Uploads the kubeadm ClusterConfiguration to a ConfigMap
  /kubelet                   Uploads the kubelet component config to a ConfigMap
upload-certs               Upload certificates to kubeadm-certs
mark-control-plane         Mark a node as a control-plane
bootstrap-token            Generates bootstrap tokens used to join a node to a cluster
addon                      Installs required addons for passing Conformance tests
  /coredns                   Installs the CoreDNS addon to a Kubernetes cluster
  /kube-proxy                Installs the kube-proxy addon to a Kubernetes cluster
```

> kubeadm init [flags]

### Options 可选参数

```
--apiserver-advertise-address string
The IP address the API Server will advertise it's listening on. If not set the default network interface will be used.
--apiserver-bind-port int32     Default: 6443
Port for the API Server to bind to.
--apiserver-cert-extra-sans stringSlice
Optional extra Subject Alternative Names (SANs) to use for the API Server serving certificate. Can be both IP addresses and DNS names.
--cert-dir string     Default: "/etc/kubernetes/pki"
The path where to save and store the certificates.
--certificate-key string
Key used to encrypt the control-plane certificates in the kubeadm-certs Secret.
--config string
Path to a kubeadm configuration file.
--cri-socket string
Path to the CRI socket to connect. If empty kubeadm will try to auto-detect this value; use this option only if you have more than one CRI installed or if you have non-standard CRI socket.
--dry-run
Don't apply any changes; just output what would be done.
--experimental-upload-certs
Upload control-plane certificates to the kubeadm-certs Secret.
--feature-gates string
A set of key=value pairs that describe feature gates for various features. Options are:
-h, --help
help for init
--ignore-preflight-errors stringSlice
A list of checks whose errors will be shown as warnings. Example: 'IsPrivilegedUser,Swap'. Value 'all' ignores errors from all checks.
--image-repository string     Default: "k8s.gcr.io"
Choose a container registry to pull control plane images from
--kubernetes-version string     Default: "stable-1"
Choose a specific Kubernetes version for the control plane.
--node-name string
Specify the node name.
--pod-network-cidr string
Specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node.
--service-cidr string     Default: "10.96.0.0/12"
Use alternative range of IP address for service VIPs.
--service-dns-domain string     Default: "cluster.local"
Use alternative domain for services, e.g. "myorg.internal".
--skip-certificate-key-print
Don't print the key used to encrypt the control-plane certificates.
--skip-phases stringSlice
List of phases to be skipped
--skip-token-print
Skip printing of the default bootstrap token generated by 'kubeadm init'.
--token string
The token to use for establishing bidirectional trust between nodes and control-plane nodes. The format is [a-z0-9]{6}\.[a-z0-9]{16} - e.g. abcdef.0123456789abcdef
--token-ttl duration     Default: 24h0m0s
The duration before the token is automatically deleted (e.g. 1s, 2m, 3h). If set to '0', the token will never expire
```

### Options inherited from parent commands 继承参数

```
--rootfs string
[EXPERIMENTAL] The path to the 'real' host root filesystem.
```

### Init workflow 初始化工作流

kubeadm init bootstraps a Kubernetes control-plane node by executing the following steps:

1. Runs a series of pre-flight checks to validate the system state before making changes. Some checks only trigger warnings, others are considered errors and will exit kubeadm until the problem is corrected or the user specifies --ignore-preflight-errors=<list-of-errors>.
2. Generates a self-signed CA (or using an existing one if provided) to set up identities for each component in the cluster. If the user has provided their own CA cert and/or key by dropping it in the cert directory configured via --cert-dir (/etc/kubernetes/pki by default) this step is skipped as described in the Using custom certificates document. The APIServer certs will have additional SAN entries for any --apiserver-cert-extra-sans arguments, lowercased if necessary.
3. Writes kubeconfig files in /etc/kubernetes/ for the kubelet, the controller-manager and the scheduler to use to connect to the API server, each with its own identity, as well as an additional kubeconfig file for administration named admin.conf.
4. Generates static Pod manifests for the API server, controller manager and scheduler. In case an external etcd is not provided, an additional static Pod manifest is generated for etcd.

Static Pod manifests are written to /etc/kubernetes/manifests; the kubelet watches this directory for Pods to create on startup.

Once control plane Pods are up and running, the kubeadm init sequence can continue.

1. Apply labels and taints to the control-plane node so that no additional workloads will run there.
2. Generates the token that additional nodes can use to register themselves with a control-plane in the future. Optionally, the user can provide a token via --token, as described in the kubeadm token docs.
3. Makes all the necessary configurations for allowing node joining with the Bootstrap Tokens and TLS Bootstrap mechanism:  
  * Write a ConfigMap for making available all the information required for joining, and set up related RBAC access rules.
  * Let Bootstrap Tokens access the CSR signing API.
  * Configure auto-approval for new CSR requests.

See kubeadm join for additional info.

1. Installs a DNS server (CoreDNS) and the kube-proxy addon components via the API server. In Kubernetes version 1.11 and later CoreDNS is the default DNS server. To install kube-dns instead of CoreDNS, the DNS addon has to configured in the kubeadm ClusterConfiguration. For more information about the configuration see the section Using kubeadm init with a configuration file bellow. Please note that although the DNS server is deployed, it will not be scheduled until CNI is installed.

### Using init phases with kubeadm

Kubeadm allows you create a control-plane node in phases. In 1.13 the kubeadm init phase command has graduated to GA from it’s previous alpha state under kubeadm alpha phase.

To view the ordered list of phases and sub-phases you can call kubeadm init --help. The list will be located at the top of the help screen and each phase will have a description next to it. Note that by calling kubeadm init all of the phases and sub-phases will be executed in this exact order.

Some phases have unique flags, so if you want to have a look at the list of available options add --help, for example:

```
sudo kubeadm init phase control-plane controller-manager --help
```

You can also use --help to see the list of sub-phases for a certain parent phase:

```
sudo kubeadm init phase control-plane --help
```

kubeadm init also expose a flag called --skip-phases that can be used to skip certain phases. The flag accepts a list of phase names and the names can be taken from the above ordered list.

An example:

```
sudo kubeadm init phase control-plane all --config=configfile.yaml
sudo kubeadm init phase etcd local --config=configfile.yaml
# you can now modify the control plane and etcd manifest files
sudo kubeadm init --skip-phases=control-plane,etcd --config=configfile.yaml
```

What this example would do is write the manifest files for the control plane and etcd in /etc/kubernetes/manifests based on the configuration in configfile.yaml. This allows you to modify the files and then skip these phases using --skip-phases. By calling the last command you will create a control plane node with the custom manifest files.

### Using kubeadm init with a configuration file 



















