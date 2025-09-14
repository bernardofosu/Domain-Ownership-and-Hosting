# 🚀 From Zero to Live: Domain → DNS → EC2 → TLS → Email (nakodtech.xyz)

A step‑by‑step, emoji‑powered guide you can print or follow next to your terminal. Swap values where you see UPPERCASE.

---

## ✅ Prerequisites

* 🪪 **Accounts:** Spaceship/Namecheap (registrar), AWS (EC2), optional Spacemail (email)
* 🖥️ **Server:** Ubuntu 22.04 EC2 instance
* 🌍 **Domain:** `nakodtech.xyz` (you already bought it 🎉)
* 🔐 **Elastic IP:** allocate one in EC2 (so your A record doesn’t break)

---

## 1) 🛒 Buy the Domain (you did this)

* Registrar checkout → enable **WHOIS privacy** (included)
* Keep **Auto‑renew** off for labs, on for production
* Verify the **ICANN email** you receive within \~15 days

---

## 2) 🌐 Choose Where DNS Lives

You have two good options—pick **one**.

### Option A — Keep DNS at **Spaceship** (fastest)

* 📍 **Library → Domain Manager → `nakodtech.xyz` → Nameservers & DNS → DNS records**
* Leave **Spaceship nameservers** selected

### Option B — Move DNS to **Route 53** (for AWS practice)

1. Route 53 → **Hosted zones → Create public hosted zone** → `nakodtech.xyz`
2. Copy the **4 NS** names Route 53 shows
3. Spaceship → Domain Manager → `nakodtech.xyz` → **Nameservers → Custom** → paste the 4 Route 53 NS → **Save**
4. You’ll now add records in **Route 53**, not Spaceship

> ⏱️ Propagation: usually minutes, sometimes longer. You can still add records immediately.

---

## 3) 📇 Add Website DNS Records

Add these **A/CNAME** records in the DNS home you chose (Spaceship *or* Route 53):

| Type           | Host/Name | Value                                   | TTL  | Purpose                                 |
| -------------- | --------- | --------------------------------------- | ---- | --------------------------------------- |
| **A**          | `@`       | `EC2_ELASTIC_IP` (e.g., `3.90.110.222`) | 300  | Root domain → your server               |
| **CNAME**      | `www`     | `nakodtech.xyz`                         | 300  | `www` follows the root                  |
| **AAAA** (opt) | `@`       | `YOUR_IPV6`                             | 300  | Only if your EC2 has IPv6               |
| **CAA** (opt)  | `@`       | `0 issue "letsencrypt.org"`             | 3600 | Restrict cert issuance to Let’s Encrypt |

> 🔎 The apex `@` **cannot be CNAME**. Use A/AAAA (or Alias in Route 53 if pointing to ALB/CloudFront later).

### Verify DNS

```bash
# Replace with your domain/IP
dig +short NS nakodtech.xyz
dig +short A nakodtech.xyz
dig +short CNAME www.nakodtech.xyz
```

---

## 4) 🔐 Open Security Group Ports

EC2 → **Security Groups** for your instance → **Inbound rules**:

* 22/tcp (SSH) from **your** IP
* 80/tcp (HTTP) from **0.0.0.0/0**
* 443/tcp (HTTPS) from **0.0.0.0/0**

---

## 5) 🧰 Install Nginx + Test Page (on the server)

```bash
sudo apt update && sudo apt install -y nginx
sudo mkdir -p /var/www/nakodtech.xyz/html
echo '<h1>nakodtech.xyz is live 🚀</h1>' | sudo tee /var/www/nakodtech.xyz/html/index.html

# Minimal HTTP vhost
sudo tee /etc/nginx/sites-available/nakodtech.xyz >/dev/null <<'CONF'
server {
  listen 80;
  server_name nakodtech.xyz www.nakodtech.xyz;
  root /var/www/nakodtech.xyz/html;
  index index.html;
}
CONF

sudo ln -s /etc/nginx/sites-available/nakodtech.xyz /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### Quick checks

```bash
curl -I http://nakodtech.xyz
```

---

## 6) 🔒 Issue a Real TLS Certificate (Let’s Encrypt)

```bash
# Install Certbot
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Obtain & install cert (auto configures Nginx + HTTPS redirect)
sudo certbot --nginx -d nakodtech.xyz -d www.nakodtech.xyz

