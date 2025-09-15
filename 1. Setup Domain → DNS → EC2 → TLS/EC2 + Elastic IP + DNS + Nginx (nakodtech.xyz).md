# 🧭 Quick Start: EC2 + Elastic IP + DNS + Nginx (nakodtech.xyz)

Follow these bite‑size steps to get a live site. Swap placeholders like **YOUR\_ELASTIC\_IP**.

A step‑by‑step, emoji‑powered guide you can print or follow next to your terminal. Swap values where you see UPPERCASE.

[Alt](./dns-records.png)

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


## 1) 🖥️ Create the EC2 VM

1. AWS Console → **EC2 → Launch instance**
2. **AMI:** Ubuntu 22.04 LTS
3. **Type:** t3.micro (lab friendly)
4. **Key pair:** create/select one
5. **Security Group (Inbound):**

   * 22/tcp (SSH) → *your IP*
   * 80/tcp (HTTP) → 0.0.0.0/0
   * 443/tcp (HTTPS) → 0.0.0.0/0
6. **Launch** ✅

> Tip: Keep this instance in the region you plan to use for the Elastic IP.

---

## 2) 📌 Allocate & Attach an Elastic IP

EC2 → **Network & Security → Elastic IPs** → **Allocate** → then **Associate** to your instance (or its primary network interface).

Why? A normal public IP can change on stop/start; an **Elastic IP** stays the same.

---

## 3) 🌐 Point the Domain (DNS)

Manage DNS at **Spaceship** *or* **Route 53** (you don’t install DNS on the VM).

Add these records:

| Host  | Type  | Value                 | TTL     |
| ----- | ----- | --------------------- | ------- |
| `@`   | A     | **YOUR\_ELASTIC\_IP** | **300** |
| `www` | CNAME | **nakodtech.xyz**     | 300     |

> If using Route 53: create a **Hosted Zone** first, add the same records, then change Nameservers at Spaceship to Route 53’s 4 NS.

**What is TTL?** Time To Live (seconds) = how long resolvers cache a record. Use **300** (5 min) while testing; raise later for stability.

---

## 4) 🍰 Install Nginx + a Simple Page (on the VM)

SSH to the instance, then run:

```bash
# Install Nginx
sudo apt update && sudo apt install -y nginx

# Site files
sudo mkdir -p /var/www/nakodtech.xyz/html
sudo tee /var/www/nakodtech.xyz/html/index.html >/dev/null <<'HTML'
<!doctype html>
<html><head><meta charset="utf-8"><title>nakodtech.xyz</title></head>
<body style="font-family:sans-serif;text-align:center;margin-top:10%">
<h1>It works! 🚀</h1>
<p>Hello from nakodtech.xyz on Nginx.</p>
</body></html>
HTML

# Minimal server block
sudo tee /etc/nginx/sites-available/nakodtech.xyz >/dev/null <<'CONF'
server {
  listen 80;
  server_name nakodtech.xyz www.nakodtech.xyz;
  root /var/www/nakodtech.xyz/html;
  index index.html;
}
CONF

# Enable & reload
sudo ln -s /etc/nginx/sites-available/nakodtech.xyz /etc/nginx/sites-enabled/ || true
sudo nginx -t && sudo systemctl reload nginx
```

Open **port 80/443** in the EC2 security group if you haven’t already.

---

## 5) ✅ Test

```bash
# DNS should resolve to your EIP
dig +short A nakodtech.xyz

# Browse to the site
# http://nakodtech.xyz  → you should see "It works! 🚀"
```

---

## 6) 🔒 Optional: Free HTTPS (Let’s Encrypt)

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx -d nakodtech.xyz -d www.nakodtech.xyz
sudo certbot renew --dry-run
```
### Enter your email
- You don’t have to enter an email, but it’s strongly recommended.
- If you enter a real, reachable email, Let’s Encrypt can send expiry / problem alerts. It’s not published on the certificate.
- If you skip it (press Enter), things will still work, but you won’t get renewal warnings.

### Type Y to agree.
When it asks about redirection, choose “Redirect” so HTTP → HTTPS automatically.

Quick checklist so it succeeds:
- Port 80 must be open (the nginx plugin uses the HTTP challenge).
- DNS A for nakodtech.xyz and CNAME www → nakodtech.xyz already point to your server (you’re good).
- If you use ufw, allow: sudo ufw allow 'Nginx Full'.

After it finishes you should see certs here:
```sh
/etc/letsencrypt/live/nakodtech.xyz/fullchain.pem
/etc/letsencrypt/live/nakodtech.xyz/privkey.pem

curl -I https://nakodtech.xyz
```

### That one’s just a newsletter opt-in for the EFF (the nonprofit behind Certbot).
It doesn’t affect your certificate at all.
- Y = share your email with EFF for news/campaigns.
- N = don’t share (recommended if you want a quiet inbox).


Certbot + the Nginx plugin did the heavy lifting for you:

🔐 Wired TLS into Nginx and wrote the server blocks.

📁 Put certs here:

/etc/letsencrypt/live/nakodtech.xyz/fullchain.pem

/etc/letsencrypt/live/nakodtech.xyz/privkey.pem

### 🔁 Set up auto-renew via a systemd timer (from the snap).
Quick health checks
```sh
# Is HTTPS working?
curl -I https://nakodtech.xyz

# Dry-run a renewal
sudo certbot renew --dry-run

