# DEVELOPMENT

**kubernetes**
tools:
- multipass - VMs
- k3sup       - cluster bootsrapping & admin
- k3s         - thin version of k8s
- kubectl     - kubernetes client

```
# if VMs don't exist: launch / create VMs
multipass launch --cpus 2 --mem 6G --disk 20G --name master-node --cloud-init multipass.yaml

multipass ls

export NODE_MASTER=$( mp ls | grep master-node | awk '{print $3}' )

export KUBECONFIG=`pwd`/kubeconfig
kubectl get node

# bootstrapping cluster
k3sup install --ip $NODE_MASTER --user ubuntu --k3s-extra-args "--cluster-init"
k3sup join --ip $NODE_MASTER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu --server
k3sup join --ip $NODE_MASTER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu

# initialize k8s objects
pushd /home/james/lab/kub-lab/k8s-intro-meetup-kit/k8s

k apply -f gamsa-service.yaml
k apply -f gamsa-deployment.yaml

popd

# run the following to get external-ip and port:
NOTE:
  you might want to use kube's svc that's defined:
  ```
    kg svc
  ```


export INGRESS_IP=$(kubectl get svc -o jsonpath='{.items[1].status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$( kg svc -o jsonpath='{.items[1].spec.ports[0].port}' )

curl $INGRESS_IP:$INGRESS_PORT/posts
```


kubectl logs -lapp=gamsa --all-containers=true -f

kubectl run -it --rm --restart=Never sqlite3
kubectl exec -it rails-dep-748f88b984-jcdhp -- rails console
kubectl exec -it rails-dep-748f88b984-jcdhp -- /bin/bash
  sqlite3 db/development.sqlite3
