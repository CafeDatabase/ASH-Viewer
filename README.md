# ASH Viewer

ASH Viewer provides graphical view of active session history data within the database.

Supported databases: Oracle, PostgreSQL

> **Fork notice** — This is the [CafeDatabase](https://github.com/cafedatabase) fork of [akardapolov/ASH-Viewer](https://github.com/akardapolov/ASH-Viewer), maintained for student use. It adds JDK 21 build compatibility (Lombok 1.18.34) and automates the `bc-fips` jar placement during `mvn package`. See [For students](#for-students) for a step-by-step walkthrough.

## Table of contents

- [Quick start](#quick-start)
- [How it works](#how-it-works)
- [Build](#build)
- [For students](#for-students)
- [Security](#security)
- [Bugs and feature requests](#bugs-and-feature-requests)
- [Downloads](#downloads)
- [Based on](#based-on)
- [License](#license)
- [Contact](#contact)

![ASH-Viewer](media/main.png)

## Quick start
- [Download the latest binary file.](https://github.com/akardapolov/ASH-Viewer/releases)
- Download JDBC driver for your database ([Oracle](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html), [PostgreSQL](https://jdbc.postgresql.org/download.html))
- Unpack the binary archive and run ASH-Viewer.jar
- Open connection dialog and populate them with data (URL for Oracle database: **jdbc:oracle:thin:@host:port:SID**)

 ![ASH-Viewer connection dialog](media/connection.png)
- Press Connect button and start to monitor your system and highlight a range to show details.

 ![ASH-Viewer Top activity](media/top.png)
- Review Raw data interface to gain a deep insight into active session history

![ASH-Viewer raw data](media/raw.png)
- Double-click on Top sql & sessions interface to get window with ASH details by sql or session ID

 ![ASH-Viewer sql details](media/sql.png)

 ![ASH-Viewer session details](media/session.png)

[Return to Table of Contents](#table-of-contents)

## How it works
Active Session History (ASH) is a view in Oracle database that maps a circular buffer in the SGA.
  The name of the view is V$ACTIVE_SESSION_HISTORY. This view is populated every second
  and will only contain data for 'active' sessions, which are defined as sessions
  waiting on a non-idle event or on a CPU.
  
ASH Viewer provides graphical Top Activity, similar Top Activity analysis and Drilldown
    of Oracle Enterprise Manager performance page. ASH Viewer store ASH data locally using
    embedded database Oracle Berkeley DB Java Edition.
    
For Oracle standard edition and PostgreSQL, ASH Viewer emulate ASH, storing active session data on local storage.
  
Please note that v$active_session_history is a part of the Oracle Diagnostic Pack and requires a purchase of the ODP license.

[Return to Table of Contents](#table-of-contents)

## Build

To compile the application into an executable jar file, do the following:

1. Install JDK version 11 or higher, Maven and Git on your local computer.
    ```shell
    java -version  
    mvn -version
    git --version 
    ``` 
2. Download the source codes of the application to your local computer using Git

    ```shell
    git clone <url source code storage system>
    cd ASH-Viewer
    ```

3. Compile the project using Maven
    ```shell
    mvn clean compile
   ```

4. Execute the Maven command to build an executable jar file with tests running
    ```shell
     mvn clean package -DskipTests=true 
    ```

An executable jar file like `ashv-<VERSION>-SNAPSHOT-jar-with-dependencies.jar` will be located at the relative path ashv/target

[Return to Table of Contents](#table-of-contents)

## For students

This fork ships two improvements over upstream that matter for first-time builders:

- Lombok was bumped to 1.18.34, so the project compiles on **JDK 16+** (tested with JDK 21). Upstream's 1.18.24 fails on JDK 16+ with `NoSuchFieldError: JCTree$JCImport.qualid`.
- The `bc-fips` jar is copied to `ashv/target/` automatically during `mvn package`. The fat jar's manifest declares `Class-Path: bc-fips-1.0.2.4.jar`, which the JVM resolves relative to the jar's own directory — without that file the app crashes at startup with `NoClassDefFoundError: BouncyCastleFipsProvider`.

### 1. Prerequisites

- JDK 11 or higher (JDK 21 verified)
- Maven 3.6+
- Git

### 2. Clone and build

```shell
git clone https://github.com/cafedatabase/ASH-Viewer.git
cd ASH-Viewer
mvn clean package -DskipTests
```

After the build, in `ashv/target/` you should see:

- `ashv-4.0-SNAPSHOT-jar-with-dependencies.jar` — runnable fat jar (~26 MB)
- `bc-fips-1.0.2.4.jar` — FIPS crypto provider, **must stay next to** the fat jar

### 3. Download the Oracle JDBC driver

The Oracle JDBC driver is not bundled (Oracle license). Download it once into a stable location of your choice:

```shell
mkdir -p ~/lib
mvn dependency:get -Dartifact=com.oracle.database.jdbc:ojdbc11:23.5.0.24.07 -Dtransitive=false
cp ~/.m2/repository/com/oracle/database/jdbc/ojdbc11/23.5.0.24.07/ojdbc11-23.5.0.24.07.jar ~/lib/ojdbc11.jar
```

`ojdbc11` works on JDK 11–21 and connects to Oracle 19c, 21c and 23ai databases.

### 4. Run

```shell
java -jar ashv/target/ashv-4.0-SNAPSHOT-jar-with-dependencies.jar
```

### 5. Connection settings (Oracle)

In the *New connection* dialog, set:

| Field        | Value |
|--------------|-------|
| Profile      | `OracleEE10g` (recommended for 19c/21c/23ai) — see note below |
| JDBC driver  | absolute path to `ojdbc11.jar` (e.g. `/home/<user>/lib/ojdbc11.jar`) |
| Driver class | `oracle.jdbc.driver.OracleDriver` |
| URL          | see formats below |
| User / Pass  | your DB credentials |

JDBC URL formats:

- **SID:** `jdbc:oracle:thin:@host:1521:SID`
- **Service name:** `jdbc:oracle:thin:@host:1521/service.name`
- **OCI PDB (private subnet service):** `jdbc:oracle:thin:@<public_ip>:1521/<pdb_name>.<subnet_dns>.<vcn_dns>.oraclevcn.com`
- **Autonomous DB (TLS + wallet):** `jdbc:oracle:thin:@<tnsname>?TNS_ADMIN=/path/to/wallet`

> **Profile choice:** `OracleEE` assumes `V$ACTIVE_SESSION_HISTORY` exposes a `SQL_OPNAME` column, but Oracle 12c+ only provides `SQL_OPCODE`. `OracleEE10g` derives `SQL_OPNAME` by joining `V$ACTIVE_SESSION_HISTORY` with `AUDIT_ACTIONS` and works correctly on 10g through 23ai. Use `OracleSE` only if your database is Standard Edition or you don't have the Oracle Diagnostic Pack license (`V$ACTIVE_SESSION_HISTORY` is part of ODP).

#### CafeDatabase course lab

If you are a CafeDatabase student, your assigned PDB on the shared Oracle 23ai lab follows the pattern `MASTERXX` (where `XX` is the number your instructor gave you, e.g. `01`, `02`, ...). Use these exact values:

| Field        | Value |
|--------------|-------|
| Profile      | `OracleEE10g` |
| JDBC driver  | `/home/<user>/lib/ojdbc11.jar` |
| Driver class | `oracle.jdbc.driver.OracleDriver` |
| URL          | `jdbc:oracle:thin:@143.47.35.135:1521/masterXX.sub09291555070.vcn23ai.oraclevcn.com` |
| User / Pass  | the credentials provided by your instructor |

Replace `masterXX` in the URL with your assigned number in lowercase (e.g. `master03`). Your DB user must hold the privileges needed to read `V$ACTIVE_SESSION_HISTORY` and `AUDIT_ACTIONS`; the lab account already has them.

Quick connectivity check from the shell before opening ASH Viewer:

```shell
nc -vz 143.47.35.135 1521
```

### 6. Linux desktop launcher (optional)

To launch ASH Viewer from your application menu, create `~/.local/share/applications/ash-viewer.desktop`:

```ini
[Desktop Entry]
Type=Application
Name=ASH Viewer
Comment=Graphical Active Session History viewer for Oracle databases
Exec=/usr/bin/java -jar /home/<user>/ASH-Viewer/ashv/target/ashv-4.0-SNAPSHOT-jar-with-dependencies.jar
Path=/home/<user>/ASH-Viewer
Icon=/home/<user>/ASH-Viewer/media/main.png
Terminal=false
Categories=Development;Database;Java;
StartupWMClass=Main
```

Replace `<user>` with your username. Validate with `desktop-file-validate ~/.local/share/applications/ash-viewer.desktop`.

[Return to Table of Contents](#table-of-contents)

## Security
Encryption and Container settings provide security for database passwords (go to Other tab -> Security block)

### Encryption
Encryption setting has AES and PBE options
- **AES** - Advanced Encryption Standard (AES) with 256-bit key encryption, from the [Bouncy Castle provider](https://www.bouncycastle.org/), [FIPS](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips#:~:text=are%20FIPS%20developed%3F-,What%20are%20Federal%20Information%20Processing%20Standards%20(FIPS)%3F,by%20the%20Secretary%20of%20Commerce.) compliant algorithm;
- **PBE** - Password based encryption (PBE) in PBEWithMD5AndDES mode with secret key (computer name or hostname). This option is weak and deprecated, please use AES when possible;

### Container
It's the way to store your encrypted data
- **DBAPI** - You sensitive data stored in Windows Data Protection API (DPAPI), maximum protected, Windows only;
- **Registry** - OS System registry storage using java preferences API - weak, but better than **Configuration**;
- **Configuration** - All data in local configuration file - weak, not recommended;

### Recommendations 
- use **AES** encryption and Windows Data Protection API (**DBAPI**) whenever possible;
- do not use **PBE** copied configuration on another host, you need to change password with a new secret key (always do it for password leak prevention).

[Return to Table of Contents](#table-of-contents)

## Bugs and feature requests
If you found a bug in the code or have a suggestion for improvement, [Please open an issue](https://github.com/akardapolov/ASH-Viewer/issues)  

[Return to Table of Contents](#table-of-contents)
 
## Downloads
- [Current version](https://github.com/akardapolov/ASH-Viewer/releases)
- [Old release 3.5.1 on github.com](https://github.com/akardapolov/ASH-Viewer/releases/tag/v3.5.1)
- [Mirror on sourceforge.net](https://sourceforge.net/projects/ashv/files/)   

[Return to Table of Contents](#table-of-contents)

## Based on
- [JFreeChart by David Gilbert](http://www.jfree.org)
- [E-Gantt Library by Keith Long](https://github.com/akardapolov/ASH-Viewer/tree/master/egantt)
- [Berkeley DB Java Edition](http://www.oracle.com/database/berkeley-db)
- [SwingLabs GUI toolkit by alexfromsun, kleopatra, rbair and other](https://en.wikipedia.org/wiki/SwingLabs)
- [Dagger 2 by Google](https://dagger.dev/)
- [AES cipher by Bouncy Castle](https://www.bouncycastle.org/)
- [Windows DPAPI Wrapper by @peter-gergely-horvath](https://github.com/peter-gergely-horvath/windpapi4j)

[Return to Table of Contents](#table-of-contents)

## License
[![GPLv3 license](https://img.shields.io/badge/License-GPLv3-blue.svg)](http://perso.crans.org/besson/LICENSE.html)

  Code released under the GNU General Public License v3.0

[Return to Table of Contents](#table-of-contents)

## Contact
  Created by [@akardapolov](mailto:akardapolov@gmail.com) - feel free to contact me!

[Return to Table of Contents](#table-of-contents)