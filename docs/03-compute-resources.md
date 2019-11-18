# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the compute resources required for running a secure and highly available Kubernetes cluster across a single [availability zone](https://cloud.yandex.com/docs/overview/concepts/geo-scope).

> Ensure a default availability zone have been set as described in the [Prerequisites](01-prerequisites.md#set-a-default-availability-zone) lab.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Private Cloud Network

In this section a dedicated [Virtual Private Cloud](https://cloud.yandex.com/docs/vpc/concepts/network#network) (VPC) network will be setup to host the Kubernetes cluster.

Create the `kubernetes-the-hard-way` custom VPC network:

```
yc vpc network create --name kubernetes-the-hard-way
```

A [subnet](https://cloud.yandex.com/docs/vpc/concepts/network#subnet) must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.

Create the `kubernetes` subnet in the `kubernetes-the-hard-way` VPC network:

```
yc vpc subnet create --name kubernetes \
  --network-name kubernetes-the-hard-way \
  --range 10.240.0.0/24
```

> The `10.240.0.0/24` IP address range can host up to 254 compute instances.

### The Kubernetes Frontend Load Balancer

Create an external load balancer with a listener to front the Kubernetes API Servers:

```
yc load-balancer network-load-balancer create --name kubernetes-the-hard-way --listener name=kubernetes-listener,port=6443,external-ip-version=ipv4
```

## Configuring SSH Access

SSH will be used to configure the controller and worker instances. You need to prepare ssh key and store it in `~/.ssh/id_rsa.pub` before creating instances as described in the [connecting to a linux VM via SSH](https://cloud.yandex.com/docs/compute/operations/vm-connect/ssh) documentation.

## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 18.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Create three compute instances which will host the Kubernetes control plane:

```
for i in 0 1 2; do
  yc compute instance create --name controller-${i} \
    --async \
    --create-boot-disk size=200,image-family=ubuntu-1804-lts \
    --image-folder-id standard-images \
    --cores 2 \
    --memory 4 \
    --network-interface subnet-name=kubernetes,address=10.240.0.1${i},nat-ip-version=ipv4 \
    --labels project=kubernetes-the-hard-way,role=controller \
    --ssh-key ~/.ssh/id_rsa.pub \
    --hostname controller-${i}
done
```

### Kubernetes Workers

Each worker instance requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise. The `pod-cidr` instance metadata will be used to expose pod subnet allocations to compute instances at runtime.

> The Kubernetes cluster CIDR range is defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial the cluster CIDR range will be set to `10.200.0.0/16`, which supports 254 subnets.

Create three compute instances which will host the Kubernetes worker nodes:

```
for i in 0 1 2; do
  yc compute instance create --name worker-${i} \
    --async \
    --create-boot-disk size=200,image-family=ubuntu-1804-lts \
    --image-folder-id standard-images \
    --cores 2 \
    --memory 4 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --network-interface subnet-name=kubernetes,address=10.240.0.2${i},nat-ip-version=ipv4 \
    --labels project=kubernetes-the-hard-way,role=worker \
    --ssh-key ~/.ssh/id_rsa.pub \
    --hostname worker-${i}
done
```

### Verification

List the compute instances in your default compute zone:

```
yc compute instances list
```

> output

```
+----------------------+--------------+---------------+---------+----------------+-------------+
|          ID          |     NAME     |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+--------------+---------------+---------+----------------+-------------+
| fhmxxxxxxxxxxxxxxxxx | controller-0 | ru-central1-a | RUNNING | XX.XXX.XXX.XXX | 10.240.0.10 |
| fhmxxxxxxxxxxxxxxxxx | controller-1 | ru-central1-a | RUNNING | XX.XXX.XXX.XX  | 10.240.0.11 |
| fhmxxxxxxxxxxxxxxxxx | controller-2 | ru-central1-a | RUNNING | XX.XXX.XXX.XX  | 10.240.0.12 |
| fhmxxxxxxxxxxxxxxxxx | worker-0     | ru-central1-a | RUNNING | XX.XXX.XXX.XXX | 10.240.0.20 |
| fhmxxxxxxxxxxxxxxxxx | worker-1     | ru-central1-a | RUNNING | XX.XXX.XXX.XXX | 10.240.0.21 |
| fhmxxxxxxxxxxxxxxxxx | worker-2     | ru-central1-a | RUNNING | XX.XXX.XXX.XXX | 10.240.0.22 |
+----------------------+--------------+---------------+---------+----------------+-------------+
```

## Testing SSH Access

Test SSH access to the `controller-0` compute instances:

```
ssh yc-user@$(yc compute instance get controller-0 \
  --format json | jq -r .network_interfaces[0].primary_v4_address.one_to_one_nat.address)
```

If this is the first time you connect to a VM, you might see a warning about an unknown host:

```
The authenticity of host '130.193.40.101 (130.193.40.101)' can't be established.
ECDSA key fingerprint is SHA256:PoaSwqxRc8g6iOXtiH7ayGHpSN0MXwUfWHkGgpLELJ8.
Are you sure you want to continue connecting (yes/no)?
```
Type `yes` in the terminal and press `Enter`.

You'll be logged into the `controller-0` instance:

```
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)
...

```

Type `exit` at the prompt to exit the `controller-0` compute instance:

```
yc-user@controller-0:~$ exit
```
> output

```
logout
Connection to XX.XXX.XXX.XXX closed
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
