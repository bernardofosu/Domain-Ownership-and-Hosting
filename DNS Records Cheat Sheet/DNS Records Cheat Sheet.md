# 🌐 DNS Records Cheat Sheet — **nakodtech.xyz**

> You’ve already nailed the big four ✅

* **A** → IPv4
* **AAAA** → IPv6
* **MX** → Mail routing
* **CNAME** → Alias (e.g., `www → nakodtech.xyz`)

---

## 🧭 SRV — *Service Location*

**What it does:** Tells clients which **host\:port** serves a named service.

**Format:** `_service._proto.name  TTL  priority  weight  port  target`

### 🔴 Real examples

**Minecraft server**

```dns
_minecraft._tcp.nakodtech.xyz.  300  0  5  25565  mc.nakodtech.xyz.
mc.nakodtech.xyz.                300  IN A  52.2.240.181
```

**SIP/VoIP (TCP 5060)**

```dns
_sip._tcp.nakodtech.xyz.         300  10 60  5060  sip1.nakodtech.xyz.
_sip._tcp.nakodtech.xyz.         300  20 40  5060  sip2.nakodtech.xyz.
```

* **Lower priority** = more preferred
* **Weight** = load-balance between same-priority targets

**XMPP (Jabber)**

```dns
_xmpp-client._tcp.nakodtech.xyz. 300  5  50 5222 xmpp.nakodtech.xyz.
_xmpp-server._tcp.nakodtech.xyz. 300  5  50 5269 xmpp.nakodtech.xyz.
```

**Active Directory / LDAP**

```dns
_ldap._tcp.nakodtech.xyz.        300  0 100  389  dc1.nakodtech.xyz.
```

> 📝 Browsers **ignore SRV for HTTP/HTTPS**. For web apps, use reverse proxy on 443 or include the port in the URL.

---

## 🧾 TXT — *Free‑form text*

Used for **SPF**, **DKIM**, **DMARC**, and ownership verifications.

### ✉️ SPF (who can send mail for your domain)

```dns
Host: @
Type: TXT
Value: "v=spf1 include:<your-mail-provider-spf> ~all"
TTL: 300
```

> Replace `<your-mail-provider-spf>` with the exact string from **Spacemail** (or your provider). Only **one** SPF TXT per domain—merge includes if needed.

### 🔐 DKIM (email signing key)

```dns
Host: s1._domainkey
Type: TXT
Value: "v=DKIM1; k=rsa; p=<very-long-public-key>"
TTL: 300
```

> Your provider gives you the **selector** (e.g., `s1`, `default`) and the public key.

### 🛡️ DMARC (policy & reporting)

```dns
Host: _dmarc
Type: TXT
Value: "v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz"
TTL: 300
```

Start with `p=none` to monitor → later move to `quarantine` or `reject`.

### ✅ Site / service verification

```dns
Host: @
Type: TXT
Value: "google-site-verification=XYZ..."
```

### 🧪 ACME DNS‑01 (Let’s Encrypt — advanced)

```dns
Host: _acme-challenge
Type: TXT
Value: "<LE-token>"
```

---

## 🧰 Other record types (nice to know)

### 🪪 CAA — *Certificate Authority Authorization*

Limit which CA may issue certs for your domain.

```dns
@  CAA 0 issue     "letsencrypt.org"      ; allow normal certs
@  CAA 0 issuewild "letsencrypt.org"      ; allow wildcard certs (if needed)
@  CAA 0 iodef     "mailto:security@nakodtech.xyz" ; optional reporting
```

> 🔒 Hardening only. Doesn’t affect HTTP/HTTPS traffic.

### 🧭 NS — *Delegate a subdomain*

```dns
dev  NS  ns-123.awsdns.com.
dev  NS  ns-456.awsdns.net.
dev  NS  ns-789.awsdns.org.
dev  NS  ns-012.awsdns.co.uk.
```

> Delegate **subdomains** to other DNS; don’t edit the apex NS unless moving the whole zone.

### 🔁 PTR — *Reverse DNS*

Set by the **IP owner** (e.g., AWS for an Elastic IP), not in your zone. Needed if you self‑host email.

### 🧷 TLSA — *DANE (bind cert to DNS)*

Advanced, requires **DNSSEC**. Common for SMTP. Skip unless you know you need it.

### 🚦 SVCB / HTTPS — *Modern service hints*

Used by some CDNs/providers for ALPN, IP hints, etc. Optional for standard Nginx-on-EC2 sites.

### 🧩 NAPTR

Used with SRV in some SIP/telephony setups.

### 🧿 ALIAS / ANAME

Provider‑specific “CNAME at apex” workaround. Not a standard RR; depends on DNS host.

---

## 🧪 Quick picks for your lab

* **SRV** → great for Minecraft, SIP/VoIP, LDAP, XMPP.
* **TXT** → essential for **SPF/DKIM/DMARC** + ownership checks.
* **TTL** → keep at **300s** (5 min) while learning; raise later for stability.

---

## ✅ Minimal working set for **nakodtech.xyz** (web + email)

```dns
@      A       52.2.240.181            ; website IPv4
www    CNAME   nakodtech.xyz           ; www → root

@      MX      10 <mx1.spacemail.example> ; from Spacemail
@      MX      20 <mx2.spacemail.example>
@      TXT     "v=spf1 include:<spacemail-spf> ~all"
_dmarc TXT     "v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz"
s1._domainkey TXT "v=DKIM1; k=rsa; p=<provider-key>"

@      CAA     0 issue "letsencrypt.org"        ; optional hardening
```

> Replace the placeholder MX/SPF/DKIM values with the exact strings from **Spacemail**.

---

## 💬 Need a tailored SRV example?

Tell me the service (e.g., Minecraft, SIP, XMPP) and the host you want to run it on—I’ll drop a **ready‑to‑paste** SRV + A set for **nakodtech.xyz**.
