
---

# Recommended Upgrade Method for Your Server

## Tomcat 9.0.87 → 9.0.117 using ZIP Package

### Safe Method:

Rename old install → extract new ZIP → restore required config

This is the best approach for your setup.

---

# Step-by-Step Procedure

---

# Step 1 — VMware Snapshot

Take your VMware snapshot first.

This gives you the fastest rollback possible.

Excellent first step.

---

# Step 2 — Backup Existing Tomcat Folder

Copy:

```text id="7ltg7u"
E:\Program Files\Apache Software Foundation\Tomcat 9.0
```

to something like:

```text id="p7it0w"
E:\Backups\Tomcat_9.0.87_Backup
```

Even with snapshot, this helps with comparisons.

---

# Step 3 — Record Existing Service Settings

Run:

```cmd id="vw1n51"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\bin\tomcat9w.exe
```

Take screenshots of:

## Java Tab

Especially:

* Java Virtual Machine path
* Initial memory pool
* Maximum memory pool
* JVM options
* Java options

## Startup Tab

## Logging Tab

This is extremely important.

---

# Step 4 — Confirm Java Version

Run:

```cmd id="r40p3s"
java -version
```

Since Java 8 is installed separately:

## Do NOT upgrade Java during this task

Just ensure Tomcat continues using the same Java path.

---

# Step 5 — Stop Tomcat Service

Run CMD as Administrator:

```cmd id="1rzb4f"
net stop Tomcat9
```

Then verify:

```cmd id="24pdmz"
sc query Tomcat9
```

You want to see:

```text id="jpjjvr"
STATE: STOPPED
```

---

# Step 6 — Rename Existing Tomcat Folder

Rename:

```text id="a4t5i0"
E:\Program Files\Apache Software Foundation\Tomcat 9.0
```

to:

```text id="svm12g"
E:\Program Files\Apache Software Foundation\Tomcat 9.0_old
```

Do NOT delete it.

This gives fast rollback.

---

# Step 7 — Download Tomcat 9.0.117 ZIP

From:

Apache Software Foundation

Use:

## Windows 64-bit ZIP package

Example:

```text id="ycr9lr"
apache-tomcat-9.0.117-windows-x64.zip
```

---

# Step 8 — Extract ZIP to Original Location

Extract into:

```text id="itj1rz"
E:\Program Files\Apache Software Foundation\
```

so the new folder becomes:

```text id="h24pd8"
E:\Program Files\Apache Software Foundation\Tomcat 9.0
```

### Important:

Keep the exact same folder name:

```text id="ltxfjy"
Tomcat 9.0
```

This helps the Windows service continue working without changes.

---

# Step 9 — Compare and Restore Configuration Files

Compare:

FROM:

```text id="u6v6xg"
E:\Program Files\Apache Software Foundation\Tomcat 9.0_old\conf\
```

TO:

```text id="3e6qu9"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\conf\
```

Especially:

* server.xml
* context.xml
* web.xml
* tomcat-users.xml
* logging.properties
* catalina.properties

---

# Critical Rule

## Do NOT overwrite the whole conf folder

Only copy the settings you actually customized.

Use:

* WinMerge
* Beyond Compare
* Notepad++

recommended.

This is the most important step.

---

# Step 10 — Restore Your Applications

Copy from:

```text id="qgzrwu"
E:\Program Files\Apache Software Foundation\Tomcat 9.0_old\webapps\
```

to:

```text id="gvz3if"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\
```

Copy only:

* your WAR files
* your deployed application folders

Avoid copying:

* docs
* examples
* manager
* host-manager
* default ROOT

unless customized.

---

# Step 11 — Restore Custom JAR Files

Check:

```text id="ytlr8r"
E:\Program Files\Apache Software Foundation\Tomcat 9.0_old\lib\
```

Copy only:

## custom JARs

Do NOT overwrite standard Tomcat libraries.

Very important.

---

# Step 12 — Restore setenv.bat

If this exists:

```text id="p2em7t"
E:\Program Files\Apache Software Foundation\Tomcat 9.0_old\bin\setenv.bat
```

copy it to:

```text id="s8weui"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\bin\
```

This often contains:

* heap settings
* JVM options
* custom variables

Very commonly forgotten.

---

# Step 13 — Start Tomcat Service

Because the path remains the same:

```text id="07ix44"
Tomcat 9.0
```

your Windows service often works immediately.

Try:

```cmd id="smlbrc"
net start Tomcat9
```

first.

---

# Step 14 — If Service Does NOT Start

Then reinstall the service:

```cmd id="8nkwyx"
cd "E:\Program Files\Apache Software Foundation\Tomcat 9.0\bin"
```

then:

```cmd id="0gbvjy"
service.bat remove
service.bat install
```

Then open:

```cmd id="1sowoe"
tomcat9w.exe
```

and restore:

* Java path
* JVM memory
* startup mode
* Java options

from your screenshots.

---

# Step 15 — Check Logs Immediately

Review:

```text id="bj20fu"
E:\Program Files\Apache Software Foundation\Tomcat 9.0\logs\
```

Look for:

* XML parsing errors
* Java path problems
* missing JARs
* SSL failures
* connector issues
* deployment errors

This is critical.

---

# Step 16 — Functional Testing

Test:

* application access
* HTTPS
* logins
* APIs
* DB connections
* uploads
* scheduled jobs
* integrations

Do not stop at “Tomcat started.”

---

# Step 17 — Confirm Upgrade Success

Logs should show:

```text id="6v9l5r"
Apache Tomcat/9.0.117
```
