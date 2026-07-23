# OpenShift Admin Interview Prep (Beginner Friendly)

This notes file follows the full Learning-Ocean Kubernetes topic list.  
Everything is written in **easy English** for beginners.  
All commands use **`oc`** (OpenShift CLI).
https://learning-ocean.com/tutorials/kubernetes/kubernetes-container-orchestration/

---

## How to use this file

1. Read one topic at a time.
2. Say the “In one line” sentence out loud.
3. Practice the commands on a lab (CRC / practice cluster).
4. Learn the “Interview question” at the end of each topic.

---

# Table of Contents

1. Container Orchestration  
2. What Is Kubernetes (and OpenShift)  
3. Minikube / Local Practice Cluster  
4. Cluster Setup  
5. Bash Completion  
6. Pods  
7. Delete Resources  
8. Labels  
9. Create Pod with YAML  
10. Create vs Apply  
11. Environment Variables  
12. Shell into a Container  
13. Multi-Container Pods  
14. Init Containers  
15. Sidecar Containers  
16. ClusterIP  
17. NodePort  
18. Services and Routes  
19. Replication Controller  
20. Deployment Overview  
21. Create a Deployment  
22. Rolling Update  
23. Recreate Strategy  
24. Namespace / Project  
25. Resource Quota  
26. Limit Range  
27. ConfigMap  
28. ConfigMap as Environment Variables  
29. ConfigMap YAML  
30. Inject ConfigMap into Pod  
31. Secrets  
32. Inject Secrets  
33. Volumes  
34. HostPath Volume  

---

# 1. Container Orchestration

## What is it?
A container is a small box that runs your app.  
If you have 100 boxes on 20 computers, managing them by hand is hard.

**Container orchestration** = software that manages those boxes for you.

## What does it do?
- Starts your app containers
- Restarts them if they crash
- Adds more copies when traffic is high
- Removes copies when traffic is low
- Moves apps to another computer if one computer dies
- Connects apps to each other
- Checks if apps are healthy

## Simple example
Without orchestration: you SSH into servers and run Docker yourself.  
With orchestration: you say “I want 3 copies of my web app” and the system does it.

## In one line
Orchestration means automatic management of containers on many machines.

## Interview question
**Q:** Why do we need container orchestration?  
**A:** Because managing many containers by hand is hard. Orchestration starts, stops, scales, heals, and connects them automatically.

---

# 2. What Is Kubernetes (and OpenShift)

## What is Kubernetes?
Kubernetes (also called **K8s**) is a free tool that orchestrates containers.

Think of Kubernetes as a **manager** for your app boxes:
- You tell it what you want
- It tries to keep that true

## What is OpenShift?
**OpenShift = Kubernetes + extra enterprise features from Red Hat.**

OpenShift gives you:
- A nice web console
- Projects (for teams)
- Routes (easy public URL for apps)
- Stronger security by default
- `oc` command line tool

## Cluster = many computers working together

| Name | Easy meaning |
|------|--------------|
| Cluster | Whole OpenShift/Kubernetes system |
| Control plane (master) | Brain of the cluster |
| Worker node | Computer that runs your apps |
| Pod | Smallest thing that runs your app |

## Brain parts (control plane)

| Part | Easy job |
|------|----------|
| API Server | Reception desk. All requests come here. |
| etcd | Notebook/database. Stores cluster data. |
| Scheduler | Chooses which worker should run a new pod. |
| Controller Manager | Watches and fixes problems (like “I need 3 pods but only 2 are running”). |

## Worker parts

| Part | Easy job |
|------|----------|
| kubelet | Agent on each worker. Starts pods and reports health. |
| kube-proxy | Helps networking / service traffic. |
| CRI-O | Software that actually runs containers on OpenShift. |

## How a request works (simple flow)
1. You type an `oc` command  
2. API Server receives it  
3. Data is saved in etcd  
4. Scheduler picks a worker  
5. kubelet starts the pod on that worker  

## In one line
Kubernetes manages containers. OpenShift is Kubernetes made ready for companies.

## Interview question
**Q:** Difference between Kubernetes and OpenShift?  
**A:** Kubernetes is the open-source base. OpenShift is built on Kubernetes and adds projects, routes, security, console, and enterprise tools. Admins use `oc`.

---

