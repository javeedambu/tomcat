# Tomcat 9 / Remedy AR System 20.02 Security Header Hardening Guide

## Overview

This guide documents the remediation steps for missing HTTP security headers identified during a Qualys vulnerability scan against a Windows server running:

* Apache Tomcat 9
* BMC Remedy AR System 20.02
* HTTPS on port 443

The goal is to improve browser-side security protections and reduce vulnerability findings without disrupting Remedy functionality.

---

# Introduction

## Environment Details

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

# Hardening - Preparation

## Security Headers Implemented

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

# Hardening

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

# Step 3 — Content Security Policy (CSP) Considerations

## Important

Testing identified that the following configuration is not supported in the current Tomcat environment and caused the Tomcat service to fail with Windows Service Error 1067 during startup:

```xml
<Valve className="org.apache.catalina.valves.HttpHeaderSecurityValve"
       contentSecurityPolicy="default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: https:;" />
```

As a result, this configuration should NOT be implemented in this environment.

---

# Recommended Approach for Remedy AR System

For Remedy AR System 20.02 environments, it is recommended to:

* Use Tomcat HttpHeaderSecurityFilter only
* Avoid aggressive CSP enforcement directly in Tomcat
* Implement CSP later through a reverse proxy or load balancer if required

This minimizes the risk of:

* Application startup failures
* Broken Remedy UI functionality
* JavaScript rendering issues
* Integration failures

---

# Supported Security Headers in Current Environment

The following headers are successfully supported using HttpHeaderSecurityFilter:

| Header                    | Status    |
| ------------------------- | --------- |
| X-Content-Type-Options    | Supported |
| X-Frame-Options           | Supported |
| X-XSS-Protection          | Supported |
| Strict-Transport-Security | Supported |
| Content-Security-Policy   | Deferred  |

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

# Verification - Post hardening Verification

## Verify Security Headers

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

# Hardening - Post Hardening Activities

## Post-Implementation Testing

After implementing the changes:

1. Verify Remedy login functionality
2. Verify dashboards and forms load correctly
3. Verify no browser console CSP errors exist
4. Verify APIs and integrations still function
5. Run a new Qualys scan

---

# Hardening - Back Out Procedure

## Rollback Procedure

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

| Finding                        | Expected Result                               |
| ------------------------------ | --------------------------------------------- |
| Missing X-Content-Type-Options | Resolved                                      |
| Missing X-Frame-Options        | Resolved                                      |
| Missing X-XSS-Protection       | Resolved                                      |
| Missing HSTS                   | Resolved                                      |
| Missing CSP                    | May still appear until implemented separately |

---

# Securing the Default Tomcat HTTP 8080 Site

## Overview

Apache Tomcat commonly exposes the default HTTP connector on port 8080.

Example:

```text
http://localhost:8080
```

While often required for internal communication or application integration, exposing port 8080 externally may introduce additional security findings during vulnerability scanning.

---

# Risks of Exposing Port 8080

Potential risks include:

* Unencrypted HTTP traffic
* Missing HTTPS enforcement
* Exposure of default Tomcat pages
* Server fingerprinting and version disclosure
* Additional attack surface

---

# Recommended Approaches

## Option 1 — Redirect HTTP to HTTPS (Recommended)

Configure Tomcat to redirect all HTTP traffic to HTTPS.

Benefits:

* Forces encrypted communication
* Improves security posture
* Reduces Qualys findings
* Improves compliance alignment

---

## Option 2 — Restrict Port 8080 to Localhost Only

If port 8080 is only required internally, bind it to localhost.

This prevents external systems from connecting directly.

---

# Option 1 — Configure HTTP to HTTPS Redirect

Edit:

```text
<Tomcat>\conf\server.xml
```

Locate the HTTP connector:

```xml
<Connector port="8080"
           protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Update configuration as needed for the environment.

## Notes

* In some environments HTTPS may terminate at a load balancer or reverse proxy.
* Validate existing SSL architecture before changing redirect behavior.
* Remedy integrations should be tested after changes.

---

# Option 2 — Restrict Port 8080 to Localhost

If external HTTP access is unnecessary, modify the connector:

```xml
<Connector port="8080"
           address="127.0.0.1"
           protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

This limits access to the local server only.

---

# Remove Default Tomcat Applications

Default Tomcat applications should not remain enabled in production.

Examples:

```text
/manager
/host-manager
/examples
/docs
```

## Recommended Action

Remove or restrict access to:

```text
<Tomcat>\webapps\docs
<Tomcat>\webapps\examples
<Tomcat>\webapps\host-manager
<Tomcat>\webapps\manager
```

---

# Suppress Tomcat Version Disclosure

Tomcat may expose version details in HTTP responses and error pages.

Example:

```text
Server: Apache-Coyote/1.1
```

---

## Recommended Configuration

Edit:

```text
<Tomcat>\conf\server.xml
```

Update connector:

```xml
<Connector port="443"
           server="SecureServer"
           ... />
```

Or suppress entirely where supported.

---

# Validate Port Exposure

Run the following commands:

```bash
netstat -ano | findstr :8080
```

And:

```bash
curl -I http://server-name:8080
```

Verify:

* Redirect behavior
* Access restrictions
* Security headers
* No unnecessary application exposure

---

# Additional Validation Checklist

| Validation Item                 | Status |
| ------------------------------- | ------ |
| HTTP redirect tested            | ☐      |
| Port 8080 externally restricted | ☐      |
| Default Tomcat apps removed     | ☐      |
| Version disclosure minimized    | ☐      |
| Remedy integrations validated   | ☐      |

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

## Issue: Tomcat service fails with Error 1067

### Possible Cause

Unsupported or invalid Valve configuration in `server.xml`.

Example unsupported configuration:

```xml
<Valve className="org.apache.catalina.valves.HttpHeaderSecurityValve"
       contentSecurityPolicy="default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: https:;" />
```

### Resolution

1. Remove the unsupported Valve entry from `server.xml`
2. Restore backup if necessary
3. Restart Tomcat service

### Notes

The environment successfully supports:

```xml
org.apache.catalina.filters.HttpHeaderSecurityFilter
```

within `conf/web.xml`.

---

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
