# Test for connection between cluster using ovs-cni
## Environment
- Ubuntu 18.04
- kubernetes v1.21.6

### Setup environment
>Setup onos, mininet, openvswitch
```
cd scripts/
sudo ./env-setup.sh
```

* Create ovs-br
```
sudo ovs-vsctl add-br br0
```

* Deploy ovs-cni daemon
>Notice: the path of `db.sock` on host may different from your host if you install ovswitch in different way
```
# in ovs.conf, we config the host `db.sock` path
sudo mkdir -p /etc/cni/net.d/ovs.d/
sudo cp conf/ovs.conf /etc/cni/net.d/ovs.d/
```

deploy the daemon
```
kubectl apply -f deploy/ovs-cni.yml
```

* Deploy network-attachment CRD
```
kubectl apply -f deploy/network-attachment.yaml
```

## Run test pods
```
kubectl apply -f deploy/pods.yaml
kubectl apply -f deploy/pods-2.yaml
```

you will found new port added on ovs-br:
```
$ sudo ovs-vsctl show
374be660-735a-42d9-9d78-8fc497a01081
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port veth4185735f
            Interface veth4185735f
        Port vetha6a6f521
            Interface vetha6a6f521
    ovs_version: "2.17.2"
```

* Set interface ip
```
kubectl exec -ti samplepod -- /bin/sh
# in samplepod
ip addr add 10.20.0.2/24 dev net1
```
set interface ip in another pod in same way

## Support

https://github.com/k8snetworkplumbingwg/ovs-cni/blob/main/docs/cni-plugin.md
