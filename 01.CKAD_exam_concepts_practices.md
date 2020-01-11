# CKAD EXAM PREPARATION 
Understand the CKAD concept, prepare for the exam and practice, practice practices!

### Documentation: 
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://github.com/dgkanatsios/CKAD-exercises

### Video Tutorial (Oreilly):
- Benjamin Muschko
https://learning.oreilly.com/live-training/courses/certified-kubernetes-application-developer-crash-course-ckad/0636920329176/

- Sander Van Vught
https://learning.oreilly.com/live-training/courses/certified-kubernetes-application-developer-ckad-crash-course/0636920318439/

### Time Management: 
19 Problems in 2 hours. Use your time wisely!
There might be weight of point for specific questions. 

### Using Alias for kubectl 
```bash
$ alias k="kubectl" 
$ alias kgetall="clear; kubectl get all --show-labels"
$ k version 
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```
### Setting Context & Namespace 
```bash
$ kubectl config current-contexts # check your current context 
$ kubectl config set-context <context-of-question> --namespace=<namespace-of-question>
$ kubectl config current-context 
$ kubectl config view | grep namespace
```

# Core Concepts 13%
- Understand Kubernetes API primitives
- Create and configure basic Pods
- (curriculum)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

### Deleting Kubernetes Objects 
Deleting object may take time, to save your time during the exam, don't wait for a graceful deletion of objects, just kill it!
```bash
$ kubectl delete pod <pod-name> --grace-period=0 --force
useful flag: --grace-period=0 --force
```
### Understand and Practice bash 
```bash
$ if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true; do echo $(date) >> ~/tmp/date.txt; sleep 5; done; 
$ while true; do kubectl get pods | grep docker-regist; sleep 3; done; 
```
### Object Management
-Imperative method : kubernetes
fast but requires detailed knowledge, no track record  
```bash
$ kubectl create namespace ckad
$ kubectl run nginx --image=nginx --restart=Never -n ckad 
```
-Declarative method: yaml
Suitable for more elaborate changes, tracks changes 
-Hybrid approach
Generate YAML file with kubectl
```bash
$ kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml 
$ vim nginx-pod.yaml 
$ kubectl apply -f nginx-pod.yaml 
```
### Pod life cycle
    -Pending
    -Running
    -Succeeded
    -Failed 
    -Unknown 

### Inspecting a Pod's status: 
-Get current status and event logs: 
```bash
$ kubectl describe pods <pod-name> | grep Status:
```
-Get current lifecycle phase: 
```bash
$ kubectl get pods <pod-name> -o yaml | grep phase 
```
### Configuring Env. Variables
Injecting runtime behaviour
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: spring-boot-app 
spec: 
  containers: 
  - image: bmuchko/spring-boot-app:1.5.3
    name: spring-boot-app 
    env:                              $ Added this 
    - name: SPRING_PROFILES_ACTIVE    $ Added this 
      value: production               $ Added this 
```
### Commands and Arguments
Running a command inside of container
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: nginx 
spec: 
  containers: 
  - image: nginx:latest 
    name: nginx 
    args:                 $ Added this 
    - /bin/sh             $ Added this 
    - -c                  $ Added this 
    - echo hello world    $ Added this 
```
### Other Useful kubectl commands 
```bash
$ kubectl logs <pod-name> 
$ kubectl exec -it <pod-name> -- /bin/sh 
```

### Exercise 1
In this exercise, you will practice the creation of a new Pod in a namespace. Once created, you will inspect it, shell into it and run some operations inside of the container.
### Creating a Pod and Inspecting it
1. Create the namespace `ckad-prep`.
2. In the namespace `ckad-prep` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.
3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.
4. Change the image of the Pod to `nginx:1.15.12`.
5. List the Pod and ensure that the container is running.
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
7. Retrieve the IP address of the Pod `mypod`.
8. Run a temporary Pod using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
9. Render the logs of Pod `mypod`.
10. Delete the Pod and the namespace.
11. Check whether any pods are failing in any available namespace. If you find the failing pods, try to find out the reason and solve it if possible. 

<details><summary> Solution 1 </summary>
<p>

```bash
$ kubectl create namespace ckad-prep 
$ kubectl get ns | grep ckad-prep 
$ kubectl run mypod --image=nginx:2.3.5 --port=80 --namespace=ckad-prep --restart=Never
$ kubectl describe pod mypod -n ckad-prep | tee pod-error.txt 
→ Error happened since the image can't be pulled (specified tag of the image doesn't exist)
$ kubectl edit pod mypod -n ckad-prep 
→ change this line to 
  image: nginx:1.15.12
$ kubectl get pod -n ckad-prep 
$ kubectl exec -it mypod -n ckad-prep -- /bin/sh 
  $ ls (inside the pod)
  $ exit 
$ kubectl get pods -n ckad-prep -o wide | grep mypod 
mypod   1/1     Running   0          93m   172.17.0.14   minikube   <none>           <none>
$ kubectl run busybox --image=busybox --rm -it --restart=Never --namespace=ckad-prep -- /bin/sh 
  $ (enter the busybox pod)
  $ wget 172.17.0.14:80
  $ (index.html created)
  $ (or execute this command to display the result) wget -O- 172.17.0.14:80
  $ exit 
  $ (busybox pod will be deleted automatically)
$ kubectl logs mypod -n ckad-prep 
$ kubectl delete pod mypod -n ckad-prep --grace-period=0 --force
$ kubectl delete namespace ckad-prep 

# Find any failing pods in available namespaces
# Get pod STATUS other than Running
$ kubectl get pods --all-namespaces | grep -v Running
# if you find the Crashloopback, error, etc, try to retrieve the logs, describe the pod's event. Solve the problem if possible (such as: edit the image tag, etc)
$ kubectl describe pod <pod-name> -n <namespace> | grep -i status
$ kubectl get pod <pod-name> -n <namespace> -o yaml | grep -i phase

# If we found error pod, check the previous logs by executing these commands: 
$ kubectl logs <pod-name> --previous 

```
</p>
</details>


### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml -n mynamespace
```

Alternatively, you can run in one line

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml | kubectl create -n mynamespace -f -
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
$ kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'env'
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c 'env' > envpod.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run
```

</p>
</details>

### Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```

</p>
</details>

### Create a pod with image nginx called nginx and allow traffic on port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

</p>
</details>

### Change pod's image to nginx:1.7.1. Observe that the pod will be killed and recreated as soon as the image gets pulled

<details><summary>show</summary>
<p>

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```
*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- wget -O- $NGINX_IP:80
``` 

</p>
</details>

### Get this pod's YAML without cluster specific information

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml --export
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>


# Configuration (ConfigMaps, SecurityContext, Secrets) 18%
- Understand ConfigMaps
- Understand SecurityContext
- Define application's resource requirements
- Create & consume Secrets
- Understand ServiceAccounts
- (curriculum)

### Centralized Configuration Data 
-Creating ConfigMap (Imperative)
(Literal values)
```bash
$ kubectl create configmap db-config ¥
  --from-literal=db=staging ¥
  --from-literal=username=jdoe

(Single file with environment variables)
$ kubectl create configmap db-config --from-env-file=config.env 

(File or directory)
$ kubectl create configmap db-config ¥
  --from-file=config.txt ¥
  --from-file=config-data.txt
```
-Creating ConfigMap (Declarative)
```yaml 
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: db-config 
data: 
  db: staging 
  username: jdoe 
```

### Exercise 2 (configmap)
In this exercise, you will first create a ConfigMap from predefined values in a file. Later, you'll create a Pod, consume the ConfigMap as environment variables and print out its values from within the container.

### Configuring a Pod to Use a ConfigMap
1. Create a new file named `config.txt` with the following environment variables as key/value pairs on each line.
- `DB_URL` equates to `localhost:3306`
- `DB_USERNAME` equates to `postgres`
2. Create a new ConfigMap named `db-config` from that file.
3. Create a Pod named `backend` that uses the environment variables from the ConfigMap and runs the container with the image `nginx`.
4. Shell into the Pod and print out the created environment variables. You should find `DB_URL` and `DB_USERNAME` with their appropriate values.
5. (Optional) Discuss: How would you approach hot reloading of values defined by a ConfigMap consumed by an application running in Pod?
- Create deployment `backend-depl` using nginx image. Use `db-config` configmap. 
- Check the current `DB_USERNAME` env. Change the `DB_USERNAME` value to `sqlite`.
- Restart your pod and make sure all container from `backend-depl` have the new value of `DB_USERNAME`

<details><summary>Solution 2 (configmap)</summary>
<p>

```bash
$ vim config.txt 
  DB_URL=localhost:3306
  DB_USERNAME=postgres 
$ kubectl create configmap db-config --from-env-file=config.txt
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > pod.yaml 
$ vi pod.yaml 
  ...
  spec: 
    containers: 
    - image: nginx 
      name: backend 
      envFrom: 
        - configMapRef: 
            name: db-config 
  ...
$ kubectl create -f pod.yaml 
$ kubectl get pods | grep backend 
$ kubectl exec -it backend -- env | grep DB_
```

```bash
# answer 5
$ kubectl create deployment backend-depl --image=nginx --dry-run -o yaml > ex2.deployment.yaml
$ vim ex2.deployment.yaml
...
spec:
  template:
    spec:
      containers:
        envFrom:
        - configMapRef:
            name: db-config
...

# edit the configmap value
$ kubectl edit cm db-config
DB_USERNAME=sqlite

$ kubectl describe cm db-config

$ kubectl create -f ex2.deployment.yaml
$ kubectl exec <pod-name> -- env | grep DB_USERNAME
```

</p>
</details>

