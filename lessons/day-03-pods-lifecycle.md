# Day 3: Pods — Anatomy, Lifecycle & Multi-Container Patterns

## Concept brief

Days 1-2 covered the cluster and the tool. Today: the **Pod**, the smallest
deployable unit in Kubernetes and the object almost every other resource
(Deployment, Job, DaemonSet...) exists to manage a template of.

A Pod is one or more containers that share a **network namespace** (same IP,
`localhost` between containers) and can share **storage** (via volumes).
Containers in a Pod are always scheduled together, on the same node, and
live/die together — you never split containers of one Pod across nodes.

**Lifecycle** — a Pod moves through phases you'll read constantly in
`kubectl get pods`:

- `Pending` — accepted by the API server, not yet scheduled/running (image
  pulling, waiting for a node, etc.)
- `Running` — bound to a node, at least one container running
- `Succeeded` — all containers terminated with exit code 0 (normal for a
  one-shot Job pod, not a long-running server)
- `Failed` — at least one container terminated non-zero and wasn't restarted
- `Unknown` — node unreachable, kubelet can't report status

Within `Running`, each container also has its own state: `Waiting`,
`Running`, or `Terminated` — visible under `status.containerStatuses` in
`-o yaml`, and the reason CKAD questions often ask you to `describe` a pod
rather than trust the one-word phase.

**Multi-container pods** exist because "one process per container" doesn't
mean "one container per Pod." Common patterns:

- **Sidecar** — a helper container running alongside the main one for the
  Pod's whole life (e.g. a log shipper reading from a shared volume).
- **Init container** (previewed today, deep-dived Day 9) — runs to
  completion *before* the main containers start; used for setup steps.
- **Ambassador/adapter** — a container that proxies or reshapes traffic/data
  between the main container and the outside world.

On the exam, multi-container pods show up as "add a sidecar that does X" —
you'll edit `spec.containers` to add a second entry, often with a shared
`emptyDir` volume mounted into both.

## kubectl command drills

Get something running fast, then inspect it:

```bash
kubectl run mypod --image=nginx
kubectl get pods
kubectl get pod mypod -o wide
kubectl describe pod mypod
kubectl get pod mypod -o jsonpath='{.status.phase}{"\n"}'
kubectl get pod mypod -o jsonpath='{.status.containerStatuses[0].state}{"\n"}'
```

Scaffold a Pod's YAML instead of writing it from scratch:

```bash
kubectl run mypod --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl run mypod --image=nginx --restart=Never --dry-run=client -o yaml
kubectl explain pod.spec.containers --recursive | less
```

Mutate an existing Pod's neighbors (Pods themselves are mostly immutable —
you can't `kubectl set image` on most Pod fields once running, so `edit`/
`patch` are for the few mutable fields, or you delete+recreate):

```bash
kubectl edit pod mypod
kubectl patch pod mypod -p '{"metadata":{"labels":{"tier":"frontend"}}}'
kubectl label pod mypod tier=frontend
kubectl annotate pod mypod owner=team-a
```

Multi-container pod — add a sidecar via YAML (generate the base with
`--dry-run=client`, then hand-edit to add the second container and shared
volume):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-sidecar
  labels:
    app: web
spec:
  containers:
    - name: web
      image: nginx
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
    - name: log-shipper
      image: busybox
      command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
  volumes:
    - name: logs
      emptyDir: {}
```

```bash
kubectl apply -f pod.yaml
kubectl get pod web-with-sidecar -o jsonpath='{.spec.containers[*].name}{"\n"}'
kubectl logs web-with-sidecar -c log-shipper
kubectl exec -it web-with-sidecar -c web -- sh
```

Clean up:

```bash
kubectl delete pod mypod web-with-sidecar
```

## Hands-on lab (run on your own local minikube)

**Task:** Create a Pod named `multi-demo` with two containers sharing an
`emptyDir` volume mounted at `/data` in both: container `writer`
(`busybox`, command that appends a timestamp to `/data/log.txt` every 2s in
a loop) and container `reader` (`busybox`, command that tails
`/data/log.txt`). Apply it, then confirm the reader sees lines the writer
produced.

```bash
kubectl apply -f pod.yaml
kubectl wait --for=condition=Ready pod/multi-demo --timeout=60s
```

**Verification command** (should print at least one line containing a
timestamp, proving cross-container shared storage works):

```bash
kubectl logs multi-demo -c reader --tail=5
kubectl get pod multi-demo -o jsonpath='{.status.containerStatuses[*].name}{"\n"}'
```
