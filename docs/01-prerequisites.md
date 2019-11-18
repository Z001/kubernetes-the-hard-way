# Prerequisites

## Yandex.Cloud

This tutorial leverages the [Yandex.Cloud](https://cloud.yandex.com/) to streamline provisioning of the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. [Sign up](https://console.cloud.yandex.com/) for [free 60 days trial](https://cloud.yandex.com/docs/free-trial/).

[Estimated cost](https://cloud.yandex.com/prices) to run this tutorial: 18.82₽ per hour (451.68₽ per day).
Detailed calculation for daily usage:

| ﻿Service               | Product                              | Unit                | Cost     |
|-----------------------|--------------------------------------|---------------------|----------|
| VPC                   | Public IP address                    | 144.00 fip*hour     | 21.95 ₽  |
| VPC                   | Public IP address of a load balancer | 24.00 fip*hour      | 3.66 ₽   |
| Compute Cloud         | Intel Cascade Lake. 100% vCPU        | 288.00 core*hour    | 215.31 ₽ |
| Compute Cloud         | Standard storage (HDD)               | 28800.06 gbyte*hour | 83.39 ₽  |
| Compute Cloud         | Intel Cascade Lake. RAM              | 576.00 gbyte*hour   | 114.05 ₽ |
| Network Load Balancer | Network load balancer                | 24.00 hour          | 13.33 ₽  |

-> The compute resources required for this tutorial exceed the [Yandex.Cloud free trial](https://cloud.yandex.com/docs/free-trial/concepts/limits).

## Quotas
```
network-hdd-total-disk-size >= 1200GB
cores >= 12
external-address-count >=7
```

## Yandex.Cloud CLI

### Install the Yandex.Cloud CLI

Follow the Yandex.Cloud CLI [documentation](https://cloud.yandex.com/docs/cli/) to install and configure the `yc` command line utility.

Verify the Yandex.Cloud CLI version is 0.43.0 or higher:

```
yc version
```

### Set a Default Availability Zone

This tutorial assumes a default compute availability zone have been configured.

If you are using the `yc` command-line tool for the first time `init` is the easiest way to do this:

```
yc init
```

## Running Commands in Parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.

Next: [Installing the Client Tools](02-client-tools.md)