# 3. Minikube / Local Practice Cluster

## Why this topic?
Before a real company cluster, beginners practice on a laptop.

| Tool | What it is |
|------|------------|
| Minikube | Small local Kubernetes |
| CRC / OpenShift Local | Small local OpenShift (best for this prep) |

## For OpenShift beginners, use CRC
CRC runs a small OpenShift cluster on your computer.

```bash
crc start
oc login -u developer https://api.crc.testing:6443
oc whoami
```

## In one line
Use a local cluster (like CRC) to practice safely before touching production.

## Interview question
**Q:** How do you practice OpenShift at home?  
**A:** I use CRC (OpenShift Local) or a training cluster. For real work, companies use installer-based or cloud OpenShift.

---

# 4. Cluster Setup (Big Picture for Admins)

You do not need full installer details for first interviews. Know the **idea**.

## Simple setup steps
1. Prepare machines / cloud / DNS
2. Install OpenShift (installer)
3. Login and check cluster is healthy
4. Configure users/login method
5. Set storage and networking
6. Create projects for teams
7. Keep cluster updated and monitored

## Useful health commands
```bash
oc login
oc whoami
oc get nodes
oc get clusterversion
oc get co
```

`oc get co` = cluster operators (OpenShift internal services).  
If many show `False` / `Degraded`, cluster has problems.

## Node status you should know
- `Ready` = good
- `NotReady` = problem
- `SchedulingDisabled` = node is cordoned (no new pods)

## In one line
Cluster setup means install OpenShift, then check nodes and operators are healthy.

## Interview question
**Q:** First checks after login to a cluster?  
**A:** `oc whoami`, `oc get nodes`, `oc get clusterversion`, `oc get co`.

---

# 5. Bash Completion

## What is it?
When you press **Tab**, the terminal finishes the command for you.

This saves time and reduces typing mistakes.

```bash
# For bash
source <(oc completion bash)

# For zsh
source <(oc completion zsh)
```

To make it permanent, add that line to your shell config file (`.bashrc` or `.zshrc`).

## In one line
Bash completion auto-completes `oc` commands with Tab.

---

# 6. Pods

## What is a Pod?
A **Pod** is the smallest unit in Kubernetes/OpenShift.

Your app container runs **inside a pod**.

Simple picture:
`Node → Pod → Container → App`

## Important beginner facts
- OpenShift does **not** run containers alone. It runs **pods**.
- One pod can have one container (most common) or more.
- All containers in one pod share:
  - same IP address
  - same network (they can talk using `localhost`)
  - optional shared storage
- If one container uses port 80, another container in the **same pod** cannot also use port 80.
- If you delete a plain pod, it stays deleted.
- For real apps, we use **Deployment** so pods come back automatically.

## Practice commands
```bash
# create a test pod
oc run firstpod --image=nginx:latest

# list pods
oc get pods

# more details (node name, IP)
oc get pods -o wide

# full details / events
oc describe pod firstpod

# see YAML
oc get pod firstpod -o yaml

# learn fields
oc explain pod
oc explain pod.spec.containers

# watch live
oc get pods -w

# all projects
oc get pods -A
```

## Common pod statuses (learn these)
| Status | Meaning | What you do |
|--------|---------|-------------|
| Pending | Waiting (no node, image, or volume yet) | `oc describe pod` |
| Running | Started | Check if Ready |
| CrashLoopBackOff | Keeps crashing | `oc logs` |
| ImagePullBackOff | Cannot download image | Check image name / registry login |
| Completed | Finished (normal for jobs) | Usually OK |
| OOMKilled | Out of memory | Give more memory or fix app |

## In one line
Pod is the smallest runnable unit. Containers live inside pods.

## Interview question
**Q:** What is a pod?  
**A:** Smallest deployable unit. It holds one or more containers that share network and can share storage.

---

# 7. Delete Resources

## Why delete carefully?
If a pod belongs to a Deployment, deleting only the pod is temporary.  
OpenShift will create a new pod again.

```bash
# delete one pod
oc delete pod firstpod

# delete using file
oc delete -f pod.yaml

# delete a deployment (this removes its pods too)
oc delete deployment myapp

# delete everything of one kind in current project
oc delete pods --all

# delete a whole project
oc delete project demo
```

