---
title: "How to Use VCF Automation API"
date: 2025-12-29 00:00:00 -0600
categories: [VMware, VCF, VCF Automation]
tags: [VCF, VCF Automation, VMware, Aria Automation, API]
description: Quick start guide to authenticate and make your first calls to the VMware Cloud Foundation (VCF) Automation API, plus a Postman setup.
hidden: true
image:
  path: /assets/img/2025-09-12-how-to-use-vcf-automation-api/Feature.png
---

This post shows a **quick, copy-paste** path to call the **VCF Automation API** (SDDC Manager API) and verify everything with a first request. It also includes a minimal Postman collection setup you can reuse.

> **Note:** Endpoints and payloads may vary by VCF version. Use these as templates and check your product’s API docs for exact paths/fields.

## Prerequisites
- VCF / SDDC Manager reachable (e.g., `https://sddc-manager.example.com`)
- An account with API permissions (often `administrator@vsphere.local`)
- Local tools: `curl` and (optional) `jq` for parsing JSON

---

# 1) Environment variables (optional but handy)

```bash
export SDDC="https://sddc-manager.example.com"
export USER="administrator@vsphere.local"
export PASS="VMware1!"

## 2) Authenticate and get a token
# Request an access token from SDDC Manager
TOKEN=$(curl -sk -X POST "$SDDC/v1/tokens" \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"$USER\",\"password\":\"$PASS\"}" | jq -r '.accessToken')

echo "$TOKEN"


If you see a long string, you’re good. If it’s empty or null, check creds/URL or SSL trust.

## 3) Your first request (sanity check)
# Example: list workload domains
curl -sk -H "Authorization: Bearer $TOKEN" "$SDDC/v1/workload-domains" | jq


Expected result: a JSON array with your domains (empty array if you have none yet).

Other common reads you can try (names can vary by version):

# SDDC Manager info
curl -sk -H "Authorization: Bearer $TOKEN" "$SDDC/v1/sddc-managers" | jq

# Inventory samples
 
 curl -sk -H "Authorization: Bearer $TOKEN" "$SDDC/v1/hosts" | jq
 curl -sk -H "Authorization: Bearer $TOKEN" "$SDDC/v1/clusters" | jq
 ```


4) Typical write operation (pattern)

Many write operations follow: POST with a JSON body → returns a task or request ID → poll task.

Pseudocode with placeholders:

REQUEST='{
  "exampleField": "value",
  "anotherField": "value"
}'

RESP=$(curl -sk -X POST "$SDDC/v1/example-operation" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "$REQUEST")

echo "$RESP" | jq
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Often the response includes: { "id": "task-123", "status": "IN_PROGRESS", ... }
{: .prompt-info }
<!-- markdownlint-restore -->
# Poll task (endpoint varies by version)
TASK_ID="$(echo "$RESP" | jq -r '.id')"
curl -sk -H "Authorization: Bearer $TOKEN" "$SDDC/v1/tasks/$TASK_ID" | jq

5) Postman quick setup (optional)

Create a New Collection → Authorization tab → Type: Bearer Token.

Add a Pre-request Script to refresh the token automatically (adjust URLs/fields to your version):

// Pre-request: fetch token and set it as {{token}}
const sddc = pm.environment.get("sddc");  // e.g. https://sddc-manager.example.com
const user = pm.environment.get("user");
const pass = pm.environment.get("pass");

pm.sendRequest({
  url: `${sddc}/v1/tokens`,
  method: 'POST',
  header: { 'Content-Type': 'application/json' },
  body: { mode: 'raw', raw: JSON.stringify({ username: user, password: pass }) }
}, (err, res) => {
  if (err) { console.log(err); return; }
  const token = res.json().accessToken;
  pm.environment.set("token", token);
  pm.request.headers.upsert({ key: 'Authorization', value: `Bearer ${token}` });
});


In each request, set Authorization → Inherit auth from parent.

Define Postman Environment vars: sddc, user, pass.

6) Bonus: calling Aria Automation (vRA) as part of a workflow

If your automation involves vRA, you’ll typically:

Obtain a token from your vRA/CSP endpoint.

Call the desired service (e.g., catalog items, deployments) with Authorization: Bearer <token>.

Keep the same structure: get token → call endpoint → (optionally) poll task.

Troubleshooting

401/403: invalid/expired token or missing role. Re-auth and verify permissions.

404: endpoint name changed in your version. Check the API reference for your VCF release.

SSL errors: add -k to curl for lab/testing or import the proper CA certs.

Wrap-up

You now have a working pattern:

get token → 2) call read endpoints → 3) trigger an operation → 4) poll task.
Clone these snippets for your day-to-day tasks and keep your env vars per customer/lab.