### Create a configmap named config with values foo=lala,foo2=lolo

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
kubectl get cm config -o yaml --export
# or
kubectl describe cm config
```

</p>
</details>

### Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

</p>
</details>

### Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml --export
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml --export
```

</p>
</details>

### Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

<details><summary>show</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # will show 'option=val5'
```

</p>
</details>

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

</p>
</details>

### Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

<details><summary>show</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

</p>
</details>

### Creating Secrets(Imperative)
kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

```bash
$ kubectl create secret generic db-creds ¥
  --from-literal=pwd=s3cre!
$ kubectl create secret generic db-creds ¥
  --from-env-file=secret.env 
$ kubectl create secret generic db-creds ¥
  --from-file=username.txt 
  --from-file=password.txt 

-Value has to be base64-encoded manually 
$ echo -n 's3cre!' | base64 
czNjcmUh=

apiVersion: v1 
kind: Secret 
metadata: 
  name: mysecret 
type: Opaque 
data: 
  pwd: czNjcmUh=

```

### Using Secret as Files from a Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:               $ secret in pod 
    - name: foo                 $ secret in pod 
      mountPath: "/etc/foo"     $ secret in pod 
      readOnly: true            $ secret in pod 
  volumes:                      $ secret in pod 
  - name: foo                   $ secret in pod 
    secret:                     $ secret in pod 
      secretName: mysecret      $ secret in pod 

```

### Using Secret as Environment Variables 

```bash
apiVersion: v1 
kind: Pod 
metadata: 
  name: mypod 
spec: 
  containers: 
  - name: mycontainer
    image: redis 
    env: 
      - name: SECRET_USERNAME
        valueFrom: 
          secretKeyRef: 
            name: mysecret 
            key: username 
      - name: SECRET_PASSWORD
        valueFrom: 
          secretKeyRef: 
            name: mysecret
            key: password 
  restartPolicy: Never

```


### Exercise 3 (secret)
In this exercise, you will first create a Secret from literal values. Next, you'll create a Pod and consume the Secret as environment variables. Finally, you'll print out its values from within the container.

### Configuring a Pod to Use a Secret
1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.
2. Create a Pod named `backend` that defines uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx`.
3. Shell into the Pod and print out the created environment variables. You should find `DB_PASSWORD` variable.
4. (Optional) Discuss: What is one of the benefit of using a Secret over a ConfigMap?

<details><summary>Solution 3 (secret)</summary>
<p>

```bash
$ kubectl create secret generic db-credentials --from-literal=db-password=passwd
$ kubectl get secrets 
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > backend-pod.yaml 
$ vim backend-pod.yaml 
...
spec: 
  containers: 
  - name: backend 
    image: nginx 
    env:
      - name: DB_PASSWORD 
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: db-password
...

$ kubectl create -f backend-pod.yaml
$ kubectl get pod 
$ kubectl exec -it backend -- /bin/sh 
  $ env | grep DB_PASSWORD

```
</p>
</details>

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml --export
echo YWRtaW4K | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d  # on MAC it is -D
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret2 # our secret's name
          key: username # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

</p>
</details>


### Security Context 
kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

- Set Security Context for a Pod 
```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: security-pod-demo 
spec: 
  securityContext:      $securityContext for Pod  
    runAsUser: 1000     $securityContext for Pod 
    runAsGroup: 3000    $securityContext for Pod 
    fsGroup: 2000       $securityContext for Pod 
  volumes:              $securityContext for Pod 
  - name: sec-ctx-vol   $securityContext for Pod 
    emptyDir: {}        $securityContext for Pod 
  containers: 
  - name: security-container-demo 
    image: busybox 
    volumeMounts:           $securityContext for Container 
    - name: sec-ctx-vol     $securityContext for Container
      mountPath: /data/demo $securityContext for Container
```
-runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000.

-runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod.

-fsGroup field specified all processes of the container are also part of the supplementary group ID 2000. The owner for volume /data/demo and any files created in that volume will be Group ID 2000.


### Exercise 4 (security context)
In this exercise, you will create a Pod that defines a filesystem group ID as security context. Based on this security context, you'll create a new file and inspect the outcome of the file creation based on the rule defined earlier.

### Creating a Security Context for a Pod
1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.
2. Files created on the volume should use the filesystem group ID 3000.
3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

<details><summary>Solution 4 (security context)</summary>
<p>

```bash
$ kubectl run secured --image=nginx --restart=Never -o yaml --dry-run > secured-pod.yaml 
$ vi secured-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secured
  name: secured
spec:
  containers:
  - image: nginx
    name: secured
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
------
...
spec: 
  securityContext: 
    fsGroup: 3000
  volumes: 
  - name: sec-ctx-vol 
    emptyDir: {}
  containers: 
  - image: nginx 
    name: secured 
    volumeMounts: 
    - name: sec-ctx-vol 
      mountPath: /data/app 
...

$ kubectl create -f secured-pod.yaml 
$ kubectl exec -it secured -- /bin/sh 
  $ cd /data/app
  $ touch logs.txt 
  $ ls -l 
  total 0
  -rw-r--r-- 1 root 3000 0 Dec  2 21:34 log.txt
  $ exit 

```
</p>
</details>

### Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>


### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    securityContext: # insert this line
      capabilities: # and this
        add: ["NET_ADMIN", "SYS_TIME"] # this as well
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

</p>
</details>


### Exercise 5 (resource boundary)
In this exercise, you will create a ResourceQuota with specific CPU and memory limits for a new namespace. Pods created in the namespace will have to adhere to those limits.

### Defining a Pod’s Resource Requirements
Create a resource quota named `apps` under the namespace `rq-demo` using the following YAML definition in the file `rq.yaml`.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500m
```

1. Create a new Pod that exceeds the limits of the resource quota requirements e.g. by defining 1G of memory. Write down the error message.
2. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

<details><summary>Solution 5 (resource boundary)</summary>
<p>

```
$ kubectl create namespace rq-demo 
$ vi rq.yaml 
apiVersion: v1 
kind: ResourceQuota 
metadata: 
  name: apps
spec: 
  hard: 
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500m

$ kubectl create -f rq.yaml -n rq-demo 
$ kubectl describe quota -n rq-demo  
$ kubectl run test-pod --image=nginx --restart=Never -o yaml > test-pod.yaml
$ vi test-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: nginx
    name: test-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
-----(edit to this one:)
...
spec: 
  containers: 
  - image: nginx 
    name: test-pod 
    resources: 
      requests: 
        memory: "1G"
        cpu: "200m"
-----

$ kubectl create -f pod.yaml -n rq-demo 
Error from server (Forbidden): error when creating "test-pod.yaml": pods "test-pod" is forbidden: exceeded quota: apps, requested: requests.cpu=1G, used: requests.cpu=0, limited: requests.cpu=2
$ vi test-pod.yaml 
...
spec: 
  resources: 
    requests: 
      memory: "200m"
      cpu: "200m"
...

$ kubectl get pods -n rq-demo 
$ kubectl create -f pod.yaml -n rq-demo 
$ kubectl describe quota --namespace=rq-demo 
Name:            apps
Namespace:       rq-demo
Resource         Used  Hard
--------         ----  ----
pods             1     2
requests.cpu     200m  2
requests.memory  200m  200m

```
</p>
</details>

### Service Account

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).
When you create a pod, if you do not specify a service account, it is automatically assigned the default service account in the same namespace.

### Exercise 6 (service account)
In this exercise, you will create a ServiceAccount and assign it to a Pod.
$$ Using a ServiceAccount
1. Create a new service account named `backend-team`.
2. Print out the token for the service account in YAML format.
3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.
4. Get a shell to the running container and print out the token of the service account.

<details><summary>Solution 6 (service account)</summary>
<p>

```bash
$ kubectl create serviceaccount backend-team 
$ kubectl get serviceaccount backend-team -o yaml --export 
→ --export $ Get a resource's YAML without cluster specific information
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: backend-team
  selfLink: /api/v1/namespaces/default/serviceaccounts/backend-team
secrets:
- name: backend-team-token-7qxd8

$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > backend-pod.yaml 
$ vi backend-pod.yaml 
...
spec: 
  serviceAccountName: backend-team
...
$ kubectl create -f backend-pod.yaml 
$ kubectl get pod backend -o yaml | less 
  → find the serviceaccount's secret name: backend-team-token-7qxd8
  → it should be somewhere here: 
  volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: backend-team-token-7qxd8
      readOnly: true
$ kubectl exec -it backend -- /bin/sh 
  $ ls /var/run/secrets/kubernetes.io/serviceaccount
  $ cat token

--or--
$ kubectl run backend2 --image=nginx --restart=Never --serviceaccount=backend-team 
$ kubectl get pod 
$ kubectl exec -it backend2 -- /bin/sh 
  $ cd /var/run/secrets/kubernetes.io/serviceaccount
  $ ls -l 
  $ cat token 
```
</p>
</details>


### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
kubectl create sa myuser
```

Alternatively:

```bash
# let's get a template easily
kubectl get sa default -o yaml --export > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### Create an nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run > pod.yaml
kubectl apply -f pod.yaml
```

or you can add manually:

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: myuser # we use pod.spec.serviceAccountName
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx # will see that a new secret called myuser-token-***** has been mounted
```


</p>
</details>

# Multi-container Pod 10%
- Understand Multi-Container Pod design patterns (ambassador, adapter, sidecar, initcontainer)
- (curriculum)

- https://blog.nillsf.com/index.php/2019/07/28/ckad-series-part-4-multi-container-pods/
1. Sidecar container 
adds functionality to your application that could be included in the main container. By hosting this logic in a sidecar, you can keep that functionality out of your main application and evolve that independently from the actual application.
2. Ambassador container 
proxies a local connection to certain outbound connection. The ambassadors brokers the connection the outside world. This can for instance be used to shard a service or to implement client side load balancing.
3. Adapter container 
takes data from the existing application and presents that in a standardized way. This is for instance very useful for monitoring data.
4. Init container
An init-container is a special container that runs before the other containers in your pod.