## Force delete (only when stuck)
```bash
oc delete pod stuck-pod --force --grace-period=0
```
Use this carefully.

## In one line
Delete the controller (like Deployment) if you want the app fully gone.

## Interview question
**Q:** I deleted a pod and it came back. Why?  
**A:** A Deployment/ReplicaSet is managing it. It recreates pods to match desired count.

---

# 8. Labels

## What are labels?
Labels are **tags** (key=value) on objects.

Example:
- `app=web`
- `env=dev`
- `tier=frontend`

## Why needed?
Services and Deployments find pods using labels.

```bash
# add label
oc label pod firstpod app=web env=dev

# find by label
oc get pods -l app=web
oc get pods -l app=web,env=dev

# remove a label
oc label pod firstpod env-
```

## In YAML
```yaml
metadata:
  name: firstpod
  labels:
    app: web
    env: dev
```

## In one line
Labels are tags used to select and group resources.

## Interview question
**Q:** How does a Service know which pods to send traffic to?  
**A:** By label selectors. Example: send traffic to all pods with `app=web`.

---

# 9. Create Pod with YAML

## What is YAML?
A text file that describes what you want.

## Simple pod YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

## Meaning of each part
- `apiVersion` / `kind` = what object type
- `metadata` = name and labels
- `spec` = desired setup
- `containers` = which images to run

```bash
# create from file
oc apply -f nginx-pod.yaml

# make YAML without creating yet
oc run nginx-pod --image=nginx:latest --dry-run=client -o yaml > nginx-pod.yaml
```

## In one line
YAML is the written desired state of a resource.

## Interview question
**Q:** What are the main sections of a pod YAML?  
**A:** `metadata` (name/labels) and `spec` (containers, volumes, etc.).

---

# 10. Create vs Apply

There are two common ways to work.

## 1) Imperative (do this now)
You type an action command.

```bash
oc run mypod --image=nginx
oc create deployment myapp --image=nginx
```

Good for quick tests.

## 2) Declarative (here is the final desired state)
You keep YAML files and apply them.

```bash
oc apply -f myapp.yaml
```

Good for real work and GitOps.

## create vs apply
| Command | Behavior |
|---------|----------|
| `oc create -f file.yaml` | Creates. Fails if object already exists. |
| `oc apply -f file.yaml` | Creates or updates. Safer for repeated use. |

## In one line
Use `oc apply` for real files. Use `oc run`/`oc create` for quick tests.

## Interview question
**Q:** Prefer create or apply?  
**A:** Prefer `apply` for YAML workflows because it can update existing objects.

---

# 11. Environment Variables

## What are they?
Settings given to the app when it starts.  
Example: `DB_HOST=mysql`, `APP_COLOR=blue`.

```bash
# set on a deployment
oc set env deployment/myapp APP_COLOR=blue
oc set env deployment/myapp --list
```

## In YAML
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: APP_COLOR
      value: "blue"
```

## Beginner tip
Do not hardcode passwords in env as plain text in Git.  
Use **Secret** for passwords. Use **ConfigMap** for normal settings.

## In one line
Env vars are runtime settings for containers.

---

# 12. Shell into a Running Container

Sometimes you need to go inside a pod to debug.

```bash
# easy way on OpenShift
oc rsh mypod

# another way
oc exec -it mypod -- /bin/sh

# if pod has many containers, choose one
oc exec -it mypod -c nginx -- /bin/sh

# run one command without full shell
oc exec mypod -- ls /app
oc exec mypod -- cat /etc/hostname
```

## When to use
- Check if files exist
- Test network (`curl`, `ping` if available)
- Read config inside container

## In one line
`oc rsh` / `oc exec` lets you enter a running container for debugging.

## Interview question
**Q:** How do you open a shell in a pod?  
**A:** `oc rsh <pod>` or `oc exec -it <pod> -- /bin/sh`.

---

# 13. Multi-Container Pods

## What is it?
One pod with **more than one container**.

They share:
- same IP
- localhost networking
- volumes (if configured)

## Good example
- Container 1: web app  
- Container 2: log collector  

They must work closely together.

## Bad example
- Web app + database + cache all forced into one pod  
(hard to scale, hard to manage)

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
  - name: logger
    image: fluent-bit:latest
```

## In one line
Multi-container pods are for tightly connected helper containers only.

