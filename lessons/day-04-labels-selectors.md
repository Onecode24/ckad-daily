# Day 4: Labels, Selectors & Annotations

## Concept brief

Days 1-3 covered the cluster, the CLI, and the Pod. Today's topic is how
Kubernetes *organizes* and *finds* objects: **labels**, **selectors**, and
**annotations**.

**Labels** are key/value pairs attached to `metadata.labels` on any object
(Pods, Deployments, Services, Nodes...). They're identifying metadata meant
to be queried — things like `app=frontend`, `tier=backend`,
`env=production`, `version=v2`. Labels are how the loosely-coupled parts of
Kubernetes find each other: a Service doesn't point at Pods by name, it
points at a **label selector**, and any Pod whose labels match is included
as an endpoint — even Pods created later. Same mechanism connects
ReplicaSets to the Pods they own, and NetworkPolicies to the Pods they
apply to.

A **selector** is a query over labels. There are two forms:

- **Equality-based**: `app=frontend`, `tier!=cache` — exact match / negation.
- **Set-based**: `environment in (production, qa)`,
  `tier notin (frontend)`, `partition` (key exists), `!partition` (key
  doesn't exist). Set-based selectors are more expressive and are what
  Deployments/ReplicaSets use internally (`matchLabels` + `matchExpressions`
  in `spec.selector`).

**Annotations** look similar (`metadata.annotations`, also key/value) but
serve a different purpose: they hold **non-identifying** metadata — build
numbers, git commit SHAs, contact emails, tool-specific config
(`kubectl.kubernetes.io/last-applied-configuration` is itself an
annotation `kubectl apply` uses for 3-way merges). Kubernetes never
selects/filters on annotations, and there are no size/character
restrictions the way there are for label values. Rule of thumb: if
something needs to be queried or matched, it's a **label**; if it's just
descriptive data for humans or tools, it's an **annotation**.

On the exam, you'll constantly use `-l`/`--selector` to filter
`kubectl get`/`delete` output, and you'll be asked to add/change labels to
make a Service pick up (or drop) a Pod, or to fix a Deployment whose
`spec.selector` no longer matches its Pod template's labels (a classic
"broken deployment" gotcha — `spec.selector.matchLabels` is **immutable**
after creation).

## kubectl command drills

Get Pods running with labels attached from the start:

```bash
kubectl run web-1 --image=nginx --labels="app=web,tier=frontend"
kubectl run web-2 --image=nginx --labels="app=web,tier=frontend"
kubectl run cache-1 --image=redis --labels="app=cache,tier=backend"
```

Query with selectors (`-l` / `--selector`), both forms:

```bash
kubectl get pods -l app=web
kubectl get pods -l 'tier in (frontend,backend)'
kubectl get pods -l 'app!=cache'
kubectl get pods -l 'app'                       # key exists, any value
kubectl get pods --show-labels
kubectl get pods -l app=web -o name
```

Expose a labeled set as a Service — the Service's `spec.selector` is what
ties it to matching Pods:

```bash
kubectl expose pod web-1 --port=80 --name=web-svc --selector="app=web"
kubectl get endpoints web-svc
```

Mutate labels/annotations on a live object (`set` doesn't cover labels —
use the dedicated `label`/`annotate` subcommands):

```bash
kubectl label pod cache-1 tier=frontend --overwrite
kubectl label pod cache-1 tier-                 # trailing dash removes a label
kubectl annotate pod web-1 build="42" commit="a1b2c3d"
kubectl annotate pod web-1 build-               # removes an annotation
```

Scale/set drills that depend on selectors matching:

```bash
kubectl scale deployment web --replicas=3
kubectl set image deployment/web nginx=nginx:1.25
```

Generate YAML scaffolding with labels, inspect/edit/patch:

```bash
kubectl run web-3 --image=nginx --labels="app=web,tier=frontend" \
  --dry-run=client -o yaml > web-3.yaml
kubectl explain pod.metadata.labels
kubectl edit pod web-3
kubectl patch pod web-3 -p '{"metadata":{"labels":{"canary":"true"}}}'
kubectl patch pod web-3 --type=json \
  -p='[{"op":"remove","path":"/metadata/labels/canary"}]'
```

Clean up:

```bash
kubectl delete pod web-1 web-2 web-3 cache-1
kubectl delete svc web-svc
```

## Hands-on lab (run on your own local minikube)

**Task:** Create three Pods — `pod-a` and `pod-b` with label `app=payments`,
and `pod-c` with label `app=inventory` (all image `nginx`). Then create a
Service named `payments-svc` that selects `app=payments` and exposes port
80. Confirm the Service's Endpoints list exactly `pod-a` and `pod-b`'s IPs
(not `pod-c`). Finally, relabel `pod-c` to `app=payments` and confirm the
Service picks it up automatically without recreating anything.

```bash
kubectl run pod-a --image=nginx --labels="app=payments"
kubectl run pod-b --image=nginx --labels="app=payments"
kubectl run pod-c --image=nginx --labels="app=inventory"
kubectl expose pod pod-a --port=80 --name=payments-svc --selector="app=payments"
```

**Verification command** (should list 2 endpoint IPs, then 3 after the
relabel):

```bash
kubectl get endpoints payments-svc -o jsonpath='{.subsets[*].addresses[*].ip}{"\n"}'
kubectl label pod pod-c app=payments --overwrite
kubectl get endpoints payments-svc -o jsonpath='{.subsets[*].addresses[*].ip}{"\n"}'
```
