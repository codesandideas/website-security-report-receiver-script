# Security Scan Report Receiver
A standalone PHP endpoint that receives security scan reports via POST and delivers them via email with beautiful HTML formatting. Designed to work with web security scanners to centralize vulnerability reporting.
## 🎯 Features
- **📧 Email Delivery**: Sends beautifully formatted HTML emails with scan results
- **🔐 API Key Authentication**: Secure endpoint with timing-safe API key comparison
- **📊 Rich Report Formatting**: Converts Markdown reports to styled HTML with syntax highlighting
- **💾 Local Storage**: Optionally saves reports to disk for archival
- **🚦 Rate Limiting**: Built-in file-based rate limiting to prevent abuse
- **🌐 IP Whitelisting**: Optional IP-based access control
- **📮 SMTP Support**: Built-in SMTP client (no dependencies) or PHP mail()
- **🎨 Responsive Design**: Mobile-friendly email templates with severity-based color coding
- **📎 Attachments**: Includes Markdown report as email attachment
- **📝 Comprehensive Logging**: Tracks all requests, successes, and failures
## 📋 Requirements
- PHP 7.4 or higher
- Web server (Apache, Nginx, or similar)
- Mail server or SMTP credentials (for email delivery)
## 🚀 Installation
1. **Upload the files** to your web server:
   ```bash
   scp receiver.php .env.example user@yourserver.com:/var/www/html/security-receiver/
   ```
2. **Set file permissions** so only the web server can read them:
   ```bash
   chown ubuntu:www-data /var/www/html/security-receiver/receiver.php
   chmod 640 /var/www/html/security-receiver/receiver.php
   ```
3. **Create and configure your `.env` file** (see Configuration section below):
   ```bash
   cp .env.example .env
   nano .env
   chown ubuntu:www-data .env
   chmod 640 .env
   ```
4. **Create the reports directory** (if using local storage):
   ```bash
   mkdir security-reports
   chown ubuntu:www-data security-reports
   chmod 770 security-reports
   ```
## ⚙️ Configuration
All credentials and settings are stored in a `.env` file — **never commit this file to version control**.

Copy the example file and edit it:
```bash
cp .env.example .env
```

### `.env` reference
```dotenv
# Authentication — use a long random string
API_KEY=CHANGE_THIS_TO_A_STRONG_RANDOM_KEY

# Email sender identity
FROM_NAME=Security Scanner
FROM_EMAIL=scanner@yourdomain.com
DEFAULT_RECIPIENT=admin@yourdomain.com

# SMTP — set SMTP_ENABLED=true to use SMTP, false to fall back to PHP mail()
SMTP_ENABLED=false
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_ENCRYPTION=tls
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
```

### SMTP providers

| Provider | Host | Port | Encryption |
|----------|------|------|------------|
| Gmail | `smtp.gmail.com` | `587` | `tls` |
| ZeptoMail | `smtp.zeptomail.in` | `465` | `ssl` |
| Mailgun | `smtp.mailgun.org` | `587` | `tls` |
| SendGrid | `smtp.sendgrid.net` | `587` | `tls` |

