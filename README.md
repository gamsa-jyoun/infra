
# DEVELOPMENT

**kubernetes**
tools:
- multipass - VMs
- k3sup       - cluster bootsrapping & admin
- k3s         - thin version of k8s
- kubectl     - kubernetes client

```
# if VMs don't exist: launch / create VMs
multipass launch --cpus 1 --mem 1G --disk 5G --name master-node --cloud-init multipass.yaml
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
k3sup install --ip $NODE_MASTER --user pi --k3s-extra-args "--cluster-init"
k3sup join --ip $AGENT_MASTER --user pi --server-ip $NODE_MASTER --server-user pi --server
k3sup join --ip $AGENT_WORKER --user pi --server-ip $NODE_MASTER --server-user pi

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

flash image onto SD card
sudo dd if=2021-03-04-raspios-buster-armhf-lite.img of=/dev/sda bs=4M conv=fsync status=progress

  https://stackoverflow.com/questions/7878707/how-to-unmount-a-busy-device
  sudo umount /dev/sda1
    NOTE: cd out of the mounted directory (/media/usb-drive)

power down:
sudo shutdown -h now

reboot
sudo shutdown -r now

adding wifi:
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

proxy server to access internet:
https://www.raspberrypi.org/documentation/configuration/use-a-roxy.md

securing:
https://www.raspberrypi.org/documentation/configuration/security.md


