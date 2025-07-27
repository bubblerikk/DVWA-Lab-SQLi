## Lab Instructions
Blind SQL Injection in DVWA (Beginner Guide)

# 🔍 What is Blind SQL Injection?
Blind SQL Injection happens when an application is vulnerable to SQL injection, but it doesn't show any error messages or query results.
In simple words:
> You can still inject malicious SQL code, but you won’t see any clear output or errors — making it “blind.”
Although it’s harder to spot, attackers can still guess and extract data by observing subtle changes in the application's behavior (like a different page load or response time).

# 🎯 What You'll Do in This Lab
In this beginner-friendly guide, you’ll learn how to:
- Set up DVWA (Damn Vulnerable Web App) and Burp Suite

- Configure a proxy using FoxyProxy in Firefox

- Intercept traffic from DVWA using Burp Suite


# Understand the setup for exploiting Blind SQL Injection
⚠️ This guide is for educational purposes only — practice in legal environments like DVWA, and always follow ethical hacking principles.

# 🛠️ Setting Up DVWA and Burp Suite
If you haven’t already installed DVWA, check out my guide:
👉 How to Install and Run DVWA in Kali Linux **README.md**

This guide walks you through:

> Installing required software

> Setting up the DVWA database

> Running DVWA on your browser via http://127.0.0.1/dvwa

# 💡 What is Burp Suite?
Burp Suite is a powerful web application testing tool created by PortSwigger (founded by Dafydd Stuttard). It helps ethical hackers inspect and manipulate requests between a browser and web server.

# Key Features:
- Intercepts HTTP/HTTPS traffic

- Modifies requests/responses

- Scans and exploits vulnerabilities

- It is a favorite among bug bounty hunters and security professionals.

# 🧰 Set Up Burp Suite with Firefox Using FoxyProxy

We’ll use FoxyProxy, a Firefox add-on, to route browser traffic through Burp Suite so we can intercept and analyze DVWA requests.

# 🔌 Step-by-Step: FoxyProxy Setup

# 1. Install FoxyProxy:

Open Firefox

Go to Add-ons → Search for FoxyProxy Standard

Click Add to Firefox



# 2. Configure a New Proxy:

Click the FoxyProxy icon in Firefox toolbar

Choose Options → Add New Proxy

# Fill in these settings:

- IP Address: 127.0.0.1

- Port: 8080

- Proxy Type: HTTP

Click Save



# 3. Enable the Proxy:

Click the FoxyProxy icon again

Select the new proxy profile you just created

# ⚙️ Configure Burp Suite to Match

1. Open Burp Suite


2. Go to Proxy → Options


3. Make sure Interface 127.0.0.1:8080 is active (this is Burp listening on localhost)



Now, when you visit websites in Firefox, Burp Suite will intercept all HTTP traffic going through 127.0.0.1:8080.

# 🧪 Testing the Proxy Setup

1. Open Firefox


2. Visit http://127.0.0.1/dvwa


3. With Burp Intercept ON, try logging in or submitting a form


4. If Burp shows the request, you’re all set!



