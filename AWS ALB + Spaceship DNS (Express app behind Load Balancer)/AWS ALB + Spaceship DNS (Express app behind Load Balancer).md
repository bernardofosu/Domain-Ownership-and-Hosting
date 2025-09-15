# 🚀 AWS ALB + Spaceship DNS (Express app behind Load Balancer)

A printable, step‑by‑step guide to deploy a tiny **Node/Express** app on **EC2**, front it with an **Application Load Balancer (ALB)**, add **TLS** via **ACM**, and connect it to your **Spaceship DNS**.

We’ll use a **subdomain** (e.g., `app.nakodtech.xyz`) via **CNAME → ALB**.

> ⚠️ Apex note: Registrars (incl. Spaceship) don’t allow **CNAME at the apex**. If you want `nakodtech.xyz` on an ALB, move DNS to Route 53 and use **ALIAS A** to the ALB, or keep apex on another host.

---

## 0) 🧭 What you’ll build

* 1× EC2 VM (Ubuntu 22.04) running Express on **:3000**
* 1× Target Group (HTTP:3000) with health checks ✅
* 1× Internet‑facing **ALB** (80/443) forwarding to the target group 🧩
* 1× ACM certificate (DNS‑validated) for `app.nakodtech.xyz` 🔒
* 1× Spaceship **CNAME**: `app → <ALB_DNS_NAME>` 🌐

---

## 1) 🔐 Security Groups (create first)

**SG-ALB** (for the Load Balancer)

* Inbound: TCP **80** from `0.0.0.0/0`, TCP **443** from `0.0.0.0/0`
* Outbound: allow all (default)

**SG-APP** (for the EC2 instance)

* Inbound: TCP **22** from *your IP* (SSH)
* Inbound: TCP **3000** **from SG-ALB** (reference the SG by ID)
* Outbound: allow all (default)

> 🎯 Referencing SG‑ALB in SG‑APP keeps port 3000 private to the load balancer.

---

## 2) 🖥️ Launch EC2

* AMI: **Ubuntu 22.04 LTS**
* Type: **t3.micro** (fine for demos)
* VPC/Subnets: same VPC as the ALB
* Security Group: **SG-APP**
* Key pair: create/select one

SSH:

```bash
ssh -i ~/.ssh/your-key.pem ubuntu@EC2_PUBLIC_IP
```

---

## 3) 🧪 Install Node & sample Express app (port 3000)

```bash
# Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Minimal app
mkdir -p ~/hello && cd ~/hello
cat > server.js <<'JS'
const express = require('express');
const app = express();
app.get('/', (_, res) => res.send('Hello from Express behind ALB! 🚀'));
app.listen(3000, () => console.log('Listening on :3000'));
JS

npm init -y
npm i express
node server.js  # quick test (Ctrl+C to stop)
```

**Systemd service** (auto‑start):

```bash
sudo tee /etc/systemd/system/hello.service >/dev/null <<'UNIT'
[Unit]
Description=Hello Express
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/hello
ExecStart=/usr/bin/node /home/ubuntu/hello/server.js
Restart=always
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
UNIT

sudo systemctl daemon-reload
sudo systemctl enable --now hello
sudo systemctl status hello --no-pager
```

Quick check: `curl -s http://127.0.0.1:3000`

---

## 4) 🎯 Target Group (HTTP :3000)

**EC2 → Target Groups → Create**

* Type: **Instances**
* Protocol/Port: **HTTP :3000**
* Health checks: **HTTP** path `/` (expects 200 OK)

Register your EC2 instance → wait for **healthy** ✅

---

## 5) 🧩 Application Load Balancer

**EC2 → Load Balancers → Create → Application Load Balancer**

* Scheme: **Internet‑facing**
* IP type: **IPv4**
* **Subnets:** select **at least two AZs**
* Security group: **SG-ALB**

**Listeners**

* Add **HTTP :80** → action **Forward** to your target group

Create ALB and note the **ALB DNS name** (e.g., `app-123456.us-east-1.elb.amazonaws.com`).