## Interview question
**Q:** Should every app use multi-container pods?  
**A:** No. Use them only when containers must share network/storage tightly. Otherwise separate pods.

---

# 14. Init Containers

## What is it?
A special container that runs **before** the main app starts.

## Easy story
Before restaurant opens (app), a worker (init) cleans tables and checks kitchen.  
Only then guests (main container) come in.

## Common uses
- Wait until database is ready
- Download config files
- Set folder permissions

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.36
    command: ['sh', '-c', 'echo waiting; sleep 5']
  containers:
  - name: app
    image: myapp:1.0
```

## Important
- Init containers run one by one
- Main app starts only after all inits succeed
- If init fails, app does not start

## In one line
Init containers prepare things before the main container starts.

## Interview question
**Q:** Difference between init and normal container?  
**A:** Init runs first and exits. Normal/app containers keep running.

---

# 15. Sidecar Containers

## What is it?
A helper container that runs **together with** the main app for the whole time.

## Easy story
Main singer (app) and sound engineer (sidecar) work during the whole concert.

## Common sidecars
- Log shipper
- Monitoring agent
- Proxy (in service mesh)

## Init vs Sidecar (very common interview ask)
| Point | Init | Sidecar |
|-------|------|---------|
| When it runs | Before app | With app |
| Does it stay running? | No | Yes |
| Purpose | Setup / wait | Ongoing help |

## In one line
Sidecar = helper container that stays beside the main app.

---

# 16. ClusterIP

## Problem first
Pods get new IPs when they restart.  
Other apps cannot keep chasing changing IPs.

## Solution
A **Service** gives a stable virtual IP and name.

**ClusterIP** is the default service type.  
It is reachable **only inside the cluster**.

```bash
oc expose deployment myapp --port=80 --target-port=8080
oc get svc
```

## YAML idea
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

Meaning:
- Other pods call port `80` on the service
- Traffic goes to pod port `8080`
- Only pods with label `app=myapp` receive traffic

## In one line
ClusterIP = internal stable access to pods.

## Interview question
**Q:** Can users on the internet use ClusterIP directly?  
**A:** No. It is internal only. Use Route/NodePort/LoadBalancer for outside access.

---

# 17. NodePort

## What is it?
A service type that opens a port on **every node**.

Example:
`http://node-ip:30080`

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

## Beginner notes
- NodePort range is usually 30000–32767
- Easy for labs
- In real OpenShift, people usually prefer **Route**

## In one line
NodePort publishes a service on a high port of each node.

---

# 18. Services and Routes

## Service (Kubernetes idea)
Stable way to reach a group of pods.

Service types:
1. **ClusterIP** – inside cluster  
2. **NodePort** – via node IP + port  
3. **LoadBalancer** – cloud load balancer  

```bash
oc get svc
oc describe svc myapp-svc
oc get endpoints myapp-svc
```

If endpoints are empty, labels probably do not match any pods.

## Route (OpenShift extra — very important)
Route gives your app a **hostname** like:
`https://myapp.apps.example.com`

```bash
# create route from service
oc expose svc myapp-svc
oc get routes
oc describe route myapp-svc
```

## Easy picture
User → Route → Service → Pods

## In one line
Service finds pods. Route exposes HTTP/HTTPS apps outside OpenShift.

## Interview question
**Q:** Service vs Route?  
**A:** Service gives stable access to pods (mostly internal). Route exposes the service to outside users with a DNS name.

---

# 19. Replication Controller

## What is it?
Old object that keeps a fixed number of pod copies running.

Example: “Always keep 3 pods.”

If one dies, it creates another.

## Today
We mostly use **Deployment** (which uses ReplicaSet).  
Replication Controller is older, but interviewers may still ask.

```bash
oc get rc
```

## In one line
Replication Controller keeps N pod copies alive (legacy). Prefer Deployment now.

## Interview question
**Q:** Is Replication Controller still used?  
**A:** Concept yes, object rarely. Modern apps use Deployment/ReplicaSet.

---

# 20. Deployment Overview

## What is a Deployment?
The normal way to run apps on OpenShift/Kubernetes.

Deployment watches your app and keeps the right number of pods.

## Object chain
`Deployment → ReplicaSet → Pods`

