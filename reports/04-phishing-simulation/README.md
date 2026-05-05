# Lab 4: Phishing Simulation

## Objective

Design, execute, and analyse a controlled phishing simulation to assess an organisation's susceptibility to social-engineering email attacks, measure employee awareness, and recommend improvements to the security awareness programme.

## Environment

| Item | Details |
|------|---------|
| Attacker OS | Kali Linux |
| Tools | GoPhish 0.12.1, Social-Engineer Toolkit (SET) 8.0 |
| Scope | Internal lab network — consenting volunteer participants only |
| Duration | 5 days (2026-04-05 to 2026-04-10) |

> **Legal notice:** This simulation was conducted with explicit written authorisation from the organisation's management and IT security team. All participants consented to the training exercise. No real credentials were collected or stored.

## Theory

### What is Phishing?

Phishing is a social-engineering attack in which an adversary sends fraudulent messages (commonly email) that appear to originate from a trusted source in order to:

- Steal credentials (usernames and passwords)
- Install malware via malicious attachments or links
- Trick users into transferring money or sensitive data

### Types of Phishing

| Type | Description |
|------|-------------|
| **Spear phishing** | Targeted attack customised for a specific individual or organisation |
| **Whaling** | Spear phishing directed at senior executives |
| **Smishing** | Phishing via SMS |
| **Vishing** | Voice/phone-based phishing |
| **Clone phishing** | Legitimate email is cloned and resent with a malicious payload |

### Phishing Kill Chain

1. **Reconnaissance** – gather target email addresses, names, roles.
2. **Weaponisation** – create a convincing pretext, build a credential-harvesting page.
3. **Delivery** – send phishing emails to targets.
4. **Exploitation** – user clicks the link or opens an attachment.
5. **Collection** – credentials or click-through data are recorded.
6. **Analysis** – measure click rate, credential submission rate, and report rate.

## Methodology

### Part A – GoPhish Campaign

GoPhish is an open-source phishing framework that provides campaign management, email sending, landing pages, and reporting dashboards.

#### Step 1 – Installation and Configuration

```bash
# Download and extract GoPhish
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip -d gophish/
cd gophish/
chmod +x gophish
```

Edit `config.json` to configure the listening interface and SMTP credentials:

```json
{
  "admin_server": {
    "listen_url": "0.0.0.0:3333",
    "use_tls": true,
    "cert_path": "gophish_admin.crt",
    "key_path":  "gophish_admin.key"
  },
  "phish_server": {
    "listen_url": "0.0.0.0:80",
    "use_tls": false
  },
  "db_name": "sqlite3",
  "db_path": "gophish.db",
  "logging": {
    "filename": "gophish.log"
  }
}
```

Start GoPhish:

```bash
sudo ./gophish
```

Access the admin dashboard at `https://127.0.0.1:3333`. Default credentials: `admin` / `gophish`.

---

#### Step 2 – Configure Sending Profile

A sending profile defines the SMTP server used to send phishing emails.

In the GoPhish dashboard:

1. Navigate to **Sending Profiles → New Profile**.
2. Set **Name:** `Lab SMTP`.
3. Set **From:** `it-support@lab-corp.local`.
4. Set **Host:** `192.168.56.50:25` (internal SMTP relay).
5. Click **Save Profile**.

---

#### Step 3 – Create a Landing Page

The landing page is a clone of the target organisation's login portal that captures submitted credentials.

In the GoPhish dashboard:

1. Navigate to **Landing Pages → New Page**.
2. Use **Import Site** and enter `http://192.168.56.101/dvwa/login.php`.
3. Tick **Capture Submitted Data** and **Capture Passwords**.
4. Set **Redirect to** `https://lab-corp.local/` (redirect after submission).
5. Click **Save Page**.

---

#### Step 4 – Create an Email Template

Design a convincing pretext email. For this simulation, the pretext is a password expiry notification from the IT team.

**Email template (HTML body):**

```html
Dear {{.FirstName}},

Your network password is due to expire in 24 hours.

To avoid disruption to your access, please update your password
by clicking the link below:

<a href="{{.URL}}">Update My Password</a>

If you have any questions, contact the IT Help Desk at
helpdesk@lab-corp.local or ext. 1234.

Kind regards,
IT Support Team
Lab Corp
```

GoPhish template variables:

| Variable | Replaced with |
|----------|--------------|
| `{{.FirstName}}` | Recipient's first name from the target group |
| `{{.URL}}` | Unique tracking URL for each recipient |

---

#### Step 5 – Import Target Group

Create a CSV file of target recipients:

```csv
First Name,Last Name,Email,Position
Alice,Smith,alice.smith@lab-corp.local,Analyst
Bob,Jones,bob.jones@lab-corp.local,Developer
Carol,White,carol.white@lab-corp.local,Manager
David,Brown,david.brown@lab-corp.local,HR Coordinator
Eve,Davis,eve.davis@lab-corp.local,Accountant
```

Import the CSV in **Users & Groups → New Group**.

