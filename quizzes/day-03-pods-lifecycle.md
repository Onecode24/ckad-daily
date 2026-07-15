# Day 3 Quiz: Pods — Anatomy, Lifecycle & Multi-Container Patterns

## Tier 1 — Recall

1. What two things do all containers within the same Pod always share?
2. List the five Pod phases and, in one phrase each, what each one means.
3. What's the difference between a sidecar container and an init container
   in terms of *when* it runs?

## Tier 2 — Application

1. Write the exact `kubectl` command to create a Pod named `debug-box`
   running the `busybox` image that does **not** restart on completion
   (i.e. `restartPolicy: Never`), without writing any YAML.
2. Write the exact `kubectl` command to print only the `status.phase` of a
   Pod named `web` in the `staging` namespace.
3. Write the exact `kubectl` command to view the logs of just the
   `log-shipper` container inside a multi-container Pod named
   `web-with-sidecar`.

## Tier 3 — Exam-sim (target: 4 minutes)

1. A Pod named `app` has two containers, `app` and `sidecar`, sharing an
   `emptyDir` volume at `/data`. `kubectl get pods` shows `app` as
   `1/2 Running`. Write the single command you'd run first to find out
   *which* container is not ready and why, then state which field in the
   output tells you the failing container's name.
2. Write a complete Pod manifest (YAML) for a Pod named `sidecar-demo` with
   two containers — `main` (image `nginx`) and `helper` (image `busybox`,
   command `sh -c "sleep 3600"`) — both mounting a shared `emptyDir` volume
   named `shared-data` at `/shared`. No extra fields beyond what's needed.

<details>
<summary>Answer key</summary>

**Tier 1**
1. The Pod's network namespace (same IP address, containers reach each
   other via `localhost`) and, if defined, any shared volumes.
2. `Pending` — accepted but not yet scheduled/running; `Running` — bound to
   a node with at least one container running; `Succeeded` — all
   containers exited 0; `Failed` — a container exited non-zero and wasn't
   restarted; `Unknown` — node/kubelet unreachable, status can't be
   determined.
3. An init container runs to completion *before* any main container
   starts (sequentially, one after another if there are several). A
   sidecar runs *alongside* the main container(s) for the Pod's entire
   lifetime, starting around the same time.

**Tier 2**
1. `kubectl run debug-box --image=busybox --restart=Never -- sleep 3600`
   (any valid non-exiting command works; `--restart=Never` is the key
   flag).
2. `kubectl get pod web -n staging -o jsonpath='{.status.phase}{"\n"}'`
3. `kubectl logs web-with-sidecar -c log-shipper`

**Tier 3**
1. `kubectl describe pod app` — the `Containers:` section lists each
   container with its own `State`/`Ready`/`Restart Count`, and the `Events:`
   section at the bottom usually shows why (e.g. `CrashLoopBackOff`,
   readiness probe failure). The failing container's name appears as the
   heading of its block under `Containers:` (or via
   `kubectl get pod app -o jsonpath='{.status.containerStatuses[?(@.ready==false)].name}'`).
2. ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: sidecar-demo
   spec:
     containers:
       - name: main
         image: nginx
         volumeMounts:
           - name: shared-data
             mountPath: /shared
       - name: helper
         image: busybox
         command: ["sh", "-c", "sleep 3600"]
         volumeMounts:
           - name: shared-data
             mountPath: /shared
     volumes:
       - name: shared-data
         emptyDir: {}
   ```

</details>
