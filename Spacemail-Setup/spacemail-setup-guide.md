# 📧 How to Set Up Spacemail on Spaceship (Step by Step with Emojis)

## 1️⃣ Buy & Connect Domain
🌐 Go to **Spaceship.com**  
🛒 Register your domain (example: **nakodtech.xyz**)  
✅ Confirm it’s active in **Domain Manager**  

## 2️⃣ Activate Spacemail Trial
📦 Go to **Spacemail Manager**  
🖱️ Click **Unbox** next to *Spacemail Starter (Trial)*  
🔑 This starts your email setup  

## 3️⃣ Create Your First Mailbox
✍️ Choose a name → e.g. **info** → `info@nakodtech.xyz`  
🔒 Set a strong password  
📌 Save credentials (you’ll need them to log in)  

## 4️⃣ DNS Setup (Automatic on Spaceship ✅)
📮 **MX records** → added automatically  
🛡️ **SPF & DKIM** → configured by Spaceship  
🔍 You can check in **Advanced DNS** if needed  

## 5️⃣ Login to Webmail
🌐 Go to **Spacemail Webmail**  
✉️ Enter your new email: `info@nakodtech.xyz`  
🔑 Use the password you set  
🎉 You’re in your inbox!  

## 6️⃣ Test Your Mailbox
📤 From Gmail/Yahoo → send mail to `info@nakodtech.xyz`  
📥 Check **Spacemail inbox** → confirm it arrives  
🔁 Reply back to Gmail → confirm sending works  

## 7️⃣ (Optional) Security Setup
Add a **CAA record** in DNS to protect certificates:  

```
CAA 0 issue "letsencrypt.org"
CAA 0 iodef "mailto:info@nakodtech.xyz"
```

🔔 Now you’ll get alerts if a CA detects misuse  

## 8️⃣ Manage Auto-Renew
⚠️ Important if you only want the free trial:  
Go to **Spacemail Manager → Settings**  
Toggle **Auto-Renew OFF**  
Or 🗑️ remove your payment method  

✅ This ensures you won’t be billed after the trial  

---

## ✅ Final Step
🎉 Done! Your domain **nakodtech.xyz** now has professional email 📧  
💼 Use it for SSL alerts, business mail, or logins  
🛡️ Stay safe: keep auto-renew off if you only test trial  
