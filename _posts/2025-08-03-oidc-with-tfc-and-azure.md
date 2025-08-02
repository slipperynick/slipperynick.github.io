---
title: "Welcome"
date: 2025-08-03 10:00:00 +0000
categories: [General]
tags: [OIDC, Terraform Cloud, Azure, JWT, AzureRM, Federated Identity, App Registration, Token Exchange, CI/CD, HashiCorp, DevSecOps]
---

Learning OIDC with Terraform Cloud and Azure: A Journey of Integration, Errors, and Resolution

Today I embarked on a deep dive into setting up OIDC (OpenID Connect) between Terraform Cloud and Azure, aiming to eliminate secrets and adopt secure, token-based authentication for infrastructure deployments. Here’s a structured recap of what I learned, the hurdles I faced, and how I overcame them.

⸻

1. Understanding OIDC and JWT

OIDC is an identity layer built on top of OAuth 2.0. It allows secure token-based authentication between parties (in our case, Terraform Cloud and Azure).

I learned that:
	•	OIDC works by issuing JWTs (JSON Web Tokens), which are signed and verifiable.
	•	A JWT contains three parts: Header, Payload, and Signature.
	•	The audience (aud) field specifies the intended recipient of the token — for Azure, this is api://AzureADTokenExchange.
	•	Azure validates these tokens against a federated identity credential linked to an App Registration.

2. The Big Picture

The goal is to have Terraform Cloud act as a workload identity that assumes a role in Azure, using short-lived OIDC tokens. This means:
	•	No long-lived credentials.
	•	No Azure CLI dependency.
	•	Cleaner, safer pipelines.

3. Step-by-Step Setup

Following the official HashiCorp documentation, here’s what I did:

a. App Registration in Azure
	•	Created a new App Registration
	•	Copied the Client ID, Tenant ID, and Subscription ID

b. Federated Identity Credential
	•	Created a new federated identity credential under the app
	•	Set Issuer to Terraform Cloud’s OIDC endpoint: https://app.terraform.io
	•	Set Subject as:
organization:<org>:project:<project>:workspace:<workspace>:run_phase:plan
	•	Set Audience to api://AzureADTokenExchange

c. IAM Role Assignment
	•	Assigned the Contributor role to the App Registration
	•	Chose “User, Group, or Service Principal” as the access type

d. Terraform Cloud Configuration
	•	Enabled Dynamic Provider Credentials in the workspace (if available)
	•	Removed any previous Azure service principals from the environment variables

4. Common Errors and Fixes

❌ Error: could not configure AzureCli Authorizer: az not found

Cause: Terraform Cloud was trying to use Azure CLI locally.

Fix: Added use_oidc = true in the provider block:

provider "azurerm" {
  features {}
  use_oidc = true
}


⸻

❌ Error: AADSTS700213 - No matching federated identity record

Cause: Subject, Audience, or Issuer did not match.

Fix:
	•	Double-checked subject string format
	•	Verified audience as api://AzureADTokenExchange
	•	Confirmed issuer matches the Terraform Cloud OIDC endpoint

5. What I Learned
	•	Azure issues short-lived access tokens, usually valid for 60 minutes, based on the OIDC JWT.
	•	Azure uses a public OIDC Discovery endpoint to fetch the public keys to validate the signature.
	•	Terraform Cloud sends the signed token, and Azure verifies its authenticity without needing Terraform’s private key — it’s a one-way trust relationship.

6. Security Note

While tenant_id and subscription_id are not secrets, they are sensitive and should be stored in GitHub secrets or environment variables — never committed to source control.

7. Final Thoughts

This was a rewarding journey through cloud identity federation. I learned not only how to integrate Terraform Cloud with Azure via OIDC, but also how the modern cloud handles identity, tokens, and secure access without traditional secrets.

Despite the confusing error messages and initial setup complexity, the end result is a clean, secure, and elegant authentication flow.

⸻

✅ Next Up: Use this setup to provision real infrastructure like resource groups, networks, or even Kubernetes clusters — all without long-lived credentials.

⸻

Keywords: OIDC, Terraform Cloud, Azure, JWT, AzureRM, Federated Identity, App Registration, Token Exchange, CI/CD, HashiCorp, DevSecOps
