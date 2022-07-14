# CKA 

Tips for Certified Kubernetes Administrator

![](https://i.ibb.co/dcczHBp/kube.png)

```
kubectl replace --force -f pod.yaml

#see the changes in real time.
kubectl get po --watch

kubectl get po --selector env=dev,bu=finance

#count how many pods exist.
kubectl get po --no-headers | wc -l

#to see which is the path of the directory holding the status pod of definition file.
cd /var/lib

kubecl drain node01

#no new pods will be put in node01
kubectl cordon node01

#put the node back in the game
kubectl uncordon node02

kubeadmin upgrade plan

#to make a backup of etcd
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
# you also can use vim /etc/kubernetes/manifests/etcd.yaml
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>


ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshotdb

# to discover network interface used by master node
# you also can see ip address alocated to master node through # inet
# ether is the mac address
ifconfig -e
cat /etc/network/interfaces

# to connect to a particular node
ssh node01

# to see docker bridge
ifconfig -a

# to see the status of a networking interface
# you can also see the name of the bridge/network created by weave
ip link 

# to see the route
ip r

# to see in which port ks is running
netstat -natulp | grep kube-scheduler

# identify network plugin configured to kubernetes
# /opt/cni/plugin is the path for cni plugins
# /etc/cnt/net.d/ is the path for cni plugin configured for kubernetes
ps aux | grep kubelet | grep network

# to see which the networking solution use this
ps aux | grep kubelet
#if you see cni for example go to /etc/cni/net.d/ to see which networkin soluton is used.

#you can see how many weave agents/kube-proxy are deployed.
#you can see the dns solution. 
#Using the describe command you can see configuration file location for dns configs. 
#See configmap related to dns to know which is root domain/zone.
#seeing the weave pods logs will show you the ip range for the pods.
#seeing the kube-proxy logs will show the type type of proxy. 
kubectl get po -n kube-system

#to see the gayeway in a pod
#remember that you can force a pod to be in a specific node using nodeName property
kubectl exec busybox -- route -n

#you can see ip range for nodes.
kubectl get nodes -o wide
ip a

# to see ip ranges for services go to /etc/kubernetes/manifests and see file related to apiserver. 
#Search for service-cluster-ip-range property

service kubelet status
service kubelet start

kubectl get pods -o=jsonpath='{.items[0].metadata.name}'
kubectl api-resources

kubectl get node --kubeconfig path
kubectl config get-contexts -o name > /opt/course/1/contexts
kubectl config current-context
cat ~/.kube/config | grep current

kubectl get pod -A --sort-by=.metadata.creationTimestamp

kubectl top node
kubectl top pod --containers=true

kubectl -n namespace auth can-i create secret

kubectl config view

kubectl set image deployment name_deploy --nginx=image --record
kubectl api-resources --namespaced -o name

service kubelet status
service kubelet start
whereis kubelet
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

#upgrade node
kubelet --version
kubeadm upgrade node

# You can create a static pod in ***/etc/kubernetes/manifests/***. Put a yaml file there.

#find certificate and validation
find /etc/kubernetes/pki | grep apiserver
openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2

#you can also use kubeadm
kubeadm certs check-expiration | grep apiserver
kubeadm certs renew apiserver

#upgrade a node to 1.19.0
ssh node01
kubectl drain node01 --ignore-daemonsets
apt update
apt install kubeadm=1.19.0-00
kubeadm upgrade apply v1.19.0
apt install kubelet=1.19.0-00
systemctl restart kubelet
kubectl uncordon node01
logout

kubectl get node node01 -o json
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}'

#the path for /var/lib/kubelet is the config of kubelet.
```

 * The only thing a scheduler does is that it sets the nodeName for a Pod declaration.
 * It was requested that the DaemonSet runs on all nodes, so we need to specify the toleration for this.
 * ssh into master node to see config files in ***/etc/kubernetes/manifests***
 * By default the kubelet looks into ***/etc/cni/net.d*** to discover the CNI plugins. 
 * Which suffix will static pods have that run on cluster1-worker1?
   * The suffix is the node hostname with a leading hyphen. It used to be -static in earlier Kubernetes versions.
 * When available cpu or memory resources on the nodes reach their limit, Kubernetes will look for Pods that are using more resources than they requested. These will be the first candidates for termination. If some Pods containers have no resource requests/limits set, then by default those are considered to use more than requested. You can see them using ***kubectl -n namespace describe pod | less -p Requests***
 * To create a user you need to create a ***CertificateSigningRequest***, Role and RoleBinding. Look at the docs.
 * Taint a node to avoid resources being scheduled on it. 
 * ETCD starts a service that listen on port 2379 by default.
 * Must watch:
   * [How to upgrade nodes](https://www.youtube.com/watch?v=3jcIN_TOc6E&ab_channel=AlokKumar).
   * [ETCD backup and restore](https://www.youtube.com/watch?v=mODkt1OJDew&ab_channel=AlokKumar).
   * [Static pods](https://www.youtube.com/watch?v=Vm_Q95RJJPU&ab_channel=AlokKumar).
   * [Taint and Tolerations](https://www.youtube.com/watch?v=_5xNAk4jOFs&ab_channel=AlokKumar).
   * [Worker node kubelet troubleshooting](https://www.youtube.com/watch?v=xvavhBWy0bI&ab_channel=MyCloudTutorials)
## LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```


```
cat myuser.csr | base64 | tr -d "\n"
kubectl certificate approve myuser
kubectl get csr
```

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
  ```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress                    # policy is only about Egress
  egress:
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db1
      ports:                        # second condition "port"
      - protocol: TCP
        port: 1111
    -                           # second rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db2
      ports:                        # second condition "port"
      - protocol: TCP
        port: 2222
```
