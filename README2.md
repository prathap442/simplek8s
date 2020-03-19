As per the `Shot13` if we want to basically put the pod then the pod would definitely be having all the tightly coupled containers in it.

* All these tightly coupled containers would interact with each other .
 
* The Smallest container that we can run on the Kubernetes is on the Pod itself.

* But to use the pod we should have the tight coupling being enabled. 


### Basically Two Object Types exist.
* Services: 
  - This is basically used for the setting up the networking.
* Pods:
  - This is basically used for setting up the tight coupling among the various containers in the cluster.

Under Services(That are basically used for the sake of the networking)
  We have subtypes for Service Object As per the `Shot14` i conclude that the subtypes can be many in number like:
  - ClusterIP
  - NodePort(Exposes a containr to the outside world(only good for dev purposes!!!))
  - LoadBalancer
  - Ingress

In the client-pod.yaml we have
```
  apiVersion: v1
  kind: service
  metadata:
  name: client-node-port
  spec:
    type: NodePort
    ports:
      - ports: 3050
        targetPort: 3000
        nodePort: 31515
    selector:
        component: web
```
As indicated in the Above file the Object kind is of Service which is used for the sake of the networking and the type is nodePort for the sake of the exposing of the contianers to the outer world ONly in development. under the spec .

------------------------------------------------------------------------------------------

Now if you take a look of the client-node-port.yaml and client-pod.yaml 

client-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers: 
    - name: client
      image: stephengrider/multi-client
      ports:
        containerPort: 3000
```

client-node-port.yaml
apiVersion: v1
kind: service
metadata:
  name: client-node-port
spec:
  type: NodePort
  ports:
    - ports: 3050
      targetPort: 3000
      nodePort: 31515
  selector:
    component: web
```


The client-node-port.yamldirects the output to that particular pod by using the using the selector component property in the client-node-port.yaml and the client-pod.yal metadata being the same and this is being done as per the files that are being shown above

Here component: web is common in two of the files .

```
 component: web
```



the ports as we need from the node-port.yml having the nodePort(that gets being exposed to the outer world). 
  and the targetPort says the container port exposing number and that is 3000 .

and now if you can see that the node-port 

------------------------------------------------------------------------------------------

# Now let us feed a config file to the kubectl and this can be done by using



What are the imperative and Declarative Deployments in the Kubernetes.

Imperative Deployments are the ones that make the kubernetes controlling to be given to the manual controlling

Declarative Deployments:
  - Most of the Deployments are declarative and they are trying to get deployed by using the yaml scripts and the complete control being given to the kubernetes on production .

