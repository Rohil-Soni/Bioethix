# ⚡ SECURITY QUICK REFERENCE

## Common Vulnerabilities & Quick Fixes

### 1. Cross-Site Scripting (XSS)

**Problem:** Attacker injects malicious scripts
```html
<!-- ❌ VULNERABLE -->
<p id="message"></p>
<script>
  document.getElementById('message').innerHTML = userInput;
</script>
```

**Fix:**
```javascript
// ✅ SAFE
document.getElementById('message').textContent = userInput;
```

---

### 2. SQL Injection

**Problem:** Attacker modifies SQL query
```sql
-- ❌ VULNERABLE
SELECT * FROM users WHERE email = '' + userEmail + '';
```

**Fix:**
```javascript
// ✅ SAFE - Use parameterized queries
db.query('SELECT * FROM users WHERE email = ?', [userEmail]);
```

---

### 3. Missing Security Headers

**Problem:** Browser doesn't know how to protect page
```html
<!-- ❌ VULNERABLE - No security headers -->
```

**Fix:**
```apache
# ✅ SAFE - In .htaccess
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
Header set Content-Security-Policy "default-src 'self'"
```

---

### 4. Exposed Sensitive Data

**Problem:** Passwords, emails, API keys visible in code
```javascript
// ❌ VULNERABLE
const email = "info@bioethix.com";
const apiKey = "sk_live_123456789";
```

**Fix:**
```javascript
// ✅ SAFE - Use environment variables and obfuscation
const email = process.env.CONTACT_EMAIL;
const encodedEmail = btoa("info@bioethix.com");  // base64
```

---

### 5. Weak Authentication

**Problem:** No password validation or rate limiting
```javascript
// ❌ VULNERABLE
if (password.length > 3) {
    login(user);
}
```

**Fix:**
```javascript
// ✅ SAFE
const MIN_PASSWORD = 12;
const hasUpperCase = /[A-Z]/.test(password);
const hasSpecial = /[!@#$%^&*]/.test(password);

if (password.length >= MIN_PASSWORD && hasUpperCase && hasSpecial) {
    attemptLogin(user);  // With rate limiting
}
```

---

### 6. CSRF (Cross-Site Request Forgery)

**Problem:** Attacker tricks user into making unwanted requests
```html
<!-- ❌ VULNERABLE -->
<form action="/delete-account" method="POST">
    <button>Delete Account</button>
</form>
```

**Fix:**
```html
<!-- ✅ SAFE -->
<form action="/delete-account" method="POST">
    <input type="hidden" name="csrf_token" value="<%= csrfToken %>">
    <button>Delete Account</button>
</form>
```

---

### 7. Insecure Dependencies

**Problem:** Using packages with known vulnerabilities
```bash
# ❌ VULNERABLE - Old version with CVE
npm install vulnerable-package@1.0.0
```

**Fix:**
```bash
# ✅ SAFE - Check and update
npm audit
npm audit fix
npm install vulnerable-package@latest
```

---

## Security Command Cheat Sheet

```bash
# 🔍 CHECK SECURITY
curl -I https://bioethix.com | grep -i "security\|x-frame\|csp"

# 🔐 TEST SSL/TLS
openssl s_client -connect bioethix.com:443 -tls1_2

# 🐛 SCAN FOR VULNERABILITIES
npm audit
npm audit fix
python -m safety check

# 📋 CHECK FOR EXPOSED SECRETS
git log -p | grep -i "password\|apikey\|secret"
grep -r "password\s*=" . --include="*.js" --include="*.env"

# 🌐 VALIDATE HEADERS
curl -s -I https://bioethix.com | grep -E "^[A-Z]"

# 📧 CHECK EMAIL EXPOSURE
curl https://bioethix.com | grep -i "@bioethix"

# 🔄 TEST REDIRECTS
curl -L -I http://bioethix.com

# 🛡️ OWASP ZAP SCAN
zaproxy -cmd -quickurl https://bioethix.com -report scan-report.html
```

---

## Security Settings by File Type

### .htaccess (Apache)
```apache
# Prevent access to sensitive files
<FilesMatch "\.(env|git|htaccess)$">
    Order allow,deny
    Deny from all
</FilesMatch>

# Add security headers
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
Header set X-XSS-Protection "1; mode=block"

# Enable gzip compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html
</IfModule>

# Cache control
<FilesMatch "\.(jpg|jpeg|png|gif|css|js)$">
    Header set Cache-Control "max-age=31536000, public"
</FilesMatch>
```

### nginx
```nginx
# Add security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Prevent access to hidden files
location ~ /\. {
    deny all;
}

# Enable gzip compression
gzip on;
gzip_types text/plain text/css application/json;
```