### Exercise 7 (InitContainer)
In this exercise, you will initialize a web application by standing up environment-specific configuration through an init container.
### Creating an Init Container
Kubernetes runs an init container before the main container. In this scenario, the init container retrieves configuration files from a remote location and makes it available to the application running in the main container. The configuration files are shared through a volume mounted by both containers. The running application consumes the configuration files and can render its values.
1. Create a new Pod in a YAML file named `business-app.yaml`. The Pod should define two containers, one init container and one main application container. Name the init container `configurer` and the main container `web`. The init container uses the image `busybox`, the main container uses the image `bmuschko/nodejs-read-config:1.0.0`. Expose the main container on port 8080.
2. Edit the YAML file by adding a new volume of type `emptyDir` that is mounted at `/usr/shared/app` for both containers.
3. Edit the YAML file by providing the command for the init container. The init container should run a `wget` command for downloading the file `https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-creating-init-container/app/config/config.json` into the directory `/usr/shared/app`.
4. Start the Pod and ensure that it is up and running.
5. Run the command `curl localhost:8080` from the main application container. The response should render a database URL derived off the information in the configuration file.
6. (Optional) Discuss: How would you approach a debugging a failing command inside of the init container?

<details><summary>Solution 7 (InitContainer)</summary>
<p>

```bash
$ kubectl run business-app --image=bmuschko/nodejs-read-config:1.0.0 --port=8080 --restart=Never -o yaml --dry-run > business-app.yaml 
$ vi business-app.yaml 
(before)
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: business-app
  name: business-app
spec:
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: business-app
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

(after)
spec:
  initContainers: 
  - name: configurer 
    image: busybox 
    volumeMounts: 
    - name: configdir 
      mountPath: /usr/shared/app
    command: 
    - wget
    - "-O"
    - "/usr/shared/app/config.json"
    - "https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-creating-init-container/app/config/config.json"
  containers: 
  - name: web 
    image: bmuschko/nodejs-read-config:1.0.0
    ports: 
    - containerPort: 8080
    resources: {}
    volumeMounts: 
    - name: configdir 
      mountPath: /usr/shared/app
  volumes: 
  - name: configdir
    emptyDir: {}

$ kubectl apply -f business-app.yaml 

-Check the log from all containers: 
$ kubectl logs business-app --all-containers=true
Connecting to raw.githubusercontent.com (151.101.108.133:443)
wget: note: TLS certificate validation not implemented
saving to '/usr/shared/app/config.json'
config.json          100% |********************************|   102  0:00:00 ETA
'/usr/shared/app/config.json' saved
Server running at http://0.0.0.0:8080/

$ kubectl exec -it business-app --container web -- /bin/sh
  $ ls -l /usr/shared/app/config.json
  $ curl localhost:8080
  Database URL: localhost:5432/customers
  $ exit 
```
</p>
</details>

### Exercise 8
In this exercise, you will implement the adapter pattern for a multi-container Pod.

### Implementing the Adapter Pattern
The adapter pattern helps with providing a simplified, homogenized view of an application running within a container. For example, we could stand up another container that unifies the log output of the application container. As a result, other monitoring tools can rely on a standardized view of the log output without having to transform it into an expected format.

1. Create a new Pod in a YAML file named `adapter.yaml`. The Pod declares two containers. The container `app` uses the image `busybox` and runs the command `while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;`. The adapter container `transformer` uses the image `busybox` and runs the command `sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;` to strip the log output off the date for later consumption by a monitoring tool. Be aware that the logic does not handle corner cases (e.g. automatically deleting old entries) and would look different in production systems.
2. Before creating the Pod, define an `emptyDir` volume. Mount the volume in both containers with the path `/var/logs`.
3. Create the Pod, log into the container `transformer`. The current directory should continuously write a new file every 20 seconds.

<details><summary> Solution 8 </summary>
<p>

```bash
$ kubectl run app --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'> pod.yaml
```
```bash
$ vi pod.yaml
```

the yaml file should be like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'
    image: busybox
    name: app
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

edit the yaml for declaring two containers with specific details:
```yaml
...
spec:
  volumes: 
  - name: configdir
    emptyDir: {}
  containers:
  - name: app
    image: busybox
    args:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'
    volumeMounts:
    - name: configdir
      mountPath: /var/logs
    resources: {}
  - name: transformer
    image: busybox
    args: 
    - /bin/sh
    - -c
    - 'sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;'
    volumeMounts:
    - name: configdir
      mountPath: /var/logs
  dnsPolicy: ClusterFirst
  restartPolicy: Never
...
```
```bash
$ kubectl create -f pod.yaml
$ kubectl get pod
$ kubectl exec -it app --container=transformer -- /bin/sh
  $ ls -l | grep transform
  -rw-r--r--    1 root     root          1800 Dec  7 10:25 2019-12-07-10-25-14-transformed.txt
  -rw-r--r--    1 root     root          1848 Dec  7 10:25 2019-12-07-10-25-34-transformed.txt
  -rw-r--r--    1 root     root          1896 Dec  7 10:25 2019-12-07-10-25-54-transformed.txt
  $ cat 2019-12-07-10-25-54-transformed.txt
    8.0K	/root
    8.0K	/root
    8.0K	/root
  $ exit
```
</p>
</details>

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit
# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>


# Observability (Probes, Logging, Monitoring, Debugging) 18%
- Understand LivenessProbes and ReadinessProbes
- Understand container logging
- Understand how to monitor applications in Kubernetes
- Understand debugging in Kubernetes
- (curriculum)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)

Understanding readiness probes
- Is application is ready to serve request?
- A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

Defining a Readiness Probe
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-app
    image: eshop:4.6.3
    readinessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 2
```

Understanding liveness probe
- Does the application function without errors?
- If the pod doesn't respond with it liveness, the kubelet kills and restarts the Container.
- 1. Liveness Command
```yaml
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
```
- 2. Liveness HTTP request
```yaml
      livenessProbe:
        httpGet: 
          path: /healthz
          port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
```
- 3. Liveness TCP probe
```yaml
      livenessProbe:
        tcpSocket:
          port: 8080
```

Defining a Liveness Probe
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: web-app
spec:
  containers:
  - name: web-app
    image: eshop:4.6.3
    livenessProbe:
      exec:
        command: 
        - cat
        - /tmp/healthy
      initialDelaySeconds: 10
      periodSeconds: 5
```


### Exercise 9 (readiness liveness)
In this exercise, you will create a Pod running a NodeJS application. The Pod will define readiness and liveness probes with different parameters.

### Defining a Pod’s Readiness and Liveness Probe
1. Create a new Pod named `hello` with the image `bmuschko/nodejs-hello-world:1.0.0` that exposes the port 3000. Provide the name `nodejs-port` for the container port.
2. Add a Readiness Probe that checks the URL path / on the port referenced with the name `nodejs-port` after a 2 seconds delay. You do not have to define the period interval.
3. Add a Liveness Probe that verifies that the app is up and running every 8 seconds by checking the URL path / on the port referenced with the name `nodejs-port`. The probe should start with a 5 seconds delay.
4. Shell into container and curl `localhost:3000`. Write down the output. Exit the container.
5. Retrieve the logs from the container. Write down the output.

<details><summary> Solution 9 (readiness liveness) </summary>
<p>

```bash
$ kubectl run hello --image=bmuschko/nodejs-hello-world:1.0.0 --restart=Never --port=3000 -o yaml --dry-run > hello-pod.yaml
```

Edit the pod yaml for Readiness and Liveness probe:
```bash
$ vim hello-pod.yaml
```
this is before the edit
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: hello
  name: hello
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello
    ports:
    - containerPort: 3000
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

after the edit
```yaml
...
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 2
    livenessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 5
      periodSeconds: 8
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
...
```

```bash
$ kubectl create -f hello-pod.yaml
$ kubectl exec -it hello -- /bin/sh
  $ curl localhost:3000
    Hello World
  $ exit
$ kubectl logs hello | tee container-log.txt
Magic happens on port 3000

```
</p>
</details>

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # run this to see that liveness probe works
kubectl delete -f pod.yaml
```

</p>
</details>

### Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the period of probing would be 10 seconds. Run it, check the probe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # get the exact names
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: 
      initialDelaySeconds: 5 # add this line
      periodSeconds: 10 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod.yaml
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
      - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod.yaml
```

</p>
</details>

## Logging
### Create a busybox pod that runs 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'. Check its logs

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```

</p>
</details>

## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```

</p>
</details>

### Get CPU/memory utilization for nodes ([metrics-server](https://github.com/kubernetes-incubator/metrics-server) must be running)

<details><summary>show</summary>
<p>

```bash
kubectl top nodes
```

</p>
</details>


## Debugging existing pods
- $ kubectl get all
- $ kubectl describe pod <pod-name>
- $ kubectl describe pod <pod-name> --container <container-name>
- $ kubectl logs <pod-name> 
- $ kubectl logs <pod-name> --container <container-name>

### Exercise 10
In this exercise, you will training your debugging skills by inspecting and fixing a misconfigured Pod.

### Fixing a Misconfigured Pod
1. Create a new Pod with the following YAML.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: failing-pod
  name: failing-pod
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do echo $(date) >> ~/tmp/curr-date.txt; sleep
      5; done;
    image: busybox
    name: failing-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
2. Check the Pod's status. Do you see any issue?
3. Follow the logs of the running container and identify an issue.
4. Fix the issue by shelling into the container. After resolving the issue the current date should be written to a file. Render the output.

<details><summary> Solution 10 </summary>
<p>