# Test auto‑renew
sudo certbot renew --dry-run
```

### TLS sanity tests

```bash
curl -I https://nakodtech.xyz
openssl s_client -connect nakodtech.xyz:443 -servername nakodtech.xyz </dev/null | head -n 20
```

---

## 7) ✉️ (Optional) Set Up Email with **Spacemail**

1. Library → **Spacemail** → start trial → create mailbox **[info@nakodtech.xyz](mailto:info@nakodtech.xyz)**
2. Spacemail shows required DNS → add in your DNS home:

   * **MX** `@` → `mx1.spacemail…` (prio 10), `mx2…` (prio 20)
   * **TXT (SPF)** `@` → `v=spf1 include:SPACEMAIL-SPF ~all`
   * **TXT (DKIM)** `selector1._domainkey` → long TXT from Spacemail
   * **TXT (DMARC)** `_dmarc` → `v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz`
3. Click **Verify** in Spacemail; then test send/receive

   * IMAP **993/TLS**, SMTP **587 or 465/TLS**

> 💡 Email works even if you have **no website**—it depends on MX/TXT records, not your web host.

---

## 8) 🧪 Final End‑to‑End Test

* Browser: visit **[https://nakodtech.xyz](https://nakodtech.xyz)** → should auto‑redirect from HTTP to HTTPS
* Send an email to **[info@nakodtech.xyz](mailto:info@nakodtech.xyz)** (if configured) and reply back
* Tools:

```bash
# DNS
nslookup nakodtech.xyz
# HTTP/HTTPS
curl -I http://nakodtech.xyz; curl -I https://nakodtech.xyz
# Email DNS (examples)
dig +short MX nakodtech.xyz
dig +short TXT _dmarc.nakodtech.xyz
```

---

## 9) 🔁 Later: Switch to **nakodtech.com**

* Buy `nakodtech.com` → create the **same DNS records** (A/CNAME/TXT/MX)
* Issue a new cert for `.com`:

```bash
sudo certbot --nginx -d nakodtech.com -d www.nakodtech.com
```

* Add a 301 redirect from `.xyz → .com` (optional):

```nginx
# In the nakodtech.xyz server block
return 301 https://nakodtech.com$request_uri;
```

---

## 🐛 Troubleshooting Quickies

* **DNS hasn’t updated** → lower TTL to 300; wait 5–30 min; `dig` the record
* **Site not reachable** → check EC2 SG ports 80/443; `sudo systemctl status nginx`
* **Cert failed** → A/CNAME must be correct; port 80 open; run `sudo certbot --nginx …` again
* **SPF/DKIM/DMARC failing** → copy provider values exactly; only **one** SPF TXT at `@`
* **EC2 email sending** → use provider SMTP (don’t send direct port 25 from EC2)

---

## 📌 Reference: Minimal Record Set (copy/paste)

```
A      @        EC2_ELASTIC_IP         TTL 300
CNAME  www      nakodtech.xyz          TTL 300
MX     @        mx1.spacemail.example  PRI 10 TTL 300
MX     @        mx2.spacemail.example  PRI 20 TTL 300
TXT    @        "v=spf1 include:spacemail._spf.example ~all"   TTL 300
TXT    selector1._domainkey  "<DKIM_LONG_VALUE>"               TTL 300
TXT    _dmarc  "v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz" TTL 300
CAA    @        0 issue "letsencrypt.org"  TTL 3600
```

---

### 🎉 You’re live!

You now own the domain, control DNS, serve a secure website on EC2 with HTTPS, and (optionally) have a working mailbox at `info@nakodtech.xyz`. Save this page and tweak as you grow.
