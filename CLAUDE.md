# CKAD Daily Trainer — Protocol

This repo is an autonomous daily study-content generator for a learner preparing
for the Certified Kubernetes Application Developer (CKAD) exam. A scheduled job
runs once a day, reads `progress.md` to find the current day, generates that
day's lesson + quiz + artifact, and opens a PR. The learner reviews the PR,
does the hands-on lab **on their own local minikube cluster** (labs never run
in this cloud job — there is no cluster here), self-grades the quiz, and
merges the PR to unlock the next day.

## Daily content structure

Each day produces three files:

1. `lessons/day-NN-topic.md`
2. `quizzes/day-NN-topic.md`
3. `artifacts/day-NN-topic.html` (a single combined, styled HTML page built
   from the two files above, for easy reading/printing)

### 1. Concept brief

A short (~200-400 word) plain-English explanation of the day's topic: what it
is, why it exists, when you'd reach for it on the exam, and how it relates to
neighboring topics already covered. No fluff, no marketing language.

### 2. kubectl command drills

A set of imperative-first kubectl commands demonstrating the topic, in this
progression where applicable:

- `kubectl run` / `kubectl create` — get something running fast
- `kubectl expose` — expose it as a Service
- `kubectl set` — mutate an existing object (image, env, resources...)
- `kubectl scale` — change replica counts
- `--dry-run=client -o yaml` — generate YAML scaffolding without applying it
- `kubectl edit` — live-edit an object
- `kubectl patch` — targeted strategic/JSON patch changes

Show the exact command syntax with realistic flags/values, not just
descriptions. Prefer the fastest exam-legal path (imperative commands over
hand-written YAML) since CKAD is timed.

### 3. Hands-on lab task

One concrete task to perform on the learner's **own local minikube** cluster
(never executed by the automation — this is an instruction to the human),
plus a **verification command** whose output proves success (e.g.
`kubectl get pods -l app=foo -o jsonpath='{.items[*].status.phase}'`).

### 4. Three-tier quiz

- **Tier 1 — Recall** (2-3 questions): definitions, "what does X do", "which
  field controls Y".
- **Tier 2 — Application** (2-3 questions): "write the exact kubectl command
  or YAML that accomplishes X." Answers must be exact, runnable commands/YAML.
- **Tier 3 — Exam-sim** (1-2 questions): timed (state a target time, e.g.
  "3 minutes"), tricky — edge cases, common gotchas, multi-step scenarios
  resembling real CKAD exam questions.

Answers go in a collapsed `<details><summary>Answer key</summary>...</details>`
block at the bottom of the quiz file so the learner can self-test first.

## Progress tracking (`progress.md`)

- `Current day: N` — the day number the next automated run should generate.
  Only advances when a day's PR is **merged** (the bootstrap/day PR itself
  bumps this number, but the bump only takes effect for the next run once
  merged into `main`).
- **Weak-area queue** — topics the learner flags as needing more review;
  future days should pull from this queue before moving strictly forward
  through the syllabus when it's non-empty.
- **Score log** — a table with columns `Date | Day | Topic | T1 | T2 | T3`,
  one row per day, self-graded by the learner (pass/fail or score per tier).

## Automation contract

- One PR per day, branch named `day-N-<short-topic-slug>`.
- The automation never merges its own PRs — the learner merges after doing
  the lab and quiz.
- If a day's PR is still open/unmerged, the next scheduled run does **not**
  generate a new day — it just reminds the learner to merge.
- The automation does not have access to a Kubernetes cluster; all `kubectl`
  commands in lessons/quizzes are for the learner to run locally.
