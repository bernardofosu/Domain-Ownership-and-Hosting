# 📁 DNS Record Groups & Subdomains — Quick Guide (nakodtech.xyz)

## 🗂️ What are “Record Groups”?

* They’re **just UI folders** to keep things tidy (Web, Email, Staging…).
* They **don’t affect DNS** resolution, priorities, routing, or propagation.
* DNS only cares about each record’s **name → type → value**.

---

## 🚀 New instance / new IP — what to do

**Most common: point a new subdomain to the new IP**

* Add **A** record → `api` → `<NEW_INSTANCE_PUBLIC_IP>` (TTL **300**)
* (Optional) **AAAA** if the instance has IPv6
* Issue a cert on that box:

```bash
sudo certbot --nginx -d api.nakodtech.xyz
```

**Move the root site to the new IP**

* Edit existing **A** for `@` → replace with new IP
* Tip: lower TTL to **300** a few hours before switching

**Simple round‑robin with two servers for one name**

* Add a **second A** for the **same host** with the second IP
* Clients randomly get one IP (no health checks)

**Keep the first site and add a separate site**

* Create a new subdomain (`jenkins`, `blog`, `vault`, …) with its own **A** to the new IP
* No new “group” required — just new records ✅

---

## 🏠 Where the root goes (apex = `@`)

Typical root records:

* `@  A/AAAA` → main site IPs (apex **cannot** be CNAME)
* `@  MX` → mail routing
* `@  TXT` → SPF, ownership verifications
* `_dmarc TXT` → DMARC policy
* `selector._domainkey TXT` → DKIM
* `@  CAA` → (optional) limit which CA can issue certs

Subdomains (examples):

* `www  CNAME  nakodtech.xyz`
* `api  A  <IP-of-API-server>`
* `gis  CNAME  mygis.hostedprovider.com`

> Groups don’t matter — put the `@` (root) records in any group; they still apply to **nakodtech.xyz**.

---

## 🔁 CNAME vs A — rules you must know

* A name can have **only one CNAME** and **no other records** at that same name.
* You **cannot** have multiple CNAMEs for the **same** name (invalid).
* Use **A/AAAA** to point a name directly to IP(s), or **CNAME** to alias to another hostname.
* Apex (`@`) **cannot** be CNAME (unless your DNS host supports ALIAS/ANAME — provider‑specific).
* For LB/HA: use **multiple A/AAAA** records, or point one CNAME to a hostname that itself resolves to many IPs (CDN/LB).

---

## 🌐 If you set these:

```
www    CNAME  nakodtech.xyz
devops CNAME  nakodtech.xyz
```

**Resolution:** `www` and `devops` first alias to `nakodtech.xyz`, then use the apex **A/AAAA** IP(s). All three names hit the **same server**.

**What users see depends on Nginx:**

```nginx
# Root + www share one site
server {
  listen 443 ssl;
  server_name nakodtech.xyz www.nakodtech.xyz;
  # ...
}

# Separate site/app for devops
server {
  listen 443 ssl;
  server_name devops.nakodtech.xyz;
  # serve another root OR proxy_pass to an internal port
}
```

**TLS note:** your certificate must include **every hostname** you serve over HTTPS. Re‑issue to include devops:

```bash
sudo certbot --nginx -d nakodtech.xyz -d www.nakodtech.xyz -d devops.nakodtech.xyz
```

---

## 🧠 Handy tips

* TTL **300s** (5 min) while learning; raise later for stability.
* Avoid accidental duplicates across groups (UI may hide them).
* Use groups purely for **visual organization** (Web, Email, Services, Staging…).

🎯 Bottom line: add/edit **records**, not groups. Groups are labels; DNS resolution depends only on the record **name/type/value**.