## What you get
- Auto restart if pod dies
- Easy scaling (`replicas: 3`)
- Easy updates
- Easy rollback

```bash
oc get deploy
oc get rs
oc get pods
oc describe deploy myapp
```

## OpenShift note
Older OpenShift also had **DeploymentConfig**.  
New work usually uses Kubernetes **Deployment**.

## In one line
Deployment manages replicas and updates for your app pods.

## Interview question
**Q:** Why not create pods directly in production?  
**A:** Bare pods do not self-heal or roll out cleanly. Deployment does.

---

# 21. Create a Deployment

```bash
# quick create
oc create deployment myapp --image=nginx:latest

# scale to 3
oc scale deployment myapp --replicas=3

# check
oc get deploy
oc get pods
```

## YAML example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

```bash
oc apply -f deploy.yaml
```

## Important beginner point
`selector.matchLabels` and pod `labels` must match.  
If they do not match, Deployment will not manage those pods.

## In one line
Create a Deployment with image + replicas + matching labels.

---

# 22. Rolling Update

## What is it?
Update pods little by little.  
Old pods and new pods can run together for some time.  
Users usually feel little or no downtime.

```bash
# change image
oc set image deployment/myapp nginx=nginx:1.26

# watch update
oc rollout status deployment/myapp

# history
oc rollout history deployment/myapp

# undo if bad
oc rollout undo deployment/myapp
```

## In YAML
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

Meaning in simple words:
- `maxUnavailable` = how many old pods can go down during update
- `maxSurge` = how many extra new pods can be added during update

## In one line
Rolling update replaces pods gradually with less downtime.

## Interview question
**Q:** How do you rollback a bad deployment?  
**A:** `oc rollout undo deployment/<name>`.

---

# 23. Recreate Strategy

## What is it?
Stop all old pods first.  
Then start new pods.

There is **downtime**.

```yaml
spec:
  strategy:
    type: Recreate
```

## When to use
- App cannot run old and new version together
- App needs exclusive access to something
- Short downtime is okay

## Rolling vs Recreate
| | Rolling | Recreate |
|--|---------|----------|
| Downtime | Usually low/none | Yes |
| Old + new together | Yes | No |
| Common for | Web apps | Special single-instance apps |

## In one line
Recreate means kill all old pods, then start new ones.

---

# 24. Namespace / Project

## What is a Namespace?
A folder-like area inside the cluster to separate resources.

Example:
- `team-a`
- `team-b`
- `dev`
- `prod`

## What is a Project in OpenShift?
**Project = Namespace + OpenShift extras**  
(easier for users/teams, with extra metadata and access control).

```bash
# create and switch
oc new-project demo
oc project demo

# list
oc get projects
oc get ns

# delete project (deletes things inside too)
oc delete project demo
```

## Rules in simple words
- Same name can exist in different projects
- One resource belongs to one project
- Nodes are not inside a project (cluster-wide)

## In one line
Project/Namespace separates teams and apps inside one cluster.

## Interview question
**Q:** Namespace vs Project?  
**A:** Namespace is Kubernetes isolation. Project is OpenShift’s version of namespace with extra features.

---

# 25. Resource Quota

## What is it?
A limit on **how much a whole project can use**.

Like a family budget:
- Max 20 pods
- Max 4 CPU
- Max 8Gi memory

```bash
oc create quota demo-quota \
  --hard=pods=20,requests.cpu=4,requests.memory=8Gi \
  -n demo

oc describe quota -n demo
```

## YAML idea
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo-quota
  namespace: demo
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: 8Gi
```

## What happens if exceeded?
New pods may fail to create.  
Error often says quota exceeded / forbidden.

## In one line
ResourceQuota limits total usage of a project.

## Interview question
**Q:** Why do admins set quotas?  
**A:** To stop one team from using all cluster CPU/memory/pods.

---

# 26. Limit Range

## What is it?
Rules for **each container/pod** inside a project.

Example:
- If user forgets memory request, give default 128Mi
- Do not allow any container above 2Gi

```bash
oc describe limitrange -n demo
```

## YAML idea
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: demo-limits
  namespace: demo
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    default:
      cpu: 500m
      memory: 512Mi
    max:
      cpu: "2"
      memory: 2Gi
```

