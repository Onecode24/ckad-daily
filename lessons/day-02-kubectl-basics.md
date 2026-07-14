# Day 2: kubectl Basics — Contexts, Namespaces, Output Formats & `explain`

## Concept brief

Day 1 covered *what* the cluster is made of. Today is about the tool you'll
use for every remaining question on the exam: `kubectl` itself, and the
handful of "meta" commands that let you navigate a cluster fast instead of
guessing.

A **context** bundles together a cluster, a user (credentials), and a
default namespace into one named entry in your kubeconfig
(`~/.kube/config`). On the real exam you're often handed several clusters
(e.g. `k8s`, `k8s-prod`) and told "solve this question in context X" — the
very first thing you do is switch context, otherwise you'll create the
right object in the wrong cluster and get zero credit. `kubectl config` is
the command family for reading and switching contexts.

A **namespace** is a logical partition inside a *single* cluster — a way to
scope names (`kubectl get pod nginx -n dev` and `-n prod` are different
pods) and apply RBAC/resource limits per team or environment. Some objects
are namespaced (Pods, Deployments, Services, ConfigMaps...), some are
cluster-scoped (Nodes, PersistentVolumes, ClusterRoles...) — `kubectl
api-resources` tells you which is which, and it's a common exam gotcha:
forgetting `-n <namespace>` silently operates on `default` instead.

**Output formats** (`-o wide/yaml/json/jsonpath/custom-columns`) matter
because CKAD is timed — you rarely have time to `describe` and read
paragraphs of text. `-o yaml` shows you the full live object (great for
copying a pattern), `-o wide` adds a couple of extra columns to `get`
(node, IP), and `jsonpath`/`custom-columns` let you extract exactly one
field for scripting or verification commands.

`kubectl explain` is the exam's built-in documentation — since you get no
internet access during the real exam, `kubectl explain <resource>.<field>`
is how you look up exact field names and types instead of guessing at
YAML structure from memory.

## kubectl command drills

Contexts and namespaces:

```bash
# Contexts
kubectl config get-contexts
kubectl config current-context
kubectl config use-context minikube
kubectl config set-context --current --namespace=dev

# Namespaces
kubectl create namespace dev
kubectl get namespaces
kubectl get ns dev -o yaml
kubectl config set-context --current --namespace=default   # switch back
```

Output formats — same `get` call, different lenses:

```bash
kubectl run web --image=nginx --dry-run=client -o yaml
kubectl create deployment web --image=nginx
kubectl get pods -o wide
kubectl get pod -l app=web -o yaml
kubectl get pod -l app=web -o json
kubectl get pods -o jsonpath='{.items[0].metadata.name}{"\n"}'
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

`kubectl explain` — building YAML from memory-free field lookups:

```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources
kubectl explain deployment.spec.strategy --recursive
```

Mutating an existing object once you know the field names:

```bash
kubectl set image deployment/web nginx=nginx:1.25
kubectl scale deployment/web --replicas=3
kubectl edit deployment web
kubectl patch deployment web -p '{"spec":{"replicas":2}}'
```

Clean up:

```bash
kubectl delete deployment web
kubectl delete namespace dev
```

## Hands-on lab (run on your own local minikube)

**Task:** Create a `dev` namespace, deploy an nginx Deployment named `web`
into it (3 replicas), then — without switching your current context's
namespace — retrieve the image of container 0 in `web` using `jsonpath`,
scoped by `-n dev`.

```bash
kubectl create namespace dev
kubectl create deployment web --image=nginx --replicas=3 -n dev
kubectl get pods -n dev -o wide
```

**Verification command** (should print `nginx` and a count of `3`):

```bash
kubectl get deployment web -n dev -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
kubectl get pods -n dev -l app=web --field-selector=status.phase=Running -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | wc -l
```
