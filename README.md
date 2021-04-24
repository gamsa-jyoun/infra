make sure router is dishing out IP addresses

network settings
  see if i can ping one of my nodes on the LAN

make sure ssh is turned on


turn off wifi on laptop and router
  disconnect and select the chord that i have plugged in
    eth0



edge max - $60

wipe the router
  reset it to factory default
  disconnect from my current router
  type in the router's ip address into the browser


  router should have DNS table

  ssh root@hostame


look up the router and see what the default IP address is
192.168.0.0.1




# DEVELOPMENT

**kubernetes**
tools:
- multipass - VMs
- k3sup       - cluster bootsrapping & admin
- k3s         - thin version of k8s
- kubectl     - kubernetes client

```
# if VMs don't exist: launch / create VMs
multipass launch --cpus 3 --mem 4G --disk 20G --name master-node --cloud-init multipass.yaml

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

sudo k3s kubectl apply -f gamsa-service.yaml
sudo k3s kubectl apply -f gamsa-deployment.yaml

popd

# run the following to get external-ip and port:
NOTE:
  you might want to use kube's svc that's defined:
  ```
    kg svc
  ```


export INGRESS_IP=$(sudo k3s kubectl get svc -o jsonpath='{.items[1].status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$( sudo k3s kubectl get svc -o jsonpath='{.items[1].spec.ports[0].port}' )

curl $INGRESS_IP:$INGRESS_PORT/posts
```


kubectl logs -lapp=gamsa --all-containers=true -f

kubectl run -it --rm --restart=Never sqlite3
kubectl exec -it rails-dep-748f88b984-jcdhp -- rails console
kubectl exec -it rails-dep-748f88b984-jcdhp -- /bin/bash
  sqlite3 db/development.sqlite3

# adding secrets / config maps with kustomize
k apply -k kustomize/secrets
k apply -k kustomize/configs


# scaling deployment
k scale deployment/rails-dep --replicas=1

curl -sfL https://get.k3s.io | sh -












# DEVELOPMENT

**kubernetes**
tools:
- multipass - VMs
- k3sup       - cluster bootsrapping & admin
- k3s         - thin version of k8s
- kubectl     - kubernetes client

```
# if VMs don't exist: launch / create VMs
multipass launch --cpus 1 --mem 2G --disk 5G --name master-node --cloud-init multipass.yaml
multipass launch --cpus 1 --mem 1G --disk 5G --name agent-master --cloud-init multipass.yaml
multipass launch --cpus 1 --mem 1G --disk 5G --name agent-worker --cloud-init multipass.yaml

multipass ls

# if VMs exist but stopped: start VMs
mp start --all

export NODE_MASTER=$( mp ls | grep master-node | awk '{print $3}' )
export AGENT_MASTER=$( mp ls | grep agent-master | awk '{print $3}' )
export AGENT_WORKER=$( mp ls | grep agent-worker | awk '{print $3}' )

export KUBECONFIG=`pwd`/kubeconfig
kubectl get node

# bootstrapping cluster
k3sup install --ip $NODE_MASTER --user ubuntu --k3s-extra-args "--cluster-init"
k3sup join --ip $AGENT_MASTER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu --server
k3sup join --ip $AGENT_WORKER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu

# initialize k8s objects
pushd /home/james/lab/kub-lab/k8s-intro-meetup-kit/k8s

k apply -f gamsa-service.yaml
k apply -f gamsa-deployment.yaml

popd

# run the following to get external-ip and port:
export INGRESS_IP=$(kubectl get svc -o jsonpath='{.items[1].status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$( kg svc -o jsonpath='{.items[1].spec.ports[0].port}' )

curl $INGRESS_IP:$INGRESS_PORT
```


kubectl logs -lapp=gamsa --all-containers=true -f



# RBAC
By default, the Kubernetes API server installs a cluster role that
allows system:unauthenticated users access to the API server’s
API discovery endpoint. For any cluster exposed to a hostile envi‐
ronment (e.g., the public internet) this is a bad idea, and there has
been at least one serious security vulnerability via this exposure.
Consequently, if you are running a Kubernetes service on the pub‐
lic internet or an other hostile environment, you should ensure that
the --anonymous-auth=false flag is set on your API server.


```
$ k auth
$ k auth reconcile -f some-rbac-config.yaml
```


# CLUSTER ADMIN
mounting USB via CLI
https://help.ubuntu.com/community/Mount/USB
  sudo fdisk -l
  sudo mount -t vfat /dev/sda1 /media/usb-drive -o uid=1000,gid=1000,utf8,dmask=027,fmask=137

  https://stackoverflow.com/questions/7878707/how-to-unmount-a-busy-device
  sudo umount /dev/sda1
    NOTE: cd out of the mounted directory (/media/usb-drive)

