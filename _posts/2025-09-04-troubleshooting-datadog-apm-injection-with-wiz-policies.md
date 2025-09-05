---
title: "Troubleshooting Datadog APM Injection with Wiz Policies: What I Learned About Admission Controllers & AI"
date: 2025-09-04 16:00:00 +0200
categories: [Tech, Reflections]
tags: [Kubernetes, Admission Controllers, Datadog, Wiz, Helm, AI, Troubleshooting]
---

# Troubleshooting Datadog APM Injection with Wiz Policies: What I Learned About Admission Controllers, AI, and Myself

Today a colleague brought me an issue: **Wiz was blocking Pods** in one of our Kubernetes clusters. The YAML looked fine, the Pods were set up with `runAsNonRoot` and `readOnlyRootFilesystem: true`. Everything should have passed. But Wiz kept blocking them.  

We traced it down and realized: **Datadog’s APM Admission Controller** was enabled. And that meant Pods were being mutated before Wiz saw them.

---

## Mutating vs Validating Admission Controllers

This was the first “aha moment.”  

- **Mutating Admission Controllers** (like Datadog’s APM injector) *change* your Pod spec on the way in. They add init containers, volumes, env vars, etc.  
- **Validating Admission Controllers** (like Wiz’s policies) check the *final mutated spec* and block if it doesn’t meet security requirements.

So even though *our YAML* was correct, Wiz wasn’t validating *our YAML*. It was validating the **mutated Pod** — which Datadog had just injected with an init container missing some hardened security settings.

---

## The Actual Issue

Datadog injects an **init container** into Pods when APM auto-instrumentation is enabled. That init container didn’t have `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, etc.  

Wiz saw that and blocked the Pod.  

The fix was to tell Datadog’s Admission Controller how to configure that injected init container. This is done through the `initSecurityContext` setting, which can be passed either as a Helm value or as an environment variable on the Cluster Agent.

At first, the environment variable looked correct in our values file, but it was in the **wrong place**. It was nested under `clusterAgent.admissionController.env` — which the Cluster Agent never reads.  

The right place was **`clusterAgent.env`**, or just use the Helm value:  

```yaml
clusterAgent:
  admissionController:
    autoInstrumentation:
      initSecurityContext: >-
        {"runAsNonRoot":true,"runAsUser":65532,"runAsGroup":65532,
         "allowPrivilegeEscalation":false,"readOnlyRootFilesystem":true,
         "seccompProfile":{"type":"RuntimeDefault"},
         "capabilities":{"drop":["ALL"]}}
```

Docs:  
- [Datadog Admission Controller – Auto Instrumentation Security Context](https://docs.datadoghq.com/tracing/guide/injectors/)  
- [Datadog Helm Chart values.yaml (GitHub)](https://github.com/DataDog/helm-charts/blob/main/charts/datadog/values.yaml)

---

## What I Learned

- The difference between mutating and validating admission controllers isn’t just academic — it explains why Wiz was blocking “valid” Pods.  
- Datadog lets you override the injected init container’s `securityContext`, but the config must be set in the right place.  
- Kubernetes troubleshooting often comes down to understanding **what the cluster sees, not what you wrote**. Tools mutate your workloads. Policies validate the result.  

---

## What Is Troubleshooting, Really?

Troubleshooting feels like three things at once:  
- **Knowledge**: knowing what levers exist, what knobs can be turned.  
- **Persistence**: sifting through docs, GitHub issues, Helm charts.  
- **Luck**: stumbling across the one clue that points you in the right direction.  

That last part — the slog, the time sink — can feel low-value. I don’t think it’s offensive to admit that. It’s the grindy part of the job.  

The higher-value parts are:  
- Framing the problem in the first place.  
- Interpreting whether a solution applies in *your* context.  
- Weighing risks, trade-offs, and next steps.  
- Communicating clearly what happened and why it matters.  

---

## The Role of AI in All This

In this case, my colleague had already set the env var. I read the docs, I saw the env var mentioned, but I couldn’t see the issue.  

I asked ChatGPT. And it spotted it in seconds: the env var was in the wrong place in the values tree.  

Could I have found it myself? Sure — with time, and some luck. But AI compressed that whole slog. It connected the dots across Helm chart structure, docs, and community examples.  

So if AI takes over the slog, what does that leave us with?  

- **Asking better questions**, because the right question matters more than a fast answer.  
- **Storytelling and alignment**, turning lessons into shared knowledge.  
- **Innovation and exploration**, because we’ve got more bandwidth to experiment.  

---

## Closing Thoughts

So here’s the fix in one line:  
👉 Put `DD_ADMISSION_CONTROLLER_AUTO_INSTRUMENTATION_INIT_SECURITY_CONTEXT` under `clusterAgent.env` **or** set `initSecurityContext` under `admissionController.autoInstrumentation`.  

But the bigger story?  
Troubleshooting isn’t going away — it’s changing. The slog can be automated. The value shifts to the creative, interpretive, human side of the work.  

And for me, that means leaning into what I do best: asking questions, telling stories, and connecting the dots.  