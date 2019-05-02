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


Feel free to modify the kubeadm config templates if you want to change / add parameters to the cluster (e.g., bindport and advertiseAddress for the API server, or the join-token).

```
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── tasks
│   └── main.yml
└── templates
    └── kubeadm.conf.j2
```

Also have a look around the defaults for the different roles in case you want to change something.