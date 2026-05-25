# 🛡️ DEVELOPER SECURITY GUIDELINES

## Overview
This guide provides security best practices for the BioEthix Pharmaceuticals website development team.

---

## 1. SECURE CODING PRACTICES

### Input Validation
```javascript
// ❌ BAD: No validation
function handleUserInput(input) {
    document.innerHTML = input;
}

// ✅ GOOD: Sanitize input
function handleUserInput(input) {
    const div = document.createElement('div');
    div.textContent = input;  // Uses text content, not HTML
    return div.innerHTML;
}
```

### Protecting Sensitive Data
```javascript
// ❌ BAD: Exposing in plain text
const apiKey = "sk_live_123456789";

// ✅ GOOD: Use environment variables and encoding
const apiKey = process.env.API_KEY;  // Server-side only
const encodedEmail = btoa("info@bioethix.com");  // Base64 for client-side
```

### Preventing XSS (Cross-Site Scripting)
```javascript
// ❌ BAD: Using innerHTML with user data
element.innerHTML = userProvidedHTML;

// ✅ GOOD: Use textContent or createElement
element.textContent = userData;
// or
const el = document.createElement('div');
el.textContent = userData;
```

### Preventing CSRF (Cross-Site Request Forgery)
```html
<!-- ❌ BAD: No protection -->
<form action="/update-profile" method="POST">
    <input type="email" name="email">
</form>

<!-- ✅ GOOD: Include CSRF token -->
<form action="/update-profile" method="POST">
    <input type="hidden" name="csrf_token" value="randomly_generated_token">
    <input type="email" name="email">
</form>
```

---

## 2. SECURE DEPENDENCIES

### Check for Vulnerabilities
```bash
# Check npm packages
npm audit
npm audit fix

# Check Python packages
pip install safety
safety check

# Check for known CVEs
npm outdated
```

### Use Trusted Sources Only
- ✅ npm.org (official registry)
- ✅ GitHub (for verified packages)
- ✅ Official CDN links (Unpkg, jsDelivr)
- ❌ Random GitHub forks
- ❌ Unofficial package sources

---

## 3. AUTHENTICATION & AUTHORIZATION

### Never Hardcode Credentials
```javascript
// ❌ BAD
const password = "admin123";
const dbConnection = "user:pass@localhost";

// ✅ GOOD
const password = process.env.ADMIN_PASSWORD;
const dbConnection = process.env.DATABASE_URL;
```

### Password Policy
- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, symbols
- Never send passwords in emails
- Use bcrypt for hashing (minimum 12 rounds)
- Implement rate limiting on login attempts

### Session Management
```javascript
// ✅ GOOD: Secure session handling
app.use(session({
    secret: process.env.SESSION_SECRET,
    cookie: { 
        secure: true,  // HTTPS only
        httpOnly: true,  // No JavaScript access
        sameSite: 'strict'  // CSRF protection
    }
}));
```

---

## 4. SECURE COMMUNICATION

### HTTPS/TLS Requirements
- 🔒 Must use TLS 1.2 or higher
- 🔒 Use certificates from trusted CAs
- 🔒 Enable HSTS (Strict-Transport-Security)
- 🔒 Redirect all HTTP to HTTPS

### Certificate Management
```bash
# Check SSL certificate expiry
openssl s_client -connect example.com:443 -showcerts | grep "notAfter"

# Generate self-signed cert for dev
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

### API Security
```javascript
// ✅ GOOD: Secure API calls
async function fetchData() {
    const response = await fetch('https://api.example.com/data', {
        method: 'GET',
        headers: {
            'Authorization': `Bearer ${apiToken}`,
            'Content-Type': 'application/json'
        }
    });
    return response.json();
}
```

---

## 5. FILE HANDLING

### Secure File Upload
```javascript
// ✅ GOOD: Validate and restrict file uploads
const allowedMimeTypes = ['image/jpeg', 'image/png'];
const maxFileSize = 5 * 1024 * 1024;  // 5MB

