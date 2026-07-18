# Day 5: ReplicaSets & the Reconciliation Loop

## Concept brief

Day 4 covered labels and selectors — the matching mechanism Kubernetes uses
to connect loosely-coupled objects. Today's topic, **ReplicaSets**, is the
first object you'll meet that actually *uses* that mechanism to keep
something running: it's Kubernetes' answer to "make sure N copies of this
Pod always exist."

A ReplicaSet's spec has three parts: `replicas` (the desired count),
`selector` (a set-based label selector — `matchLabels`/`matchExpressions` —
identifying which Pods it owns), and `template` (the Pod spec used to create
new Pods when needed). The ReplicaSet controller runs a continuous
**reconciliation loop**: watch the current state (how many Pods exist that
match the selector), compare it to desired state (`replicas`), and take the
smallest action to close the gap — create Pods if short, delete Pods if
over. This observe-diff-act loop is the core pattern behind *every*
Kubernetes controller (Deployments, Jobs, DaemonSets all work the same way
at a higher level), so understanding it here pays off for the rest of the
syllabus.

A subtlety that trips people up (and shows up on the exam): a ReplicaSet
doesn't "contain" its Pods in any structural sense — it *counts* Pods
matching its selector, the same dynamic label-matching from Day 4. If you
manually create a Pod with matching labels, the ReplicaSet adopts it and
counts it toward `replicas` (possibly deleting one of its own to compensate
if you're now over). If you relabel a Pod away from the selector, the
ReplicaSet treats it as gone and creates a replacement — the orphaned Pod
keeps running, just unmanaged.

In practice you will almost never create a bare ReplicaSet directly — you
create a **Deployment**, which manages ReplicaSets for you (one per
rollout, enabling rolling updates/rollback, coming in Week 4). But CKAD
expects you to know what's underneath a Deployment, to read/troubleshoot
`kubectl get rs` output, and to know why deleting a Pod a ReplicaSet owns
doesn't actually reduce your replica count.

## kubectl command drills

There's no `kubectl run --replicas` for ReplicaSets directly — imperative
creation isn't supported for this object, so the fastest exam-legal path is
`--dry-run=client -o yaml` scaffolding (or generating it from a Deployment
and stripping it down). Generate the YAML:

```bash
kubectl create deployment web --image=nginx --replicas=3 \
  --dry-run=client -o yaml > web-deploy.yaml
```

Or hand-write a minimal ReplicaSet YAML directly (there's no imperative
generator for `kind: ReplicaSet`, so this is the one object type where
you'll type YAML from memory):

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```

Apply and inspect:

```bash
kubectl apply -f web-rs.yaml
kubectl get rs
kubectl get rs web-rs -o wide
kubectl describe rs web-rs
kubectl get pods -l app=web --show-labels
```

Scale (imperative, same verb as Deployments/Services):

```bash
kubectl scale rs web-rs --replicas=5
kubectl scale rs web-rs --replicas=2
kubectl get rs web-rs -o jsonpath='{.status.replicas}/{.spec.replicas}{"\n"}'
```

Mutate the Pod template (note: editing `spec.template` on a live ReplicaSet
does **not** roll out to existing Pods — only new Pods created afterward get
it, since a bare ReplicaSet has no rollout mechanism, unlike a Deployment):

```bash
kubectl set image rs/web-rs nginx=nginx:1.25
kubectl edit rs web-rs
kubectl patch rs web-rs -p '{"spec":{"replicas":4}}'
```

Prove the reconciliation loop live — delete a managed Pod and watch it get
replaced, then orphan one by relabeling it:

```bash
kubectl delete pod -l app=web --field-selector=status.phase=Running \
  --dry-run=client   # sanity-check the selection first
kubectl get pods -l app=web -w &
kubectl delete pod $(kubectl get pods -l app=web -o jsonpath='{.items[0].metadata.name}')
# a replacement Pod appears within seconds — Ctrl+C the watch
kubectl label pod <one-remaining-pod> app=web-orphan --overwrite
# ReplicaSet creates a new Pod to replace the one it lost; the relabeled
# Pod keeps running but is no longer counted
```

Clean up:

```bash
kubectl delete rs web-rs
```

## Hands-on lab (run on your own local minikube)

**Task:** Create a ReplicaSet named `payments-rs` with `replicas: 3`,
selector `app=payments`, running `nginx`. Confirm 3 Pods are running. Delete
one Pod directly and confirm the ReplicaSet replaces it (still 3 Pods,
different Pod name set). Then create a **fourth**, unmanaged Pod by hand
with a **non-matching** label (`app=payments-extra`) and confirm the
ReplicaSet ignores it (still shows `replicas: 3`, desired count unaffected).
Finally, relabel that fourth Pod to `app=payments` and confirm the
ReplicaSet deletes one Pod to bring the count back down to 3 (it now owns 4
matching Pods but only wants 3).

```bash
cat <<'EOF' > payments-rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: payments-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payments
  template:
    metadata:
      labels:
        app: payments
    spec:
      containers:
        - name: nginx
          image: nginx
EOF
kubectl apply -f payments-rs.yaml
kubectl run payments-extra --image=nginx --labels="app=payments-extra"
```

**Verification command** (run before and after each step to observe the
reconciliation loop in action):

```bash
kubectl get rs payments-rs -o jsonpath='{.spec.replicas}/{.status.replicas}{"\n"}'
kubectl get pods -l app=payments -o jsonpath='{.items[*].metadata.name}{"\n"}'
```
