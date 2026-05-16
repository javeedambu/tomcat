# Tomcat 9 / Remedy AR System 20.02 Security Header Hardening Guide

## Overview

This guide documents the remediation steps for missing HTTP security headers identified during a Qualys vulnerability scan against a Windows server running:

* Apache Tomcat 9
* BMC Remedy AR System 20.02
* HTTPS on port 443

The goal is to improve browser-side security protections and reduce vulnerability findings without disrupting Remedy functionality.

---

# Environment Details

| Component        | Details                    |
| ---------------- | -------------------------- |
| Web Server       | Apache Tomcat 9            |
| Application      | BMC Remedy AR System 20.02 |
| Operating System | Microsoft Windows          |
| Protocol         | HTTPS                      |
| Port             | 443                        |
| Scanner          | Qualys                     |

---

# Vulnerability Summary

## Qualys Finding

| Item            | Value                             |
| --------------- | --------------------------------- |
| QID             | 11827                             |
| Severity        | 2                                 |
| Title           | HTTP Security Header Not Detected |
| Affected Header | X-Content-Type-Options            |

Additional missing security headers may also be detected during the scan.

---

# Risk Description

Missing HTTP security headers may allow browsers to:

* Perform MIME type sniffing
* Allow clickjacking attacks
* Reduce protection against reflected cross-site scripting (XSS)
* Ignore strict HTTPS enforcement

While these findings are typically classified as low or medium severity, implementing the headers is considered security best practice.

---

# Headers Implemented

The following security headers are recommended and implemented:

| Header                    | Purpose                            |
| ------------------------- | ---------------------------------- |
| X-Content-Type-Options    | Prevent MIME type sniffing         |
| X-Frame-Options           | Protect against clickjacking       |
| X-XSS-Protection          | Legacy browser XSS protection      |
| Strict-Transport-Security | Force HTTPS usage                  |
| Content-Security-Policy   | Restrict content execution sources |

---

# Important Remedy Considerations

BMC Remedy AR System uses:

* Inline JavaScript
* Dynamic content rendering
* Embedded resources
* Browser-side scripting

Because of this, an overly restrictive Content Security Policy (CSP) may break application functionality.

A relaxed CSP configuration is recommended initially.

---

# Remediation Steps

## Step 1 — Backup Existing Configuration

Before making changes:

1. Stop Tomcat service
2. Backup configuration files:

```text
<Tomcat>\conf\web.xml
<Tomcat>\conf\server.xml
```

Example:

```text
copy web.xml web.xml.bak
copy server.xml server.xml.bak
```

---

# Step 2 — Configure Security Headers Filter

Edit:

```text
<Tomcat>\conf\web.xml
```

Add the following configuration inside the `<web-app>` section.

```xml
<filter>
  <filter-name>securityHeadersFilter</filter-name>
  <filter-class>org.apache.catalina.filters.HttpHeaderSecurityFilter</filter-class>

  <!-- Prevent MIME sniffing -->
  <init-param>
    <param-name>xContentTypeOptions</param-name>
    <param-value>nosniff</param-value>
  </init-param>

  <!-- Clickjacking protection -->
  <init-param>
    <param-name>antiClickJackingEnabled</param-name>
    <param-value>true</param-value>
  </init-param>

  <init-param>
    <param-name>antiClickJackingOption</param-name>
    <param-value>SAMEORIGIN</param-value>
  </init-param>

  <!-- Legacy browser XSS protection -->
  <init-param>
    <param-name>xssProtectionEnabled</param-name>
    <param-value>true</param-value>
  </init-param>

  <!-- Enable HSTS -->
  <init-param>
    <param-name>hstsEnabled</param-name>
    <param-value>true</param-value>
  </init-param>

  <init-param>
    <param-name>hstsMaxAgeSeconds</param-name>
    <param-value>31536000</param-value>
  </init-param>
</filter>

<filter-mapping>
  <filter-name>securityHeadersFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

---

# Step 3 — Configure Content Security Policy (CSP)

Edit:

```text
<Tomcat>\conf\server.xml
```

Inside the `<Host>` section add:

```xml
<Valve className="org.apache.catalina.valves.HttpHeaderSecurityValve"
       contentSecurityPolicy="default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: https:;" />
