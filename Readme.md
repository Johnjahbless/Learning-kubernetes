# Adding labels to the application

## Chapter Goal
1. Adding labels during build time
2. Viewing labels
2. Adding labels to running pods
3. Deleting a label
4. Searching by labels
5. Extending the label concept to deployments/services

### Adding labels during build time

You can add labels to pods, services and deployments either at build time or at run time. If you're adding labels at build time, you can add a label section in the metadata portion of the YAML as shown below:

```
```

You can deploy the code above by using the command `kubectl create -f helloworld-pod-with-labels.yml`


### Viewing labels

You have a pod with labels. Super! But how do you see them? You can add the `--show-labels` option to your kubectl get command as shown here: `kubectl get pods --show-labels`.

```
```

### Adding labels to running pods
To add labels to a running pod, you can use the `kubectl label` command as follows: `kubectl label po/helloworld app=helloworld`. This adds the label `app` with the value `helloworld` to the pod.

To update the value of a label, use the `--overwrite` flag in the command as follows: `kubectl label po/helloworld app=helloworldapp --overwrite` 

### Deleting a label
To remove an existing label, just add a `-` to the end of the label key as follows: `kubectl label po/helloworld app-`. This will remove the app label from the helloworld pod.

```
```

### Searching by labels
Creating, getting and deleting labels is nice, but the ability to search using labels helps us identify what's going on in our infrastructure better. Let's take a look. First, we're going to deploy a few pods that will constitute what a small org might have. 

`kubectl create -f kubectl create -f sample-infrastructure-with-labels.yml`

```
```

Looking at these applications running from a high level makes it hard to see what's going on with the infrastructure.

`kubectl get pods`

`kubectl get pods --show-labels`

```
```

You can search for labels with the flag `--selector` (or `-l`). If you want to search for all the pods that are running in production, you can run `kubectl get pods --selector env=production` as shown below:

`kubectl get pods --selector env=production`


```
```

Similarly, to get all pods by dev lead Karthik, you'd add `dev-lead=karthik` to the selector as shown below.

`kubectl get pods --selector dev-lead=karthik`

```
```

You can also do more complicated searches, like finding any pods owned by Karthik in the development tier, by the following query `dev-lead=karthik,env=staging`:

`kubectl get pods -l dev-lead=karthik,env=staging`

```
```

Or, any apps not owned by Karthik in staging (using the ! construct):

`kubectl get pods -l dev-lead!=karthik,env=staging`

```
```

Querying also supports the `in` keyword

`kubectl get pods -l 'release-version in (1.0,2.0)'`

```
```

Or a more complicated example:

`kubectl get pods -l "release-version in (1.0,2.0),team in (marketing,ecommerce)"`

```
```

The opposite of "in" is "notin". Surprise. "Notin" is supported as well, as shown in this example:

`kubectl get pods -l 'release-version notin (1.0,2.0)'`

```
```

Finally, sometimes your label might not have a value assigned to it, but you can still search by label name.

`kubectl get pods -l 'release-version'`  

```
```

### Extending the label concept to deployments/services

As a bonus, labels will also work with `kubectl get services/deployments/all --show-labels` and will return labels for your services, deployments or all objects.

`kubectl get all --show-labels`

And, you can delete pods, services or deployments by label as well! For example `kubectl delete pods -l dev-lead=karthik` will delete all pods who's dev-lead was Karthik. 


To summarize, labels in Kubernetes is a powerful concept! Use the labeling feature to your advantage to build your infrastructure in an organized fashion.


# Application health checks

## Chapter Goal
1. Add HTTP health checks to the helloworld deployment
2. Simulate a failing deployment that fails a readiness probe
3. Simulate a failing deployment that fails a liveness probe

### Add HTTP health check to the helloworld deployment

We will take our existing hellworld deployment, and add a readiness and liveness probe healthchecks.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
```

A readiness probe is used to know when a container is ready to start accepting traffic.

The yaml has the following parameters:
```
readinessProbe:
  # length of time to wait for a pod to initialize
  # after pod startup, before applying health checking
  initialDelaySeconds: 10
  # Amount of time to wait before timing out
  initialDelaySeconds: 1
  # Probe for http
  httpGet:
    # Path to probe
    path: /
    # Port to probe
    port: 80
```

A liveness probe is used to know when a container might need to be restarted.

A liveness probe yaml has the following parameters that need to be filled out:

```
livenessProbe:
  # length of time to wait for a pod to initialize
  # after pod startup, before applying health checking
  initialDelaySeconds: 10
  # Amount of time to wait before timing out
  timeoutSeconds: 1
  # Probe for http
  httpGet:
    # Path to probe
    path: /
    # Port to probe
    port: 80