Create pod's yaml file
```bash
$ vim pod.yaml
(copy paste the yaml from the problem)
...

$ kubectl create -f pod.yaml
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
failing-pod   1/1     Running   0          48s

$ kubectl logs failing-pod
  /bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory
  /bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory
  /bin/sh: can't create /root/tmp/curr-date.txt: nonexistent directory
$ kubectl exec -it failing-pod -- /bin/sh
  $ ls ~/tmp/curr-date.txt
    ls: /root/tmp/curr-date.txt: No such file or directory
  $ ls ~/tmp
    ls: /root/tmp: No such file or directory
  $ mkdir ~/tmp
  $ cd ~/tmp
  $ ls -l
    total 4
    -rw-r--r--    1 root     root           140 Dec  7 13:38 curr-date.txt
  $ cat curr-date.txt
    Sat Dec 7 13:38:00 UTC 2019
    Sat Dec 7 13:38:05 UTC 2019
    Sat Dec 7 13:38:10 UTC 2019
  $ exit
```
</p>
</details>



# Pod Design 20%
- Understand how to use Labels, Selectors, and Annotations
- Understand Deployments and how to perform rolling updates
- Understand Deployments and how to perform rollbacks
- Understand Jobs and CronJobs
- (curriculum)

## Labels
### Purpose of Labels
- Essential to querying, filtering and sorting Kubernetes objects

### Assigining Labels
- Defined in the metadata section of a Kubernetes object definition
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: pod1
  labels: 
    tier: backend
    env: prod
    app: miracle
```

Querying multiple label is work as boolean search
```bash
$ kubectl get pods --show-labels

$ kubectl get pods -l tier=frontend,env=dev --show-labels

# Has the label with key "version"
$ kubectl get pods -l version --show-labels

# Tier label is frontend or backend, and Environment is development
$ kubectl get pods -l 'tier in (frontend,backend),env=dev' --show-labels
```

## Selectors
- Grouping resources by label selectors
```yaml
spec: 
  ...
  selector: 
    tier: frontend
    env: dev
```

```yaml
spec: 
...
  selector: 
    matchLabels:
      version: v2.1
    matchExpressions:
    - {key: tier, operator: In, values: {frontend,backend}}
```

## Annotations
- Purpose of annotations : descriptive metadata without the ability to be queryable

### Assigining Annotations
- Defined in the metadata section of a Kubernetes object definition
```yaml
...
kind: Pod
metadata:
  annotations:
    commit: 866a8dc
    author: 'D Fahmi'
    branch: 'ff/bugfix'
```

### Exercise 11
In this exercise, you will exercise the use of labels and annotations for a set of Pods.

### Defining and Querying Labels and Annotations
1. Create three different Pods with the names `frontend`, `backend` and `database` that use the image `nginx`.
2. Declare labels for those Pods as follows:
- `frontend`: `env=prod`, `team=shiny`
- `backend`: `env=prod`, `team=legacy`, `app=v1.2.4`
- `database`: `env=prod`, `team=storage`

3. Declare annotations for those Pods as follows:
- `frontend`: `contact=John Doe`, `commit=2d3mg3`
- `backend`: `contact=Mary Harris`

4. Render the list of all Pods and their labels.
5. Use label selectors on the command line to query for all production Pods that belong to the teams `shiny` and `legacy`.
6. Remove the label `env` from the `backend` Pod and rerun the selection.
7. Render the surrounding 3 lines of YAML of all Pods that have annotations.

<details><summary> Solution 11 </summary>
<p>

```bash
$ kubectl run frontend --image=nginx --restart=Never -o yaml --dry-run > frontend.yaml
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > backend.yaml
$ kubectl run database --image=nginx --restart=Never -o yaml --dry-run > database.yaml
```

declare labels and annotations for frontend.yaml
```yaml
...
metadata:
  creationTimestamp: null
  name: frontend
  labels:
    run: frontend
    env: prod
    team: shiny
  annotations:
    contact: 'John Doe'
    commit: 2d3mg3
...
```

declare labels and annotations for backend.yaml
```yaml
metadata:
  creationTimestamp: null
  name: backend
  labels:
    run: backend
    env: prod
    team: legacy
    app: v1.2.4
  annotations:
    contact: 'Mary Harris'
```

declare labels for database.yaml
```yaml
metadata:
  creationTimestamp: null
  name: database
  labels:
    run: database
    env: prod
    team: storage
```

```bash
# Create all pods
$ kubectl create -f frontend.yaml
$ kubectl create -f backend.yaml
$ kubectl create -f database.yaml

$ kubectl get pods --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
backend    1/1     Running   0          75s   app=v1.2.4,env=prod,run=backend,team=legacy
database   1/1     Running   0          70s   env=prod,run=database,team=storage
frontend   1/1     Running   0          81s   env=prod,run=frontend,team=shiny

$ kubectl get pods -l 'env=prod,team in (shiny,legacy)' --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
backend    1/1     Running   0          4m46s   app=v1.2.4,env=prod,run=backend,team=legacy
frontend   1/1     Running   0          4m52s   env=prod,run=frontend,team=shiny

# Remove label env from backend Pod
$ kubectl label pods backend env-
pod/backend labeled

$ kubectl get pods -l 'env=prod,team in (shiny,legacy)' --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
frontend   1/1     Running   0          12m   env=prod,run=frontend,team=shiny

$ kubectl get pods -o yaml | grep 'annotations' -C 3
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      contact: Mary Harris
    creationTimestamp: 2019-12-07T23:20:01Z
    labels:
--
--
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      commit: 2d3mg3
      contact: John Doe
    creationTimestamp: 2019-12-07T23:19:55Z
```

</p>
</details>


## Deployment
- Scaling and replication features for pods

### Creating deployment
```bash
$ kubectl create deployment nginx-deployment --image=nginx --dry-run -o yaml > deploy.yaml
$ vim deploy.yaml
$ kubectl create -f deploy.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Change the pod's image and record the change to the rollout history
```bash
# deployment-name: deploy-test1, image: nginx to nginx:latest
$ kubectl set image deployment deploy-test1 nginx=nginx:latest --record

$ kubectl rollout history deployment deploy-test1 
deployment.apps/deploy-test1
REVISION  CHANGE-CAUSE
2         <none>
3         kubectl set image deployment deploy-test1 nginx=nginx:latest --record=true
```


### Exercise 12 (Deployment)
In this exercise, you will create a Deployment with multiple replicas. After inspecting the Deployment, you will update its parameters. Furthermore, you will use the rollout history to roll back to a previous revision.

### Performing Rolling Updates for a Deployment
1. Create a Deployment named `deploy` with 3 replicas. The Pods should use the `nginx` image and the name `nginx`. The Deployment uses the label `tier=backend`. The Pods should use the label `app=v1`.
2. List the Deployment and ensure that the correct number of replicas is running.
3. Update the image to `nginx:latest`.
4. Verify that the change has been rolled out to all replicas.
5. Scale the Deployment to 5 replicas.
6. Have a look at the Deployment rollout history.
7. Revert the Deployment to revision 1.
8. Ensure that the Pods use the image `nginx`.
9. (Optional) Discuss: Can you foresee potential issues with a rolling deployment? How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?

<details><summary> Solution 12 (Deployment) </summary>
<p>

```bash
$ kubectl create deployment deploy --image=nginx -o yaml --dry-run > deploy.yaml

# Check the created deploy.yaml
$ vim deploy.yaml

```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploy
  name: deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

edit deploy.yaml to meet the exam requirement:
```yaml
# set the Deployment's label to tier=backend
metadata:
  creationTimestamp: null
  labels:
    tier: backend
  name: deploy

# set replicas to 3, and The Pods should use the label app=v1.
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: v1
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
```

```bash
# apply the yaml file
$ kubectl create -f deploy.yaml
deployment.apps/deploy created

$ kubectl get deployment
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
deploy   3/3     3            3           13s
```

Update the image to nginx:latest.
```bash
$ kubectl set image deployment/deploy nginx=nginx:latest
deployment.apps/deploy image updated

# Get more information about the current deployment revision
$ kubectl rollout history deploy --revision=2 
deployment.apps/deploy with revision #2
Pod Template:
  Labels:	app=v1
	pod-template-hash=8cfcc49bc
  Containers:
   nginx:
    Image:	nginx:latest
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

```

Now scale the Deployment to 5 replicas.

```shell
$ kubectl scale deployments deploy --replicas=5
deployment.extensions/deploy scaled
```

Roll back to revision 1. You will see the new revision. Inspecting the revision should show the image `nginx`.

```shell
$ kubectl rollout undo deployment/deploy --to-revision=1
deployment.extensions/deploy

$ kubectl rollout history deploy
deployment.extensions/deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

$ kubectl rollout history deploy --revision=3
deployment.extensions/deploy with revision #3
Pod Template:
  Labels:	app=v1
	pod-template-hash=454670702
  Containers:
   nginx:
    Image:	nginx
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

### Optional

> Can you foresee potential issues with a rolling deployment?

A rolling deployment ensures zero downtime which has the side effect of having two different versions of a container running at the same time. This can become an issue if you introduce backward-incompatible changes to your public API. A client might hit either the old or new service API.

> How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?

You can configure the deployment use the `Recreate` strategy. This strategy first kills all existing containers for the deployment running the current version before starting containers running the new version.

</p>
</details>



### Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

<details><summary>show</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

</p>
</details>

### Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
```

</p>
</details>

### Get the label 'app' for the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
```

</p>
</details>

### Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
```

</p>
</details>

### Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -lapp app-
```

</p>
</details>

### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

We can use the 'nodeSelector' property on the Pod YAML:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the slection label
```

You can easily find out where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

</p>
</details>

### Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'
```

</p>
</details>

### Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx1 | grep -i 'annotations'
```

As an alternative to using `| grep` you can use jsonPath like `-o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx:1.7.8 --replicas=2 --port=80
```

**However**, `kubectl run` for Deployments is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.7.8/image: nginx:1.7.8\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

</p>
</details>

### View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### Update the nginx image to nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.7.9
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.7.8
```

</p>
</details>

### Do an on purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and Enter, to navigate quickly
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull'
```

</p>
</details>


### Return the deployment to the second revision (number 2) and verify the image is nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

</p>
</details>

### Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### Update the image to nginx:1.9.1 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.9.1
# or
kubectl edit deploy nginx
# change the image to nginx:1.9.1
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### Resume the rollout and check that the nginx:1.9.1 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
```

</p>
</details>

### Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

## Jobs

### Create a job with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
kubectl run pi --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

**However**, `kubectl run` for Job is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=OnFailure -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

**However**, `kubectl run` for Job is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>
  
```bash
kubectl create job busybox --image=busybox --dry-run -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```
  
Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=OnFailure --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

**However**, `kubectl run` for CronJob is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>


# State Persistence (SP) 8%
- Understand PersistentVolumeClaims for storage
- (curriculum)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)


### SP1. Defining and Mounting a PersistentVolume
In this exercise, you will create a PersistentVolume, connect it to a PersistentVolumeClaim and mount the claim to a specific path of a Pod.

1. Create a Persistent Volume named `pv`, access mode `ReadWriteMany`, storage class name `shared`, 5MB of storage capacity and the host path `/data/config`.
2. Create a Persistent Volume Claim named `pvc` that requests the Persistent Volume in step 1. The claim should request 1MB. Ensure that the Persistent Volume Claim is properly bound after its creation.
3. Mount the Persistent Volume Claim from a new Pod named `app` with the path `/var/app/config`. The Pod uses the image `nginx`.
4. Check the events of the Pod after starting it to ensure that the Persistent Volume was mounted properly.

<details><summary> Show SP1 </summary>
<p>

Create a YAML file for the Persistent Volume and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 5m
  accessModes:
    - ReadWriteMany
  storageClassName: shared
  hostPath:
    path: /data/config
```
- If you are using minikube, make sure the /data/config folder is exist
```bash
# ssh the minikube
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ls -ld /data/config
drwxr-xr-x 2 root root 4096 Dec 14 10:17 /data/config

# directory is existed!
# try creating one file for the next checking
$ sudo su -
$ echo "this file is created by MINIKUBE" > minikube_created_file.txt
$ ls 
  minikube_created_file.txt
```

You will see that the Persistent Volume has been created but and is available to be claimed.

```shell
$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv     5m         RWX            Retain           Available           shared                  119s
```

Create a YAML file for the Persistent Volume Claim and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1m
  storageClassName: shared
```

You will see that the Persisten Volume Claim has been created and has been bound to the Persisten Volume.

```shell
$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc    Bound    pv       512m       RWX            shared         2s

$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
pv     512m       RWX            Retain           Bound    default/pvc   shared                  1m
```

Create pod with yaml declarative method
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pvc
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: "/var/app/config"
          name: pv-storage
```

```bash
$ kubectl create -f pod.yaml

$ kubectl get pods 
NAME   READY   STATUS    RESTARTS   AGE
app    1/1     Running   0          2m44s

```


```shell
# Enter to the pod and make sure the created directory exist
$ kubectl exec -it app -- /bin/bash
  $ cd /var/app/config
  $ ls -l
  total 8
  -rw-r--r-- 1 root root 42 Dec 14 10:42 minikube_created_file.txt
  # the file we created is exist 
  $ cat minikube_created_file.txt
  this is file created by MINIKUBE minikube

```
</p>
</details>

### SP2. Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

<details><summary>show SP2. </summary>
<p>

*This question is probably a better fit for the 'Multi-container-pods' section but I'm keeping it here as it will help you get acquainted with state*

Easiest way to do this is to create a template pod with:

```bash
$ kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'sleep 3600' > pod.yaml
$ vi pod.yaml
```
Copy paste the container definition and type the lines that have a comment in the end:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - name: busybox
    args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```
```bash
$ kubectl -f pod.yaml
```

Connect to the second container:

```bash
$ kubectl exec -it busybox -c busybox2 -- /bin/sh
$ cat /etc/passwd 
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false

$ cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd 

$ cat /etc/foo/passwd # confirm that stuff has been written successfully
root
daemon
bin
sys
sync
mail
www-data
operator
nobody

$ exit
```

Connect to the first container:

```bash
$ kubectl exec -it busybox -c busybox -- /bin/sh
$ mount | grep foo # confirm the mounting
$ cat /etc/foo/passwd
$ exit

$ kubectl delete po busybox --grace-period=0 --force
```

</p>
</details>


### SP3.1. Create a PersistentVolume of 1Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

<details><summary>show SP3.1.</summary>
<p>

```bash
$ vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Show the PersistentVolumes:

```bash
$ kubectl create -f pv.yaml
# will have status 'Available'

$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
myvolume   1Gi        RWO,RWX        Retain           Available           normal                  3s

$ kubectl describe pv
Name:            myvolume
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    normal
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO,RWX
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/foo
    HostPathType:
Events:            <none>
```

</p>
</details>

### SP3.2. Create a PersistentVolumeClaim for this storage class, called mypvc, a request of 1MB and an accessMode of ReadWriteOnce and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show SP3.2.</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1m
```

Create it on the cluster:

```bash
$ kubectl create -f pvc.yaml
```

Show the PersistentVolumeClaims and PersistentVolumes:

```bash
$ kubectl get pvc # will show as 'Bound'
$ kubectl get pv # will show as 'Bound' as well
```

</p>
</details>

### SP3.3. Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show SP3.3.</summary>
<p>

Create a skeleton pod:

```bash
$ kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'sleep 3600' > pod.yaml
$ vi pod.yaml
```

Add the lines that finish with a comment:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Create the pod:

```bash
$ kubectl create -f pod.yaml
```

Connect to the pod and copy '/etc/passwd' to '/etc/foo/passwd':

```bash
$ kubectl exec busybox -it -- /bin/sh
  $ cp /etc/passwd /etc/foo/passwd
  $ exit
```

</p>
</details>

### SP3.4. Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup

<details><summary>show SP3.4.</summary>
<p>

Create the second pod, called busybox2:

```bash
$ vim pod.yaml
# change 'metadata.name: busybox' to 'metadata.name: busybox2'
$ kubectl create -f pod.yaml
$ kubectl exec busybox2 -- ls /etc/foo # will show 'passwd'
# cleanup
$ kubectl delete po busybox busybox2
```

</p>
</details>

### SP4. Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>show SP4.</summary>
<p>

```bash
$ kubectl run busybox --image=busybox --restart=Never -- sleep 3600
$ kubectl cp busybox:/etc/passwd ./passwd # kubectl cp command
# previous command might report an error, feel free to ignore it since copy command works
$ cat passwd
```

</p>
</details>


# Service & Networking (SN) 13%
- Understand Services
- Demonstrate basic understanding of NetworkPolicies
- (curriculum)

### EXERCISES FOR SERVICE & NETWORKING

### Exercise 14 (Service)
SN1. In this exercise, you will create a Deployment and expose a container port for its Pods. You will demonstrate the differences between the service types ClusterIP and NodePort.

### Routing Traffic to Pods from Inside and Outside of a Cluster
1. Create a deployment named `myapp` that creates 2 replicas for Pods with the image `nginx`. Expose the container port 80.
2. Expose the Pods so that requests can be made against the service from inside of the cluster.
3. Create a temporary Pods using the image `busybox` and run a `wget` command against the IP of the service.
4. Change the service type so that the Pods can be reached from outside of the cluster.
5. Run a `wget` command against the service from outside of the cluster.
6. (Optional) Discuss: Can you expose the Pods as a service without a deployment?
7. (Optional) Discuss: Under what condition would you use the service type `LoadBalancer`?


<details><summary> Solution 14 (Service) </summary>
<p>

#### SN1.1. Create a deployment named `myapp` that creates 2 replicas for Pods with the image `nginx`. Expose the container port 80.

```bash
# Imperative method to create deployment
$ kubectl create deployment myapp --image=nginx --dry-run -o yaml > myapp.yaml
```

```yaml
# Edit the myapp.yaml 
$ vim myapp.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 2   # edit replicas
  selector:
    matchLabels:
      app: myapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: myapp
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:                # add container port
        - containerPort: 80   # add container port
        resources: {}
status: {}
```
```bash
# create the deployment 
$ kubectl create -f myapp.yaml
deployment.apps/myapp created

$ kubectl get deployment,pod
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   2/2     2            2           73s

NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-6568fd68c9-n89fv   1/1     Running   0          72s
pod/myapp-6568fd68c9-nzr58   1/1     Running   0          73s
```

#### SN1.2. Expose the Pods so that requests can be made against the service from inside of the cluster.
```bash
$ kubectl expose deployment myapp --target-port=80
service/myapp exposed

$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
myapp        ClusterIP   10.97.67.223    <none>        80/TCP    22s

$ kubectl describe services myapp
Name:              myapp
Namespace:         default
Labels:            app=myapp
Annotations:       <none>
Selector:          app=myapp
Type:              ClusterIP        # Type of Service
IP:                10.97.67.223     # IP address to access the service
Port:              <unset>  80/TCP
TargetPort:        80/TCP           # Port to access the service
Endpoints:         172.17.0.2:80,172.17.0.3:80
Session Affinity:  None
Events:            <none>
```
#### SN1.3. Create a temporary Pods using the image `busybox` and run a `wget` command against the IP of the service.
```bash
$ kubectl run temp-pod --image=busybox --restart=Never -it --rm -- /bin/bash 
  # enter the temp-pod container
  $ wget -O- 10.97.67.223:80

  Connecting to 10.97.67.223:80 (10.97.67.223:80)
  writing to stdout
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>

  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>

  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
  - 100% |*******************|   612  0:00:00 ETA
  written to stdout
