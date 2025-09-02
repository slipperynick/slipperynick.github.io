---
title: "When Scripts Leak Secrets: API Credentials on macOS Endpoints"
date: 2025-09-01 10:00:00 +0000
categories: [Security]
tags: [macOS, Jamf, Intune, Workspace ONE, Endpoint Security, API Security, UEM]
---

# When Scripts Leak Secrets: API Credentials on macOS Endpoints

One of the lesser-discussed security risks in endpoint management is the exposure of API credentials in scripts. I’ve run into this issue personally while working with Jamf, Microsoft Intune, and VMware Workspace ONE — and it deserves some attention.

## Why this matters

Modern UEM (Unified Endpoint Management) platforms — Jamf, Intune, Workspace ONE — depend on API calls to manage macOS endpoints. Admins often automate actions using shell scripts that call these APIs directly.

Here’s the problem: on macOS (and other Unix-like systems), the process table (ps) is visible to all local users. If your script exports or passes API tokens as environment variables, those secrets can appear in the process list.  

That means any user with local access can see valid credentials for your management platform — sometimes with very broad privileges.

## The risk in practice

Here’s a simple scenario:

- A script runs on a Mac, exporting JAMF_API_TOKEN=abcdef123.  
- The script itself might not be visible to the user, but the process table is.  
- A user runs ps aux and sees the token in plain text.  

With that token in hand, an attacker could authenticate directly to your MDM/UEM platform. Depending on the token’s scope, they could:

- Delete or re-enroll devices  
- Push or update policies  
- Exfiltrate inventory or environment details  

This isn’t malware — it’s just basic process monitoring. But it’s enough to harvest exposed tokens in real time.

## How to check your environment

You can verify whether this affects you:

1. Trigger a script on your endpoint that makes API calls.  
2. In another terminal, run:  

   ps aux | grep API  

3. Watch for exported variables or arguments that include credentials.  

If you see your token, it’s exposed.

4. Alternatively, use this script to monitor - [repo here](https://github.com/slipperynick/endpoint-secrets-poc)
```bash
#!/bin/bash

SECRETS=("secret" "token" "HUMAN" "API_TOKEN" "API_KEY" "SECRET_KEY" "ACCESS_TOKEN" "AUTH_TOKEN" "API_SECRET" "PASSWORD" "azureBlobToken" "clientSecret")  # Replace with your desired secrets

while true; do
  for SECRET in "${SECRETS[@]}"; do
    # Get the list of process IDs (PIDs) for processes containing the secret
    PIDS=$(pgrep -f "$SECRET")

    if [ -n "$PIDS" ]; then
      # Secret found in process command-line options
      echo "Secret '$SECRET' found in processes:"
      ps -wwp "$PIDS" -o pid,ppid,command
    fi
  done

  sleep 1  # Adjust the interval between checks as needed
done

```

## Real-world lessons

I’ve seen this play out in a couple different ways:

- **Jamf:** Years ago, I noticed brute-force attempts against the Jamf API. At the time, Jamf didn’t seem to block or even detect these attempts (hopefully that has improved today).  
- **Workspace ONE:** While implementing VMware Workspace ONE’s OS auto-update functionality, I discovered that their updater script (MacOS Update Utility / mUU) exported API credentials in this exact way. I raised the issue — and to VMware’s credit, [they fixed it] (https://github.com/euc-oss/euc-samples/blob/main/UEM-Samples/Utilities%20and%20Tools/macOS/macOS%20Updater%20Utility/README.md#api-credentials).  

These experiences reinforced an important point: vendors don’t always get this right. Sometimes it’s up to admins to spot and report flaws.

## Mitigations

Here are some practical steps:

- Restrict token scope. If you must use them on endpoints, keep them read-only.  
- Review vendor scripts. Don’t assume shipped scripts handle secrets correctly.  
- Treat endpoints as hostile. Anything executed locally can be inspected.  

## Closing thoughts

This issue may look small, but it cuts across endpoint security, API security, and trust in management tooling. If your MDM automation leaks credentials, you’ve effectively handed attackers the keys to your environment. Once an attacker has access to your MDM, all your endpoints are compromised and will possibly lead to lateral movement.

The takeaway is simple: don’t let your scripts be the weakest link. Audit them, protect your tokens, and push vendors to follow better practices.  
