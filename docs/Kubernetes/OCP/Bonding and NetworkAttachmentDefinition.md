---
tags: tech/openshift
created: 2024-12-22
modified: 2026-01-28
share: true
---

# Bonding and NetworkAttachmentDefinition

## Creating bond

1. Verify the network interfaces available first by logging into OCP web console, or from nmtui into individual nodes
2. Create NodeNetworkConfigurationPolicy for **each worker nodes**
   
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: bond1-worker-node-name-policy
spec:
  nodeSelector:
    kubernetes.io/hostname: <node-name>
  desiredState:
    interfaces:
    - name: bond1
      type: bond
      state: up
      ipv4:
        dhcp: false
        address:
        - ip: <ip address>
          prefix-length: 24
        enabled: true
      link-aggregation:
        mode: active-backup
        options:
          miimon: '140'
        port:
          - ens32f0np0
          - ens34f0np0
          - ens36f0np0
          - ens38f0np0
      mtu: 1450
```
3. Apply the configuration `oc create -f bond1-<node-name>-policy`

## Create NetworkAttachmentPolicy

1. Create new namespace `oc create ns test-project`
2. Create manifests for NAD
   
```yaml title="test-ipvlan.yaml" hl_lines=10,15
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: test-ipvlan
  namespace: test-project
spec:
  config: |
    { 
      "cniVersion": "0.3.1",
      "name": "ipvlan-net",
      "type": "ipvlan",
      "master": "bond1",
      "ipam": {
        "type": "whereabouts",
        "range": "x.x.x.x/24"
      }
    }
```
3. Apply the configuration`oc create -f test-ipvlan.yaml`