## Quota vs LimitRange (memorize)
| | ResourceQuota | LimitRange |
|--|---------------|------------|
| Controls | Total for project | Each pod/container |
| Example | Project max 20 pods | One container max 2Gi |

## In one line
LimitRange sets defaults and max/min for each container.

---

# 27. ConfigMap

## What is it?
A place to store **normal config** (not passwords).

Examples:
- theme color
- feature flags
- config files
- app mode = dev/prod

```bash
# from literal values
oc create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod

# from file
oc create configmap app-config --from-file=app.properties

# from env file
oc create configmap app-config --from-env-file=config.env

# view
oc get cm
oc get cm app-config -o yaml
oc describe cm app-config
```

## Why useful?
You can change config without rebuilding the image.

## In one line
ConfigMap stores non-secret settings for apps.

## Interview question
**Q:** ConfigMap vs hardcoding config in image?  
**A:** ConfigMap keeps config outside image, so same image can run in dev/test/prod with different settings.

---

# 28. ConfigMap as Environment Variables

Give ConfigMap values to the container as env vars.

```bash
oc set env deployment/myapp --from=configmap/app-config
```

## YAML way (all keys)
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
```

## YAML way (one key)
```yaml
env:
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR
```

## In one line
You can load ConfigMap keys as environment variables.

---

# 29. ConfigMap YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
  app.properties: |
    color=blue
    mode=prod
```

```bash
oc apply -f configmap.yaml
```

## Beginner tip
Under `data:`, values are plain text.  
(For Secrets, values are usually base64 in YAML.)

## In one line
ConfigMap YAML has `data:` with your keys and values.

---

# 30. Inject ConfigMap into Pod

Two beginner ways:

## Way A: as environment variables
Good for simple settings.

## Way B: as files (volume mount)
Good for config files.

```yaml
volumes:
- name: config-vol
  configMap:
    name: app-config
containers:
- name: app
  image: myapp:1.0
  volumeMounts:
  - name: config-vol
    mountPath: /etc/config
```

Then inside container you may see:
- `/etc/config/APP_COLOR`
- `/etc/config/APP_MODE`

## Important beginner note
If you change ConfigMap used as env vars, restart the pod to pick up changes.

## In one line
Inject ConfigMap as env vars or as mounted files.

## Interview question
**Q:** Ways to use ConfigMap in a pod?  
**A:** Environment variables, or mount as files in a volume.

---

# 31. Secrets

## What is it?
Like ConfigMap, but for **sensitive data**:
- passwords
- API tokens
- private keys

```bash
oc create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=S3cret!

oc create secret generic db-secret --from-file=password.txt
oc create secret generic db-secret --from-env-file=secret.env

oc get secret
oc describe secret db-secret
```

## Important truth for interviews
Secret data is **base64 encoded** in YAML.  
Encoding is not full encryption by itself.  
Still better than putting passwords in normal ConfigMaps or images.

```bash
# decode example
oc get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

## Common secret types
- generic (opaque)
- tls
- docker-registry

## In one line
Secrets store sensitive values for pods to use.

## Interview question
**Q:** Are Kubernetes/OpenShift Secrets fully encrypted by default?  
**A:** They are base64 encoded in API objects. At rest encryption depends on cluster setup. Still use Secrets, not ConfigMaps, for passwords.

---

# 32. Inject Secrets

## As environment variable
```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

## As file mount
```yaml
volumes:
- name: secret-vol
  secret:
    secretName: db-secret
containers:
- name: app
  volumeMounts:
  - name: secret-vol
    mountPath: /etc/secret
    readOnly: true
```

```bash
oc set env deployment/myapp --from=secret/db-secret
```

## Beginner best practices
- Do not put secrets in Git in plain text
- Prefer Secret object over hardcoding
- Give pods only the secrets they need

## In one line
Inject secrets as env vars or mounted files, same idea as ConfigMap.

---

# 33. Volumes

## What is a volume?
Storage attached to a pod.

Container files are temporary by default.  
If container restarts, files inside can be lost.  
Volumes help keep or share data.

## Common beginner volume types
| Type | Easy meaning |
|------|--------------|
| emptyDir | Temporary folder for life of pod |
| configMap | Mount config as files |
| secret | Mount secret as files |
| PVC | Persistent storage request (real apps) |
| hostPath | Use folder from the worker node |