```

## Notes

This CSP is intentionally relaxed to maintain compatibility with Remedy AR System.

A stricter CSP should only be implemented after application testing.

---

# Step 4 — Restart Tomcat

Restart the Tomcat service.

Example:

```text
services.msc
```

Or:

```text
net stop Tomcat9
net start Tomcat9
```

---

# Step 5 — Validate Headers

Run the following command from a system with curl installed:

```bash
curl -k -I https://your-server-name
```

Expected headers:

```text
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: https:
```

---

# Validation Checklist

| Validation Item                    | Status |
| ---------------------------------- | ------ |
| web.xml updated                    | ☐      |
| server.xml updated                 | ☐      |
| Backup completed                   | ☐      |
| Tomcat restarted                   | ☐      |
| Application functionality tested   | ☐      |
| Headers verified with curl/browser | ☐      |
| Qualys rescan completed            | ☐      |

---

# Post-Implementation Testing

After implementing the changes:

1. Verify Remedy login functionality
2. Verify dashboards and forms load correctly
3. Verify no browser console CSP errors exist
4. Verify APIs and integrations still function
5. Run a new Qualys scan

---

# Rollback Procedure

If issues occur:

1. Restore backup files
2. Restart Tomcat

Example:

```text
copy web.xml.bak web.xml
copy server.xml.bak server.xml
```

Then restart Tomcat.

---

# Expected Qualys Improvements

The following findings should be reduced or eliminated:

| Finding                        | Expected Result     |
| ------------------------------ | ------------------- |
| Missing X-Content-Type-Options | Resolved            |
| Missing X-Frame-Options        | Resolved            |
| Missing X-XSS-Protection       | Resolved            |
| Missing HSTS                   | Resolved            |
| Missing CSP                    | Resolved or Reduced |

---

# Recommended Next Hardening Steps

After completing header remediation:

1. Disable TLS 1.0 and TLS 1.1
2. Configure strong TLS cipher suites
3. Remove default Tomcat pages and examples
4. Suppress Tomcat version disclosure
5. Enable Secure and HttpOnly cookie flags
6. Review weak SSL/TLS findings from Qualys
7. Restrict administrative interfaces

---

# Change Management Recommendations

## Suggested Change Window

Implement changes during a scheduled maintenance window to minimize potential impact to Remedy users.

Recommended activities:

1. Notify application owners and support teams
2. Confirm backup availability
3. Validate rollback procedures
4. Perform changes in lower environments first when possible

---

# Recommended Implementation Order

| Order | Activity                      |
| ----- | ----------------------------- |
| 1     | Backup configuration files    |
| 2     | Apply web.xml changes         |
| 3     | Apply server.xml changes      |
| 4     | Restart Tomcat                |
| 5     | Validate Remedy functionality |
| 6     | Validate headers              |
| 7     | Run Qualys rescan             |

---

# Troubleshooting Guide

## Issue: Remedy pages fail to load

### Possible Cause

Content Security Policy is too restrictive.

### Resolution

Temporarily relax CSP policy:

```xml
contentSecurityPolicy="default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: https:;"
```

Restart Tomcat and retest.

---

## Issue: Browser console CSP errors

### Resolution

Open browser developer tools and review:

* Console tab
* Network tab

Identify blocked domains or scripts and update CSP accordingly.

---

## Issue: Headers not appearing

### Possible Causes

* Tomcat not restarted
* Reverse proxy overwriting headers
* Configuration placed outside `<web-app>` section

### Resolution

1. Verify Tomcat restart completed successfully
2. Verify XML syntax is valid
3. Verify reverse proxy configuration
4. Test directly against Tomcat if possible

---

# Security Impact Summary

Implementing these headers improves:

* Browser-side attack resistance
* Clickjacking protection
* MIME type enforcement
* HTTPS enforcement
* Baseline web application hardening

These changes also improve overall vulnerability scan posture and reduce recurring Qualys findings.

---

# Executive Summary

A Qualys scan identified missing HTTP security headers on the Apache Tomcat / Remedy AR System environment.

The implemented remediation introduces industry-standard HTTP response headers to strengthen browser security protections while maintaining compatibility with Remedy AR System 20.02.

The recommended configuration:

* Resolves low-severity Qualys findings
* Improves secure browser behavior
* Introduces HSTS enforcement for HTTPS
* Adds baseline Content Security Policy protection
* Maintains compatibility with Remedy application functionality

The implementation includes rollback procedures, validation guidance, and operational testing recommendations.

---

# References

* Apache Tomcat Security Documentation
* BMC Remedy AR System Documentation
* Qualys Vulnerability KnowledgeBase
* OWASP Secure Headers Project