```

#### SN1.4. Change the service type so that the Pods can be reached from outside of the cluster.
```bash
$ kubectl edit service myapp
```
(before edit)
```yaml
spec:
  clusterIP: 10.97.67.223
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: myapp
  sessionAffinity: None
  type: ClusterIP     # edit this part to NodePort
```
(after edit)
```yaml
spec:
  clusterIP: ~
  ports:
  - port: ~
    protocol: ~
    targetPort: ~
  selector:
    app: ~
  sessionAffinity: ~
  type: NodePort
```

```bash
# Check the service changed to NodePort
$ kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        70d
myapp        NodePort    10.103.17.121   <none>        80:31151/TCP   7m35s
```

#### SN1.5. Run a `wget` command against the service from outside of the cluster.
```bash
# Try to connect the service from our local machine
# For Minikube
$ minikube service myapp
|-----------|-------|-------------|-----------------------------|
| NAMESPACE | NAME  | TARGET PORT |             URL             |
|-----------|-------|-------------|-----------------------------|
| default   | myapp |             | http://192.168.99.112:31907 |
|-----------|-------|-------------|-----------------------------|
🎉  Opening service default/myapp in default browser...

OR, GET THE MINIKUBE IP WITH: 
$ minikube ip
192.168.99.112

# Try with wget or curl
$ wget -O- 192.168.99.112:31907

--2019-12-13 23:27:43--  http://192.168.99.112:31907/
Connecting to 192.168.99.112:31907... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11 [text/html]
Saving to: `STDOUT'
- 0%[   ]   0  --.-KB/s         Hello World
- 100%[=========>]      11  --.-KB/s    in 0s
2019-12-13 23:27:43 (826 KB/s) - written to stdout [11/11]

$ curl 192.168.99.112:31907
Hello World
```
</p>
</details>


### SN2.1. Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

- answer type 1
```bash
$ kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# observe that a pod as well as a service are created
```
- answer type 2
```bash
# create pod with port 80
$ kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx.yaml
$ kubectl -f nginx.yaml 

# expose
$ kubectl expose pod nginx
```
</p>
</details>


### SN2.2. Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
$ kubectl get svc nginx # services
$ kubectl get ep # endpoints
```

</p>
</details>

### SN2.3. Get pod's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
$ kubectl get svc nginx # get the IP (something like 10.108.93.130)
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.110.80.139   <none>        80/TCP    15m

$ kubectl run busybox --rm --image=busybox -it --restart=Never -- sh
  $ wget -O- 10.110.80.139:80
  $ exit
```

</p>
or
<p>

```bash
$ IP=$(kubectl get svc nginx --template={{.spec.clusterIP}}) # get the IP (something like 10.108.93.130)
$ kubectl run busybox --rm --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80
```

</p>
</details>

### SN2.4. Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

<details><summary>show</summary>
<p>

```bash
kubectl edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # change cluster IP to nodeport
status:
  loadBalancer: {}
```

```bash
kubectl get svc
```

```
# result:
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx        NodePort    10.107.253.138   <none>        80:31931/TCP   3m
```

```bash
wget -O- NODE_IP:31931 # if you're using Kubernetes with Docker for Windows/Mac, try 127.0.0.1
# For Minikube, try `minikube ip` to get the node ip such as 192.168.99.117
```

```bash
kubectl delete svc nginx # Deletes the service
kbuectl delete pod nginx # Deletes the pod
```
</p>
</details>

### SN3.1. Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --dry-run -o yaml > foo.yml

vi foo.yml
```

Update the yaml to update the replicas and add container port.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: foo
  name: foo
spec:
  replicas: 3 # Update this
  selector:
    matchLabels:
      app: foo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: foo
    spec:
      containers:
      - image: dgkanatsios/simpleapp
        name: simpleapp
        ports:                   # Add this
          - containerPort: 8080  # Add this
        resources: {}
status: {}
```
</p>
</details>

### SN3.2. Get the pod IPs. Create a temp busybox pod and trying hitting them on port 8080

<details><summary>show</summary>
<p>


```bash
$ kubectl get pods -l app=foo -o wide # 'wide' will show pod IPs
NAME                   READY   STATUS    RESTARTS   AGE   IP        
foo-54d949bdc7-hl46z   1/1     Running   0          29m   172.17.0.8
foo-54d949bdc7-k7rd9   1/1     Running   0          29m   172.17.0.2
foo-54d949bdc7-wtvcr   1/1     Running   0          29m   172.17.0.3
```
or, try below command to retrieve IPs from all pods

```bash
$ kubectl describe `kubectl get pods -l app=foo -o name` | grep IP
IP:                 172.17.0.8
IP:                 172.17.0.2
IP:                 172.17.0.3
```

```bash
$ kubectl run busybox --image=busybox --restart=Never -it --rm -- sh

wget -O- POD_IP:8080 # do not try with pod name, will not work
# try hitting all IPs to confirm that hostname is different
exit
```

</p>
</details>

### SN3.3. Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>

- !! Pay attention on the difference of PORT and TARGETPORT!!
- port: service foo receives request from this port number
- targetport: the connection from service foo is forwarded to containers on this port
```bash
$ kubectl expose deploy foo --port=6262 --target-port=8080
$ kubectl get service foo # you will see ClusterIP as well as port 6262
$ kubectl get endpoints foo # you will see the IPs of the three replica nodes, listening on port 8080
```

</p>
</details>

### SN3.4. Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. 

<details><summary>show</summary>
<p>

```bash
$ kubectl get svc # get the foo service ClusterIP
$ kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
  $ wget -O- foo:6262 # DNS works! run it many times, you'll see different pods responding
  $ wget -O- SERVICE_CLUSTER_IP:6262 # ClusterIP works as well
# you can also kubectl logs on deployment pods to see the container logs
```

</p>
</details>

### SN3.5. Change the Service type from ClusterIP to NodePort. Get the NodePort IP and confirm the connection to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster.

<details><summary>show</summary>
<p>

```bash
$ kubectl edit service foo
# edit the type: ClusterIP to type: NodePort

$ kubectl get svc # get the foo service ClusterIP
NAME   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
foo    NodePort   10.109.67.237   <none>        6262:31017/TCP   3m23s

$ minikube ip # If you are using Minikube, get Node IP from this command
192.168.99.112

$ curl 192.168.99.112:31017 # execute this command many times and you'll get these three different hostname result:
Hello world from foo-54d949bdc7-bm77q and version 2.0
Hello world from foo-54d949bdc7-n8x6x and version 2.0
Hello world from foo-54d949bdc7-ltqnd and version 2.0
```

Delete service and deployment
```bash
$ kubectl delete svc foo
$ kubectl delete deployment foo
```
</p>
</details>


## Network Policy
- youtube.com/watch?v=3gGpMmYeEO8
- github.com/ahmetb/kubernetes-network-policy-recipes


### SN4. Restricting Access to and from a Pod
- In this exercise, you will set up a NetworkPolicy to restrict access to and from a Pod.

Let's assume we are working on an application stack that defines three different layers: a frontend, a backend and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file `app-stack.yaml`. The application needs to run in the namespace `app-stack`.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: frontend
  namespace: app-stack
  labels:
    app: todo
    tier: frontend
spec:
  containers:
    - name: frontend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: backend
  namespace: app-stack
  labels:
    app: todo
    tier: backend
spec:
  containers:
    - name: backend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: database
  namespace: app-stack
  labels:
    app: todo
    tier: database
spec:
  containers:
    - name: database
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: example
```

1. Create the required namespace.
2. Copy the Pod definition to the file `app-stack.yaml` and create all three Pods. Notice that the namespace has already been defined in the YAML definition.
3. Create a network policy in the YAML file `app-stack-network-policy.yaml`.
4. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.
5. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

<details><summary> Show SN4 </summary>
<p>

#### SN4.1. Create the required namespace.
```bash
$ kubectl create namespace app-stack
```

#### SN4.2. Copy the Pod definition to the file `app-stack.yaml` and create all three Pods. Notice that the namespace has already been defined in the YAML definition.
```bash
$ vim app-stack.yaml
# copy paste all three pods in one yaml

$ kubectl create -f app-stack.yaml
pod/frontend created
pod/backend created
pod/database created
```

#### SN4.3. Create a network policy in the YAML file `app-stack-network-policy.yaml`.
#### SN4.4. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.
#### SN4.5. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

```bash
$ vim app-stack-network-policy.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database      # network policy for database pod. don't define the frontend since must disallow incoming from these pods
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: todo
          tier: backend   # allowed only for backend pods
    ports:
    - protocol: TCP   
      port: 3306
```
```bash
$ kubectl create -f app-stack-network-policy.yaml

$ kubectl get networkpolicy -n app-stack
NAME                       POD-SELECTOR             AGE
app-stack-network-policy   app=todo,tier=database   2m1s

$ kubectl describe networkpolicy app-stack-network-policy -n app-stack
Name:         app-stack-network-policy
Namespace:    app-stack
Created on:   2019-12-14 14:32:01 +0900 JST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=todo,tier=database
  Allowing ingress traffic:
    To Port: 3306/TCP
    From:
      PodSelector: app=todo,tier=backend
  Allowing egress traffic:
    <none> (Selected pods are isolated for egress connectivity)
  Policy Types: Ingress, Egress
