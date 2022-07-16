# CKA 

Tips for Certified Kubernetes Administrator

![](https://i.ibb.co/dcczHBp/kube.png)

```
#to see cpu/memory consumption
kubectl top pod

#If node01 is nodeNotReady describe that node. 
#If you see Kubelet stopped posting node status you will need to ssh into the node. See the file #/etc/kubernetes/kubelet.conf. 
#Correct the file if you find anything wrong. And restart the service. 
#You can try only restart the service. Some questions require only that.
systemctl daemon-reload
systemctl restart kubelet

#see the result with:
systemctl status kubelet
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
   * [ETCD backup and restore](https://www.youtube.com/watch?v=mODkt1OJDew&ab_channel=AlokKumar). In the exam you willneed the [Doc](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster).
   * [Static pods](https://www.youtube.com/watch?v=Vm_Q95RJJPU&ab_channel=AlokKumar).
   * [Taint and Tolerations](https://www.youtube.com/watch?v=_5xNAk4jOFs&ab_channel=AlokKumar).
   * [Worker node kubelet troubleshooting](https://www.youtube.com/watch?v=xvavhBWy0bI&ab_channel=MyCloudTutorials)
   * [NetworkPolicy](https://www.youtube.com/watch?v=KK49iNc_W4I&ab_channel=MyCloudTutorials)
## LimitRange

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