```

Thus, our deployment yaml now becomes:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment-with-probe
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
        readinessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # Amount of time to wait before timing out
          initialDelaySeconds: 1
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 80
        livenessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # Amount of time to wait before timing out
          timeoutSeconds: 1
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 80

```

We run this yaml the same as we had done before: `kubectl create -f helloworld-deployment-with-probes`


### Simulate a failing deployment that fails a readiness probe
We will now try to simulate a bad helloworld pod that fails a readiness probe. Instead of checking port 80 like the last example, we will run a readiness check on port 90 to simulate a failing scenario. Thus, our yaml now is:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment-with-bad-readiness-probe
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
        readinessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # Amount of time to wait before timing out
          initialDelaySeconds: 1
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 90
```

We will run this yaml with the command `kubectl create -f helloworld-with-bad-readiness-probe.yaml`. After about a minute, we will notice that our pod is still not in a read state when we run the `kubectl get pods` command.

```
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm   0/1       Running   0          1m
```

Describing the pod will show that the pod failed readiness:
```
MacbookHome:04_02 Application health checks karthik$ kubectl describe po helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm
Name:   helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm
Namespace:  default
Node:   minikube/192.168.99.101
Start Time: Sun, 29 Oct 2017 01:35:08 -0500
Labels:   app=helloworld
    pod-template-hash=4220863004
Annotations:  kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"helloworld-deployment-with-bad-readiness-probe-8664db7448","uid"...
Status:   Running
IP:   172.17.0.4
Controllers:  ReplicaSet/helloworld-deployment-with-bad-readiness-probe-8664db7448
Containers:
  helloworld:
    Container ID: docker://61fa8b27fe2e8239d5dc84fc097af17c9d4decc90b3bea09103cc9d73302f235
    Image:    karthequian/helloworld:latest
    Image ID:   docker-pullable://karthequian/helloworld@sha256:165f87263f1775f0bf91022b48a51265357ba9f36bc3882f5ecdefc7f8ef8f6d
    Port:   80/TCP
    State:    Running
      Started:    Sun, 29 Oct 2017 01:35:10 -0500
    Ready:    False
    Restart Count:  0
    Readiness:    http-get http://:90/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-x8qf6 (ro)
Conditions:
  Type    Status
  Initialized   True
  Ready   False
  PodScheduled  True
Volumes:
  default-token-x8qf6:
    Type: Secret (a volume populated by a Secret)
    SecretName: default-token-x8qf6
    Optional: false
QoS Class:  BestEffort
Node-Selectors: <none>
Tolerations:  <none>
Events:
  FirstSeen LastSeen  Count From      SubObjectPath     Type    Reason      Message
  --------- --------  ----- ----      -------------     --------  ------      -------
  5m    5m    1 default-scheduler         Normal    Scheduled   Successfully assigned helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm to minikube
  5m    5m    1 kubelet, minikube         Normal    SuccessfulMountVolume MountVolume.SetUp succeeded for volume "default-token-x8qf6"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Pulling     pulling image "karthequian/helloworld:latest"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Pulled      Successfully pulled image "karthequian/helloworld:latest"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Created     Created container
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Started     Started container
  4m    1m    20  kubelet, minikube spec.containers{helloworld} Warning   Unhealthy   Readiness probe failed: Get http://172.17.0.4:90/: dial tcp 172.17.0.4:90: getsockopt: connection refused
MacbookHome:04_02 Application health checks karthik$
```

### Simulate a failing deployment that fails a liveness probe

Next, we will simulate a bad helloworld pod that fails a liveness probe. Instead of checking port 80 like the last example, we will run a liveness check on port 90 to simulate a failing scenario. Thus, our yaml now is:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment-with-bad-liveness-probe
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
        livenessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # How often (in seconds) to perform the probe.
          periodSeconds: 5
          # Amount of time to wait before timing out
          timeoutSeconds: 1
          # Kubernetes will try failureThreshold times before giving up and restarting the Pod
          failureThreshold: 2
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 90
```


We will run this yaml with the command `kubectl create -f helloworld-with-bad-liveness-probe.yaml`. After about a minute, we will notice that our pod is still not in a read state when we run the `kubectl get pods` command.

