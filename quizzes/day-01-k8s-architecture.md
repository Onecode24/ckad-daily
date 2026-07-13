# Day 1 Quiz: Kubernetes Architecture Overview & Cluster Components

## Tier 1 — Recall

1. Which control plane component is the *only* one that talks directly to
   etcd?
2. What is the difference in responsibility between `kube-scheduler` and
   `kube-controller-manager`?
3. Name the two agents that run on every worker node and briefly say what
   each does.

## Tier 2 — Application

1. Write the exact `kubectl` command to list every pod in the `kube-system`
   namespace with wide output (showing node and IP).
2. Write the exact `kubectl` command to print only the kubelet version of
   the first node in the cluster, using `jsonpath`.
3. Write the exact `kubectl` command(s) to switch your current context's
   default namespace to `dev` without changing the context itself.

## Tier 3 — Exam-sim (target: 3 minutes)

1. A pod is stuck in `Pending` and `kubectl describe pod <name>` shows an
   event `0/1 nodes are available: 1 Insufficient cpu`. Which control plane
   component produced that event, and what two things would you check next
   to unstick it? (You don't need to fix it — just name the component and
   your next two diagnostic commands.)
2. True or false, and justify in one sentence: if `kube-apiserver` goes
   down but `kubelet` and the container runtime are still running on a
   node, all Pods already running on that node immediately stop.

<details>
<summary>Answer key</summary>

**Tier 1**
1. `kube-apiserver` — it is the sole component with direct read/write
   access to etcd; every other component (including `kubectl`) goes
   through it.
2. `kube-scheduler` only *decides which node* a new, unscheduled Pod
   should run on. `kube-controller-manager` runs the ongoing reconciliation
   loops (Deployment, ReplicaSet, Node, etc.) that keep actual state
   matching desired state over time.
3. `kubelet` — receives Pod specs for its node from the API server and
   ensures the described containers are actually running via the
   container runtime. `kube-proxy` — maintains per-node network rules
   (iptables/IPVS) so traffic to a Service's ClusterIP reaches the correct
   backend Pod(s).

**Tier 2**
1. `kubectl get pods -n kube-system -o wide`
2. `kubectl get nodes -o jsonpath='{.items[0].status.nodeInfo.kubeletVersion}'`
3. `kubectl config set-context --current --namespace=dev`

**Tier 3**
1. Produced by `kube-scheduler` (it emits the `FailedScheduling` event
   when no node satisfies the Pod's resource requests). Next checks:
   `kubectl describe nodes` (look at `Allocatable`/`Allocated resources`
   to see actual headroom) and `kubectl get pod <name> -o yaml` (check the
   Pod's `resources.requests.cpu` to see what it's asking for — it may
   simply be requesting more CPU than any node has free).
2. **False.** The API server being down does not stop already-running
   containers — `kubelet` and the container runtime keep existing Pods
   alive independently. What breaks is *new* scheduling, config updates,
   and `kubectl` access, since those all require the API server.

</details>
