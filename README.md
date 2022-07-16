# CKA 

Tips for Certified Kubernetes Administrator

![](https://i.ibb.co/dcczHBp/kube.png)

```
#to see cpu/memory consumption
kubectl top pod

#If node01 is nodeNotReady describe that node. 
kubectl describe node01
#If you see Kubelet stopped posting node status you will need to ssh into the node. See the file #/etc/kubernetes/kubelet.conf. 
#Correct the file if you find anything wrong. And restart the service. 
#You can try only restart the service. Some questions require only that.
systemctl daemon-reload
systemctl restart kubelet

#see the result with:
systemctl status kubelet
```

 * In the exam, when you are using vim press 'i' to enter in insert mode (insert key will not work).
 * Must watch:
   * [How to upgrade nodes](https://www.youtube.com/watch?v=3jcIN_TOc6E&ab_channel=AlokKumar).
   * [ETCD backup and restore](https://www.youtube.com/watch?v=mODkt1OJDew&ab_channel=AlokKumar). In the exam you will need the [Doc](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster).
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