```
MacbookHome:04_02 Application health checks karthik$ kubectl create -f helloworld-with-bad-liveness-probe.yaml
deployment "helloworld-deployment-with-bad-liveness-probe" created
MacbookHome:04_02 Application health checks karthik$ kubectl get deployments
NAME                                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment-with-bad-liveness-probe   1         1         1            1           5s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS        RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4    1/1       Running       0          9s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS        RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4    1/1       Running       1          26s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   1          32s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   2          38s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   3          50s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS             RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   0/1       CrashLoopBackOff   3          1m
```

When we describe our pod, we notice that it is failing the liveness probe:

```
MacbookHome:04_02 Application health checks karthik$ kubectl describe po helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4
Name:   helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4
Namespace:  default
Node:   minikube/192.168.99.101
Start Time: Sun, 29 Oct 2017 01:47:59 -0500
Labels:   app=helloworld
    pod-template-hash=3331596669
Annotations:  kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"helloworld-deployment-with-bad-liveness-probe-77759fbbbf","uid":...
Status:   Running
IP:   172.17.0.4
Controllers:  ReplicaSet/helloworld-deployment-with-bad-liveness-probe-77759fbbbf
Containers:
  helloworld:
    Container ID: docker://41cfe81a3327b03831440e861eb2164b8a9f601ce630f024c76412d903781922
    Image:    karthequian/helloworld:latest
    Image ID:   docker-pullable://karthequian/helloworld@sha256:165f87263f1775f0bf91022b48a51265357ba9f36bc3882f5ecdefc7f8ef8f6d
    Port:   80/TCP
    State:    Running
      Started:    Sun, 29 Oct 2017 01:48:47 -0500
    Last State:   Terminated
      Reason:   Completed
      Exit Code:  0
      Started:    Sun, 29 Oct 2017 01:48:31 -0500
      Finished:   Sun, 29 Oct 2017 01:48:46 -0500
    Ready:    True
    Restart Count:  3
    Liveness:   http-get http://:90/ delay=10s timeout=1s period=5s #success=1 #failure=2
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-x8qf6 (ro)
Conditions:
  Type    Status
  Initialized   True
  Ready   True
  PodScheduled  True
Volumes:
  default-token-x8qf6:
    Type: Secret (a volume populated by a Secret)
    SecretName: default-token-x8qf6
    Optional: false
QoS Class:  BestEffort
Node-Selectors: <none>
Tolerations:  <none>
Events:
  FirstSeen LastSeen  Count From      SubObjectPath     Type    Reason      Message
  --------- --------  ----- ----      -------------     --------  ------      -------
  58s   58s   1 default-scheduler         Normal    Scheduled   Successfully assigned helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4 to minikube
  58s   58s   1 kubelet, minikube         Normal    SuccessfulMountVolume MountVolume.SetUp succeeded for volume "default-token-x8qf6"
  47s   12s   4 kubelet, minikube spec.containers{helloworld} Warning   Unhealthy   Liveness probe failed: Get http://172.17.0.4:90/: dial tcp 172.17.0.4:90: getsockopt: connection refused
  58s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Pulling     pulling image "karthequian/helloworld:latest"
  57s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Pulled      Successfully pulled image "karthequian/helloworld:latest"
  57s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Created     Created container
  41s   11s   3 kubelet, minikube spec.containers{helloworld} Normal    Killing     Killing container with id docker://helloworld:Container failed liveness probe.. Container will be killed and recreated.
  57s   10s   4 kubelet, minikube spec.containers{helloworld} Normal    Started     Started container
MacbookHome:04_02 Application health checks karthik$
```

And finally, when we check the deployment, we notice that deployment is not available as well.
```
MacbookHome:04_02 Application health checks karthik$ kubectl get deployments
NAME                                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment-with-bad-liveness-probe   1         1         1            0           3m
```

To summarize, we've learned that we can use readiness and liveness probes to check the status of our pods and use them to restart pods when necessary and check pod health.


# Handling application upgrades

## Chapter Goals
1. Upgrade a deployment from 1 version to another
2. Rollback the deployment to the 1st version

### Upgrade a deployment from 1 version to another

Let's deploy our initial version of our application `kubectl create -f helloworld-black.yaml --record`. After the deployment and service has completed, let's expose this via a nodeport by `minikube service navbar-service`. This will bring up the helloworld UI that we have seen before. We added the `--record` to this because we want to record our rollout history that I'll talk about later on.

Looking at the deployment, we see that there are 3 desired, current and ready replicas.