**Gmail**: Enable 2FA then generate an [App Password](https://support.google.com/accounts/answer/185833) — use that as `SMTP_PASSWORD`, not your account password.

### Security & storage settings
These are set directly in `receiver.php` (not sensitive, safe to commit):
```php
'allowed_ips'    => [],   // Empty = allow all. Ex: ['1.2.3.4', '5.6.7.8']
'rate_limit'     => 10,   // Max requests per hour per IP
'max_payload_mb' => 10,   // Max POST size in MB
'save_reports'   => true, // Save reports to disk
```
## 📤 Usage
### Testing the Endpoint
Test that the endpoint is working:
```bash
curl -X GET https://yourdomain.com/security-receiver.php
```
Expected response:
```json
{
  "status": "success",
  "message": "Security Scan Report Receiver is running",
  "version": "1.0"
}
```
### Sending a Test Report
```bash
curl -X POST https://yourdomain.com/security-receiver.php \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_SECRET_KEY",
    "email": "admin@example.com",
    "hostname": "web01.example.com",
    "scan_target": "/var/www/html",
    "risk_level": "MEDIUM",
    "frameworks": "WordPress 6.4, PHP 8.1",
    "total_issues": 5,
    "critical": 0,
    "high": 1,
    "medium": 3,
    "low": 1,
    "info": 0,
    "report": "# Security Scan Report\n\n## Summary\n\nFound 5 security issues.\n\n## Issues\n\n- SQL Injection risk in login.php\n- Outdated library detected"
  }'
```
### Integration with Security Scanners
Example usage with a hypothetical security scanner:
```bash
sudo bash web-security-scanner.sh /var/www/html \
  --webhook https://yourdomain.com/security-receiver.php \
  --api-key YOUR_SECRET_KEY \
  --email admin@yoursite.com
```
## 📡 API Reference
### Endpoint: POST /
Receives and processes security scan reports.
#### Request Headers
```
Content-Type: application/json
```
#### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `api_key` | string | ✅ | Authentication key (must match config) |
| `report` | string | ✅ | Markdown-formatted security report |
| `email` | string | ⚪ | Recipient email (falls back to default_recipient) |
| `hostname` | string | ⚪ | Server hostname |
| `scan_target` | string | ⚪ | Path or URL that was scanned |
| `risk_level` | string | ⚪ | Overall risk: CRITICAL, HIGH, MEDIUM, LOW, CLEAN |
| `frameworks` | string | ⚪ | Detected frameworks/technologies |
| `total_issues` | integer | ⚪ | Total number of issues found |
| `critical` | integer | ⚪ | Number of critical issues |
| `high` | integer | ⚪ | Number of high severity issues |
| `medium` | integer | ⚪ | Number of medium severity issues |
| `low` | integer | ⚪ | Number of low severity issues |
| `info` | integer | ⚪ | Number of informational items |
#### Response
**Success (200)**:
```json
{
  "status": "success",
  "message": "Report received and emailed successfully",
  "emailed_to": "admin@example.com",
  "risk_level": "MEDIUM",
  "issues": 5,
  "saved": true
}
```
**Authentication Error (401)**:
```json
{
  "status": "error",
  "message": "Invalid API key"
}
```
**Rate Limited (429)**:
```json
{
  "status": "error",
  "message": "Rate limit exceeded. Try again later."
}
```
**Partial Success (207)**: Report saved but email failed
```json
{
  "status": "success",
  "message": "Report saved locally but email failed",
  "error": "SMTP connection timeout",
  "saved": true
}
```
## 🎨 Email Template Features
The generated emails include:
- **Modern, Responsive Design**: Works on desktop and mobile
- **Risk Level Badges**: Color-coded (CRITICAL=red, HIGH=orange, MEDIUM=yellow, LOW=blue, CLEAN=green)
- **Statistics Dashboard**: Visual summary of issues by severity
- **Markdown Rendering**: Supports headers, lists, tables, code blocks, and more
- **Syntax Highlighting**: Code blocks with dark theme styling
- **Tables**: Automatically styled with alternating rows
- **Expandable Sections**: Support for HTML `<details>` elements
- **Markdown Attachment**: Full report included as `.md` file
## 🔍 Supported Markdown Features
The report converter supports:
- **Headers** (`#`, `##`, `###`, `####`)
- **Bold** (`**text**`) and *Italic* (`*text*`)
- **Code blocks** (` ```language ... ``` `)
- **Inline code** (`` `code` ``)
- **Tables** (GitHub-flavored)
- **Lists** (ordered and unordered)
- **Checkboxes** (`- [ ]` and `- [x]`)
- **Blockquotes** (`> text`) - styled as recommendation boxes
- **Links** (`[text](url)`)
- **Horizontal rules** (`---`)
- **Details/Summary** (`<details><summary>...</summary>...</details>`)
## 🛡️ Security Considerations
1. **Never commit `.env`** — it contains your API key and SMTP password. It is listed in `.gitignore` by default
2. **Use a strong API key** — generate one with `openssl rand -hex 32`
3. **Use HTTPS** to encrypt data in transit
4. **Restrict file permissions**: `chmod 640` on `receiver.php` and `.env`, with group set to your web server user
5. **Enable IP whitelisting** if your scanner runs from a fixed IP
6. **Review rate limits** based on your expected traffic
7. **Secure the reports directory** with `.htaccess` (automatically created)
8. **Monitor logs** regularly for suspicious activity
9. **Use App Passwords** for Gmail SMTP (not your account password)
## 📊 Logging
Logs are stored in the file specified by `$config['log_file']` (default: `security-receiver.log`)
Log format:
```
[2024-02-19 10:30:45] [INFO] [192.168.1.100] Report received: web01.example.com | Risk: MEDIUM | Issues: 5 (C:0 H:1 M:3)
[2024-02-19 10:30:46] [INFO] [192.168.1.100] Email sent to admin@example.com
[2024-02-19 10:30:46] [INFO] [192.168.1.100] Saved: scan_web01_2024-02-19_10-30-45.md
```
## 🐛 Troubleshooting
### 500 Internal Server Error
**Problem**: Script returns 500 on any request
**Solutions**:
1. Check file permissions — the web server user must be able to read `receiver.php`:
   ```bash
   chown ubuntu:www-data receiver.php && chmod 640 receiver.php
   ```
2. Check server error logs: `tail -f /var/log/apache2/error.log`
3. Verify PHP syntax: `php -l receiver.php`

### Email Not Sending
**Problem**: Email delivery fails
**Solutions**:
1. Confirm `.env` exists and `SMTP_ENABLED=true` is set
2. Verify SMTP credentials in `.env` are correct
3. Check error logs: `tail -f security-receiver.log`
4. For Gmail: Ensure you're using an App Password, not your account password
5. Test SMTP connectivity: `openssl s_client -connect smtp.yourhost.com:465`

### 401 Unauthorized
**Problem**: API key rejected
**Solutions**:
1. Confirm `API_KEY` in `.env` matches the key sent in the request (case-sensitive)
2. Check for leading/trailing whitespace in `.env`
3. Review logs for authentication attempts

### 429 Rate Limited
**Problem**: Too many requests
**Solutions**:
1. Increase `rate_limit` in `receiver.php`
2. Wait one hour for the rate limit window to reset
3. Check `/tmp/secrecv_rate_*` files for rate limit state

### Reports Not Saved
**Problem**: `saved: false` in response
**Solutions**:
1. Ensure `save_reports` is `true` in `receiver.php`
2. Fix directory ownership and permissions:
   ```bash
   chown ubuntu:www-data security-reports && chmod 770 security-reports
   ```
3. Check disk space: `df -h`

### IP Blocked
**Problem**: 403 Forbidden
**Solutions**:
1. Add your IP to the `allowed_ips` array in `receiver.php`
2. Set `allowed_ips` to `[]` to allow all IPs
3. Check proxy/load balancer config for correct IP forwarding
## 📝 File Storage Structure
When `save_reports` is enabled, reports are saved as:
```
security-reports/
├── .htaccess (auto-generated, denies web access)
├── scan_web01_2024-02-19_10-30-45.md
├── scan_web02_2024-02-19_11-15-23.md
└── scan_database_2024-02-19_14-22-10.md
```
## 🔄 Version History
- **v1.0** - Initial release
  - Email delivery with HTML formatting
  - API key authentication
  - Rate limiting
  - Local storage
  - SMTP support
  - Markdown to HTML conversion
## 📄 License
This script is provided as-is for security monitoring purposes. Review and customize according to your needs.
## 🤝 Support
For issues or questions:
1. Check the troubleshooting section above
2. Review the log file for error details
3. Verify all configuration settings
4. Test with the provided curl examples
---
**Generated by**: Universal Web Security Scanner v3.0  
**Note**: Always manually review flagged security items before taking action.