function validateUpload(file) {
    if (!allowedMimeTypes.includes(file.type)) {
        throw new Error('Invalid file type');
    }
    if (file.size > maxFileSize) {
        throw new Error('File too large');
    }
    return true;
}
```

### Secure File Storage
- Never store uploaded files in web-accessible directories
- Use random filenames to prevent enumeration
- Validate file contents, not just extension
- Use virus scanning for production

---

## 6. DATABASE SECURITY

### Prevent SQL Injection
```sql
-- ❌ BAD: String concatenation
SELECT * FROM users WHERE email = '' + userInput + '';

-- ✅ GOOD: Parameterized queries
SELECT * FROM users WHERE email = ?
// Parameter: userInput
```

### Database Access Control
- Use least privilege principle
- Different credentials for read/write operations
- Never store passwords in code
- Encrypt sensitive database fields
- Regular backup and recovery testing

---

## 7. ENVIRONMENT CONFIGURATION

### .env File Management
```bash
# .env (NEVER commit this file)
DATABASE_URL=postgresql://user:pass@localhost/db
API_KEY=sk_live_123456
SESSION_SECRET=random_secret_key

# .env.example (Safe to commit)
DATABASE_URL=postgresql://user:pass@localhost/db
API_KEY=your_api_key_here
SESSION_SECRET=your_session_secret
```

### .gitignore Setup
```
.env
.env.local
.env.*.local
node_modules/
.vscode/
dist/
build/
*.log
.DS_Store
```

---

## 8. LOGGING & MONITORING

### Secure Logging
```javascript
// ❌ BAD: Logging sensitive data
console.log(`User login: ${password}`);
console.log(`Card number: ${cardNumber}`);

// ✅ GOOD: Log safely
console.log(`User login attempt for: ${email}`);
console.log(`Card ending in: ${cardNumber.slice(-4)}`);
```

### What to Monitor
- Login attempts (especially failures)
- Permission changes
- Admin actions
- API rate limiting
- Error patterns
- Unusual database queries

---

## 9. DEPENDENCY & LIBRARY UPDATES

### Regular Updates Schedule
```bash
# Check for outdated packages
npm outdated

# Update packages safely
npm update  # Minor/patch updates
npm install package@latest  # Major updates

# Security patches only
npm audit fix
```

### Before Committing Dependencies
1. Check package reputation
2. Review package.json changes
3. Run security audit
4. Test the application
5. Commit lock file (package-lock.json)

---

## 10. THIRD-PARTY INTEGRATIONS

### Vet Before Integration
- ✅ Check GitHub stars and recent activity
- ✅ Review security audit history
- ✅ Check for known CVEs
- ✅ Test in staging environment
- ❌ Use unmaintained or suspicious packages

### Subresource Integrity (SRI)
```html
<!-- ✅ GOOD: Verify external scripts -->
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-ABC123..."
        crossorigin="anonymous"></script>
```

Generate SRI hash:
```bash
curl https://cdn.example.com/lib.js | openssl dgst -sha384 -binary | base64
```

---

## 11. SECURITY HEADERS CHECKLIST

| Header | Purpose | Value |
|--------|---------|-------|
| Content-Security-Policy | Prevent XSS | `default-src 'self'` |
| X-Frame-Options | Prevent clickjacking | `SAMEORIGIN` |
| X-Content-Type-Options | Prevent MIME sniffing | `nosniff` |
| Strict-Transport-Security | Force HTTPS | `max-age=31536000` |
| Referrer-Policy | Control referrer | `strict-origin-when-cross-origin` |
| Permissions-Policy | Restrict features | `geolocation=(), microphone=()` |

---

## 12. INCIDENT RESPONSE

### If Breach is Suspected
1. **Immediate** - Take system offline if necessary
2. **Investigation** - Determine scope and impact
3. **Notification** - Alert stakeholders and affected users
4. **Containment** - Limit spread of compromise
5. **Eradication** - Remove malicious code
6. **Recovery** - Restore from clean backup
7. **Post-Incident** - Review and improve security

### Emergency Contacts
```
Security Team Lead: [Contact Info]
CTO: [Contact Info]
Legal: [Contact Info]
Public Relations: [Contact Info]
```

---

## 13. TESTING SECURITY

### Manual Testing
```bash
# Check security headers
curl -I https://bioethix.com

