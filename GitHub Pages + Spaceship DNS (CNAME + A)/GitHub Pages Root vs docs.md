# GitHub Pages: Root vs `/docs` (Quick Guide) 🧑‍💻✨

## 🧠 TL;DR

* GitHub Pages serves **one publish folder** per repo.
* Your **homepage must be `index.html`** inside that publish folder.
* Choose in **Settings → Pages → Deploy from a branch**:

  * **Folder:** `/ (root)` or **`/docs`** (only these two without Actions)

---

## 🏠 User/Org site (repo = `yourname.github.io`)

* Put **`index.html` at the repo root** (default branch).
* That becomes `https://yourname.github.io/`.

## 📦 Project site (any other repo)

1. Go to **Settings → Pages**.
2. **Source:** *Deploy from a branch*
3. **Branch:** e.g. `main`
4. **Folder:**

   * **`/ (root)`** → put `index.html` at repo root
   * **`/docs`** → create a folder **`docs/`** (exact name) and put `index.html` there

> ✅ If you select **`/docs`**, it must be a **top‑level folder named exactly `docs/`** (not `src/docs`, not `Doc`).

---

## 📁 If your HTML lives in another folder (e.g., `site/`)

You’ve got three options:

1. **Move** `site/index.html` into the publish folder (root or `docs/`).
2. **Rename** `site/` → **`docs/`**, then select `/docs` in Pages.
3. Use **`gh-pages` branch** or a **GitHub Action** that builds/copies output into the publish location.

### 🚪 Make the root redirect (if you keep files in a subfolder)

Put this as `/index.html` in the publish root:

```html
<!doctype html>
<meta charset="utf-8">
<meta http-equiv="refresh" content="0; url=/site/">
<title>Redirecting…</title>
<a href="/site/">Click here if you are not redirected.</a>
```

---

## 🌐 Custom domain tips

* After you set a **custom domain** in **Settings → Pages**, GitHub creates a **`CNAME` file** in the publish folder. **Don’t delete it.**
* Point DNS:

  * **CNAME** `www → <username>.github.io`
  * **A** records for apex (optional): `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`

---

## 🔧 Jekyll & special folders

* If you use folders that start with `_` (e.g., `_assets`), add an empty file called **`.nojekyll`** to the publish root so Pages won’t run Jekyll processing.

---

## ✅ Verify it’s live

1. Commit & push `index.html` to the chosen publish folder.
2. Wait 1–2 minutes → open the URL shown in **Settings → Pages**.
3. If using a custom domain, verify the DNS has propagated (TTL \~5–30 min typically).

---

## 🧯 Quick Troubleshooting

* **404 or old page?** Ensure `index.html` is **in the publish folder** you selected.
* **Wrong folder?** Switch between `/ (root)` and `/docs` and move files accordingly.
* **Assets 404?** Use relative paths (`./css/style.css`) and commit those files.
* **Custom domain not working?** Check the `CNAME` file exists and DNS records are correct.

---

## 📝 Checklist

* [ ] Decide: **root** or **`/docs`**
* [ ] Put **`index.html`** in that exact folder
* [ ] (Optional) Add `.nojekyll` if you use `_`-prefixed folders
* [ ] (Optional) Set **custom domain** → keep the `CNAME` file
* [ ] Test the URL from **Settings → Pages**

Happy publishing! 🚀
