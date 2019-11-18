# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.yandex.com/docs/vpc/concepts/static-routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## The Routing Table

In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.

Print the internal IP address and Pod CIDR range for each worker instance:

```
for instance in worker-0 worker-1 worker-2; do
  yc compute instance get --full ${instance} \
  	--format json | jq -r '[.network_interfaces[0].primary_v4_address.address,.metadata."pod-cidr"] | join(" ")'
done
```

> output

```
10.240.0.20 10.200.0.0/24
10.240.0.21 10.200.1.0/24
10.240.0.22 10.200.2.0/24
```

## Routes

Create route table with network routes for each worker instance:

```
yc vpc route-table create --name=kubernetes-route-table \
  --network-name=kubernetes-the-hard-way \
  --route destination=10.200.0.0/24,next-hop=10.240.0.20 \
  --route destination=10.200.1.0/24,next-hop=10.240.0.21 \
  --route destination=10.200.2.0/24,next-hop=10.240.0.22
```

List the routes in the `kubernetes-route-table` route table:

```
yc vpc route-table get kubernetes-route-table --format json | jq .static_routes
```

> output

```
[
  {
    "destination_prefix": "10.200.0.0/24",
    "next_hop_address": "10.240.0.20"
  },
  {
    "destination_prefix": "10.200.1.0/24",
    "next_hop_address": "10.240.0.21"
  },
  {
    "destination_prefix": "10.200.2.0/24",
    "next_hop_address": "10.240.0.22"
  }
]
```

Link the routing table to one of the subnets

```
yc vpc subnet update kubernetes --route-table-name kubernetes-route-table
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
