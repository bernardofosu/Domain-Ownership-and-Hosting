# 🔐 CAA DNS Records – Complete Cheat Sheet (with Emojis)

---

## 📌 The 3 CAA Tags Explained

### 1️⃣ issue  
📌 Meaning → Which CA(s) can issue normal certs for your domain (and wildcard, unless overridden).  
📝 Value → CA’s domain name (no https://).  

✅ Example (allow Let’s Encrypt):  
```dns
CAA 0 issue "letsencrypt.org"
```  
🎯 Use when: You want to allow a specific CA (e.g., Let’s Encrypt) to issue certs for **nakodtech.xyz**.

---

### 2️⃣ issuewild  
📌 Meaning → Policy for wildcard certs (`*.nakodtech.xyz`).  
📝 Value → CA’s domain.  

✅ Example (allow LE wildcard certs):  
```dns
CAA 0 issuewild "letsencrypt.org"
```  

🚫 Forbid wildcard issuance:  
```dns
CAA 0 issuewild ";"
```  

🎯 Use when: You want explicit control over wildcard cert issuance.

---

### 3️⃣ iodef  
📌 Meaning → Where CAs should send reports if they detect mis-issuance attempts.  
📝 Value → `mailto:` or `https://` URL.  

✅ Examples:  
```dns
CAA 0 iodef "mailto:security@nakodtech.xyz"
CAA 0 iodef "https://reports.nakodtech.xyz/caa"
```  

🎯 Use when: You want alerts about certificate issues. *(Does not control issuance)*.

---

## 🧾 CAA Record Components Table

| 🏷️ Field        | 📝 What it means | 🔧 What you put here | 📌 Example |
|-----------------|------------------|----------------------|------------|
| **Name / Host** | Which domain/subdomain the rule applies to. | `@` for root (nakodtech.xyz) or subdomain. | `@` → nakodtech.xyz <br> `www` → www.nakodtech.xyz |
| **Type**        | Record type. Must be CAA. | Always `CAA`. | CAA |
| **Flags** (0/128) | Defines if the tag is critical. <br>0 = normal. <br>128 = critical (rare). | Usually `0`. | `0` |
| **Tag**         | Purpose of record. | Choose: <br>• `issue` → allow CA <br>• `issuewild` → wildcard <br>• `iodef` → send reports | issue |
| **Value**       | The CA domain or report address. | If tag = issue → CA domain. <br>If issuewild → CA domain or `;`. <br>If iodef → `mailto:` or `https://`. | letsencrypt.org <br> mailto:security@nakodtech.xyz |
| **TTL**         | Time-to-live (cache time). | Common: 1800s (30m) or 3600s (1h). | 30 min |

---

## 🌍 Real-World CAA Record Examples

| 🏷️ Tag (Purpose) | 🔧 Value (What you put) | 📌 What it means in practice | 🌍 Example |
|------------------|--------------------------|-----------------------------|------------|
| **issue**        | letsencrypt.org | Only Let’s Encrypt can issue certificates. | `CAA 0 issue "letsencrypt.org"` |
| **issue**        | digicert.com | Only DigiCert can issue certs. | `CAA 0 issue "digicert.com"` |
| **issuewild**    | letsencrypt.org | Allow Let’s Encrypt to issue wildcard certs. | `CAA 0 issuewild "letsencrypt.org"` |
| **issuewild**    | ; | Forbid all wildcard certs. | `CAA 0 issuewild ";"` |
| **iodef**        | mailto:security@nakodtech.xyz | If suspicious request → send email. | `CAA 0 iodef "mailto:security@nakodtech.xyz"` |
| **iodef**        | https://reports.nakodtech.xyz/caa | Reports go to HTTPS endpoint. | `CAA 0 iodef "https://reports.nakodtech.xyz/caa"` |

---

## 🛠️ Recommended Setup for nakodtech.xyz (Let’s Encrypt)

```dns
CAA 0 issue "letsencrypt.org"                  # allow LE to issue certs
CAA 0 issuewild "letsencrypt.org"              # allow wildcard certs explicitly
CAA 0 iodef "mailto:security@nakodtech.xyz"    # send problem reports to email
```

---

## ⚡ Key Notes

- 🔢 **Flags** → Use `0` (non-critical). Use `128` only for strict enforcement.  
- 🕑 **TTL** → 30 mins or 1 hour is safe.  
- 🔍 Verify with:  
```bash
dig +short CAA nakodtech.xyz
```
👉 Expected → `0 issue "letsencrypt.org"`

✨ In short:  
- `issue` → Who can issue.  
- `issuewild` → Who can issue wildcards.  
- `iodef` → Who gets reports.  