```


</p>
</details>


### SN5. Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: true' can access the deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

<details><summary>show</summary>
<p>

```bash
$ kubectl create deployment nginx --image=nginx --dry-run -o yaml > deployment-nginx.yaml
$ vim deployment-nginx.yaml
# change the replicas to 2

$ kubectl create -f deployment-nginx.yaml
```

Create the NetworkPolicy
```bash
$ vim access-nginx.yaml
```
```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # pick a name
spec:
  podSelector:
    matchLabels:
      run: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: 'true' # 'true' *needs* quotes in YAML, apparently
```

```bash
# Create the NetworkPolicy
$ kubectl create -f policy.yaml

# Check if the Network Policy has been created correctly
# make sure that your cluster's network provider supports Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
# For minikube should be carefull since you need to configure the NetworkPolicy plugin
$ kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80                       
# This should not work

$ kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=true -- wget -O- http://nginx:80  
# This should be fine
```

</p>
</details>


## Hard level  exercise

### Todays Task: Pod Creation
Create a pod of image bash which runs once to execute the command `hostname > /tmp/hostname && sleep 30`.
Edit its yaml to add a label `my-label: test`.
Run the pod from the yaml file.
Connect via ssh to the pod. Make sure its hostname is written into file `/tmp/hostname`.
Delete the pod.

### Solution

```bash
kubectl run pod1 --restart=Never --image=bash -- bash -c "hostname >> /tmp/hostname && sleep 30"
kubectl get pod pod1 -o yaml --export > pod1.yaml
kubectl delete pod pod1
```
Then edit pod1.yaml to make it look ~like this (I removed unimportant lines):
```yaml
apiVersion: v1
kind: Pod 
metadata:
  labels:
    run: pod1
    my-label: test
  name: pod1
spec:
  containers:
  - args:
    - bash
    - -c
    - hostname >> /tmp/hostname && sleep 30
    image: bash
    name: pod1
  restartPolicy: Never
```

Check out the my-label definition. Now we continue using our updated pod1.yaml:

```bash
kubectl create -f pod1.yaml
kubectl get pod -l my-label
  NAME   READY   STATUS      RESTARTS   AGE
  pod1   0/1     Completed   0          39s
kubectl exec -it pod1 bash
> bash-5.0# cat /tmp/hostname
> pod1
> exit
kubectl delete pod pod1
```

## Task: Namespaces, Deployments and Services
- Create a new namespace k8s-challenge-2-a and assure all following operations (unless different namespace is mentioned) are done in this namespace.
- Create a deployment named nginx-deployment of three pods running image nginx with a memory limit of 64MB.
- Expose this deployment under the name nginx-service inside our cluster on port 4444, so point the service port 4444 to pod ports 80.
- Spin up a temporary pod named pod1 of image cosmintitei/bash-curl, ssh into it and request the default nginx page on port 4444 of our nginx-service using curl.
- Spin up a temporary pod named pod2 of image cosmintitei/bash-curl in namespace k8s-challenge-2-b, ssh into it and request the default nginx page on port 4444 of our nginx-service in namespace k8s-challenge-2-a using curl.

## Solution
### Namespace
```bash
kubectl create namespace k8n-challenge-2-a
kubectl config current-context # show current context
  docker-for-desktop
# now set default namespace for context
kubectl config set-context docker-for-desktop--namespace=k8n-challenge-2-a
# or simply use the --current flag, thanks to Anton!
kubectl config set-context --current --namespace=k8n-challenge-2-a
```

### Deployment
```bash
kubectl run nginx-deployment --image=nginx --replicas 3 -o yaml --dry-run > nginx-deployment.yaml
```

Now edit nginx-deployment.yaml to define our memory restrictions and our containerPort:
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "64Mi"
      restartPolicy: Always
```

```bash
kubectl create -f nginx-deployment.yaml
```

Our watch command should output something like this:

### Expose / Service
```bash
kubectl expose deployment nginx-deployment --name=nginx-service --port=4444 --target-port=80
```

### Pod1
```bash
kubectl run -it pod1 --image=cosmintitei/bash-curl --restart=Never --rm
```

### Pod2
```bash
kubectl create namespace k8n-challenge-2-b
kubectl run -it pod2 --image=cosmintitei/bash-curl --restart=Never --namespace=k8n-challenge-2-b --rm
```

- So here we can reach our nginx-service using curl http://nginx-service.k8n-challenge-2-a:4444 .
- End
- We learned how to use namespaces and how pods from different namespaces can reach each other via DNS. We also looked at spinning up pods with/without deployment to fit different use cases.

## Task: CronJobs and Volumes
- Create a static PersistentVolume of 50MB to your nodes/localhosts `/tmp/k8s-challenge-3` directory.
- Create a PersistentVolumeClaim for this volume for 40MB.
- Create a CronJob which runs two instances every minute of: a pod mounting the PersistentStorageClaim into `/tmp/vol` and executing the command `hostname >> /tmp/vol/storage`.
- We only need to keep the last 4 successful executed jobs in the cronjob history.
- Check your local filesystem for the hostnames of these pods with `tail -f /tmp/k8s-challenge-3/storage`.

## Solution
### Create Volume:
- Create pv.yaml to create the volume, I copy pasted from the docs:
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    id: vol1
spec:
  storageClassName: manual
  capacity:
    storage: 50Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/k8s-challenge-3"
```

```bash
kubectl create -f pv.yaml
```

### Create Volume Claim:
- Create `pvc.yaml`, I copy pasted from the docs:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi
  selector:
    matchLabels:
      id: vol1
```

```bash
kubectl create -f pvc.yaml
```

- Make sure you give your volume a label and select this in your claim, else you may run into unwanted side effects.
- Make sure your PV and PVC use the same storageClassName, else they won’t bind. As far as I understand, creating a PVC with no StorageClass defined uses the default one (kubectl get storageclasses). PVs have an empty StorageClass by default. More about this here and here.

- Now our cluster monitoring command or kubectl get pv,pvc should show something like this:
- Also, if you get an error that the manual storage class doesn’t exist you can try using storageClassName: “”.

### Create CronJob1:
```bash
kuberctl run --dry-run -o yaml cronjob1 --schedule="*/1 * * * *" --restart=Never --image=bash -- bash -c "hostname >> /tmp/vol/storage" > cronjob1.yaml
```

- Adjust the cronjob1.yaml with our requirements:

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  labels:
    run: cronjob1
  name: cronjob1
spec:
  concurrencyPolicy: Allow
  jobTemplate:
    metadata:
      creationTimestamp: null
    spec:
      parallelism: 2
      template:
        metadata:
          creationTimestamp: null
          labels:
            run: cronjob1
        spec:
          volumes:
            - name: cron-vol
              persistentVolumeClaim:
                claimName: task-pv-claim
          containers:
          - args:
            - bash
            - -c
            - hostname >> /tmp/vol/storage
            image: bash
            name: cronjob1
            resources: {}
            volumeMounts:
              - name: cron-vol
                mountPath: /tmp/vol
          restartPolicy: Never
  schedule: '*/1 * * * *'
  successfulJobsHistoryLimit: 4
```

- Check the mounted volumes, the parallelism and successfulJobsHistoryLimit.

```bash
kubectl create -f cronjob1.yaml
# wait for first run after one minute
tail -f /tmp/k8s-challenge-3/storage
cronjob1-1553541240-jpt6t
cronjob1-1553541240-2skbd
cronjob1-1553541300-s7g6k
cronjob1-1553541300-qpbbp
```

- We see 4 jobs in our history and 8 pods (2 per job) using kubectl get job,pod:

## Task: Deployment, Rollouts and Rollbacks
1. Create a deployment of 15 pods with image nginx:1.14.2 in namespace one.
2. Confirm that all pods are running that image.
3. Edit the deployment to change the image of all pods to nginx:1.15.10.
4. Confirm that all pods are running image nginx:1.15.10.
5. Edit the deployment to change image of all pods to nginx:1.15.666.
6. Confirm that all pod are running image nginx:1.15.666 and have no errors. Show error if there is one.
7. Woops! Something went crazy wrong here! Rollback the change, so all pods should run nginx:1.15.10 again.
8. Confirm that all pods are running image nginx:1.15.10.

## Solution:
1. 
```bash
kubectl create namespace one
kubectl config set-context $(kubectl config current-context) --namespace=one
kubectl create deployment nginx1 --image=nginx:1.14.2 --replicas=15
```
2. 
```bash
kubectl get pods -o yaml | grep 1.14.2
kubectl get pods -o yaml | grep 1.14.2 | wc -l
# should be 30 as the image is also listed in the status output
```

3. 
```bash
kubectl edit deployments nginx1
# then manually edit the container image to nginx:1.15.10
# OR
kubectl set image deployment/nginx1 nginx=nginx:1.15.10
```

4. 
```bash
kubectl get pods -o yaml | grep 1.15.10 | wc -l
```

5. 
```bash
kubectl edit deployments nginx1
# then manually edit the container image to nginx:1.15.666
# OR
kubectl set image deployment/nginx1 nginx=nginx:1.15.666
```

6. 
```bash
kubectl get pods -o yaml | grep 1.15.666 | wc -l
# something wrong
kubectl get pods
```

```bash
kubectl log nginx1-64cfc7f765-24kwb
```

- Oups! Let’s roll back.

7. 
```bash
kubectl rollout history deployment nginx1 # show history
kubectl rollout undo deployment nginx1
```

8. 
```bash
kubectl get pods -o yaml | grep 1.14.2 | wc -l
```

## Task: Secrets and ConfigMaps
1. Create a secret1.yaml. The yaml should create a secret called secret1 and store password:12345678. Create that secret.
2. Create a pod1.yaml which creates a single pod of image bash . This pod should mount the secret1 to /tmp/secret1. This pod should stay idle after boot. Create that pod.
3. Confirm pod1 has access to our password via file system.
4. On your local create a folder drinks and its content: mkdir drinks; echo ipa > drinks/beer; echo red > drinks/wine; echo sparkling > drinks/water
5. Create a ConfigMap containing all files of folder drinks and their content.
6. Make these ConfigMaps available in our pod1 using environment variables.
7. Check on pod1 if those environment variables are available.

## Solution
1. 
```bash
kubectl create secret generic secret1 --from-literal=password=12345678 --dry-run -o yaml > secret1.yml
kubectl -f secret1.yml create
```

2. 
```bash
kubectl run pod1 --image=bash --dry-run -o yaml --restart=Never sleep 9999 > pod1.yaml
```
Edit the `pod1.yaml` to something like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  volumes:
    - name: sec-vol
      secret:
        secretName: secret1
  containers:
  - args:
    - sleep
    - "9999"
    image: bash
    name: pod1
    resources: {}
    volumeMounts:
      - name: sec-vol
        mountPath: /tmp/secret1
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
kubectl -f pod1.yaml create
```

