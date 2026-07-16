# Day 4 Quiz: Labels, Selectors & Annotations

## Tier 1 — Recall

1. What is the key difference in purpose between a label and an annotation?
2. Name the two forms of label selectors and give one example of each.
3. Which field on a Service ties it to a set of Pods, and how does
   Kubernetes decide which Pods are included?

## Tier 2 — Application

1. Write the exact `kubectl` command to create a Pod named `api-1` with
   image `nginx` and labels `app=api` and `tier=backend` set at creation
   time (no YAML file).
2. Write the exact `kubectl` command to list all Pods whose `tier` label is
   either `frontend` or `backend`, using a set-based selector.
3. Write the exact `kubectl` command to remove the `tier` label from a Pod
   named `api-1` without touching its other labels.

## Tier 3 — Exam-sim (target: 3 minutes)

1. A Deployment named `web` has `spec.selector.matchLabels: {app: web}` and
   its Pod template has label `app: web`. Someone ran
   `kubectl label pod web-abc123 app=web-old --overwrite` directly on one
   of the running Pods. Will the Deployment's ReplicaSet still count that
   Pod toward its desired replica count? What will happen shortly after,
   and what single command would you run to see it happen?
2. You need to edit a Deployment's `spec.selector.matchLabels` to add a new
   key. You run `kubectl edit deployment web` and add
   `version: v2` under `matchLabels`, save, and get an error. Why does this
   fail, and what would you actually have to do to change which Pods a
   Deployment selects?

<details>
<summary>Answer key</summary>

**Tier 1**
1. Labels are identifying metadata meant to be *selected/queried on*
   (Services, ReplicaSets, NetworkPolicies match against them); annotations
   are non-identifying metadata for humans/tools (build info, commit SHAs,
   config) that Kubernetes never filters or selects by.
2. Equality-based (e.g. `app=web`, `tier!=cache`) and set-based (e.g.
   `environment in (production, qa)`, `tier notin (frontend)`, or bare
   `partition` for key-exists).
3. `spec.selector` on the Service. Kubernetes includes any Pod whose
   `metadata.labels` match every key/value in that selector — matching is
   dynamic and re-evaluated continuously, so Pods created later are picked
   up automatically.

**Tier 2**
1. `kubectl run api-1 --image=nginx --labels="app=api,tier=backend"`
2. `kubectl get pods -l 'tier in (frontend,backend)'`
3. `kubectl label pod api-1 tier-`

**Tier 3**
1. No — the ReplicaSet's `spec.selector.matchLabels` still requires
   `app=web`, and this Pod's label no longer matches, so the ReplicaSet no
   longer counts it as one of "its" Pods even though it still exists and is
   running. Because the ReplicaSet is now short one matching Pod versus its
   desired replica count, it will create a brand-new replacement Pod with
   the correct `app=web` label shortly after (the orphaned Pod with
   `app=web-old` is left running, unmanaged, until manually deleted).
   Command to watch it happen: `kubectl get pods -l app=web -w` (or
   `kubectl get pods --show-labels -w`) to see the new Pod appear while the
   relabeled one stays orphaned.
2. `spec.selector.matchLabels` (and `matchExpressions`) on a Deployment is
   **immutable** after creation — the API server rejects updates to it, so
   `kubectl edit` will save-and-error (or refuse) on that field. To
   actually change which Pods a Deployment selects, you must delete and
   recreate the Deployment with the new selector (optionally with
   `--cascade=orphan` to keep existing Pods/ReplicaSet alive during the
   swap), since there's no in-place mutation path for that field.

</details>
