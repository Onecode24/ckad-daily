# Day 1: Kubernetes Architecture Overview & Cluster Components

## Concept brief

A Kubernetes cluster has two kinds of machines: **control plane nodes** and
**worker nodes**. You don't need to administer the control plane for the
CKAD exam (that's CKA territory), but you do need to know what each piece
does so you can reason about where things run and why something is broken.

**Control plane components:**
- `kube-apiserver` — the front door. Every `kubectl` command, every
  controller, everything talks to the cluster *only* through the API
  server. It validates and persists objects to etcd.
- `etcd` — the cluster's key-value store. The single source of truth for
  all cluster state (every object you create lives here).
- `kube-scheduler` — watches for Pods with no node assigned and picks a
  node for them based on resource requests, affinity rules, taints, etc.
- `kube-controller-manager` — runs the reconciliation loops (Deployment
  controller, ReplicaSet controller, Node controller, etc.) that constantly
  compare *desired state* (what you asked for) to *actual state* (what's
  running) and act to close the gap.

**Worker node components:**
- `kubelet` — the agent on every node. Talks to the API server, receives
  Pod specs assigned to its node, and makes sure the containers described
  in those specs are actually running (via the container runtime).
- `kube-proxy` — maintains network rules on each node so traffic to a
  Service's ClusterIP gets routed to the right backend Pods.
- **Container runtime** — containerd (or another CRI-compliant runtime)
  that actually pulls images and runs containers.

**The core mental model for CKAD:** you describe *desired state* in a
manifest (or via `kubectl run`/`create`), the API server stores it, and a
controller loop continuously drives *actual state* toward it. Nothing you
do in this exam is a one-shot imperative action under the hood — it's
always "update desired state, let a controller reconcile." This is why
`kubectl scale`, `kubectl set image`, and editing a Deployment's YAML all
work the same way: they change desired state, and the Deployment/ReplicaSet
controllers do the rest.

## kubectl command drills

Get oriented in a cluster (run these against your local minikube):

```bash
# Cluster & node info
kubectl cluster-info
kubectl get nodes -o wide
kubectl describe node minikube

# See control-plane components running as pods (kubeadm-style clusters;
# on minikube these are static pods in kube-system)
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide

# Inspect the API server's view of resource types
kubectl api-resources | head -20
kubectl api-versions

# Watch reconciliation happen live: create something and watch events
kubectl create deployment demo --image=nginx --dry-run=client -o yaml
kubectl create deployment demo --image=nginx
kubectl get events --sort-by=.lastTimestamp | tail -10
kubectl get pods -w   # Ctrl+C once you see it go Running

# Clean up
kubectl delete deployment demo
```

Useful context/namespace commands you'll use every single day from here on:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl config set-context --current --namespace=default
kubectl get all -A
```

## Hands-on lab (run on your own local minikube)

**Task:** Start minikube if it isn't running, then identify every control
plane pod and confirm the kubelet + container runtime versions match what
the API server reports.

```bash
minikube start   # if not already running
kubectl get pods -n kube-system -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
kubectl version --short
kubectl get nodes -o jsonpath='{.items[0].status.nodeInfo.kubeletVersion} {.items[0].status.nodeInfo.containerRuntimeVersion}{"\n"}'
```

**Verification command** (should print `Running` for every kube-system pod
and a non-empty kubelet/runtime version pair):

```bash
kubectl get pods -n kube-system --field-selector=status.phase!=Running
# ^ empty output = every kube-system pod is Running (success)
kubectl get nodes -o jsonpath='{.items[0].status.nodeInfo.kubeletVersion} {.items[0].status.nodeInfo.containerRuntimeVersion}{"\n"}'
# ^ should print two non-empty version strings
```
