# Introduction

This document defines the engineering procedure for upgrading Apache Tomcat from version **9.0.87** to **9.0.117** on the Windows-based on-premise BMC Remedy Action Request System environment supporting **Remedy AR System 20.02**.

The current environment uses Tomcat as the web application server hosting the Remedy Mid Tier (`/arsys`), providing browser-based access to the Remedy platform over HTTPS (port 443). This service is business-critical, as it enables user authentication, incident management, service request processing, and operational workflows across the platform.

## Background

The existing Remedy platform version (**20.02**) is now outside of vendor support:

* **Full Support End Date:** 21-Feb-2024
* **End of Support Date:** 21-Feb-2025

The platform currently runs with the following technology stack:

* Apache Tomcat – Version 9.0.87
* Java Platform, Standard Edition – Version 1.8.0 Update 45

Although Apache Tomcat version 9 remains within the supported major release family for Remedy 20.02, the currently installed version (**9.0.87**) is significantly behind current security patch levels and has been identified by vulnerability scanning as containing multiple known security issues.

## Security Risk and Vulnerability Exposure

Recent vulnerability assessments identified multiple high and medium severity Common Vulnerabilities and Exposures (CVEs) affecting the installed Tomcat version. These include:

* Denial of Service (DoS) vulnerabilities
* HTTP/2 protocol abuse vulnerabilities
* Directory traversal vulnerabilities
* File upload and resource exhaustion vulnerabilities
* Rewrite rule bypass vulnerabilities
* Windows installer side-loading vulnerabilities

Examples include:

* CVE-2024-34750
* CVE-2024-38286
* CVE-2025-48989
* CVE-2025-55752
* CVE-2025-52520
* CVE-2025-31650
* CVE-2025-31651
* CVE-2025-49124
* CVE-2025-53506
* CVE-2025-52434

All currently identified vulnerabilities are resolved by Apache Tomcat version **9.0.109** or later.

These vulnerabilities present operational and security risks including service disruption, denial of service, unauthorised file access, and increased exposure of the Remedy web platform.

## Upgrade Justification

To address the identified security risks and reduce operational exposure, Tomcat will be upgraded from:

**Current Version:** 9.0.87
**Target Version:** 9.0.117

Version **9.0.117** has been selected rather than the minimum remediation version (9.0.109) to ensure the platform is brought to the latest available stable patch level within the Tomcat 9.x release stream, providing:

* Resolution of all currently identified vulnerabilities
* Additional security and stability fixes introduced after 9.0.109
* Improved operational resilience
* Reduced likelihood of immediate follow-up remediation activity

This approach ensures the environment remains on the latest maintained Tomcat 9 release while preserving compatibility with the existing Remedy architecture.

## Validation Approach

As all environments operate as single-instance Tomcat deployments, an in-place upgrade approach was assessed and validated within the Development environment where an extended downtime window was acceptable.

The Development upgrade was completed successfully, including:

* Tomcat binary replacement
* Preservation of existing configuration and SSL settings
* Successful restart of the Tomcat service
* Validation of Remedy Mid Tier access via `/arsys`
* Functional verification of login, ticket access, and application workflows

This successful implementation established the documented upgrade procedure, rollback steps, and realistic service downtime expectations.

The same validated process will now be applied to the Test and Production environments in a controlled manner, reducing implementation risk and ensuring consistency across all environments.

## Scope

This document covers the engineering procedure for upgrading the Tomcat application server across Development, Test, and Production environments, including:

* Pre-upgrade validation
* Backup and rollback preparation
* Tomcat binary replacement
* Service validation
* Mid Tier functionality verification
* Post-upgrade operational checks

The Development implementation serves as the proven reference model for execution in Test and Production.
