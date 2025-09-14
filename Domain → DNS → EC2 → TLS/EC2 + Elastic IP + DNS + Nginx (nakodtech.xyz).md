# 🧭 Quick Start: EC2 + Elastic IP + DNS + Nginx (nakodtech.xyz)

Follow these bite‑size steps to get a live site. Swap placeholders like **YOUR\_ELASTIC\_IP**.

---

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

> Bonus hardening: add a **CAA** record `0 issue "letsencrypt.org"` at the domain apex to restrict certificate issuers.

---

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
