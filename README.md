# vagrant-k8s-cni

Sets up a kubernetes cluster without CNI. Just run `vagrant up`. You might also want to run `vagrant ssh-config >> ~/.ssh/config` as vagrant ansible provisioner is a little lacking and it needs manual configuration for the hosts and the private keys.

## Vagrant Configuration

VMs configuration can be modified in the `devConfig.yml` file. The current config is below:

```yaml
---
- box:
    vb: "ubuntu/xenial64"
  name: "k8s-master"
  nics:
    - type: "private_network"
      ip_addr: "10.0.0.10"
  ram: "2000"
  vcpu: "2"


- box:
    vb: "ubuntu/xenial64"
  name: "k8s-worker"
  nics:
    - type: "private_network"
      ip_addr: "10.0.0.11"
  ram: "1000"
  vcpu: "2"
```

## Ansible Configuration

Update the hosts file (./ansible/hosts) with the correct VM ip addresses:

```bash
k8s-master ansible_host=10.0.0.10
k8s-worker ansible_host=10.0.0.11
```

Feel free to modify the kubeadm config templates if you want to change / add parameters to the cluster (e.g., bindport and advertiseAddress for the API server, or the join-token).

```console
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── tasks
│   └── main.yml
└── templates
    └── kubeadm.conf.j2
```

Also have a look around the defaults for the different roles in case you want to change something. The most important configuration file is the `kubeadm.conf` which looks like this:

```yaml
---
apiServer:
  certSANs:
  - localhost
apiVersion: kubeadm.k8s.io/v1beta1
clusterName: cni-test
controllerManager:
  extraArgs:
    enable-hostpath-provisioner: "true"
kind: ClusterConfiguration
kubernetesVersion: v1.14.1
networking:
  podSubnet: {{ clusterCIDR }}
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: InitConfiguration
bootstrapTokens:
- token: {{ joinToken }}
nodeRegistration:
  kubeletExtraArgs:
    "node-ip" : "{{ hostIP }}"
localAPIEndpoint:
  advertiseAddress: {{ apiserverAdvertiseAddress }}
  bindPort: 6443
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: JoinConfiguration
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
evictionHard:
  imagefs.available: 0%
  nodefs.available: 0%
  nodefs.inodesFree: 0%
imageGCHighThresholdPercent: 100

---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
clusterCIDR: {{ clusterCIDR }}
```

## Testing CNIs

To test a specific CNI, all what you need to do is apply the yaml for the corresponding CNI. For example to use Calico, do:

```bash
kubectl apply -f https://docs.projectcalico.org/v3.6/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

Please note that CNI plugins also define POD or Cluster CIDR, be sure to make the Pod CIDR consistent across the following components:

- kube-controller-manager
- kube-proxy
- CNI plugin

## Remove Taint from Master if needed

Remove taint from master node if you want to use it as a worker:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## To join workers, an additional step is needed

Add your node IP to the kubelet

```console
sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

[Service]
...
...
..
Environment="KUBELET_EXTRA_ARGS=--node-ip=<e.g., 10.0.0.11>"
ExecStart=
...
...
```

then you can join the cluster from the node:

```bash
sudo kubeadm join 10.0.0.10:6443 --token  abcdef.0123456789abcdef --discovery-token-unsafe-skip-ca-verification --ignore-preflight-errors=all
```

## TODO

- Find a decent way to set the node-ip extra-flag for the kubelet (for each host) in the kubeadm.conf.


## Notes 

If you see this error:

```error
fatal: [k8s-master]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host 10.0.0.10 port 22: Operation timed out\r\n", "unreachable": true}
```

Just re-run with `vagrant up --provision` after a few seconds (the machine will become available).