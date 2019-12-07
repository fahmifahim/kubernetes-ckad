★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

# CKAD CRASH COURSE

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

--Documentation: 
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://matthewpalmer.net/kubernetes-app-developer/$purchase-the-ebook
https://github.com/dgkanatsios/CKAD-exercises

--Video Tutorial (Oreilly):
Benjamin Muschko
https://learning.oreilly.com/live-training/courses/certified-kubernetes-application-developer-crash-course-ckad/0636920329176/

Sander Van Vught
https://learning.oreilly.com/live-training/courses/certified-kubernetes-application-developer-ckad-crash-course/0636920318439/

--Time Management: 
19 Problems in 2 hours. Use your time wisely!
There might be weight of point for specific questions. 

--Using Alias for kubectl 
$ alias k=kubectl 
$ k version 
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}

--Setting Context & Namespace 
$ kubectl config set-context <context-of-question> --namespace=<namespace-of-question>

--Deleting Kubernetes Objects 
Deleting object may take time, to save your time during the exam, don't wait for a graceful deletion of objects, just kill it!
$ kubectl delete pod <pod-name> --grace-period=0 --force
useful flag: --grace-period=0 --force

--Understand and Practice bash 
$ if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true; do echo $(date) >> ~/tmp/date.txt; sleep 5; done; 
$ while true; do kubectl get pods | grep docker-regist; sleep 3; done; 

--Object Management
-Imperative method : kubernetes
> fast but requires detailed knowledge, no track record  
$ kubectl create namespace ckad
$ kubectl run nginx --image=nginx --restart=Never -n ckad 
-Declarative method: yaml
> Suitable for more elaborate changes, tracks changes 
-Hybrid approach
>Generate YAML file with kubectl
$ kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml 
$ vim nginx-pod.yaml 
$ kubectl apply -f nginx-pod.yaml 

--Pod life cycle
-Pending
-Running
-Succeeded
-Failed 
-Unknown 

--Inspecting a Pod's status: 
-Get current status and event logs: 
$ kubectl describe pods <pod-name> | grep Status:

-Get current lifecycle phase: 
$ kubectl get pods <pod-name> -o yaml | grep phase 

--Configuring Env. Variables
Injecting runtime behaviour
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

--Commands and Arguments
Running a command inside of container
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

--Other Useful kubectl commands 
$ kubectl logs <pod-name> 
$ kubectl exec -it <pod-name> -- /bin/sh 


# Exercise 1
In this exercise, you will practice the creation of a new Pod in a namespace. Once created, you will inspect it, shell into it and run some operations inside of the container.
## Creating a Pod and Inspecting it
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

# Solution 1 
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


--Centralized Configuration Data 
-Creating ConfigMap (Imperative)
(Literal values)
$ kubectl create configmap db-config ¥
  --from-literal=db=staging 
  --from-literal=username=jdoe

(Single file with environment variables)
$ kubectl create configmap db-config --from-env-file=config.env 

(File or directory)
$ kubectl create configmap db-config ¥
  --from-file=config.txt
  --from-file=config-data.txt

-Creating ConfigMap (Declarative)
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: db-config 
data: 
  db: staging 
  username: jdoe 


# Exercise 2
In this exercise, you will first create a ConfigMap from predefined values in a file. Later, you'll create a Pod, consume the ConfigMap as environment variables and print out its values from within the container.

## Configuring a Pod to Use a ConfigMap
1. Create a new file named `config.txt` with the following environment variables as key/value pairs on each line.
- `DB_URL` equates to `localhost:3306`
- `DB_USERNAME` equates to `postgres`
2. Create a new ConfigMap named `db-config` from that file.
3. Create a Pod named `backend` that uses the environment variables from the ConfigMap and runs the container with the image `nginx`.
4. Shell into the Pod and print out the created environment variables. You should find `DB_URL` and `DB_USERNAME` with their appropriate values.
5. (Optional) Discuss: How would you approach hot reloading of values defined by a ConfigMap consumed by an application running in Pod?01234567:02-creating-using-configmap fahmi$

# Solution 2
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


--Creating Secrets(Imperative)
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

--Using Secret as Files from a Pod
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

--Using Secret as Environment Variables 
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
    


# Exercise 3
In this exercise, you will first create a Secret from literal values. Next, you'll create a Pod and consume the Secret as environment variables. Finally, you'll print out its values from within the container.