---

#### Step 6 – Launch the Campaign

1. Navigate to **Campaigns → New Campaign**.
2. Select the email template, landing page, sending profile, and target group.
3. Set **Launch Date** and click **Launch Campaign**.

---

#### Step 7 – Monitor Results

The GoPhish dashboard provides real-time campaign statistics:

| Metric | Count | Percentage |
|--------|-------|-----------|
| Emails sent | 5 | 100% |
| Emails opened | 3 | 60% |
| Links clicked | 2 | 40% |
| Credentials submitted | 1 | 20% |
| Reports to IT | 1 | 20% |

**Timeline:**

```
Day 1 (09:00) – Campaign launched; emails delivered.
Day 1 (09:47) – First email opened (Carol White).
Day 1 (10:12) – First link click (Carol White).
Day 1 (10:13) – Credentials submitted (Carol White).
Day 2 (08:30) – Email forwarded to IT security team (Bob Jones) — reported as phishing.
Day 3 (11:00) – Two more emails opened (Alice Smith, David Brown).
```

---

### Part B – Credential Harvesting with SET (Social-Engineer Toolkit)

SET is a Python-based framework bundled with Kali Linux for social-engineering attacks. This section demonstrates a credential-harvesting attack using SET's Website Attack vector.

#### Step 1 – Launch SET

```bash
sudo setoolkit
```

Navigate through the menus:

```
1) Social-Engineering Attacks
2) Website Attack Vectors
3) Credential Harvester Attack Method
2) Site Cloner
```

#### Step 2 – Clone Target Website

Enter the attacker's IP address and the site to clone:

```
Enter the IP address for the POST back in Harvester/Tabnabbing:
192.168.56.100

Enter the url to clone:
http://192.168.56.101/dvwa/login.php
```

SET will:

1. Clone the login page.
2. Start an HTTP server on port 80.
3. Wait for victims to visit `http://192.168.56.100/` and submit their credentials.

#### Step 3 – Deliver the Link

Send the victim a crafted link via email or instant message:

```
Subject: Urgent – Account Verification Required

Please verify your account immediately:
http://192.168.56.100/
```

#### Step 4 – Capture Credentials

When the victim submits the form, SET displays the captured credentials:

```
[*] WE GOT A HIT! Printing the output:
POSSIBLE USERNAME FIELD FOUND: username=admin
POSSIBLE PASSWORD FIELD FOUND: password=password
[*] WHEN YOU'RE DONE, HIT CONTROL-C TO GENERATE A REPORT.
```

---

## Results Summary

### GoPhish Campaign Results

| Finding | Value |
|---------|-------|
| Email open rate | 60% |
| Link click rate | 40% |
| Credential submission rate | 20% |
| Phishing report rate | 20% |
| Most susceptible department | Finance / HR |

### Key Observations

1. **High open rate (60%)** — The IT-branded password expiry pretext was convincing.
2. **One credential submission (20%)** — One user submitted their credentials without verifying the URL or sender.
3. **One correct report (20%)** — The developer (Bob Jones) correctly identified the email as phishing and reported it, demonstrating effective awareness training.
4. **No malware detection training** — No attachment-based test was run; recommend including this in future simulations.

---

## Remediation Recommendations

| Recommendation | Priority |
|---------------|----------|
| Deploy an email gateway with anti-phishing filters (DMARC, DKIM, SPF) | High |
| Implement multi-factor authentication (MFA) for all accounts | **Critical** |
| Conduct quarterly phishing simulations to measure awareness trends | High |
| Deliver targeted security awareness training to users who clicked | High |
| Establish a clear phishing reporting process (dedicated email/button) | Medium |
| Enable login anomaly alerts to detect credential-stuffing attempts | Medium |

## Phishing Indicators of Compromise (IoCs) Reference

Real-world phishing emails often contain the following indicators:

| Indicator | Example |
|-----------|---------|
| Mismatched sender domain | `support@paypa1.com` (note the `1`) |
| Urgent / threatening language | "Your account will be locked in 24 hours" |
| Generic greeting | "Dear Customer" instead of your name |
| Suspicious hyperlinks | Hovering reveals a different URL to the displayed text |
| Unexpected attachments | `.exe`, `.zip`, `.docm` files |
| Poor grammar / spelling | Unusual phrasing or misspellings |

## Conclusion

The phishing simulation revealed that 40% of targeted employees clicked a malicious link and 20% submitted credentials. While a 20% reporting rate indicates some existing security awareness, the overall results highlight the need for improved training and technical controls such as MFA and email authentication protocols. Regular simulations are essential for maintaining a security-conscious culture.

## References

- GoPhish documentation: https://docs.getgophish.com/
- Social-Engineer Toolkit: https://github.com/trustedsec/social-engineer-toolkit
- SANS Phishing Awareness: https://www.sans.org/security-awareness-training/resources/phishing/
- NIST SP 800-115 (Technical Guide to Information Security Testing): https://csrc.nist.gov/publications/detail/sp/800-115/final
