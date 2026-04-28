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

---

# Installation Preparation

Prior to upgrading Apache Tomcat from version **9.0.87** to **9.0.117**, a full review of the existing installation and configuration was completed to ensure a controlled in-place upgrade with a clear rollback path.

As the environment is a single-instance deployment supporting the BMC Remedy Action Request System Mid Tier service, preserving the existing configuration and ensuring rapid recovery capability were critical preparation steps.

---

## 1. Identify Existing Installation Locations

The following installation paths were confirmed on the server:

### Remedy AR System

```text id="r1"
E:\Program Files\BMC Software\
```

This contains the core Remedy application server components and supporting services.

### Java Runtime

```text id="r2"
E:\Program Files\Java\
```

This provides the Java runtime required by Tomcat and Remedy Mid Tier.

### Apache Tomcat

```text id="r3"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\
```

This hosts the Remedy Mid Tier web application and provides browser access to Remedy via the `/arsys` context.

---

## 2. Confirm Existing Software Versions

The currently installed software versions were verified prior to upgrade:

| Component                        | Version            |
| -------------------------------- | ------------------ |
| Apache Tomcat                    | 9.0.87             |
| BMC Remedy Action Request System | 20.02.00 Patch 006 |
| Java Platform, Standard Edition  | 1.8.0 Update 45    |

This ensured compatibility validation before proceeding with the Tomcat upgrade.

---

## 3. Validate Existing Service Access

Prior to any maintenance activity, successful access to both the local Tomcat service and the external Remedy Mid Tier URL was confirmed.

### Local Tomcat Validation

```text id="r4"
http://localhost:8080
```

This verified that the local Tomcat service was operational and responding correctly.

### Remedy Mid Tier Validation

```text id="r5"
https://myremedy.domain.local/
```

This confirmed that the Mid Tier service was accessible externally and that user access to Remedy was functioning normally before the upgrade.

This baseline validation is important for post-upgrade comparison.

---

## 4. Download Target Tomcat Version

The upgrade package was obtained directly from the official Apache Software Foundation source.

### Download Source

### Selected Package

The **Windows 64-bit ZIP distribution** was selected rather than the Windows installer to allow a controlled in-place binary replacement while preserving the existing service configuration.

---

## 5. Extract New Tomcat Version

The downloaded ZIP package was extracted to the following staging location:

```text id="r6"
E:\Program Files\Apache Software Foundation\Tomcat 9.0.117\
```

This allowed the new binaries to be prepared and reviewed prior to replacing the live installation.

The existing production installation remained untouched during this stage.

---

## 6. Stop Tomcat Service and Create Full Backup

Before any modification to the live environment, the existing Tomcat service was stopped to ensure file consistency.

### Service Stopped

```text id="r7"
Apache Tomcat 9
```

### Full Installation Backup Created

The entire existing Tomcat installation folder was archived as a ZIP backup:

### Source

```text id="r8"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\
```

### Backup Location

```text id="r9"
E:\Media\Backup\
```

This backup provides the primary rollback path in the event of upgrade failure.

---

## 7. Record Existing Java Configuration

The existing Java service configuration was documented prior to upgrade.

### Confirmed Java Options

Key parameters included:

```text id="r10"
-Dcatalina.home=E:\Program Files\Apache Software Foundation\Tomcat 9.0
-Dcatalina.base=E:\Program Files\Apache Software Foundation\Tomcat 9.0
-Djava.io.tmpdir=...
-Djava.util.logging.config.file=...
```

This confirmed that the environment uses a **single-instance Tomcat deployment** where both `CATALINA_HOME` and `CATALINA_BASE` point to the same installation directory.

These values must be preserved after upgrade to avoid service startup issues.

---

## 8. Review Existing server.xml Configuration

The existing `server.xml` file was analysed to identify all custom configuration that must be retained during the upgrade.

Key findings included:

* HTTPS connector configured on **port 443**
* SSL enabled using Java keystore
* Custom keystore path:

```text id="r11"
E:\Remedy\Keystore\remedydev.jks
```

* Existing default host configuration under:

```xml id="r12"
<Service name="Catalina">
<Engine name="Catalina">
<Host appBase="webapps">
```

Preserving this configuration is critical, as loss of SSL or connector settings would prevent access to the Remedy Mid Tier application.

---

## Preparation Outcome

At completion of the preparation phase:

* Existing environment state was fully documented
* Backup and rollback capability was established
* Target upgrade binaries were staged
* Java and SSL configuration dependencies were identified
* Service validation baseline was confirmed

This ensured the upgrade could proceed safely with controlled risk and a clear recovery path if required.

---
# Upgrade Procedure

