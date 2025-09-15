# 🧩 Final, clean DNS to keep (nakodtech.xyz)

Below is a **simple, copy‑exact** set you can mirror in Spaceship DNS — formatted like your panel. I kept it minimal and tidy.

---

## 🌐 Apex & Web

| Host    | Type      | Value                    | TTL    |
| ------- | --------- | ------------------------ | ------ |
| @       | **A**     | `54.242.67.35`           | 30 min |
| `www`   | **CNAME** | `nakodtech.xyz`          | 30 min |
| `pages` | **CNAME** | `bernardofosu.github.io` | 5 min  |

> ✅ **Tip:** Keep TTL small (5–30 min) while testing; raise later for stability.

---

## 📮 Mail delivery (MX)

> ⚠️ **Let Spacemail create these automatically. Do not add duplicates.**

| Host | Type   | Value               | Pref | TTL   |
| ---- | ------ | ------------------- | ---- | ----- |
| @    | **MX** | `mx1.spacemail.com` | 10   | 5 min |
| @    | **MX** | `mx2.spacemail.com` | 20   | 5 min |



## 📝 Notes

* 🟢 If Gmail places your mail in **Spam**, keep sending normal, short **plain‑text** messages; avoid link‑only mails, large images, or raw HTML. Reputation improves quickly.
* 🟣 Only **one set** of MX records should exist (no “spaceshipmail” vs “spacemail” mixes).
* 🔁 If you ever move the site root off EC2, update only the **A** record for `@` (and keep `www → CNAME → nakodtech.xyz`).

That’s it — clean and conflict‑free. ✅

## 🔐 Authentication (TXT)

Set up **SPF**, **DKIM**, and **DMARC** so mail from `@nakodtech.xyz` lands in Inbox (not Spam).

### ✅ What to create (copy values exactly)

| Host (name)          | Type | Value (one line)                                                                                          | TTL    |
| -------------------- | ---- | --------------------------------------------------------------------------------------------------------- | ------ |
| `@`                  | TXT  | `v=spf1 include:spf.spacemail.com ~all`                                                                   | 30 min |
| `default._domainkey` | TXT  | `v=DKIM1; k=rsa; p=MIIBIjANBgkqh...FULL_RSA_KEY_HERE...`                                                  | 30 min |
| `_dmarc`             | TXT  | `v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz; ruf=mailto:dmarc@nakodtech.xyz; fo=1; adkim=r; aspf=r` | 30 min |

> 📝 **Notes**
>
> * DKIM must be the **full public key on a single line** (no spaces/newlines added by the UI).
> * You can start with `p=none` for DMARC while testing, then tighten to `p=quarantine` or `p=reject` after a few days of clean reports.
> * Leave the **MX** records that Spacemail created (don’t duplicate them): `mx1.spacemail.com (10)`, `mx2.spacemail.com (20)`.

### 🛠️ How to add each TXT in Spaceship DNS

1. **DNS → Add record → TXT**.
2. For **SPF**: set **Host** = `@`, **Value** = `v=spf1 include:spf.spacemail.com ~all`, **TTL** = 30 min → **Save**.
3. For **DKIM**: set **Host** = `default._domainkey`, **Value** = paste the **entire** RSA key Spacemail shows, **TTL** = 30 min → **Save**.
4. For **DMARC**: set **Host** = `_dmarc`, **Value** = `v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz; ruf=mailto:dmarc@nakodtech.xyz; fo=1; adkim=r; aspf=r`, **TTL** = 30 min → **Save**.

### 🔎 Verify from a terminal

```bash
# SPF
dig +short TXT nakodtech.xyz

# DKIM
dig +short default._domainkey.nakodtech.xyz TXT

# DMARC
dig +short _dmarc.nakodtech.xyz TXT
```

### 📬 After DNS propagates

* Send a **plain‑text** test from Spacemail to Gmail/Outlook.
* If it still lands in Spam, check:

  * SPF/DKIM/DMARC dig results match the table above.
  * Message isn’t only an image or empty HTML (send normal text).
  * Give Gmail time to trust the sender (star/“Not spam” a few messages).


## 🔐 Authentication (TXT)

> Copy these **exact** values. DKIM must be the **full single‑line key**.

### SPF (host **@**)

```
v=spf1 include:spf.spacemail.com ~all
```

### DKIM (host **default.\_domainkey**)

```
v=DKIM1; k=rsa; p=MIIBIjANBgkqh...FULL_RSA_KEY_HERE...
```

### DMARC (host **\_dmarc**)

```
v=DMARC1; p=none; rua=mailto:dmarc@nakodtech.xyz; ruf=mailto:dmarc@nakodtech.xyz; fo=1; adkim=r; aspf=r
```

---

## 🧪 Verify from terminal

Use any resolver (here I use Cloudflare 1.1.1.1):

```bash
dig +noall +answer nakodtech.xyz A @1.1.1.1
dig +noall +answer www.nakodtech.xyz CNAME @1.1.1.1
dig +noall +answer pages.nakodtech.xyz CNAME @1.1.1.1

dig +noall +answer nakodtech.xyz MX @1.1.1.1

dig +short TXT nakodtech.xyz @1.1.1.1         # SPF
dig +short default._domainkey.nakodtech.xyz TXT @1.1.1.1  # DKIM
dig +short _dmarc.nakodtech.xyz TXT @1.1.1.1   # DMARC
```

---

## 📝 Notes

* 🟢 If Gmail places your mail in **Spam**, keep sending normal, short **plain‑text** messages; avoid link‑only mails, large images, or raw HTML. Reputation improves quickly.
* 🟣 Only **one set** of MX records should exist (no “spaceshipmail” vs “spacemail” mixes).
* 🔁 If you ever move the site root off EC2, update only the **A** record for `@` (and keep `www → CNAME → nakodtech.xyz`).

That’s it — clean and conflict‑free. ✅
