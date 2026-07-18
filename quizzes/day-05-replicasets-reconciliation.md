# Day 5 Quiz: ReplicaSets & the Reconciliation Loop

## Tier 1 — Recall

1. What three fields make up a ReplicaSet's `spec`, and what does each one
   control?
2. Describe the reconciliation loop in one sentence: what does the
   controller observe, compare, and do?
3. Why does a bare ReplicaSet ignore updates you make to `spec.template`
   for Pods that already exist?

## Tier 2 — Application

1. Write the exact `kubectl` command to scale a ReplicaSet named `web-rs`
   to 5 replicas.
2. Write the exact `kubectl` command (no YAML file) to check how many Pods
   `web-rs` currently desires versus how many currently exist, using
   `jsonpath`.
3. Write a minimal ReplicaSet YAML (`kind: ReplicaSet`) named `cache-rs`
   with 2 replicas, selector `app=cache`, running image `redis`.

## Tier 3 — Exam-sim (target: 4 minutes)

1. A ReplicaSet `web-rs` has `replicas: 3` and selector `matchLabels:
   {app: web}`. You run `kubectl get pods -l app=web` and see exactly 3
   Pods. You then manually create a fourth Pod named `web-manual` with
   label `app=web` (same selector match) directly via `kubectl run`.
   What happens within seconds, and which Pod is most likely to be
   deleted — the pre-existing one or `web-manual`? Why does the order
   matter here from a troubleshooting perspective?
2. You inherited a ReplicaSet where `kubectl get rs web-rs` shows
   `DESIRED: 3` and `CURRENT: 3`, but `kubectl get pods -l app=web` shows
   only 2 Pods, both `Running`. What field/mismatch would explain this gap,
   and what single command would you run first to diagnose it?

<details>
<summary>Answer key</summary>

**Tier 1**
1. `replicas` (desired Pod count), `selector` (set-based label selector —
   `matchLabels`/`matchExpressions` — identifying which Pods this
   ReplicaSet counts as its own), and `template` (the Pod spec used to
   create new Pods when the controller needs more).
2. It continuously watches how many existing Pods match its `selector`,
   compares that count to `spec.replicas`, and creates or deletes the
   minimum number of Pods needed to close the gap.
3. Because a bare ReplicaSet has no rollout/update mechanism — it only
   acts when the *count* of matching Pods differs from `replicas`. Editing
   `spec.template` only affects Pods created *after* the edit (e.g. to
   replace a deleted one); it never touches Pods that already exist and
   still count toward the desired total. (Rolling updates to existing Pods
   require a Deployment, covered in Week 4.)

**Tier 2**
1. `kubectl scale rs web-rs --replicas=5`
2. `kubectl get rs web-rs -o jsonpath='{.spec.replicas}/{.status.replicas}{"\n"}'`
3. ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: cache-rs
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: cache
     template:
       metadata:
         labels:
           app: cache
       spec:
         containers:
           - name: redis
             image: redis
   ```

**Tier 3**
1. The ReplicaSet now counts 4 matching Pods against a desired count of 3,
   so it deletes exactly one to bring the count back down — there's no
   contractual guarantee of which one (ReplicaSets don't prefer
   "manually created" vs. "controller created" Pods; deletion order is
   effectively unspecified/implementation-defined, though in practice
   newer/not-yet-ready Pods are often preferred for deletion). The order
   matters for troubleshooting because it means `web-manual` could survive
   while one of the "original" three gets deleted instead — so you can't
   assume the Pod you just created is safe, and you can't assume the
   ReplicaSet is doing anything wrong just because a Pod you expected to
   keep is gone; check `kubectl describe rs web-rs` (Events section) to see
   which Pod was actually deleted and why.
2. This points to a `selector`/label mismatch or a stuck/terminating Pod
   that the count hasn't caught up on yet — most commonly, `CURRENT: 3`
   reflects the ReplicaSet's last known count from its own status, but one
   of the "counted" Pods may be `Terminating` (not shown as Running) or a
   third Pod exists but doesn't match the label selector shown in `get
   pods -l app=web` (e.g. it has an extra required label from
   `matchExpressions` that the simple `-l app=web` filter doesn't
   reflect). First diagnostic command: `kubectl describe rs web-rs` — its
   Events section shows recent create/delete actions and its full selector,
   letting you compare against `kubectl get pods --show-labels`.

</details>
