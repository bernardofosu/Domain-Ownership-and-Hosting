# 🚀 nakodtech.xyz — It’s Working!

Your latest checks show everything is healthy:

* ✅ **DNS**: `nakodtech.xyz` & `www` resolve to **54.242.67.35**
* ✅ **Nginx**: service is **active (running)**
* ✅ **HTTPS**: `curl -I https://nakodtech.xyz` → **200 OK**
* ℹ️ Direct IP over HTTPS fails by design (certs are for names, not raw IPs)

---

## 🔎 When you see “connection timed out”

It’s almost always **reachability** (ports / firewall) or **DNS cache** still using the old IP.

### 1) Verify `www` resolves to the same IP

```bash
dig +short www.nakodtech.xyz @1.1.1.1
dig +short www.nakodtech.xyz @8.8.8.8
```

Expected: `54.242.67.35`. If not, wait up to TTL.

### 2) Check EC2 Security Group (most common)

Inbound rules must include:

* 🌍 **TCP 80** from `0.0.0.0/0`
* 🌍 **TCP 443** from `0.0.0.0/0`

### 3) Check OS firewall (UFW)

```bash
sudo ufw status verbose
```

Should be **inactive** or allow **80, 443**.

### 4) Confirm Nginx listeners

```bash
sudo ss -ltnp | egrep ':80|:443'
```

Expect listeners on `0.0.0.0:80` and `0.0.0.0:443`.

### 5) Quick curl tests

**From the server:**

```bash
curl -I http://127.0.0.1
curl -I -k https://127.0.0.1
```

**From anywhere (bypasses DNS):**

```bash
curl -I --connect-timeout 5 http://54.242.67.35
curl -I --connect-timeout 5 https://54.242.67.35 -k
```

If these work but the hostname fails → DNS cache.

### 6) Ensure HTTP → HTTPS redirect

```nginx
server {
  listen 80;
  server_name nakodtech.xyz www.nakodtech.xyz;
  return 301 https://$host$request_uri;
}
```

### 7) Check logs if needed

```bash
sudo tail -n 100 /var/log/nginx/access.log
sudo tail -n 100 /var/log/nginx/error.log
```

---

## 🧠 Notes & Tips

* ⏱️ **TTL matters**: with TTL 30m, some clients may keep the old A record for \~30 min.
* 📌 Consider an **Elastic IP** to avoid future DNS switches.
* 🔐 Cert must include **both names** (`nakodtech.xyz` & `www.nakodtech.xyz`). Check:

  ```bash
  sudo certbot certificates
  ```

  If `www` is missing, reissue:

  ```bash
  sudo certbot --nginx -d nakodtech.xyz -d www.nakodtech.xyz
  sudo nginx -t && sudo systemctl reload nginx
  ```
* 🔁 **Auto‑renew** runs via snap’s systemd timer (twice daily):

  ```bash
  systemctl list-timers | grep certbot
  sudo certbot renew --dry-run
  ```
* 🌐 Add IPv6 later? Create an **AAAA** record and ensure Nginx listens on IPv6.

---

## 🛠️ Handy commands

```bash
# See live DNS answers + TTL
dig +noall +answer nakodtech.xyz
dig +noall +answer www.nakodtech.xyz

# Health checks
curl -I https://nakodtech.xyz
curl -I https://www.nakodtech.xyz
```

You’re live and healthy. 🎉 If anyone still can’t reach it, it’s their DNS cache—give it a little time or have them hard refresh (Ctrl/Cmd+Shift+R).