As developers, we're required to make changes to our applications and get these deployed. The rollout functionality of Kubernetes assists with upgrades because it allows us to upgrade the code without any downtime. In our application, I'd like to update the nav bar to a blue color rather than what it is right now. I'll package this in a container with a new label called "blue".

To update the image, I run this command: `kubectl set image deployment/navbar-deployment helloworld=karthequian/helloworld:blue`. This sets the image from what it is currently (karthequian/helloworld:black) to karthequian/helloworld:blue.

The deployment will update, and if you look at the webpage, you'll see the updated text.

Let's take a look at what happened here. When the deployment was edited, a new result set was created for it. Running the `kubectl get rs` command shows us this. One result set with 3 desired, current and ready pods, and another with 0.

We can also take a look at the rollout history by typing `kubectl rollout history deployment/navbar-deployment`.

### Rollback the deployment to the 1st version
To rollback the deployment, we use the rollout undo command `kubectl rollout undo deployment/navbar-deployment`. This will revert our changes back to the previous version.

Our webpage will be back to the Lionel version of the deployment.

In a real world setting, you might have a longer history, and might want to rollback to a specific version. To do this, add a `--to-revision=version` to the specific version you want to.


# Running a more real world example

In this chapter, we'll take the popular Kubernetes guestbook, and attempt to run it! You can read more about the guestbook here: [https://kubernetes.io/docs/tutorials/stateless-application/guestbook/]

Run the guestbook by executing `kubectl create -f guestbook.yaml`

You'll see something like this:

```
$ kubectl create -f guestbook.yaml
deployment.apps/redis-master created
service/redis-master created
deployment.apps/redis-slave created
service/redis-slave created
deployment.apps/frontend created
service/frontend created
```

Load up the guestbook by running the command `minikube service frontend`. The output would look like this:
```
$ minikube service frontend
|-----------|----------|-------------|---------------------------|
| NAMESPACE |   NAME   | TARGET PORT |            URL            |
|-----------|----------|-------------|---------------------------|
| default   | frontend |          80 | http://192.168.64.2:31824 |
|-----------|----------|-------------|---------------------------|
🎉  Opening service default/frontend in default browser...
```

# Working with configmaps

## Chapter Goals
1. Learn how to declare a configmap
2. Understand how to call a configmap from a deployment

### Learn how to declare a configmap
Applications require a way for us to pass data to them that can be changed at deploy time. Examples of this might be log-levels or urls of external systems that the application might need at startup time. Instead of hardcoding these values, we can use a configmap in kubernetes, and pass these values as environment variables to the container.

We will take an example of "log_level", and pass the value "debug" to a pod via a configmap in this example.

To create a configmap for this literal type `kubectl create configmap logger --from-literal=log_level=debug`

To see all your configmaps: `kubectl get configmaps`

To read the value in the logger configmap: `kubectl get configmap/logger -o yaml`

To edit the value, we can run `kubectl edit configmap/logger`

### Understand how to call a configmap from a deployment




# Jobs in Kubernetes

## Chapter Goals
1. How to run jobs
2. How to run cron jobs

### How to run jobs
Jobs are a construct that run a pod once, and then stop. However, unlike pods in deployments, the output of the job is kept around until you decide to remove it.

Running a job is similar to running a deployment, and we can create this by `kubectl create -f simplejob.yaml`

To see the output of the job: `kubectl get jobs`

You can find the pod that ran by doing a `kubectl get pods`, and then get the logs from it as well.

### How to run cron jobs
Cron jobs are like jobs, but they run periodically.

Start your cron by running `kubectl create -f cronjob.yaml`

We can use the cronjob api to view your cronjobs: `kubectl get cronjobs`. It adds the last schedule date



# Working with secrets

## Chapter Goals
1. Learn how to declare a secret
2. Understand how to add a secret to a deployment

### Learn how to declare a secret
Just like configuration data, applications might also require other data that might be of more sensitive in nature- for example database passwords, or API tokens. Passing these in the yaml for a deployment or pod would make them visible to everyone.

In these usecases, use a secret to encapsulate sensitive data.

To create a secret: `kubectl create secret generic apikey --from-literal=api_key=123456789`

Notice that we can't read the value of the secret directly:
`kubectl get secret apikey -o yaml`

### Understand how to add a secret to a deployment

Adding a secret to a deployment is similar to what we did for configmaps. You can add a secret to the env portion, and start up the deployment with:
`kubectl create -f secretreader-deployment.yaml`







# Running the Kubernetes Dashboard

