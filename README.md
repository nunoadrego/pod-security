# pod-security
Playing with Pod Security

## Create cluster with audit logging enabled
```
kind create cluster --config kind-config.yaml
```

## Add Pod Security Standards to a namespace

```
kubectl label namespace default \
    pod-security.kubernetes.io/enforce=baseline \
    pod-security.kubernetes.io/enforce-version=v1.29 \
    pod-security.kubernetes.io/audit=restricted \
    pod-security.kubernetes.io/audit-version=v1.29 \
    pod-security.kubernetes.io/warn=restricted \
    pod-security.kubernetes.io/warn-version=v1.29
```

## Check with kubectl
```
kubectl run test-pss --image nginx --namespace default --dry-run=server
kubectl run test-pss --image nginx --namespace default 
```

## Check audit logs
```
docker exec kind-control-plane cat /var/log/kubernetes/kube-apiserver-audit.log | jq '. | select(.objectRef.name=="test-pss")'
docker exec kind-control-plane cat /var/log/kubernetes/kube-apiserver-audit.log | jq '. | select(.objectRef.name=="test-pss") | .responseStatus'
docker exec kind-control-plane cat /var/log/kubernetes/kube-apiserver-audit.log | jq '. | select(.objectRef.name=="test-pss") | .annotations // empty | with_entries(select(.key | startswith("pod-security")))'
docker exec kind-control-plane cat /var/log/kubernetes/kube-apiserver-audit.log | jq '. | select(.objectRef.namespace=="default") | .annotations // empty | with_entries(select(.key | startswith("pod-security")))'
``` 

## Cleanup
```
kind delete cluster
```
