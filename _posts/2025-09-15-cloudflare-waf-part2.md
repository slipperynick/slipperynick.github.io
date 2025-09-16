---
title: "Cloudflare WAF (Part 2): Terraform rulesets"
date: 2025-09-15 22:00:00 +0000
categories: [cloud, security, terraform, waf]
tags: [cloudflare, terraform, waf, zero-trust, IaC]
---

# Cloudflare WAF (Part 2): Terraform rulesets

In [Part 1](/posts/cloudflare-waf-part1/) I created some basic Cloudflare WAF rules using the dashboard. It was a good way to test quickly, but for a real project I want **repeatable, version-controlled infrastructure**.  

In this post, I’ll show how I used **Terraform + Cloudflare provider v5** to manage my custom WAF rules as code. Along the way I’ll share the gotchas I ran into (like the mysterious *phase* error) and how I fixed them.

---

## Goal (pew pew)

Implement four simple WAF rules using Terraform:

1. **Block RU traffic** (geo-block)  
2. **Block curl user-agents** (easy to test locally)  
3. **Block `/admin` paths**  
4. **Block queries containing `<script>`** (very simple XSS-like filter)

---

## Terraform Setup

First, configure the provider. I created a **zone-scoped API token** in the Cloudflare dashboard with:

- Zone → WAF: Write  
- (Optional) Zone → DNS: Write  

Then I exported it:

```bash
export CLOUDFLARE_API_TOKEN=your-token
```

My `main.tf` looks like this:

```hcl
terraform {
  required_providers {
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 5"
    }
  }
}

provider "cloudflare" {}

variable "zone_id" {
  default = "" # removed in case :D
}

resource "cloudflare_ruleset" "custom" {
  zone_id = var.zone_id
  name    = "default"                      # keep as-is to avoid replacement
  kind    = "zone"
  phase   = "http_request_firewall_custom"

  rules = [
    {
      action      = "block"
      description = "Block RU (test)"
      expression  = "(ip.geoip.country eq \"RU\")"
      enabled     = true
    },
    {
      action      = "block"
      description = "Block User-Agent curl"
      expression  = "(http.user_agent contains \"curl\")"
      enabled     = true
    },
    {
      action      = "block"
      description = "Block /admin path prefix"
      expression  = "(http.request.uri.path contains \"/admin\")"
      enabled     = true
    },
    {
      action      = "block"
      description = "Block requests with script in query"
      expression  = "(http.request.uri.query contains \"<script>\")"
      enabled     = true
    }
  ]
}
```
---

## Learnings Along the Way

- **Phases matter**  
  Each Cloudflare request goes through stages called *phases*. For custom rules you must use the phase `http_request_firewall_custom`. And crucially, only **one ruleset per phase** is allowed per zone.  
  → If you try to create a new ruleset while one already exists, you’ll hit: exceeded maximum number of zone rulesets for phase http_request_firewall_custom. The fix is to **import the existing ruleset** and manage it in Terraform, not create a duplicate.

- **Importing existing rules**  
I had to run:

```bash
terraform import cloudflare_ruleset.custom \
  "zones/zoneid/ruleid"

(Format is always `zones/<ZONE_ID>/<RULESET_ID>`.)  

- **Keep the name as `default`**  
Cloudflare created the ruleset with name `"default"`. If you try to rename it, Terraform will plan a destructive replacement. I left it as `default` and only edited the rules.

- **Provider v5 syntax**  
Rules must be defined as a **list of objects**:

```hcl
rules = [
  { action = "block", expression = "(...)" }
]
```

Old inline `rules { ... }` blocks won’t work anymore.

---

## Testing the Rules

Here’s how I verified each rule:

Rule 1: **RU geo-block:** connect via VPN exit in RU → see Cloudflare block page.  
- **User-agent curl:**  

    ```bash
    curl -A "curl/8.4.0" -I https://harripersad.org/
    ```

    Got a `403 Forbidden`.


**Rule 2: Path `/admin`:**  

    ```bash
    curl -I https://harripersad.org/admin/settings
    ```

    Got a `403`.

**Rule 3: Script in query:**  

    ```bash
    curl -I "https://harripersad.org/?q=<script>"
    ```
    Got a `403`.

All rules triggered exactly as expected. You can also watch events in the dashboard under **Security → WAF → Events**.

---

## Next Steps

This was a simple set of blocking rules. Perhaps next...

- Managed rulesets (Cloudflare’s OWASP Core Rules). 
- Rate limiting rules (protect login, APIs).  
- Logging into SIEM (Elastic, Datadog).  
- Tuning for false positives.  

But my core idea is clear: **WAF as code, under Terraform, version-controlled.**