# SQL injection test
curl "https://bioethix.com/search?q='; DROP TABLE users;--"

# XSS test
curl -X POST -d 'name=<script>alert(1)</script>' https://bioethix.com/api
```

### Automated Testing
```bash
# OWASP ZAP
zaproxy -cmd -quickurl https://bioethix.com

# npm audit
npm audit

# Snyk
snyk test --all-projects
```

### Professional Penetration Testing
Schedule quarterly with certified security firm focusing on:
- Web application penetration testing
- Infrastructure security
- Social engineering assessment
- Vulnerability scanning

---

## 14. COMPLIANCE & STANDARDS

### Applicable Standards
- 🏥 **HIPAA** - Health Insurance Portability and Accountability Act
- 🌍 **GDPR** - General Data Protection Regulation (if serving EU users)
- 🔒 **ISO 27001** - Information Security Management
- 🏭 **IEC 62443** - Industrial Cybersecurity
- 🇮🇳 **India Data Protection Act** - Personal data protection
- 📱 **OWASP Top 10** - Web application security risks

### Documentation Requirements
- [ ] Privacy Policy
- [ ] Terms of Service
- [ ] Security Policy
- [ ] Incident Response Plan
- [ ] Data Retention Policy
- [ ] Breach Notification Policy

---

## 15. SECURITY CHECKLIST FOR DEPLOYMENT

### Before Going Live
- [ ] Enable HTTPS/TLS with valid certificate
- [ ] Security headers configured
- [ ] Email obfuscated (base64 encoding)
- [ ] No hardcoded secrets or credentials
- [ ] .env file not committed to git
- [ ] npm audit passing (no high/critical vulnerabilities)
- [ ] Database backups tested
- [ ] Monitoring and alerting enabled
- [ ] Incident response plan documented
- [ ] Legal/compliance review complete

### Regular Maintenance
- [ ] Weekly: Check logs and monitoring alerts
- [ ] Monthly: Dependency security updates
- [ ] Quarterly: Penetration testing
- [ ] Annually: Full security audit and compliance review

---

## 16. USEFUL RESOURCES

### Security Tools
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP ZAP](https://www.owasp.org/index.php/ZAP)
- [npm audit](https://docs.npmjs.com/cli/audit)
- [Snyk](https://snyk.io/)
- [SSL Labs](https://www.ssllabs.com/)

### Learning Resources
- OWASP WebGoat (interactive training)
- Cybrary (free security courses)
- HackTheBox (CTF challenges)
- TryHackMe (security labs)

### Standards & Compliance
- [ISO 27001](https://www.iso.org/isoiec-27001-information-security-management.html)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CWE Top 25](https://cwe.mitre.org/top25/)

---

## 17. DEVELOPER RESPONSIBILITIES

### Every Developer Must:
1. ✅ Follow secure coding practices
2. ✅ Keep dependencies updated
3. ✅ Review security advisories
4. ✅ Never commit secrets
5. ✅ Sanitize user input
6. ✅ Use HTTPS in all connections
7. ✅ Report security issues immediately
8. ✅ Participate in security training
9. ✅ Review security checklists
10. ✅ Test security implications

### Report Security Issues
🚨 Found a vulnerability?
- DO NOT post publicly
- Email: security@bioethix.com
- Include: Description, severity, reproduction steps
- Expect response within 24 hours

---

**Last Updated:** May 25, 2026  
**Next Review:** June 25, 2026

---

*This guide is a living document. Updates reflect emerging threats and best practices.*