# See the timer (snap install)
systemctl list-timers | grep certbot
systemctl status snap.certbot.renew.timer
```

Keep it renewing smoothly
- Leave port 80 open (HTTP-01 challenge uses it).
- DNS A and www CNAME must keep pointing to this server.
- Server time should be correct (Ubuntu handles this via systemd-timesyncd).

### Where to put your site files
Serve content from: you copy any of the HTML Templates
```sh
/var/www/nakodtech.xyz/html
```

After changes:
```sh
sudo nginx -t && sudo systemctl reload nginx
```


### View sites-available
```sh
cat /etc/nginx/sites-available/nakodtech.xyz
```
```sh
server {
  server_name nakodtech.xyz www.nakodtech.xyz;
  root /var/www/nakodtech.xyz/html;
  index index.html;

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nakodtech.xyz/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nakodtech.xyz/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.nakodtech.xyz) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = nakodtech.xyz) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  server_name nakodtech.xyz www.nakodtech.xyz;
    return 404; # managed by Certbot
}
```

### 🔐 CAA record (Certification Authority Authorization)

What it is: A DNS record that tells the world which Certificate Authorities (CAs) are allowed to issue TLS certificates for your domain. Browsers/clients ask CAs to check CAA before issuing—if your CA isn’t allowed, issuance is refused. This reduces the risk of a rogue/accidental cert.

#### CAA doesn’t control HTTP/HTTPS traffic.
It only tells certificate authorities which ones are allowed to issue TLS certificates for your domain.

If you set:
```sh
CAA 0 issue "letsencrypt.org"
```
it means only Let’s Encrypt is allowed to issue new certs for nakodtech.xyz (and subdomains unless overridden).
HTTP (port 80) and HTTPS traffic still work as usual; browsers don’t care about your CAA when visiting the site—they just use whatever valid cert you present.

Existing certs from other CAs keep working until they expire. CAA affects future issuance, not currently installed certs.

If you also want to allow another CA (e.g., Cloudflare’s, DigiCert, Google Trust Services), add more lines:
```sh
CAA 0 issue "pki.goog"
CAA 0 issue "digicert.com"
```

For wildcard certs, add:
```sh
CAA 0 issuewild "letsencrypt.org"
```

Optional notifications:
```sh
CAA 0 iodef "mailto:security@nakodtech.xyz"
```
So: CAA = who may issue certs, not whether HTTP/HTTPS works.




### Add a CAA record: 0 issue "letsencrypt.org".

Add HSTS (after you’re confident):
```sh
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```


## 🧰 Quick Troubleshooting

* DNS not resolving? Lower TTL to 300s, wait a few minutes, re‑check with `dig`.
* Site down? Check security group, `sudo systemctl status nginx`, and that **A** points to the correct **EIP**.
* Cert errors? Ensure port 80 is open and DNS is correct before running `certbot`.

🎉 Done! You now have a live site at **nakodtech.xyz** with clean DNS and Nginx.

---

## ✉️ Email DNS Cheat‑Sheet with Emojis

### 🧠 What is TTL?

**TTL (Time To Live)** = how long DNS resolvers cache a record.

* Use **300s (5 min)** while testing or when IPs may change.
* For production, **1–4 hours** is common.

### 📫 Records for Site + Email (nakodtech.xyz)

| Host / Name                 | Type  | Value                                                  | Priority |     TTL | Purpose                               |
| --------------------------- | ----- | ------------------------------------------------------ | -------: | ------: | ------------------------------------- |
| `@`                         | A     | **YOUR\_PUBLIC\_IP** (or Elastic IP)                   |        – | **300** | 🌐 Root domain → your server          |
| `www`                       | CNAME | **nakodtech.xyz**                                      |        – | **300** | 🔁 `www` follows root                 |
| *(optional)* `@`            | AAAA  | `YOUR_IPV6`                                            |        – |     300 | 🌿 IPv6 if available                  |
| `@`                         | MX    | **<MX1 from Spacemail>**                               |   **10** | **300** | 📥 Primary inbound mail server        |
| `@`                         | MX    | **<MX2 from Spacemail>**                               |   **20** | **300** | 🛟 Backup inbound mail server         |
| `@`                         | TXT   | **v=spf1 include:<Spacemail-SPF> \~all**               |        – | **300** | 🛡️ SPF: who can send mail            |
| `<selector1>._domainkey`    | TXT   | **<DKIM public key>**                                  |        – | **300** | ✍️ DKIM signature key                 |
| `_dmarc`                    | TXT   | **v=DMARC1; p=none; rua=mailto\:dmarc\@nakodtech.xyz** |        – | **300** | 🔎 DMARC monitoring                   |
| *(optional)* `@`            | CAA   | **0 issue "letsencrypt.org"**                          |        – |    3600 | 🔐 Allow only Let’s Encrypt for certs |
| *(optional)* `autoconfig`   | CNAME | **<autoconfig host>**                                  |        – |     300 | ⚙️ Client auto‑config                 |
| *(optional)* `autodiscover` | CNAME | **<autodiscover host>**                                |        – |     300 | 🧭 Outlook/Apple auto‑discover        |

> 📨 **Mailboxes (e.g., `info@nakodtech.xyz`) are created in Spacemail** — you do **not** put them in MX. MX must point to **mail server hostnames** from Spacemail.

### ✅ Quick Verify

```bash
# DNS resolution
dig +short A nakodtech.xyz
dig +short CNAME www.nakodtech.xyz

# Mail DNS
dig +short MX nakodtech.xyz
dig +short TXT nakodtech.xyz | grep spf
dig +short TXT _dmarc.nakodtech.xyz
```

Need me to fill these with your real values? Share your **public IP** and Spacemail’s **MX/SPF/DKIM** strings and I’ll drop a ready‑to‑paste block here. ✍️🚀


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
