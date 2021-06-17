# rancher-node-policies

We are building Rancher policies based on the documentation provided by Rancher - https://rancher.com/docs/rke/latest/en/os/#ports
If any of the ports or protocols are not correctly matched, I would contact the team at Rancher for an update on this.

This tutorial assumes that you already have a tier called 'rancher-nodes' in Calico Cloud:

```
cat << EOF > rancher-nodes.yaml
apiVersion: projectcalico.org/v3
kind: Tier
metadata:
  name: rancher-nodes
spec:
  order: 350
EOF  
```

```
kubectl apply -f rancher-nodes.yaml
```

# etcd-nodes

Once the tier is created, build a policy for the ETCD nodes:


```
cat << EOF > etcd-nodes.yaml
apiVersion: projectcalico.org/v3
kind: StagedGlobalNetworkPolicy
metadata:
  name: rancher-nodes.etcd-nodes
spec:
  tier: rancher-nodes
  order: 0
  selector: (environment == "development"&&node == "etcd")
  namespaceSelector: ''
  serviceAccountSelector: ''
  ingress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '2376'
          - '2379'
          - '2380'
          - '9099'
          - '10250'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  egress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '443'
          - '2379'
          - '2380'
          - '6443'
          - '9099'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  doNotTrack: false
  applyOnForward: false
  preDNAT: false
  types:
    - Ingress
    - Egress
EOF  
```

```
kubectl apply -f etcd-nodes.yaml
```

# Control-plane-nodes (Master Node)

Now proceed to build a policy for the master nodes:


```
cat << EOF > control-plane-nodes.yaml
apiVersion: projectcalico.org/v3
kind: StagedGlobalNetworkPolicy
metadata:
  name: rancher-nodes.control-plane-nodes
spec:
  tier: rancher-nodes
  order: 100
  selector: (environment == "development"&&node == "master")
  namespaceSelector: ''
  serviceAccountSelector: ''
  ingress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '80'
          - '443'
          - '2376'
          - '6443'
          - '9099'
          - '10250'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  egress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '443'
          - '2379'
          - '2380'
          - '9099'
          - '10250'
          - '10254'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  doNotTrack: false
  applyOnForward: false
  preDNAT: false
  types:
    - Ingress
    - Egress
EOF  
```

```
kubectl apply -f control-plane-nodes.yaml
```

# worker-nodes

Finally, we can build a policy for the worker nodes:


```
cat << EOF > worker-nodes.yaml
apiVersion: projectcalico.org/v3
kind: StagedGlobalNetworkPolicy
metadata:
  name: rancher-nodes.worker-nodes
spec:
  tier: rancher-nodes
  order: 200
  selector: (environment == "development"&&node == "worker")
  namespaceSelector: ''
  serviceAccountSelector: ''
  ingress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '22'
          - '3389'
          - '80'
          - '443'
          - '2376'
          - '9099'
          - '10250'
          - '10254'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  egress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '443'
          - '6443'
          - '9099'
          - '10254'
    - action: Allow
      protocol: UDP
      source: {}
      destination:
        ports:
          - '8472'
  doNotTrack: false
  applyOnForward: false
  preDNAT: false
  types:
    - Ingress
    - Egress
EOF  
```

```
kubectl apply -f worker-nodes.yaml
```
