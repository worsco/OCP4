# Infrastructure node setup

### Create a new machineconfigpool infra
```
oc create -f infra-machineconfigpool.yaml
```
### List the existing worker machineconfigs
```
oc get mc|grep worker|grep -v rendered|awk '{print $1}'
```
### Get the the existing worker machineconfigs
```
oc get mc 00-worker -o yaml > 00-infra.yaml
```
```
oc get mc 01-worker-container-runtime -o yaml > 01-infra-container-runtime.yaml
```
```
oc get mc 01-worker-kubelet -o yaml > 01-infra-kubelet.yaml
```
```
oc get mc 99-worker-<HEX-VALUE-TO-REPLACE>-registries -o yaml > 99-infra-<HEX-VALUE-TO-REPLACE>-registries.yaml
```
```
oc get mc 99-worker-ssh -o yaml > 99-infra-ssh.yaml
```
### Replace worker with infra and delete useless lines
```
sed -i \
-e '/annotations/,+1d' \
-e '/creationTimestamp/d' \
-e'/generation/d' \
-e '/ownerReference/,+6d' \
-e '/resourceVersion/d' \
-e '/selfLink/d' \
-e '/uid/ {/data/!d}' \
-e 's/worker/infra/' \
00-infra.yaml
```
```
sed -i \
-e '/annotations/,+1d' \
-e '/creationTimestamp/d' \
-e'/generation/d' \
-e '/ownerReference/,+6d' \
-e '/resourceVersion/d' \
-e '/selfLink/d' \
-e '/uid/ {/data/!d}' \
-e 's/worker/infra/' \
01-infra-container-runtime.yaml
```
```
sed -i \
-e '/annotations/,+1d' \
-e '/creationTimestamp/d' \
-e'/generation/d' \
-e '/ownerReference/,+6d' \
-e '/resourceVersion/d' \
-e '/selfLink/d' \
-e '/uid/ {/data/!d}' \
-e 's/worker/infra/' \
01-infra-kubelet.yaml
```
```
sed -i \
-e '/annotations/,+1d' \
-e '/creationTimestamp/d' \
-e'/generation/d' \
-e '/ownerReference/,+4d' \
-e '/resourceVersion/d' \
-e '/selfLink/d' \
-e '/uid/ {/data/!d}' \
-e 's/worker/infra/' \
99-infra-<HEX-VALUE-TO-REPLACE>-registries.yaml
```
```
sed -i \
-e '/annotations/,+1d' \
-e '/creationTimestamp/d' \
-e'/generation/d' \
-e '/ownerReference/,+6d' \
-e '/resourceVersion/d' \
-e '/selfLink/d' \
-e '/uid/ {/data/!d}' \
-e 's/worker/infra/' \
99-infra-ssh.yaml
```
### Create the new infra machineconfigs
```
oc create -f 00-infra.yaml
```
```
oc create -f 01-infra-container-runtime.yaml
```
```
oc create -f 01-infra-kubelet.yaml
```
```
oc create -f 99-infra-<HEX-VALUE-TO-REPLACE>-registries.yaml
```
```
oc create -f 99-infra-ssh.yaml
```
### Relabel worker machines as infra (3 are needed)
```
oc label node <NODE-NAME> node-role.kubernetes.io/infra=
```
```
oc label node <NODE-NAME> node-role.kubernetes.io/worker-
```
### Move the internal routers to the infra nodes
```
oc patch ingresscontrollers.operator.openshift.io default -n openshift-ingress-operator \
-p '{"spec":{"nodePlacement":{"nodeSelector":{"matchLabels":{"node-role.kubernetes.io/infra":""}}}}}' \
--type=merge
```
### Define 3 routers (default are 2!)
```
oc patch ingresscontrollers.operator.openshift.io default -n openshift-ingress-operator \
-p '{"spec":{"replicas": 3}}' \
--type=merge
```
### Move the internal image registry to the infra nodes
```
oc patch configs.imageregistry.operator.openshift.io/cluster \
-p '{"spec":{"nodeSelector":{"node-role.kubernetes.io/infra": ""}}}' \
--type=merge
```
