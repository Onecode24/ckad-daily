# Day 2 Quiz: kubectl Basics — Contexts, Namespaces, Output Formats & `explain`

## Tier 1 — Recall

1. What three things does a kubeconfig "context" bundle together?
2. Name two resource types that are cluster-scoped (not namespaced), and
   one command you'd use to check whether a given resource is namespaced.
3. What problem does `kubectl explain` solve during the actual (offline)
   exam?

## Tier 2 — Application

1. Write the exact `kubectl` command to switch the *current context's*
   default namespace to `prod` (without changing which context is active).
2. Write the exact `kubectl` command to print just the `status.phase` of
   every pod in the `dev` namespace, one per line, using `jsonpath`.
3. Write the exact `kubectl explain` command you'd run to look up the
   valid fields under a Deployment's `spec.strategy`, showing all nested
   fields at once (not one level at a time).

## Tier 3 — Exam-sim (target: 3 minutes)

1. You're told: "In the `k8s-prod` context, find the container image used
   by the Deployment named `payments` in the `billing` namespace, without
   changing your current context." Write the single command that does
   this using `--context` and `-n` flags directly (no `use-context`
   switch), with `-o jsonpath` to extract only the image string.
2. You run `kubectl get pod mypod` and get `Error from server (NotFound):
   pods "mypod" not found`, but `kubectl get pod mypod -n staging` finds
   it immediately. Explain in one sentence what happened, and name the
   single command you'd run first to prevent this mistake for the rest of
   the session.

<details>
<summary>Answer key</summary>

**Tier 1**
1. A cluster (API server address/CA), a user (credentials), and a default
   namespace.
2. Cluster-scoped examples: `Node`, `PersistentVolume`, `ClusterRole`,
   `Namespace` itself (any two of these). Check with:
   `kubectl api-resources -o wide` (the `NAMESPACED` column shows
   `true`/`false`), or `kubectl api-resources --namespaced=false`.
3. There's no internet access during the real exam, so you can't look up
   docs or Stack Overflow. `kubectl explain <resource>.<field>` is the
   only in-cluster reference for exact field names, types, and valid
   values when writing YAML from memory.

**Tier 2**
1. `kubectl config set-context --current --namespace=prod`
2. `kubectl get pods -n dev -o jsonpath='{range .items[*]}{.status.phase}{"\n"}{end}'`
3. `kubectl explain deployment.spec.strategy --recursive`

**Tier 3**
1. `kubectl get deployment payments -n billing --context=k8s-prod -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'`
2. The current context's default namespace was not `staging` (likely
   `default`), so the first `get pod` looked in the wrong namespace and
   found nothing — it wasn't actually missing. Run
   `kubectl config set-context --current --namespace=staging` first
   (or always pass `-n <namespace>` explicitly) to avoid repeating this.

</details>
