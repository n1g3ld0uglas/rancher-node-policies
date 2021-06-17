# rancher-node-policies

We are building Rancher policies based on the documentation provided by Rancher - https://rancher.com/docs/rke/latest/en/os/#ports
If any of the ports or protocols are not correctly matched, I would contact the team at Rancher for an update on this.

I'm building this scenario around a generic 3 node cluster - master, worker and etcd node:


![Screenshot 2021-06-17 at 14 17 46](https://user-images.githubusercontent.com/82048393/122404102-d74a6680-cf76-11eb-88a3-8ee1219280e9.png)


Automatically register your nodes as Host Endpoints (HEPS). To enable automatic host endpoints, edit the default KubeControllersConfiguration instance, and set spec.controllers.node.hostEndpoint.autoCreate to true:

```
kubectl patch kubecontrollersconfiguration default --patch='{"spec": {"controllers": {"node": {"hostEndpoint": {"autoCreate": "Enabled"}}}}}'
```

<img width="1560" alt="Screenshot 2021-06-17 at 14 14 55" src="https://user-images.githubusercontent.com/82048393/122403683-7753c000-cf76-11eb-9016-ac84ce09297c.png">

to add the label kubernetes-host to all nodes and their host endpoints:

```
kubectl label nodes --all kubernetes-host=
```

![Screenshot 2021-06-17 at 14 20 09](https://user-images.githubusercontent.com/82048393/122404637-4de76400-cf77-11eb-81d2-f63bb46b2779.png)



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

<img width="637" alt="Screenshot 2021-06-17 at 14 22 27" src="https://user-images.githubusercontent.com/82048393/122404854-7a9b7b80-cf77-11eb-96cf-55caf84d8353.png">


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
  selector: has(kubernetes-host) && environment == 'dev'
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

<img width="637" alt="Screenshot 2021-06-17 at 14 23 41" src="https://user-images.githubusercontent.com/82048393/122405073-a74f9300-cf77-11eb-865a-8ac3ffcb077f.png">


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
  selector: has(kubernetes-host) && environment == 'dev'
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

<img width="621" alt="Screenshot 2021-06-17 at 14 25 14" src="https://user-images.githubusercontent.com/82048393/122405314-da922200-cf77-11eb-8b74-a088b5ed16ad.png">


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
  selector: has(kubernetes-host) && environment == 'dev'
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

<img width="615" alt="Screenshot 2021-06-17 at 14 26 22" src="https://user-images.githubusercontent.com/82048393/122405523-09a89380-cf78-11eb-8295-509a8ff953f2.png">