### Node.js/Express
```javascript
const helmet = require('helmet');
const app = require('express')();

// Use helmet for security headers
app.use(helmet());

// Set specific headers
app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    next();
});

// Rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
});
app.use(limiter);
```

---

## HTTP Security Headers Reference

| Header | Value | Purpose |
|--------|-------|---------|
| **X-Frame-Options** | SAMEORIGIN | Prevent clickjacking |
| **X-Content-Type-Options** | nosniff | Prevent MIME sniffing |
| **X-XSS-Protection** | 1; mode=block | XSS attack mitigation |
| **Content-Security-Policy** | default-src 'self' | Prevent XSS |
| **Strict-Transport-Security** | max-age=31536000 | Force HTTPS |
| **Referrer-Policy** | strict-origin-when-cross-origin | Control referrer info |
| **Permissions-Policy** | geolocation=(), camera=() | Restrict features |

---

## Testing URLs for BioEthix

```bash
# Test main pages
curl https://bioethix.com
curl https://bioethix.com/about.html
curl https://bioethix.com/products.html
curl https://bioethix.com/contact.html

# Check for email exposure
curl https://bioethix.com | grep -o "[a-zA-Z0-9._%+-]*@bioethix.com"

# Check for sensitive files
curl https://bioethix.com/.env
curl https://bioethix.com/.git/config
curl https://bioethix.com/web.config

# Verify robots.txt
curl https://bioethix.com/robots.txt

# Check API responses
curl -X POST https://bioethix.com/api/contact -H "Content-Type: application/json" \
  -d '{"message":"test"}'

# Test rate limiting
for i in {1..100}; do curl https://bioethix.com; done
```

---

## Before Every Commit

```bash
# ✅ SECURITY CHECKLIST

# 1. Check for secrets
grep -r "password\|api_key\|secret" . --include="*.js" --include="*.py"

# 2. Check for console logs with sensitive data
grep -r "console.log\|print(" . --include="*.js" --include="*.py" | \
  grep -i "password\|email\|token"

# 3. Check for eval() or similar
grep -r "eval\|Function(" . --include="*.js"

# 4. Ensure .env is gitignored
cat .gitignore | grep -q "^\.env$" || echo "❌ .env not in .gitignore"

# 5. Run security audit
npm audit

# 6. Check for hardcoded URLs
grep -r "http://\|ftp://" . --include="*.js" | grep -v "^Binary" | grep -v "//"
```

---

## Emergency Security Response

### If Compromise Suspected:

```bash
# 1. STOP - Take site offline
systemctl stop apache2  # or nginx

# 2. PRESERVE - Don't modify anything
find . -type f -newermt "2 hours ago" -ls > suspicious_files.txt

# 3. INVESTIGATE - Check logs
tail -1000 /var/log/apache2/access.log > access.log.backup
tail -1000 /var/log/apache2/error.log > error.log.backup

# 4. SCAN - Look for backdoors
grep -r "eval\|system\|passthru\|shell_exec" . --include="*.php" --include="*.js"
find . -type f -name "*.php" -o -name "*.jsp" -o -name "*.aspx" | xargs file

# 5. RESTORE - Restore from clean backup
rm -rf /var/www/html/*
tar -xzf /backups/website-clean.tar.gz -C /var/www/html/

# 6. SECURE - Fix vulnerability
# [Apply patches here]

# 7. VALIDATE - Run security checks
./verify-security.sh https://bioethix.com
```

---

## Security Incident Report Template

```
SECURITY INCIDENT REPORT
========================

Date Discovered: [DATE]
Severity: [CRITICAL/HIGH/MEDIUM/LOW]

Description:
[What happened?]

Impact:
[Who is affected? What data?]

Root Cause:
[Why did it happen?]

Actions Taken:
- [ ] System taken offline
- [ ] Investigation completed
- [ ] Users notified
- [ ] Vulnerability patched
- [ ] Backup verified
- [ ] Monitoring enhanced

Timeline:
- [TIME] - Issue discovered
- [TIME] - Investigation started
- [TIME] - Fix implemented
- [TIME] - System restored

Lessons Learned:
[What will prevent this in future?]

Signed by: [SECURITY OFFICER]
Date: [DATE]
```

---

## Quick Security Score

Score yourself:

- ✅ HTTPS enabled: +10 points
- ✅ Security headers configured: +10 points
- ✅ Email obfuscated: +5 points
- ✅ No exposed secrets: +10 points
- ✅ npm audit clean: +10 points
- ✅ Strong password policy: +10 points
- ✅ Regular backups: +10 points
- ✅ Monitoring enabled: +10 points
- ✅ Incident plan documented: +10 points
- ✅ Security training current: +5 points

**Your Score: ___/100**

*Target: 80+ for production*

---

**Last Updated:** May 25, 2026