## Chapter Goals
1. Run the Kubernetes Dashboard in minikube
2. Explore the dashboard and look for the features it supports
3. Create a deployment from the dashboard

### Run the Kubernetes Dashboard in minikube
The Kubernetes Dashboard is a simple, web user interface for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as manage the cluster itself.

One of the common asks in the container world is for the need for a ui to visualize what is going on with clusters. It was one of the most requested features of Docker when it first started, and is a common requirement for devops engineers to manage clusters.

Kubernetes learned from this lesson early on, and created the Kubernetes Dashboard for this purpose, and it allows you to monitor and view clusters from an operational perspective.

One thing I like about minikube are the addons, and the Kubernetes Dashboard comes bundled as an addon. You can view all addons by typing `minikube addons list`. Let's enable the dashboard by typing `minikube addons enable dashboard`.

We will also enable the metrics server, to see cluster memory and CPU usage. This is enabled on minikube with `minikube addons enable metrics-server`

To start up the dashboard, type `minikube dashboard`

This will bring up the dashboard in your browser.


### 2. Explore the dashboard and look for the features it supports
Explore the cluster sections, available namespaces.

Visit the deployments section, and drill down into a deployment, and notice how you can scale, edit or delete a deployment. Visit the replica set for the deployment to see all the pods associated with it.

Go into the pod section to see all the pods running. Notice how you can fetch pod logs from the ui directly, or exec into a pod if it has it available.

### 3. Create a deployment from the dashboard
Create a helloworld deployment with the UI; specify an app name, and container image karthequian/helloworld:latest with 2 pods. See the deployment and pods running in the UI.

#### Handout Links

[Kubernetes Dashboard doc] https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
[Dashboard - Full Featured Web Interface for Kubernetes] http://blog.kubernetes.io/2016/07/dashboard-web-interface-for-kubernetes.html
[Dashboard Github] https://github.com/kubernetes/dashboard





# Basic troubleshooting techniques

## Chapter Goals
1. Kubernetes Techniques
2. Looking at Log files
3. Executing commands in a container

### Overview
Sometimes things don't work as they should in your deployments, and you'd like to take a closer look to debug issues or understand what's going on. There are 3 techniques I use from a day to day basis when I work with kubernetes. Let me show you what these are.

### Kubernetes Techniques
When things are not deploying as expected, or things seem to be taking a while, I describe the deployments and pods associated with the deployments to look for errors.

Let's run the helloworld application that is bundled with this section by typing `kubectl create -f helloworld-with-bad-pod.yaml`.

As it's starting up, we can run a `kubectl get deployments` and a `kubectl describe deployment bad-helloworld-deployment`.

We notice that we have 0 available pods in the deployment that signals that there is something going on with the pod.

If we introspect pods with a `kubectl get pods`, we see that the `bad-helloworld-deployment` pod is in an image pull backoff state and isn't ready.

Describing the pod with `kubectl describe pod bad-helloworld-deployment-7bb4b7466-f6nkm`, will show me that kubernetes is having trouble pull the pod from the repository, either because it doesn't exist, or because we're missing the repository credentials.

### Looking at log files
Another technique I end up using a lot to track pod progress is looking at the log files for a pod. If you write your logs to standard out, you can get to them by the command `kubectl logs <pod_name>`. This will return the log statements that are being written by your application in the pod.

### Executing commands in a container
Finally, sometimes it is necessary to exec into the actual container running the pod to look for errors, or state. To do this, run the exec command `kubectl exec -it <pod-name> -c <container-name> /bin/bash` where -it is an interactive terminal and -c is the flag to specify the container name. Finally we want a bash style terminal.

This drops us into the container, and we can introspect into the details of our application.




# Daemonsets and Statefulsets

## Chapter Goals
1. How to run daemonsets
2. How to run statefulsets

## Daemonsets

Daemonsets: A DaemonSet ensures that all Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. Examples of a daemon set would be running your logging or monitoring agent on your nodes.

First, you'll want to make sure you tag minikube with a label of `kubectl label node/minikube infra=development` to label the node first.

In the example, I will just run a simple busybox image as a daemonset, and then run daemonset examples to show how you can tag things to run on specific nodes
`kubectl create -f daemonset.yaml` will run on the nodes

`kubectl create -f daemonset-infra-development.yaml`  will only run on nodes labeled `infra=development` (as shown above in the label)

`kubectl create -f daemonset-infra-prod.yaml`  will only run on nodes labeled `infra=production`


## Statefulsets

Statefulsets Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods.