---

## 6) 🔒 HTTPS with ACM (automatic, free)

**ACM (same region as ALB) → Request certificate**

* Public cert for `app.nakodtech.xyz`
* Validation: **DNS**

ACM shows a **CNAME** (Name → Value) for validation.

### Add the validation CNAME in Spaceship DNS

* Host: paste **Name** from ACM
* Type: **CNAME**
* Value: paste **Value/Target** from ACM
* TTL: **300**

Wait until ACM status = **Issued** ✅

### Add HTTPS listener

Back to **ALB → Listeners** → **Add :443**

* SSL cert: select your ACM cert for `app.nakodtech.xyz`
* Default action: **Forward** → target group

(Optional) In the **HTTP (80)** listener, add a rule to **Redirect to HTTPS**.

---

## 7) 🌐 Spaceship DNS → CNAME to the ALB

Create a **CNAME**:

| Host | Type  | Value (ALB DNS name)                     | TTL |
| ---- | ----- | ---------------------------------------- | --- |
| app  | CNAME | `app-123456.us-east-1.elb.amazonaws.com` | 300 |

> 🧠 Use the ALB **DNS name** (not ARN or IP). Apex (`@`) cannot be CNAME.

---

## 8) ✅ Verify

```bash
# DNS
dig +short app.nakodtech.xyz CNAME
dig +short app.nakodtech.xyz

# HTTP(S)
curl -I http://app.nakodtech.xyz
curl -I https://app.nakodtech.xyz
```

You should see **200 OK**, and browsing to **[https://app.nakodtech.xyz](https://app.nakodtech.xyz)** shows: *Hello from Express behind ALB! 🚀*

---

## 9) 🧹 Troubleshooting

* **Target unhealthy?** From the instance: `curl -I http://127.0.0.1:3000`. Check logs: `journalctl -u hello -f`. Ensure **SG-APP** allows **3000 from SG-ALB**.
* **HTTPS pending?** ACM DNS CNAME must match exactly; wait until **Issued**, then attach to ALB :443.
* **Timeouts?** ALB must be **Internet‑facing**, SG‑ALB must allow 80/443, and subnets need an **Internet Gateway**.
* **HTTP works, HTTPS doesn’t?** Ensure :443 listener + ACM cert, or add 80→443 redirect.

---

## 10) ✨ Extras

* **PM2**: `npm i -g pm2 && pm2 start server.js && pm2 save && pm2 startup systemd`
* **HSTS** in Express:

  ```js
  app.use((req,res,next)=>{res.set('Strict-Transport-Security','max-age=31536000; includeSubDomains');next();});
  ```
* **Scale‑out**: put the instance in an **Auto Scaling Group** targeting your TG.

---

## 11) 🧭 Example DNS layouts (Spaceship)

**Hybrid (apex on EC2, app on ALB)**

```
@      A      52.2.240.181                  ; EC2 (root)
www    CNAME  nakodtech.xyz                 ; www → apex
app    CNAME  app-123456.us-east-1.elb.amazonaws.com
```

**All via ALB (needs Route 53 for apex)**

```
@   ALIAS A   app-123456.us-east-1.elb.amazonaws.com
www CNAME     nakodtech.xyz
```

---

## 12) 💸 Costs

* **ALB**: hourly + LCU usage
* **ACM**: free when attached to ALB
* **EC2** + **data transfer** as usual (great for AWS credits)

---

### ✅ TL;DR checklist

* [ ] SG‑ALB (80/443 from world) & SG‑APP (3000 from SG‑ALB, 22 from your IP)
* [ ] EC2 running Express on :3000 (systemd)
* [ ] Target Group healthy
* [ ] ALB with :80 → TG and :443 (ACM) → TG
* [ ] ACM cert for `app.nakodtech.xyz` is **Issued**
* [ ] Spaceship CNAME `app → <ALB_DNS_NAME>`
* [ ] `https://app.nakodtech.xyz` returns your app 🎉
