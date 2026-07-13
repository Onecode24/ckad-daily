# CKAD 60-Day Syllabus

Target exam date: **2026-09-13**. One topic per day, ordered by CKAD exam
domain weight so heavier domains get more repetition and appear earlier and
more often:

| Domain | Weight |
|---|---|
| Application Environment, Configuration & Security | ~25% |
| Application Design & Build | ~20% |
| Application Deployment | ~20% |
| Services & Networking | ~20% |
| Application Observability & Maintenance | ~15% |

Each day = one `lessons/day-NN-*.md` + `quizzes/day-NN-*.md` + a combined
`artifacts/day-NN-*.html`. Days pull sequentially from this list unless the
**Weak-area queue** in `progress.md` is non-empty, in which case the next
day revisits a queued topic instead of moving forward.

## Week 1 — Foundations (Days 1-7)

1. Kubernetes architecture overview & cluster components
2. kubectl basics: contexts, namespaces, `-o wide/yaml/json`, `explain`
3. Pods: anatomy, lifecycle, multi-container pods
4. Labels, selectors, annotations
5. ReplicaSets & the reconciliation loop
6. Namespaces & resource scoping
7. Week 1 review + mini mock quiz

## Week 2 — Foundations continued (Days 8-14)

8. API resources & API versions (`kubectl api-resources`, `api-versions`)
9. Init containers
10. Sidecar containers & multi-container patterns (ambassador, adapter)
11. Container images: pull policy, tags, private registries
12. Resource requests & limits (CPU/memory)
13. Pod design: restart policies, `kubectl run` deep dive
14. Week 2 review + mini mock quiz

## Week 3 — Configuration, Secrets & Security (Days 15-21)

15. ConfigMaps: create from literal/file/env, consume as env vars
16. ConfigMaps as mounted volumes + reload behavior
17. Secrets: generic/tls/docker-registry types, encoding vs encryption
18. Consuming Secrets as env vars and volumes
19. SecurityContext: runAsUser, runAsNonRoot, fsGroup, capabilities
20. ServiceAccounts & RBAC basics (Roles, RoleBindings)
21. Week 3 review + mock: config & security scenarios

## Week 4 — Deployment Objects (Days 22-28)

22. Deployments: create, rollout, rollout history
23. Rolling updates & rollback strategies (maxSurge/maxUnavailable)
24. Deployment strategies: blue/green & canary (manual patterns)
25. Jobs: completions, parallelism, backoffLimit
26. CronJobs: schedule syntax, concurrencyPolicy, history limits
27. DaemonSets & StatefulSets (conceptual + basic manifests)
28. Week 4 review + mock: deployment scenarios

## Week 5 — Scaling, Helm & Kustomize (Days 29-35)

29. Manual scaling & HorizontalPodAutoscaler basics
30. Helm fundamentals: repos, install/upgrade/rollback, values
31. Writing/customizing a Helm chart (values.yaml overrides)
32. Kustomize basics: bases, overlays, patches
33. Kustomize: configMapGenerator/secretGenerator, images transformer
34. Combining Helm + Kustomize in a workflow
35. Week 5 review + mock: scaling & packaging scenarios

## Week 6 — Probes & Debugging (Days 36-42)

36. Liveness probes (exec/http/tcp)
37. Readiness probes & startup probes
38. Probe tuning: initialDelaySeconds, periodSeconds, failureThreshold
39. Debugging pods: `kubectl logs`, `describe`, `exec`, ephemeral containers
40. Debugging: `kubectl debug`, crash loops, OOMKilled, image pull errors
41. Troubleshooting a broken Deployment end-to-end (multi-fault scenario)
42. Week 6 review + mock: observability & debugging scenarios

## Week 7 — Services & Networking (Days 43-49)

43. Services: ClusterIP, NodePort, ExternalName
44. Services: LoadBalancer, headless services, endpoints
45. Ingress: rules, paths, TLS basics
46. Ingress controllers & annotations (nginx-style)
47. NetworkPolicy: default-deny, ingress/egress rules
48. NetworkPolicy: namespace & pod selectors, ports
49. Week 7 review + mock: services & networking scenarios

## Week 8 — Storage & Mock Exams (Days 50-60)

50. Volumes: emptyDir, hostPath basics
51. PersistentVolumes & PersistentVolumeClaims
52. StorageClasses & dynamic provisioning
53. Volume access modes & reclaim policies
54. StatefulSets + PVCs (volumeClaimTemplates)
55. Multi-topic mixed review (config + deployment)
56. Multi-topic mixed review (services + observability)
57. Full-length mock exam #1 (timed, mixed domains)
58. Full-length mock exam #2 (timed, mixed domains)
59. Weak-area deep dive (pulled from `progress.md` queue)
60. Final review + exam-day checklist & logistics

## Notes for the generator

- If `progress.md`'s Weak-area queue is non-empty on a given day, generate
  content for the **oldest queued weak area** instead of the next syllabus
  day, then remove it from the queue. The day number still increments.
- Review/mock days (7, 14, 21, 28, 35, 42, 49, 57, 58) should synthesize
  multiple prior topics into scenario-style questions rather than
  introducing new material.