## Configuring a Pod to Use a Secret
1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.
2. Create a Pod named `backend` that defines uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx`.
3. Shell into the Pod and print out the created environment variables. You should find `DB_PASSWORD` variable.
4. (Optional) Discuss: What is one of the benefit of using a Secret over a ConfigMap?

# Solution 3 
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


--Security Context 
-Set Security Context for a Pod 
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

-runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000.
-runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod.
-fsGroup field specified all processes of the container are also part of the supplementary group ID 2000. The owner for volume /data/demo and any files created in that volume will be Group ID 2000.


# Exercise 4
In this exercise, you will create a Pod that defines a filesystem group ID as security context. Based on this security context, you'll create a new file and inspect the outcome of the file creation based on the rule defined earlier.

## Creating a Security Context for a Pod
1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.
2. Files created on the volume should use the filesystem group ID 3000.
3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

# Solution 4
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


--Resource Boundaries


# Exercise 5
In this exercise, you will create a ResourceQuota with specific CPU and memory limits for a new namespace. Pods created in the namespace will have to adhere to those limits.

## Defining a Pod’s Resource Requirements
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

# Solution 5 
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


--Service Account
When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).
When you create a pod, if you do not specify a service account, it is automatically assigned the default service account in the same namespace.

# Exercise 6
In this exercise, you will create a ServiceAccount and assign it to a Pod.
$$ Using a ServiceAccount
1. Create a new service account named `backend-team`.
2. Print out the token for the service account in YAML format.
3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.
4. Get a shell to the running container and print out the token of the service account.

# Solution 6 
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


--Multi-container Pod 
https://blog.nillsf.com/index.php/2019/07/28/ckad-series-part-4-multi-container-pods/
1. Sidecar container 
adds functionality to your application that could be included in the main container. By hosting this logic in a sidecar, you can keep that functionality out of your main application and evolve that independently from the actual application.
2. Ambassador container 
proxies a local connection to certain outbound connection. The ambassadors brokers the connection the outside world. This can for instance be used to shard a service or to implement client side load balancing.
3. Adapter container 
takes data from the existing application and presents that in a standardized way. This is for instance very useful for monitoring data.
4. Init container
An init-container is a special container that runs before the other containers in your pod.


# Exercise 7 (InitContainer)
In this exercise, you will initialize a web application by standing up environment-specific configuration through an init container.
## Creating an Init Container
Kubernetes runs an init container before the main container. In this scenario, the init container retrieves configuration files from a remote location and makes it available to the application running in the main container. The configuration files are shared through a volume mounted by both containers. The running application consumes the configuration files and can render its values.
1. Create a new Pod in a YAML file named `business-app.yaml`. The Pod should define two containers, one init container and one main application container. Name the init container `configurer` and the main container `web`. The init container uses the image `busybox`, the main container uses the image `bmuschko/nodejs-read-config:1.0.0`. Expose the main container on port 8080.
2. Edit the YAML file by adding a new volume of type `emptyDir` that is mounted at `/usr/shared/app` for both containers.
3. Edit the YAML file by providing the command for the init container. The init container should run a `wget` command for downloading the file `https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-creating-init-container/app/config/config.json` into the directory `/usr/shared/app`.
4. Start the Pod and ensure that it is up and running.
5. Run the command `curl localhost:8080` from the main application container. The response should render a database URL derived off the information in the configuration file.
6. (Optional) Discuss: How would you approach a debugging a failing command inside of the init container?

# Solution 7 (InitContainer)
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


# Exercise 8
In this exercise, you will implement the adapter pattern for a multi-container Pod.

## Implementing the Adapter Pattern
The adapter pattern helps with providing a simplified, homogenized view of an application running within a container. For example, we could stand up another container that unifies the log output of the application container. As a result, other monitoring tools can rely on a standardized view of the log output without having to transform it into an expected format.

1. Create a new Pod in a YAML file named `adapter.yaml`. The Pod declares two containers. The container `app` uses the image `busybox` and runs the command `while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;`. The adapter container `transformer` uses the image `busybox` and runs the command `sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;` to strip the log output off the date for later consumption by a monitoring tool. Be aware that the logic does not handle corner cases (e.g. automatically deleting old entries) and would look different in production systems.
2. Before creating the Pod, define an `emptyDir` volume. Mount the volume in both containers with the path `/var/logs`.
3. Create the Pod, log into the container `transformer`. The current directory should continuously write a new file every 20 seconds.