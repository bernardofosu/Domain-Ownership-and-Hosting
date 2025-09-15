# 🔐 HSTS & Let’s Encrypt — Emoji Guide (nakodtech.xyz)

## 🛡️ What is HSTS?

**HSTS = HTTP Strict Transport Security.**

It’s an HTTP response header that tells browsers:

> “For the next **max-age** seconds, only talk to this site over **HTTPS**.
> If the user types `http://`, automatically upgrade to `https://` and never allow a downgrade.”

### ⭐ Why use it?

* 🧱 Stops SSL‑strip / downgrade attacks
* 🍪 Prevents cookies/content from ever being sent over HTTP after the first HTTPS visit
* ⚡ With **preload**, protection even on the **first** visit

---

## ✅ What “be confident” means (checklist)

Turn HSTS on **only when**:

* ✅ Your site works end‑to‑end on **HTTPS** (valid cert, 301 redirect from HTTP → HTTPS).
* ✅ All assets (images/JS/CSS/fonts) load via **https\://** (no *mixed content*).
* ✅ If you add **`includeSubDomains`**, **every subdomain** you use now or in the future will also be HTTPS (e.g., `www`, `api`, `blog`, admin panels, device UIs, etc.).

  * ℹ️ HSTS affects **HTTP/S web traffic** only (not SMTP/IMAP), but it **will** break any web UI that’s still plain HTTP.
* ✅ You won’t need to serve anything over HTTP for a while. Long **max‑age** means browsers cache the rule and you can’t quickly undo it.

---

## 🧪 Safe rollout plan

Start small (no subdomains yet):

```nginx
add_header Strict-Transport-Security "max-age=86400" always;  # 1 day
```

Watch for issues → increase to **1 week**, then **1 month**, then:

```nginx
add_header Strict-Transport-Security "max-age=31536000" always;  # 1 year
```

When **all subdomains** are HTTPS, add:

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

After you’ve run like this for a while without problems, you **may** add preload and submit the domain at **[https://hstspreload.org](https://hstspreload.org)**:

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

⚠️ **Preload is hard to undo**; only do it if you’re **100% sure** everything under your domain will always be HTTPS.

---

## 🧩 Where to put it (Nginx)

Add the header **inside your HTTPS (port 443) server block**:

```nginx
server {
  listen 443 ssl;
  server_name nakodtech.xyz www.nakodtech.xyz;

  add_header Strict-Transport-Security "max-age=86400" always;  # start small

  # ... rest of your config ...
}
```

Reload Nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

### 🔍 Verify / rollback

Verify header is served:

```bash
curl -I https://nakodtech.xyz | grep -i strict-transport-security
```

Roll back (reduce cache on new visits):

```nginx
add_header Strict-Transport-Security "max-age=0" always;
```

Then reload Nginx and give browsers time to re‑fetch over HTTPS and clear the cached policy.

---

## 🪪 Quick reminder: CAA vs HSTS

* **CAA** (`CAA 0 issue "letsencrypt.org"`) limits **which CA** may issue certificates for your domain. 🔏
* **HSTS** forces browsers to use **HTTPS**. 🌐
* They’re complementary and independent.

---

## 🆓 Let’s Encrypt — Free certs & renewal

* 💸 **Free** TLS certificates
* ⏳ Validity: **90 days** per certificate
* 🔁 Tools like **Certbot** attempt renewal around **day 60**

Where your certs live (on this box):

```
/etc/letsencrypt/live/nakodtech.xyz/fullchain.pem
/etc/letsencrypt/live/nakodtech.xyz/privkey.pem
```

### 🔄 Auto‑renew (snap install)

A systemd timer runs twice daily and renews any cert with ≤30 days left, then reloads Nginx.

Check it:

```bash
sudo certbot renew --dry-run
systemctl status snap.certbot.renew.timer
systemctl list-timers | grep certbot
```

### 🧰 Optional cron (not needed if timer is present)

```cron
0 3,15 * * * /snap/bin/certbot renew --quiet
```

> Runs at **03:00** and **15:00** daily; renews only when within 30 days of expiry. **Don’t run both** timer and cron.

### 📌 Requirements for renewal success

* Port **80** open (HTTP‑01 challenge)
* DNS **A/CNAME** still point to this server
* System time correct

### 🆘 If expired / manual re‑issue

```bash
sudo certbot --nginx -d nakodtech.xyz -d www.nakodtech.xyz
sudo nginx -t && sudo systemctl reload nginx
```

🎉 That’s it—use HSTS carefully for rock‑solid HTTPS, and Let’s Encrypt + Certbot to keep certs fresh automatically.
