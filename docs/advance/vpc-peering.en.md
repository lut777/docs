# VPC Peering

VPC peering provides a mechanism for bridging two VPC networks through logical routes so that workloads within two VPCs 
can access each other through private addresses as if they were on the same private network, without the need for NAT forwarding through a gateway.

## Prerequisites
1. This feature is only available for customized VPCs.
2. To avoid route overlap the subnet CIDRs within the two VPCs cannot overlap.
3. Currently, only interconnection of two VPCs is supported.

## Usage

First create two non-interconnected VPCs with one Subnet under each VPC, 
and the CIDRs of the Subnets do not overlap with each other.

```yaml
kind: Vpc
apiVersion: kubeovn.io/v1
metadata:
  name: vpc-1
spec: {}
---
kind: Subnet
apiVersion: kubeovn.io/v1
metadata:
  name: net1
spec:
  vpc: vpc-1
  cidrBlock: 10.0.0.0/16
---
kind: Vpc
apiVersion: kubeovn.io/v1
metadata:
  name: vpc-2
spec: {}
---
kind: Subnet
apiVersion: kubeovn.io/v1
metadata:
  name: net2
spec:
  vpc: vpc-2
  cidrBlock: 172.31.0.0/16
```

Add `vpcPeerings` and the corresponding static routes within each VPC:

```yaml
kind: Vpc
apiVersion: kubeovn.io/v1
metadata:
  name: vpc-1
spec: 
  vpcPeerings:
    - remoteVpc: vpc-2
      localConnectIP: 169.254.0.1/30
  staticRoutes:
    - cidr: 172.31.0.0/16
      nextHopIP: 169.254.0.2
      policy: policyDst
---
kind: Vpc
apiVersion: kubeovn.io/v1
metadata:
  name: vpc-1
spec:
  vpcPeerings:
    - remoteVpc: vpc-2
      localConnectIP: 169.254.0.2/30
  staticRoutes:
    - cidr: 10.0.0.0/16
      nextHopIP: 169.254.0.1
      policy: policyDst
```

- `remoteVpc`: The name of another peering VPC.
- `localConnectIP`: As the IP address and CIDR of the interconnection endpoint. Note that both IPs should belong to the same CIDR and should not conflict with existing subnets.
- `cidr`：CIDR of the peering Subnet.
- `nextHopIP`：The `localConnectIP` on the other end of the peering VPC.