# This proxy setup is essential because DVWA is running locally, and Burp Suite needs to see the traffic between Firefox and the DVWA server.
![1_gdN9EU643TUodSo5yQokOQ](https://github.com/user-attachments/assets/07948bff-9921-485d-9c35-9a694e734de9)

  
#  🧠 What Happens After Proxy Is Set?

With the proxy connection set up, Burp Suite now intercepts all browser traffic going through 127.0.0.1:8080.
This means when you visit http://127.0.0.1/dvwa, Burp will see and capture the full HTTP request, which is very useful for finding vulnerabilities — like Blind SQL Injection.


---

# 🛠️ SQLMap – The SQL Injection Tool

# ❓ What is SQLMap?

SQLMap is a powerful open-source tool that helps automate the process of detecting and exploiting SQL injection vulnerabilities in web applications.

It can:

Detect different types of SQLi (including Blind SQLi)

- Dump database names, tables, columns.
- Extract data (like usernames, passwords)
- Take over the database (read/write access)
- It saves a lot of manual effort — especially when the SQLi is Blind and slow to test by hand.


---

# 💾 Installing SQLMap

SQLMap usually comes pre-installed in Kali Linux.

But if it’s missing, you can install it by running this command in the terminal:

- `sudo apt-get install sqlmap`

After installation, check the version using:

- `sqlmap --version`

✅ The version I’m using for this tutorial: 1.7.6#stable
![1_zgseZVBTVE2LNEVqmPE9qg](https://github.com/user-attachments/assets/ac9136c6-382c-460b-ad4b-8de0092a50bd)

---

# 🍪 Cookie Harvesting using Burp Suite

After logging into DVWA (Damn Vulnerable Web Application):

1. Navigate to the “**SQL Injection (Blind)**” page from the sidebar.
2. In the input field labeled id, enter the value:
`2`

3. Click the **Submit** button.

However, you’ll notice that nothing happens in the browser right away.
That’s because Burp Suite is intercepting the request before it reaches the DVWA server.

💡 Burp works as a man-in-the-middle proxy, so when Intercept is ON, it holds the request until you decide what to do with it.


---

# ✅ What To Do Next
- Open Burp Suite
- Go to the Proxy tab
- In the Intercept sub-tab, you’ll see the captured **GET** request
- Click “**Forward**” to send the request to the DVWA server

Now the page will respond and show the output in the browser.

This process also helps us capture the full request (including cookies) — which is useful when running tools like SQLMap, as authentication is often handled via session cookies.
![1_rqAL4eNbqeFUwotOOhv9mw](https://github.com/user-attachments/assets/3197b95a-f1c5-4ff9-a7f9-694646846209)





# 📍 Identifying Session Cookies in Burp Suite
Now, let’s proceed to the “Proxy” tab in Burp Suite.
1. After submitting the form in DVWA, Burp Suite will intercept the request.
2. In the Intercept tab, look carefully at the Request Headers.
3. You will see something like this:

 - `GET /dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit HTTP/1.1`
 - `Host: 127.0.0.1`
 - `User-Agent: Mozilla/5.0 ...`
 - `Cookie: security=low; PHPSESSID=fe2rj2i5gud7ch1i0at3o9gkt0` < **copy this**

🔒 The line starting with Cookie: contains your session identifiers — usually PHPSESSID and security.


---

✂️ Copy This Cookie Header
Highlight and copy the entire Cookie: line.
We will use this value in the next step when sending the request to SQLMap.

📌 Example copied cookie:
`security=low; PHPSESSID=fe2rj2i5gud7ch1i0at3o9gkt0`

> This tells SQLMap who you are — without it, the request might be rejected or redirected to a login page.
![1_Ni_HEE2BaZfx3vNNAiDqiQ](https://github.com/user-attachments/assets/3cf5df61-a224-459d-ae65-55b93a3bc8a9)



# 🔗 Copy the Updated URL After Forwarding
After clicking the “Forward” button in Burp Suite, the request is sent to the DVWA server.

Now, go back to your Firefox browser.
You’ll see that the URL in the address bar has changed, and looks something like this:

`http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit`

> 📌 This updated URL includes the injected id parameter and the Submit value — it’s the exact request DVWA received.


---

# ✅ What You Need to Do
1. Copy the full URL from your browser’s address bar.
2. Save it somewhere temporarily — we will use it with SQLMap in the next step.

> This URL tells SQLMap where the vulnerable input is and allows it to test and exploit the Blind SQL Injection vulnerability.
![1_2HHtL8QIQEU_K1g92EG8lQ (1)](https://github.com/user-attachments/assets/2bf9546e-3fb7-47aa-b128-752e7bef4323)


# 🔍 Scanning for Vulnerabilities
In your terminal window, execute the following command:

<pre><code>`sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" </code></pre>

> ⚠️ Note: Be careful not to miss the --cookie double dash. In your original line, it was mistakenly a single dash - cookie, which will throw an error.



# 🛠 What Happens Next?
SQLmap will initiate a connection to the provided URL.
It checks whether the id GET parameter is dynamic—in other words, whether it can be exploited or not.
You can observe the actual payloads being submitted by SQLmap during this scan.

✅ Output Result:
- From the output, SQLmap identifies that the id parameter is vulnerable to Boolean-based blind SQL injection.
> (sql map output)
![1_2xRL3i6rqQdqIU-FDZcylg](https://github.com/user-attachments/assets/ede1c2f1-8ee1-460f-b2b6-ff6848c20b93)

> (sql map output)
![1_GT6TDpjaHXtPIplQ9FZplA](https://github.com/user-attachments/assets/008e291f-611b-4811-846e-6ec9f3daa760)

> (sql map output)
![1_aVa8JX6mKDyw4OlzjFQoCQ](https://github.com/user-attachments/assets/35840a0a-760a-4c2e-9fbd-6a6abf5b0622)

# 🧠 Fingerprinting the Target
From the SQLMap output, we can gather valuable information about the target environment:
- Operating System: Linux Debian
- Web Server: Apache 2.4.57
- Back-End DBMS: MySQL

This level of fingerprinting is incredibly useful for understanding the stack you're attacking and preparing further exploitation steps.
> 💡 You can avoid interactive prompts during scanning by adding the --batch flag to your SQLMap commands. This is useful for automated or scripted execution.

---

# 🗂️ Revealing Database Schemas

To enumerate all available database schemas, run the following command:

`sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" \
--cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" \
--batch --schema `

> 🖼️ A zoomed-in image of the schema discovery output can be included here if you are adding visuals in your GitHub repo for clarity.

> schema information
![1_xQgPsVR0ZR0fxYf6St8zcg](https://github.com/user-attachments/assets/ff3974fa-2265-43f9-bd2d-df1a3c26e0b1)


---

# 🔍 Revealing the dvwa Tables

Upon enumeration, two database names were retrieved: dvwa and information_schema. However, our focus is solely on the dvwa database, as it contains the relevant application data.

To retrieve the tables in the dvwa database, run the following command:

<pre><code>sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" -D dvwa --tables</code></pre>

# ✅ Explanation

-D dvwa: Specifies the target database.

--tables: Enumerates all the tables within the specified database.

> DVWA tables
![1_-UAn7bGd0aO4W2X8H-nbig](https://github.com/user-attachments/assets/d26d53f8-3a28-4a09-a1b2-37bdda48d5b6)



# 🧱 Enumerating the Columns in the users Table

After running SQLMap, we find that the dvwa database contains two tables:
- guestbook
- users ✅ (our target)

# Now, let's enumerate the columns inside the users table to understand its structure.
# 💻 SQLMap Command:

<pre><code>sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" --columns -T users --batch</code></pre>

# ✅ Explanation:

-T users → Specifies the target table (users).

--columns → Retrieves column names from the specified table.

--batch → Uses default answers to skip interactive prompts.

> tables users columns
![1_OKx38RRGzG63qgrgRNjxkw](https://github.com/user-attachments/assets/2b6cb405-65de-4678-955b-0a59a5a1bb56)


# 🔐 Dumping Usernames and Passwords
From our SQLMap output, we observe that the users table contains 8 columns. However, for our purpose, we are only interested in:
- user
- password
> These two columns typically store the user credentials.

# 💻 SQLMap Command to Dump Data:

<pre><code>sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" --dump -T users -C user,password --batch</code></pre>

✅ Explanation:
- --dump → Extracts actual data from the specified table/columns.

- -T users → Indicates the target table is users.

- -C user,password → Specifies the columns we want to extract.

- --batch → Automatically confirms default prompts.

> ⚠️ Ensure DVWA is set to low security and the PHP session is valid when executing the above command.

> Tables Users entries
![1_YWz3FmCz7Sc4lECiH3ZRVA](https://github.com/user-attachments/assets/f0035d0d-7e82-4422-864e-2b3eb3b77762)



# 🔓 Cracking Password Hashes with SQLMap

After successfully dumping the data, SQLMap detects that the password column contains hashed values. It then prompts:
- Whether to store the hashes (Default: No)
- Whether to crack them using a dictionary attack (Default: Yes)


# If agreed, SQLMap proceeds to:
- Use its built-in dictionary (wordlist.txt)
- Optionally use a custom wordlist (e.g., rockyou.txt)
- Ask about using common password suffixes (Default: No)

# 🧠 Behind the Scenes:
> SQLMap performs a dictionary-based attack to crack weak or commonly used passwords from the hashes. Once successful, it displays the plaintext credentials.

# 💻 Example Command with Custom Wordlist:

<pre><code>sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=9tp5mn3hpkfgh6fddmbampps7m" --dump -T users -C user,password --batch --passwords --wordlist=/usr/share/wordlists/rockyou.txt</code></pre>

---

# ✅ Conclusion
- We successfully:
1. Detected SQL injection using blind SQLi
2. Enumerated database structure using --tables and --columns
3. Dumped and cracked user credentials from the users table
> This demonstrates how vulnerable applications like DVWA can be exploited if proper input validation and security practices are not followed.

---

🙏 Thank You
> Thank you for taking the time to read this guide.
Your feedback, suggestions, and contributions are always welcome!

# Feel free to ⭐ star this repository, raise issues, or contribute improvements.