Following successful preparation and validation in the Development environment, the in-place upgrade of Apache Tomcat from version **9.0.87** to **9.0.117** was performed using a controlled binary replacement approach.

As this is a single-instance deployment supporting the BMC Remedy Action Request System Mid Tier service, preserving configuration consistency and maintaining a clear rollback path were critical throughout the implementation.

---

## 1. Shutdown and Infrastructure Protection

Prior to any system changes, the server was gracefully shut down to ensure a clean system state and to allow infrastructure-level rollback if required.

### Actions Performed

* Server shutdown completed
* Full VMware snapshot taken for rollback protection

This provides a full machine-level recovery option in addition to the Tomcat file backup created during preparation.

---

## 2. Post-Snapshot Validation

Once the server was powered back on, a final validation was performed to confirm the environment remained healthy before beginning the upgrade.

The following checks were completed successfully:

### Local Tomcat Access

```text id="u1"
http://localhost:8080
```

### Remedy Mid Tier Access

```text id="u2"
https://myremedy.domain.local/
```

### User Authentication Validation

Successful login to the Remedy Mid Tier application was confirmed.

This ensured the pre-upgrade baseline remained valid and that the snapshot represented a known-good working state.

---

## 3. Stop Application Services

Before replacing Tomcat binaries, all related services were stopped to prevent file locking and ensure a clean transition.

### Services Stopped

* Apache Tomcat 9
* BMC Remedy services (as applicable to the environment)

This included the Remedy application services required to fully release file handles and prevent startup inconsistencies.

---

## 4. Preserve Existing Tomcat Installation

The current Tomcat installation folder was retained by renaming the existing directory rather than deleting it.

### Existing Folder Renamed

### From

```text id="u3"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\
```

### To

```text id="u4"
E:\Program Files\Apache Software Foundation\Tomcat 9.0.87\
```

This provides a fast local rollback option if required.

---

## 5. Promote New Tomcat Version to Live Installation

The previously extracted Tomcat 9.0.117 staging folder was renamed to match the original production path.

### New Folder Renamed

### From

```text id="u5"
E:\Program Files\Apache Software Foundation\Tomcat 9.0.117\
```

### To

```text id="u6"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\
```

This ensures the Windows service continues to reference the correct path without requiring changes to service configuration or Java options.

At this stage, both versions existed on disk:

* `Tomcat 9.0.87` (previous version retained)
* `Tomcat 9.0` (new live version containing 9.0.117 binaries)

---

## 6. Compare Old and New Installations

A detailed folder and file comparison was performed using **Beyond Compare** to identify environment-specific configuration that needed to be preserved.

This step is critical because standard Tomcat ZIP extraction does not include custom SSL, application context, or Remedy-specific configuration.

---

## 7. Copy Required Configuration from Old Installation

Based on validation in the Development environment, only the following items required migration from the old installation to the new installation.

### Required File Copy

### Copy:

```text id="u7"
..\Tomcat 9.0.87\conf\server.xml
```

### To:

```text id="u8"
..\Tomcat 9.0\conf\
```

This preserves:

* HTTPS connector configuration
* Port 443 listener
* SSL keystore settings
* existing connector tuning

---

### Required Folder Copy

### Copy:

```text id="u9"
..\Tomcat 9.0.87\conf\Catalina\
```

### To:

```text id="u10"
..\Tomcat 9.0\conf\
```

This preserves host-specific and application-specific configuration used by the Remedy Mid Tier deployment.

---

## 8. Remove Unused Default Tomcat Applications

The following default Tomcat folders existed in the new installation but were not present in the original environment and were therefore removed to maintain consistency.

### Deleted Folders

```text id="u11"
..\Tomcat 9.0\webapps\examples\
..\Tomcat 9.0\webapps\host-manager\
```

This avoids introducing unnecessary administrative or demonstration applications into the production environment.

---

## 9. Environment-Specific Validation

The Development environment required only the files listed above.

However, Test and Production environments may contain additional customisation depending on historical changes, SSL configuration, or deployment-specific settings.

Therefore:

### Additional review is mandatory for:

* `conf\`
* `webapps\`
* SSL keystore references
* custom certificates
* service wrapper settings
* logging configuration
* Remedy-specific context configuration

Any additional differences identified during comparison must be reviewed and copied where required.

No assumptions should be made that all environments are identical.

---

## Upgrade Outcome

At completion of the upgrade phase:

* Tomcat binaries were successfully upgraded to version 9.0.117
* Existing service paths remained unchanged
* SSL and Remedy Mid Tier configuration were preserved
* Local rollback remained available via retained 9.0.87 folder
* Infrastructure rollback remained available via VMware snapshot

The environment was then ready for post-upgrade validation and operational testing.
