---
tags: 
title: A. Core Concepts
author: ndy2
date: 2024-02-06
description: ""
---

# Questions  

###### List all the namespaces in the cluster

```
kubectl get namespaces
kubectl get ns
```





###### List all the pods in all namespaces

```
k get po -A
```

###### List all the pods in the `particular` namespace

```
k get po -n particular
```



###### List all the services in the `particular` namespace

```
k get svc -n particular
```

###### List all the pods showing name and namespace with a json path expression

```
k get po -A -o jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"
```

###### List all the pods with name and namespace with custom column with "NAME", "NAMESPACE"

```
k get po -A -o custom-columns=NAME:.metadata.name, NAMESPACE:.metadata.namespace
```

###### Create an nginx pod in a default namespace and verify the pod running

```
k run nginx --image=nginx --restart=Never
k get po
```
###### Create the same nginx pod with a yaml file

```
k run nginx --image=nginx --restart=Never -o yaml --dry-run=client > nginx.yaml
k create -f nginx.yaml
```

###### Output the yaml file of the pod you just created

```
k get po nginx -o yaml
```

###### Output the yaml file of the pod you just created without the cluster-specific information

```
k get po nginx -o yaml --export
```

--export is deprecated!

###### Get the complete details of the pod you just created

```
k describe po nginx
```

###### Delete the pod you just created

```
k delete po nginx
```

###### Delete the pod you just created without any delay

```
k delete po nginx --grace-period=0 --force
```

```
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "nginx" force deleted
```

###### Create the nginx pod with version 1.17.4 and expose it on port 80

```
k run nginx --image=nginx:1.17.4 --restart=Never --port=80
```

--port : the port that this container exposes

###### Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated

```
k set image pod/nginx nginx=nginx:1.15-alpine
k describe po nginx | grep Image
```

###### Change the Image version back to 1.17.1 for the pod you just updated and observe the changes

```
k set image pod nginx=nginx:1.17.1
k describe po nginx | grep Image
```

###### Create the nginx pod and execute the simple shell on the pod

```
k run nginx --image=nginx --restart=Never
k exec -it nginx -- /bin/bash
```

###### Get the IP Address of the pod you just created

```
k get po nginx -o wide
```

###### Create a busybox pod and run command ls while creating it and check the logs

```
k run busybox --image=busybox --restart=Never -- ls
k logs busybox
```



###### If pod crashed check the previous logs of the pod

```
k logs busybox -p
```

previous/ p - If true, print the logs for the previous instance of the container in a pod if it exists.

###### Create a busybox pod with command sleep 3600

```
k run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600" "echo hello"
```

###### Check the connection of the nginx pod from the busybox pod

```
k get po nginx -o wide
k exec -it busybox -- curl telnet://<NGINX-IP-ADDRESS>>
```
###### Create a busybox pod and echo message 'How are you' and delete it manually

```
k run busybox --image=busybox --restart=Never --it -- echo "How are you"
k delete po busybox
```

###### Create a busybox pod and echo message 'How are you' and delete it immediately

```
k run busybox --image=busybox --restart=Never -it --rm -- echo "How are you"
```

rm - If true, delete resources created in this command for **attached containers**.

###### Create an nginx pod and list the pod with different levels of verbosity

```
k run nginx --image=nginx
k get po nginx
k get po nginx -o wide
k get po nginx --v=7
k get po nginx --v=8
k get po nginx --v=9
```

###### List the nginx pod with custom columns POD_NAME and POD_STATUS

```
k get po nginx -o custom-columns="POD_NAME:.metadata.name,POD_STATUS:.status.containersStatuses[].state"
```

###### List all the pods sorted by name

```
k get po --sort-by=.metadata.name
```

###### List all the pods sorted by created timestamp

```
k get po --sort-by=.metadata.createdTimestamp
```

# Kubectl Commands

## run

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### get

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

`--watch`, `-w`