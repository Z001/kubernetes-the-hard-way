# Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances

Delete the controller and worker compute instances:

```
for instance in controller-0 controller-1 controller-2 \
  worker-0 worker-1 worker-2; do
  yc compute instance delete ${instance}
done
```

## Networking

Delete the external load balancer network resources:

```
{
  yc load-balancer network-load-balancer delete kubernetes-the-hard-way

  yc load-balancer target-group delete kubernetes-target-group
}
```

Delete the `kubernetes-the-hard-way` network VPC:

```
{
  yc vpc subnet delete kubernetes

  yc vpc route-table delete kubernetes-route-table

  yc vpc network delete kubernetes-the-hard-way
}
```