3. 
```bash
kubectl exec pod1 cat /tmp/secret1/password
```
4. 
```bash
kubectl create configmap drink1 --from-file ./drinks
```
- Edit the pod1.yaml to something like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  volumes:
    - name: sec-vol
      secret:
        secretName: secret1
  containers:
  - args:
    - sleep
    - "9999"
    image: bash
    name: pod1
    resources: {}
    volumeMounts:
      - name: sec-vol
        mountPath: /tmp/secret1
    envFrom:
      - configMapRef:
          name: drink1
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```
5. 
```bash
kubectl exec pod1 env
beer=ipa
water=sparkling
wine=red
```

## Task: Simulate migration phase of external service
- Let’s say we have a running cluster but one service is still outside. We pretend www.google.com:80 is our external service which we would like to migrate into the cluster in a few phases. Between each phase there could be days or weeks passing, but we simulate this in one go today.
1. Create a deployment called compute of three replicas of image byrnedo/alpine-curl which keep running for a while. Make sure they can reach our external service: curl www.google.com:80
2. We will move this external service soon inside the cluster. But till then we create an ExternalName service called webapi which points to www.google.com. Make sure our pods can reach that internal service and get forwarded to our external one. curl webapi --header “Host: www.google.com"
3. Create a deployment called nginx of image nginx. We pretend this is a replica of our external www.google.com:80
4. Change the internal webapi service to no longer point to the external DNS but our internal nginx pod. Test the connection from compute pods to webapi service: curl webapi

## Solution
1. Cluster connected to external service
```bash
kubectl run --help
kubectl run compute --image=byrnedo/alpine-curl --replicas=3 --command -- /bin/sh -c "sleep 10d"
kubectl exec compute-66f769c5d9-6chc9 -- curl www.google.com:80
```

- We now have a running cluster which can reach our external web service.

2. Migration phase with ExternalName service
```bash
kubectl create service --help
kubectl create service externalname --help
kubectl create service externalname webapi --external-name www.google.com
kubectl describe service webapi
kubectl exec compute-fd98796c6-27w65 -- ping webapi
kubectl exec compute-fd98796c6-27w65 -- curl webapi --header "Host: www.google.com" # we can reach home of www.google.com
```

Instead of using the external DNS name we can now use our internal service name webapi. This means we can redirect all internal requests already to this dns name although it’s still outside the cluster.

3. Migrate the external service inside the cluster
```bash
kubectl run nginx --image nginx # might be more complex in prod :D
```

We imagine the internal nginx deployment is a running replica of our external service www.google.com:80, both are now running simultaneously in this transitioning phase.

4. Point webapi service to internal nginx
- Now it’s time to no longer send requests to www.google.com:80 but our internal nginx pod.
```bash
kubectl get service webapi -o yaml --export > webapi.yaml
```

- Now edit webapi.yaml to something like this (we change the service type from ExternalName to ClusterIP):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapi
spec:
  selector:
    run: nginx
  ports:
    - name: "80"
      port: 80
      targetPort: 80
  type: ClusterIP
```

- Make sure the selector refers to the correct labels of your nginx pod. Then apply the changes and check connectivity:

```bash
kubectl delete -f webapi.yaml
kubectl create -f webapi.yaml
kubectl exec compute-fd98796c6-27w65 -- ping webapi
kubectl exec compute-fd98796c6-27w65 -- curl webapi
```

- The curl webapi did still point to www.google.com, probably because of some DNS caching on the pod I thought. So I tried recreating all pods (kubectl delete pod --all) but this didn’t change. The webapi name was still resolved even if I removed the webapi service completely! So it must be Kubernetes dns caching.

- To flush dns resolution I identified the kube-dns pod and forced its recreation:
```bash
kubectl get pod -n kube-system | grep dns
kubectl delete pod kube-dns-86f4d74b45-z8lns
kubectl exec compute-fd98796c6-27w65 -- curl webapi # this now returns the default nginx page instead of www.google.com !
```

- Nice, we finished the migration. As with all good migrations we had some little hickups but were able to solve this by “clearing” the kube dns cache (aka killing the kube-dns pod). Let me know if there is a cleaner solution for this!
- Now we can shut down our outside service www.google.com:80 ;)

### End
- The ExternalName service is really useful when migrating an existing infrastructure into Kubernetes. You can already create your whole service structure and use internal name resolution. Then, step by step, you can change the services to point to inside objects just by changing the service type.


## Scenario Setup
```bash
kubectl create -f https://raw.githubusercontent.com/wuestkamp/k8s-challenges/master/9/scenario.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx
    spec:
      volumes:
      - name: logs
        emptyDir: {}
      containers:
      - image: nginx
        name: nginx
        resources: {}
        volumeMounts:
          - name: logs
            mountPath: /var/log/nginx
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  ports:
  - port: 1234
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: LoadBalancer
status:
  loadBalancer: {}
```

- We have a deployment of one nginx instance and a LoadBalancer service to expose this. So you should be able to access nginx on your external ip:

```bash
curl localhost:1234
```

- If you can’t access the nginx via localhost just create a temporary pod to use curl to connect.
- The nginx pod has an emptyDir volume setup which is mounted at /var/log/nginx. You should be able to see access logs with:

```bash
kubectl exec nginx-54d8ff86dc-tthzg -- tail -f /var/log/nginx/access.log
```

### Task: Create a sidecar container for logging
1. Add a sidecar container of image bash to the nginx pod
2. Mount the pod scoped volume named logs into the sidecar, so same as nginx container does.
3. Our sidecar should pipe the content of file access.log (that’s inside the volume logs because nginx container writes it there) to stdout
4. Check if you can access the logs of your sidecar using kubectl logs.

### 1,2,3 Create a sidecar container in the nginx pod, mount log volume and pipe those to output
- It’s not (yet) possible to create a new container in a running pod using kubectl edit, so we dump and make yaml changes:
```bash
kubectl get deployment nginx -o yaml --export > d_nginx.yaml
```

- Now we edit the d_nginx.yaml and add a new container:
```yaml

...
spec:
  containers:
  - image: bash
    name: sidecar
    volumeMounts:
    - mountPath: /tmp/logs
      name: logs
    command:
      - "/bin/sh"
      - "-c"
      - "tail -f /tmp/logs/access.log"
  - image: nginx
    imagePullPolicy: Always
    name: nginx
...

```

- Then we apply:

```bash
kubectl delete deployment nginx
kubectl create -f d_nginx.yaml
```

4. Check if you can access the logs of your sidecar using

```bash
curl localhost:1234
kubectl logs nginx-5c989bbd58-wzc6b sidecar
```

### Recap
- Sidecars can be helpful to collect logging information or other data and provide it using a standardized method across your cluster for other services to collect, this could be called the Adapter Pattern.
- They can also be useful for debugging purposes like investigating network issues. It would be awesome to be able to add a new container which has all the debugging tools needed into a running pod. It seems kubectl-debug let’s you do this though.

## Task: Hacking a Deployment
- Create a deployment of image nginx with 3 replicas and check the kubectl get events that this caused.
- Manually create a single pod of image nginx and try to smuggle it into the custody of the existing deployment (without changing the deployment configuration).
- Did it work? Are there now 4 pods? Check kubectl get events for what happened.

## Solution
1. 
```bash
kubectl run nginx --image nginx --replicas 3
kubectl get events | tail -n 10
```

2. 
- Pods belong to ReplicaSets based on labels, so we need to find that label. We can do this in different ways:
```bash
kubectl describe pod nginx-65899c769f-67jjw | grep -i label
kubectl describe replicaset nginx-65899c769f| grep -i label
```

- Next we either set the labels directly during pod creation:
```bash
kubectl run nginx --image=nginx --restart=Never --labels="run=nginx,pod-template-hash=2145573259"
kubectl get pods # not pod there
```

- or we add the label afterwards:
```bash
kubectl run nginx --image=nginx --restart=Never
kubectl get pods # there it is!
kubectl label pods nginx pod-template-hash=2145573259
kubectl get pods # and it's gone
```
- When we add the label afterwards we don’t need to add the run=nginx because it was added automatically by kubectl run.

3. 
```bash
kubectl get events | tail -n 10
```
- We can see that the awesome ReplicaSet-Controller jumped in, noticed that there were more replicas available and deleted the new one. No way to sneak our pod into this deployment! :D

### Recap
- Should we call that hacking? No, but sounds good for sure ;) This should just show a little how a replication controller works.








## github code template: 
```bash
code for bash
```

```yaml
code for yaml
```

# Main Topic
## Sub Topic
### Small topic
- detail point 1
- detail point 2

## Exercise 9
text...

### Detail of exercise 
text...

<details><summary> Solution number </summary>
<p>

```bash

```
</p>
</details>