## emptyDir example
```yaml
volumes:
- name: cache
  emptyDir: {}
containers:
- name: app
  volumeMounts:
  - name: cache
    mountPath: /cache
```

## For real apps (admin answer)
Use **PVC + StorageClass** so data can survive pod restarts and be managed properly.

```bash
oc get sc
oc get pv
oc get pvc
```

## In one line
Volumes give pods storage that can be shared or persistent.

## Interview question
**Q:** Is container disk permanent?  
**A:** No. Use volumes/PVC for data you need to keep.

---

# 34. HostPath Volume

## What is it?
Mount a folder from the **worker computer (node)** into the pod.

```yaml
volumes:
- name: host-data
  hostPath:
    path: /data
    type: DirectoryOrCreate
containers:
- name: app
  volumeMounts:
  - name: host-data
    mountPath: /mnt/data
```

## Why beginners should be careful
- Pod becomes tied to that node’s disk
- Security risk (pod can touch host files)
- OpenShift security (SCC) often **blocks** hostPath

## Better option
Use PVC for application data.

## In one line
HostPath mounts node local folders. Useful for demos, risky for normal apps.

## Interview question
**Q:** Why avoid hostPath in OpenShift?  
**A:** Security risk and node coupling. SCC may block it. Use PVC instead.

---

# Final Beginner Cheat Sheet

```bash
# login and where am I
oc login
oc whoami
oc project

# see things
oc get nodes
oc get pods
oc get pods -o wide
oc get deploy,svc,route
oc get cm,secret
oc get pvc

# debug
oc describe pod <name>
oc logs <name>
oc rsh <name>
oc get events

# change apps
oc apply -f file.yaml
oc delete -f file.yaml
oc scale deploy/myapp --replicas=3
oc set image deploy/myapp nginx=nginx:1.26
oc rollout undo deploy/myapp
```

---

# Easy End-to-End Story (connects all topics)

1. Admin creates a **Project**  
2. Admin sets **Quota** and **LimitRange**  
3. Dev creates a **Deployment** (which creates **Pods**)  
4. Config goes in **ConfigMap**, passwords in **Secret**  
5. App gets a **Service** (ClusterIP)  
6. OpenShift **Route** gives public URL  
7. Update app with **Rolling update**  
8. If update is bad, **rollout undo**

If you can explain this story, you already sound strong in a junior OpenShift admin interview.

---

# 20 Very Common Interview Q&A (Quick Revision)

1. **What is OpenShift?** Kubernetes plus enterprise features.  
2. **What is a Pod?** Smallest unit that runs containers.  
3. **What is a Node?** A worker machine in the cluster.  
4. **What is etcd?** Cluster database.  
5. **What does scheduler do?** Chooses node for new pods.  
6. **Why labels?** To select and group resources.  
7. **Service purpose?** Stable access to changing pods.  
8. **ClusterIP vs NodePort?** Internal vs node-port access.  
9. **What is Route?** OpenShift way to expose HTTP app externally.  
10. **Deployment vs Pod?** Deployment manages and heals pods.  
11. **Rolling vs Recreate?** Gradual update vs stop-all-then-start.  
12. **Project vs Namespace?** Project is OpenShift namespace+.  
13. **Quota vs LimitRange?** Project total vs per-container rules.  
14. **ConfigMap for what?** Non-secret config.  
15. **Secret for what?** Passwords/tokens/keys.  
16. **Init vs Sidecar?** Before app vs beside app.  
17. **How to debug pod?** describe → logs → rsh/exec.  
18. **Why pod Pending?** No resources, image pull, PVC, scheduling rules.  
19. **Why CrashLoopBackOff?** App keeps crashing; check logs.  
20. **Why avoid hostPath?** Security and node dependency; use PVC.

---

# Suggested Learning Order (Beginner Path)

**Week idea:**
1. Topics 1–5 (basics + cluster idea)  
2. Topics 6–12 (pods and daily commands)  
3. Topics 13–18 (multi-container + networking)  
4. Topics 19–23 (deployments and updates)  
5. Topics 24–26 (projects and limits)  
6. Topics 27–34 (config, secrets, storage)  

Read → practice command → speak answer out loud.

---

You now have the **full TOC**, in **simple beginner English**, with examples and interview answers for each topic.
