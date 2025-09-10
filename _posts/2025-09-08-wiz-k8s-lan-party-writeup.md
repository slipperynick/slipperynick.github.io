---
title: "Wiz Kubernetes LAN Party – Challenge Write-Up"
date: 2025-09-08 22:00:00 +0000
categories: [CTF, Kubernetes, Security]
tags: [Wiz, Kubernetes, Istio, Kyverno, CTF, Cloud Security, Service Mesh]
---

# Wiz Kubernetes LAN Party – Challenge Write-Up

I spent some time working through the **K8s LAN Party** CTF challenges. They were tricky but really fun, and each one tested a different piece of Kubernetes knowledge. Here’s my write-up for the five challenges: what I did, and my notes.  


---

## Challenge 1: Recon

**Summary:**  
The first step was classic reconnaissance inside a cluster. I used `dnsscan` to sweep the cluster DNS service and enumerate hidden services. This revealed several `.svc.cluster.local` endpoints, one of which led to the next challenge.

**Notes:**
- DNS is often the easiest recon entry point inside a Kubernetes pod.  
- Tools like `dnsscan` can quickly enumerate `svc.cluster.local` services.  
- TODO: Dig deeper into how CoreDNS implements service resolution in multi-tenant clusters.

---

## Challenge 2: Finding Neighbors

**Summary:**  
This was about detecting a **sidecar container** (an Envoy proxy). I used `tcpdump` in my pod to sniff traffic headed to a service and saw HTTP responses stamped with `server: istio-envoy`. The flag was embedded in the request/response stream.

**Notes:**
- Sidecars share the same network namespace as the app container, so `tcpdump` inside the pod captures their traffic.  
- Envoy is Istio’s data plane; seeing `istio-envoy` headers means traffic is being intercepted.  
- TODO: Review more on how Istio transparently injects sidecars and how iptables handles the redirection.

---

## Challenge 3: Data Leakage

**Summary:**  
This focused on an **NFS (EFS) share** exposed inside the cluster. By checking mounts, I found an NFS mount. Using `nfs-ls` and `nfs-cat` (with `uid=0`, `gid=0`, `version=4` in the connection string), I bypassed local file permissions and directly read a file containing the flag.

**Notes:**
- NFS relies heavily on **network reachability**; if you can connect, you can often act as root.  
- `nfs-ls` and `nfs-cat` are great for exploring shares without system mounting.  
- TODO: Review why UID/GID mapping is still weak in NFSv4 and how cloud providers (like AWS EFS) attempt to secure it.

---

## Challenge 4: Bypassing Boundaries

**Summary:**  
This challenge involved Istio **AuthorizationPolicy**. The policy denied GET/POST traffic for pods labeled a certain way. Normally, traffic is routed through the sidecar, which attaches identity. By running `curl` as the `istio` user (UID 1337), I bypassed iptables interception and connected directly to the service, retrieving the flag.

**Notes:**
- Istio redirects traffic with iptables except for traffic from UID 1337.  
- Bypassing Envoy strips away the identity, so some AuthorizationPolicy DENYs don’t trigger.  
- TODO: Revisit how mTLS identity is propagated in Istio and why bypassing the sidecar causes policies to misapply.

---

## Challenge 5: Lateral Movement (Work-in-Progress)

**Summary:**  
This one revolved around **Kyverno**, a policy engine that uses admission webhooks. Without a Kubernetes API token, I couldn’t create pods directly, so I attempted to POST **AdmissionReview** JSONs straight to Kyverno’s `/mutate` and `/validate` endpoints. I saw `allowed: true` but no patch with a flag. Likely the intended trick is to craft a Pod with specific labels/names (or running as root) that would trigger a mutate policy injecting the flag. I didn’t finish this one fully, but I know the direction.

**Notes:**
- Admission webhooks don’t create objects; they only return allow/deny decisions and patches.  
- To simulate them, you must wrap your Pod in an AdmissionReview request.  
- TODO: Explore Kyverno mutate policies in more detail and test label/annotation combinations that could trigger hidden rules.

---

# Final Thoughts

Each challenge focused on a different piece of the Kubernetes ecosystem:

- **Recon** showed how DNS can reveal the landscape.  
- **Finding Neighbors** highlighted the power of network inspection inside pods.  
- **Data Leakage** reminded me of the risks in legacy storage systems like NFS.  
- **Bypassing Boundaries** revealed how service mesh policies can be sidestepped.  
- **Lateral Movement** forced me to think about admission controls and their complexity.  

This CTF reinforced that Kubernetes security isn’t just about pods and RBAC — the glue between components (DNS, service mesh, NFS, admission controllers) is where real risks live.

---

[My K8s LAN Party Certificate](https://k8slanparty.com/certificate/aw3skPGL)