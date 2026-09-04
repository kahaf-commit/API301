# Lab Setup | First Attempt

# Setting Up the Environment

# 1. Install Kali Linux

Download and set up the Kali Linux 64-bit VM via VMware from the [official Kali website](https://www.kali.org/get-kali/#kali-virtual-machines) .

# 2. Update Kali

Ensure your Kali installation is up-to-date with the following commands:

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

# 3. Update User Accounts

After setting up the OS, updating default credentials is a security best practice.

# 4. Install Essential Tools

# **Burp Suite Community Edition**

Burp Suite is crucial for intercepting and analyzing HTTP requests.

```bash
sudo apt-get install burpsuite -y
```

Download Jython from [jython.org](https://www.jython.org/download.html) and add the .jar file to the Extender Options in Burp Suite.

# **FoxyProxy Standard**

- FoxyProxy lets you control traffic routing, especially useful when switching between Burp and other proxies.
    1. **Install FoxyProxy from** [here](https://addons.mozilla.org/en-US/firefox/addon) **.**
    2. **Configure Proxies:**
        1. BurpSuite Proxy: Set IP to **`127.0.0.1`**, Port to **`8080`**.
        2. Postman Proxy: Set IP to **`127.0.0.1`**, Port to **`5555`**.
    
    1. **Burp Suite Certificate**
        1. Using FoxyProxy, select the BurpSuite proxy.
        2. Visit [**http://burpsuite**](http://burpsuite/) and download the CA certificate.
        3. Import the certificate into Firefox.

# **OWASP ZAP**

Another essential tool for security testing.

```bash
sudo apt install zaproxy
```

Navigate to Manage Add-Ons (CTRL+U) to update Fuzzer and OpenAPI Support.

# **MITMweb Certificate Setup**

- Start MITMweb: `mitmweb`
- Use FoxyProxy to route traffic to BurpSuite.
- Download the `mitmproxy-ca-cert.pem` from [mitm.it](https://mitm.it/) .
- Import the certificate into Firefox.

# **Install Postman**

```bash
sudo wget https://dl.pstmn.io/download/latest/linux64 -O postman-linux-x64.tar.gz
sudo tar -xvzf postman-linux-x64.tar.gz -C /opt
sudo ln -s /opt/Postman/Postman /usr/bin/postman
```

# **Install mitmproxy2swagger**

```bash
sudo pip3 install mitmproxy2swagger
```

# **Install Git**

```bash
sudo apt-get install git
```

# **Install Docker**

```bash
sudo apt-get install docker.io docker-compose
```

# **Install Go**

```bash
sudo apt install golang-go
```

# **Install JWT Toolkit**

```bash
cd /opt
sudo git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool
pip3 install termcolor cprint pycryptodomex requests
```

# **Install Kiterunner**

```bash
sudo git clone https://github.com/assetnote/kiterunner.git
cd kiterunner
sudo make build
sudo ln -s /opt/kiterunner/dist/kr /usr/bin/kr\
```

# **Install Arjun**

```bash
sudo apt install arjun
```

# 5. Useful Wordlists

# **SecLists**

```bash
sudo apt install seclists
```

---

# Configuring the Hacking Lab

# **crAPI**

[crAPI](https://github.com/OWASP/crAPI) is an API security testing lab from OWASP.

- Create a lab directory and set up crAPI:
    
    ```bash
    cd ~
    mkdir labs
    sudo curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml
    sudo docker-compose pull
    sudo docker-compose -f docker-compose.yml --compatibility up -d
    ```
    
    If you encounter installation issues, consider using the development version available on GitHub or APIsec’s hosted instance.
    

---

# API Reconnaissance Techniques

# 1. Passive Reconnaissance

# **Google Dorking**

```markdown
inurl:/wp-json/wp/v2/users
inurl:"/api/v1" intext:"Index of /"
inurl:/api/v1 intext:index of /
intitle:index.of intext:api.txt
intitle:"index of" intext:"api.txt"
intitle:index of api_key OR api key OR apiKey -pool
```

# **Git Dorking**

```jsx
api key
api keys
apikey
authorization: Bearer
access_token
token
```

# **Shodan Queries**

```jsx
hostname:targetname.com
content-type: application/json
content-type: application/xml
200 OK
wp-json
```

# **Wayback Machine**

Compare older API documentation for differences in endpoints. Test old endpoints during active testing.

# 2. Active Reconnaissance

# **Nmap**

```bash
nmap -sC -sV <target> -oA output.txt
nmap -p- <target> -oA output-allports.txt
nmap -sV --script=http-enum <target> -p 80,443,8000,8080
```

# **Amass**

```bash
amass enum -active -d <target>
sudo amass enum -active -d example.com | grep api
```

# **Directory Brute-force**

- **Gobuster:**
    
    ```bash
    gobuster dir -u http://target-name.com:8000 -w /usr/share/dirb/common.txt
    ```
    
- **ffuf:**
    
    ```bash
    ffuf -u http://target/FUZZ -w /usr/share/dirb/common.txt
    ```
    

# **DevTools in Browser**

- Open DevTools with `F12` or `Ctrl+Shift+I` to inspect requests, check for API calls, and examine responses.

# Conclution

# crAPI Lab Setup & API Pentesting Notes

Personal lab notes for setting up a Python virtual environment, running `mitmproxy2swagger`, deploying crAPI (Completely Ridiculous API) via Docker, and configuring Firefox/FoxyProxy for intercepting local traffic with Burp Suite / OWASP ZAP.

---

## 1. Python Virtual Environment (`myenv`)

### Cleaning up pip/apt cache

```bash
sudo pip cache purge
sudo apt clean
sudo apt autoremove -y
```

### Activating the environment later

Open a terminal and make sure you're in the folder where `myenv` was created (usually your home `~` directory), then run:

```bash
# 1. Activate the virtual environment
source myenv/bin/activate

# 2. Run the tool directly
mitmproxy2swagger --help
```

### Deactivating when done

```bash
deactivate
```

### Shortcut (skip activation every time)

Run the tool directly from Kali's base system in one line:

```bash
~/myenv/bin/mitmproxy2swagger --help
```

---

## 2. Preventing `mitmweb` from Auto-Opening a Browser

### Method 1 — Disable auto-open, open manually (easiest)

```bash
mitmweb --no-web-open-browser
```

Alternative form:

```bash
mitmweb --set web_open_browser=false
```

Once running, the terminal will show a link (usually `http://127.0.0.1:8081`). Open Firefox manually and paste that URL into the address bar.

### Method 2 — Change Kali's default browser

If you want every link click to open in Firefox instead of Chrome/Chromium:

```bash
sudo update-alternatives --config xdg-open
```

or, to change the browser directly:

```bash
sudo update-alternatives --config x-www-browser
```

---

## 3. Deploying crAPI (Docker)

```bash
cd ~
mkdir labs
sudo curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml
sudo docker-compose pull
sudo docker-compose -f docker-compose.yml --compatibility up -d
```

Then open Firefox to access the app.

---

## 4. Configuring Burp Suite / OWASP ZAP as a Proxy

### Step 1 — Check proxy tool port

Interception tools typically run on these default ports:

| Tool | Host/IP | Port |
| --- | --- | --- |
| Burp Suite | `127.0.0.1` | `8080` |
| OWASP ZAP | `127.0.0.1` | `8081` |

### Step 2 — Configure FoxyProxy

1. Click the FoxyProxy icon in Firefox → **Options**.
2. Click **Add** to create a new proxy profile.
3. Enter:
    - **Title:** `Burp/ZAP Proxy` (any name)
    - **Proxy Type:** `HTTP`
    - **Proxy IP Address:** `127.0.0.1`
    - **Port:** `8080` (or your tool's port)
4. **Save**, then click the FoxyProxy icon and select this new profile to enable it.

### Step 3 — Disable Firefox's localhost proxy bypass (⚠️ Important)

By default, Firefox sends `localhost` / `127.0.0.1` traffic directly instead of through the proxy (FoxyProxy). This must be disabled to test crAPI locally:

1. In the Firefox address bar, type `about:config` and press Enter.
2. Click **Accept the Risk and Continue** on the warning screen.
3. Search for: `network.proxy.allow_hijacking_localhost`
4. This defaults to `false` — double-click it to set it to `true`.

**Alternative:** instead of changing Firefox's internal config, use your machine's LAN IP (e.g. `http://192.168.1.5:8888`) or a custom domain via a hosts-file entry (e.g. `http://crapi.local:8888`) instead of `http://localhost:8888`.

---

## 5. Recon Commands & Search Dorks (API Pentesting)

### Google dorks — exposed API index listings

```
inurl:"/api/v1" intext:"Index of /"
```

### Google dorks — exposed `api.txt` files

```
intitle:"index of" intext:"api.txt"
```

### GitHub code/repo search — leaked API keys

```
api key
```

### GitHub code search — leaked access tokens

```
access_token
```

### GitHub code search — leaked bearer tokens

```
Authorization: Bearer
```

### Subdomain enumeration with `amass` (crAPI lab)

```bash
sudo amass enum -active -d example.com | grep api
```

> Replace `example.com` with your actual authorized target/lab domain.
`-active` performs active enumeration (DNS resolution, possible port scans) — only run against domains/scopes you're authorized to test.
The `grep api` filter narrows results to subdomains containing "api" (e.g. `api.example.com`, `api-staging.example.com`).
> 

---

## Notes / Reminders

- All active scanning (`amass -active`, Burp/ZAP intercepting, etc.) should only target lab environments (crAPI) or scopes you have explicit authorization to test.
- FoxyProxy + `about:config` proxy bypass changes are local Firefox settings — revert `network.proxy.allow_hijacking_localhost` back to `false` if you no longer need local proxy interception, to avoid accidentally routing other localhost traffic through a proxy.