![alt text](https://github.com/prathap442/parking_lot_pracs/blob/master/shot11.png)


----------------------------------------------------------------------------------------------------

![shot12](https://github.com/prathap442/simplek8s/blob/master/images/shot12.png)


Now let us try to update the pod that contains the image of `multi-client` to contain `multi-worker`

Changed From
client-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-client
      ports:
        - containerPort: 9999
```

Changed To
client-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-worker
      ports:
        - containerPort: 9999
```


Now as we have updated the pod we need to apply this to the kubernetes and tell the cluster that we are running to replace the pods that have multi-client running in them to now run the multi-worker and for this what we need to do is configure the pod once again 

When we do that here comes the crucial part like how does the pod containing the containers of the multi-client come to know that it has to be update to the multi-worker is by means of 

ids like
```
  -name(In the metadata)
  -kind
```

So the kubectl master whenever being imprinted with the command

```
 kubectl apply -f client-pod.yaml
```

then the master takes the input and then finds the container with the name "client-pod" and the kind "Pod" and then it targets that particular pod and updates the containers in them with the specs being


```
  kind: `Pod`
  metadata:
    name: `client-pod`
    labels:
      component: web
  specs:
    -containers:
      name: client
      image: stephengrider/multi-worker
      ports:
        - containerPort: 9999
```

so these two specs in the client-pod.yaml are concentrated and then images are updated in those containers


Now what if the id is not found then the pod adds the additional containers with the different specifications being mentioned .


before configuration 

```
 $ kubectl get pods 

  NAME         READY   STATUS    RESTARTS   AGE
  client-pod   1/1     Running   0          1m
```

after configuration of kubectl apply -f client-pod.yaml with the updated image o worker from the client then

```
 $ kubectl get pods 

  NAME         READY   STATUS    RESTARTS   AGE
  client-pod   1/1     Running   1         2m
```

no as we see the restarts have increased by one count because of the new configuration that came into picture .


In order to inspect a very specific pod in that particular node  we use the command

```
$ kubectl describe <objecttype> <objectname>
```

which is 

```
  NAME         READY   STATUS    RESTARTS   AGE
  client-pod   1/1     Running   1         2m

$ kubectl describe pod client-pod 
```

and here is the ton of the information presented for the client-pody

```
Name:         client-pod
Namespace:    default
Node:         minikube/10.0.2.15
Start Time:   Sun, 23 Feb 2020 16:24:52 +0530
Labels:       component=web
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"component":"web"},"name":"client-pod","namespace":"default"},"spec...
Status:       Running
IP:           172.17.0.4  
IPs:          <none>
Containers:
  client:
    Container ID:   docker://c9cdd5c7667ad8cbce9d980203df122de31c511598398936bc40f58776ad9761
    Image:          stephengrider/multi-worker
    Image ID:       docker-pullable://stephengrider/multi-worker@sha256:5fbab5f86e6a4d499926349a5f0ec032c42e7f7450acc98b053791df26dc4d2b
    Port:           9999/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 23 Feb 2020 16:41:57 +0530
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sun, 23 Feb 2020 16:41:08 +0530
      Finished:     Sun, 23 Feb 2020 16:41:53 +0530
    Ready:          True
    Restart Count:  4
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v58l8 (ro)
Conditions:
  Type           Status
  Initialized    True 
  Ready          True 
  PodScheduled   True 
Volumes:
  default-token-v58l8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v58l8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age                From               Message
  ----    ------                 ----               ----               -------
  Normal  Scheduled              34m                default-scheduler  Successfully assigned client-pod to minikube
  Normal  SuccessfulMountVolume  34m                kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-v58l8"
  Normal  Killing                18m (x2 over 34m)  kubelet, minikube  Killing container with id docker://client:Container spec hash changed (2628785887 vs 2173011808).. Container will be killed and recreated.
  Normal  Pulling                18m (x2 over 34m)  kubelet, minikube  pulling image "stephengrider/multi-client"
  Normal  Pulled                 18m (x2 over 33m)  kubelet, minikube  Successfully pulled image "stephengrider/multi-client"
  Normal  Killing                17m (x2 over 33m)  kubelet, minikube  Killing container with id docker://client:Container spec hash changed (2173011808 vs 2628785887).. Container will be killed and recreated.
  Normal  Pulling                17m (x3 over 34m)  kubelet, minikube  pulling image "stephengrider/multi-worker"
  Normal  Created                17m (x5 over 34m)  kubelet, minikube  Created container
  Normal  Started                17m (x5 over 34m)  kubelet, minikube  Started container



-----------------
So basically the command has described us with the information of the lifecycle of the events that it has undergone through .



So this is the approach of the declarative saying when we want the change then just update the client-pod.yaml
```


Now let us do a small change in the specifications of the containers ports let say the container is updated to expose the port from 9999 to 3000 .

Then i'm being displayed with a huge message from the command when i apply the command as follows

```
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-worker
      ports:
        - containerPort: 3000
```


```
kubectl apply -f client-pod.yaml
```

this displays us basically with the following errors

```

The Pod "client-pod" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
{"Volumes":[{"Name":"default-token-v58l8","HostPath":null,"EmptyDir":null,"GCEPersistentDisk":null,"AWSElasticBlockStore":null,"GitRepo":null,"Secret":{"SecretName":"default-token-v58l8","Items":null,"DefaultMode":420,"Optional":null},"NFS":null,"ISCSI":null,"Glusterfs":null,"PersistentVolumeClaim":null,"RBD":null,"Quobyte":null,"FlexVolume":null,"Cinder":null,"CephFS":null,"Flocker":null,"DownwardAPI":null,"FC":null,"AzureFile":null,"ConfigMap":null,"VsphereVolume":null,"AzureDisk":null,"PhotonPersistentDisk":null,"Projected":null,"PortworxVolume":null,"ScaleIO":null,"StorageOS":null}],"InitContainers":null,"Containers":[{"Name":"client","Image":"stephengrider/multi-worker","Command":null,"Args":null,"WorkingDir":"","Ports":[{"Name":"","HostPort":0,"ContainerPort":

A: 3000,"Protocol":"TCP","HostIP":""}],"EnvFrom":null,"Env":null,"Resources":{"Limits":null,"Requests":null},"VolumeMounts":[{"Name":"default-token-v58l8","ReadOnly":true,"MountPath":"/var/run/secrets/kubernetes.io/serviceaccount","SubPath":"","MountPropagation":null}],"VolumeDevices":null,"LivenessProbe":null,"ReadinessProbe":null,"Lifecycle":null,"TerminationMessagePath":"/dev/termination-log","TerminationMessagePolicy":"File","ImagePullPolicy":"Always","SecurityContext":null,"Stdin":false,"StdinOnce":false,"TTY":false}],"RestartPolicy":"Always","TerminationGracePeriodSeconds":30,"ActiveDeadlineSeconds":null,"DNSPolicy":"ClusterFirst","NodeSelector":null,"ServiceAccountName":"default","AutomountServiceAccountToken":null,"NodeName":"minikube","SecurityContext":{"HostNetwork":false,"HostPID":false,"HostIPC":false,"ShareProcessNamespace":null,"SELinuxOptions":null,"RunAsUser":null,"RunAsGroup":null,"RunAsNonRoot":null,"SupplementalGroups":null,"FSGroup":null},"ImagePullSecrets":null,"Hostname":"","Subdomain":"","Affinity":null,"SchedulerName":"default-scheduler","Tolerations":[{"Key":"node.kubernetes.io/not-ready","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300},{"Key":"node.kubernetes.io/unreachable","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300}],"HostAliases":null,"PriorityClassName":"","Priority":null,"DNSConfig":null}

B: 9999,"Protocol":"TCP","HostIP":""}],"EnvFrom":null,"Env":null,"Resources":{"Limits":null,"Requests":null},"VolumeMounts":[{"Name":"default-token-v58l8","ReadOnly":true,"MountPath":"/var/run/secrets/kubernetes.io/serviceaccount","SubPath":"","MountPropagation":null}],"VolumeDevices":null,"LivenessProbe":null,"ReadinessProbe":null,"Lifecycle":null,"TerminationMessagePath":"/dev/termination-log","TerminationMessagePolicy":"File","ImagePullPolicy":"Always","SecurityContext":null,"Stdin":false,"StdinOnce":false,"TTY":false}],"RestartPolicy":"Always","TerminationGracePeriodSeconds":30,"ActiveDeadlineSeconds":null,"DNSPolicy":"ClusterFirst","NodeSelector":null,"ServiceAccountName":"default","AutomountServiceAccountToken":null,"NodeName":"minikube","SecurityContext":{"HostNetwork":false,"HostPID":false,"HostIPC":false,"ShareProcessNamespace":null,"SELinuxOptions":null,"RunAsUser":null,"RunAsGroup":null,"RunAsNonRoot":null,"SupplementalGroups":null,"FSGroup":null},"ImagePullSecrets":null,"Hostname":"","Subdomain":"","Affinity":null,"SchedulerName":"default-scheduler","Tolerations":[{"Key":"node.kubernetes.io/not-ready","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300},{"Key":"node.kubernetes.io/unreachable","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300}],"HostAliases":null,"PriorityClassName":"","Priority":null,"DNSConfig":null}




```




the above errors basically describes saying that we are not allowed to change the fields other than image and some other fields .
So we cannot change now next we conclude that for a service of kind pod we cannot changed the container specifications once fixed apart from small changes like image attribute.



Now let us look at the differences between the Pod and the Deployment where the Deployment allows to change event ports of the container even though they are once created by using the template


Please have a look at the `shot37.png` and `shot38.png` for the sake of the examples .

Shot39.png is basically showing us the template of the pod in the Object type Deployment this basiccally has the attributes that can be changed 

containers: 1
name: client
port 3000
image multi-worker


```
A Deployment basically in kubernetes is the one that runs a set of multiple containers .

A Deployment monitors the state of each pod,updating when necessary

Good For Development
Good For Production
```



bash@git/simplek8s$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
client-pod   1/1     Running   6          16h
```

# Deleting the pod

Now i wanted to delete the pod then for the sake of removing we have got different syntax
```
kubectl delete -f <file-for-the-pod-creation>
```
so the same file that is used for the sake of the pod configuration should be used when the pod needs to be removed
```
  kubectl delete -f client-pod.yaml`
  pod "client-pod" deleted
```

kubectl get pods now gives me with 
```
$ kubectl get pods

No resources found in default namespace.
```

# Creating the Pods using the Deployment object


* This is possible by using the command

```
kubectl apply -f client-deployment.yaml 
```
deployments/client-deployment got created



Experiments on the local Machine

```
kubectl apply -f ./client-deployment.yaml 
deployment.apps/client-deployment created
prathap@prathap-HP-EliteBook-840-G1:~/Documents/stephen-grider-learnings/simplek8s$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
client-deployment-588947887b-7jccw   0/1     ContainerCreating   0          1s

prathap@prathap-HP-EliteBook-840-G1:~/Documents/stephen-grider-learnings/simplek8s$ kubectl apply -f ./client-deployment.yaml 
deployment.apps/client-deployment configured

prathap@prathap-HP-EliteBook-840-G1:~/Documents/stephen-grider-learnings/simplek8s$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
client-deployment-588947887b-7jccw   1/1     Running             0          31s
client-deployment-78cdc5df45-96hkc   0/1     ContainerCreating   0          3s

prathap@prathap-HP-EliteBook-840-G1:~/Documents/stephen-grider-learnings/simplek8s$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
client-deployment-78cdc5df45-96hkc   1/1     Running   0          44s

```


    So the client-deployment.yaml when i have changed the containers port from 3000 to 3001 so the template is changed and then the new pod got created and then old pod got deleted .






# Template has got the prominent importance in the client-deployment.yaml

```
# In this file for keys camel case is being used
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          image: stephengrider/multi-client
          ports:
            - containerPort: 3001
```


* Here template has got the metadata involved in int that is the metadata component: web having the labels 

* Here the selector exists actually the selector exists to have the control on the pods .

* For having the handle on the pods then the pods should need to be identitified by some property with the properties of the matchlabels of the 
```
  component: web
```
$ kubectl get deployments

```
NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
client-deployment   1         1         1            1           1d
```




# Scenario of updating the pods with the images that are latest can be done

```
  $ kubectl set image <objecttype>/<nameoftheobject(this exists in the metadatasection)> <container-name>=<dockerusername/image:tag>  
```
```
  kubectl set image deployment/client-deployment client = stephengrider/multi-client:v5
```


There can be 3 kinds of the solutions to this kind of the probleme those are 

Solution1. To delete the pods so that kubernetes refetches those images and builds the pods again

Solution2. To update the Configuration file as shown by tagging the image with a specific version 
```
# In this file for keys camel case is being used
apiVersion: apps/v1
kind: Deployment # this is the object type
metadata:
  name: client-deployment #thisis the object name
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          >>>>>>>>>>>>>change observer>>>>>>>>>>>>>>>
          image: stephengrider/multi-client:v2
          >>>>>>>>>>>>>>>>>>>>>>>>>>>>
          ports:
            - containerPort: 3000
```




Solution3: The imperative approach where the imperative command is being used 




