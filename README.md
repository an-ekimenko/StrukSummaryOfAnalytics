# Excel to Text ETL Automation Utility

[![Build Status](https://shields.io)](#)
[![Java Version](https://shields.io)](#)
[![Dependency](https://shields.io)](#)
[![Framework](https://shields.io)](#)

An enterprise-grade, resilient CLI utility designed to automate data extraction, validation, and transformation workflows. Managed by a Google Guice dependency injection bootstrap layer, the application securely fetches high-volume Excel workbooks from cloud repositories, executes strict infrastructure checks, and exports optimized data packages to enterprise shared storage (SMB/UNC).

---

## Key Enterprise Features

* **Execution Path Independence:** Implements dynamic runtime code-source location decoding via `ProtectionDomain` and `URI` normalization. The utility can be invoked from any active working directory or scheduled tasks seamlessly.
* **Guice Dependency Injection Layer:** Uses a pre-initialization layer (`AppModule`) to resolve environment paths, bootstrap logging contexts, and handle strict binding overrides for system thresholds.
* **Lombok-Optimized Data Layer:** Leverages compile-time annotation processing via Lombok (`@Getter`/`@Setter`) within the configuration DTO layers (`stk.dto`), ensuring a clean, boilerplate-free codebase.
* **Dynamic Runtime TLS/SSL Validation:** Seamlessly integrates with the enterprise `by.beloil.crypto:crypto-cert-loader` package. Evaluates inbound endpoint signatures in real-time, executing runtime keystore extensions to eliminate certificate expiration or self-signed trust disruptions (`PKIX path building failed`).
* **Unified Enterprise Logging (Log4j2/SLF4J):** Replaces legacy console logging streams with an integrated Log4j2 engine. For streamlined enterprise monitoring, all operational metrics and debug diagnostics are replicated concurrently into both the console output and the persistent file repository `logs/common.log`.
* **Granular Third-Party Log Filtering:** Explicitly orchestrates boundaries and logging levels targeting high-volume vendor sub-systems like `com.spire` and `org.asynchttpclient`. Suppresses verbose operational trace noise, ensuring audit stability in production environments.
* **Formula-Driven Network Resiliency:** Features an automated multi-attempt loop to survive transient network drops. Handshake connection intervals and retry cool-down delays are calculated dynamically based on an exponential progression scale:
  \[\text{Timeout} = \text{initial\_conn\_timeout\_ms} \times (1 + \text{attempt}^{(\text{attempt} - 1)})\]
* **Large File Optimization:** Configured with an intentional unlimited Read Timeout parameter combined with a formula-driven Handshake Timeout. This shields high-volume file transmissions (~10MB+) from premature session kills caused by network fluctuation.
* **Decoupled Footprint Observability (YAML Report):** Guarantees the persistence of process metrics to `exchange/result.yaml` inside a global `finally` block, providing down-stream analytical orchestrators with crisp operational footprints containing execution timestamps and status codes.
* **Resource Leak Prevention & Auto-Cleanup:** Implements explicit memory cleanup routines via `workbook.dispose()`. Temporary source `.xlsx` payloads are automatically wiped from the disk storage arrays immediately following verified conversions.
* **Root-Level Workspace Compilation:** Automatically detaches the generated executable `.jar` file from the temporary `target/` storage and places it directly into the application home folder upon Maven pack success, ensuring shorter and absolute invocation scripts.

---

## Architectural Workflow

```text
       [Cloud Storage] 
              │
              ▼ (Pre-Flight: Trust Validation via crypto-cert-loader Module)
       [Local System Memory] ──► [Log4j2 Pipeline Engine] ──► logs/common.log
              │
              ▼ (Dynamic Path Validation & OS Write Permissions Check)
       [Target Network Share] ──► [Spire.XLS Processing Engine] ──► Optimized Text Output (# Delimited)
              │
              ▼ (Global Lifecycle Finalization Block)
       [Footprint Exported] ──► Write metrics package to exchange/result.yaml
```

---

## Project Structure

```text
StrukSummaryOfAnalytics/
├── config/
│   ├── Log4j2.xml              # Programmatic logger routing architecture (Editable)
│   ├── system.properties       # Local runtime states (IGNORED BY GIT - Secret)
│   └── system.properties.dist  # Public distribution template for environment setup
├── logs/
│   └── common.log              # Persistent log outputs tracking pipeline execution
├── exchange/
│   └── result.yaml             # Decoupled monitoring automation descriptor
├── dependencies/               # Copied isolated external classpath runtimes
├── src/
│   └── main/
│       └── java/
│           └── stk/
│               ├── dto/
│               │   └── ApiConfig.java # Lombok-optimized configuration DTO
│               ├── Main.java          # Core business logic and ETL engine
│               └── AppModule.java     # Guice bootstrapping & TLS-validation layer
├── .gitignore                  # Strict Git filtering boundary rules
├── pom.xml                     # Maven version management matrix
└── StrukSummaryOfAnalytics-0.0.1-SNAPSHOT.jar # Root-level workspace artifact
```

---

## Configuration Management

The application segregates code from environment states using an externalized configuration policy. The environmental property and logger definition files reside in the root `config/` directory relative to the project deployment home and are fully exposed for modifications without recompiling the executable module.

### Security & Deployment Policy
To protect intellectual property and enterprise infrastructure data, `system.properties` is explicitly ignored by version control. 

**Deployment Step:** To configure a new runtime environment, clone the repository, navigate to the `config/` directory, copy `system.properties.dist` into a new file named `system.properties`, and configure exactly one authentication option block by uncommenting it.

### 1. Environment Properties Configuration (`config/system.properties.dist`)
* `original_xls`: The HTTP/HTTPS web entry point targeting the production cloud document repository.
* `output_dir`: The destination enterprise path (local disk array or UNC path share). Backslashes must be safely escaped.
* `max_retries`: *(Optional)* Defines the total download network limits before crashing. If omitted or malformed, defaults to `3`.
* `initial_conn_timeout_ms`: *(Optional)* Base connect handshake interval threshold in milliseconds. If omitted or malformed, defaults to `30000`.

```properties
# Cloud storage URLs
original_xls=YOUR_URL

# Output directories (UNC share paths must use safely escaped backslashes)
output_dir=YOUR_OUTPUT_DIR

# Cryptographic and Authentication Settings
# Choose EXACTLY ONE authentication block by uncommenting it:
auth=none

#auth=token
#auth_token=YOUR_SECURE_TOKEN_HERE

#auth=basic
#auth_username=YOUR_CORPORATE_USERNAME
#auth_password=YOUR_CORPORATE_PASSWORD

# Fault tolerance configurations
max_retries=3
initial_conn_timeout_ms=30000
```

### 2. Logging Engine Architecture Configuration (`config/Log4j2.xml`)
Manages asynchronous and synchronous appender routing limits. Suppresses runtime debug overhead originating from underlying Excel parsers (`com.spire`) and network handlers (`org.asynchttpclient`) to maintain clear and indexed execution records.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n</Property>
    </Properties>

    <Appenders>
        <!-- Console Tracking Appender -->
        <Console name="ConsoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </Console>
        
        <!-- Persistent File Log Appender -->
        <File name="FileAppender" fileName="logs/common.log" append="true">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </File>
    </Appenders>
    
    <Loggers>
        <!-- Suppress verbose third-party operational framework logs -->
        <Logger name="com.spire" level="WARN"/>
        <Logger name="org.asynchttpclient" level="WARN"/>
        
        <!-- Root Logger Routing -->
        <Root level="INFO">
            <AppenderRef ref="ConsoleAppender"/>
            <AppenderRef ref="FileAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

---

## Deployment & Execution

### 1. Compilation
Build and auto-deploy the binary to the root directory using Maven:
```bash
mvn clean package
```

### 2. Execution Matrix
Thanks to the dynamic path resolution layer and the root-level deployment plugin, the application supports universal absolute path invocations. This single command structure ensures seamless execution whether triggered manually via terminal interfaces or orchestrated autonomously by OS schedulers (such as Windows Task Scheduler or Linux CRON daemons):

```bash
java -jar "<path_to_application_home>\StrukSummaryOfAnalytics-0.0.1-SNAPSHOT.jar"
```

---

## Automation Process Return Codes

The application implements enterprise-level exit matrix codes to deliver clean operational observability for external SIEM engines, wrapper automation scripts, or monitoring agents. 

Upon lifecycle termination, execution performance indicators are automatically persisted to the detached file checkpoint `exchange/result.yaml` to ensure automated downstream systems can seamlessly audit the result:

```yaml
output:
  datetime: 202610-30:11:28:41
  returned_code: 0
```

| Exit Code | Classification | Root Cause & Resolution |
| :---: | :--- | :--- |
| **0** | `SUCCESS` | Lifecycle completed. File fetched, transformed, and disk space freed. |
| **1** | `INITIALIZATION_FAIL` | System runtime path context error, or configuration files are missing/malformed. |
| **2** | `IO_DIRECTORY_DENIED` | Target network path validation rejected. Missing OS directory privileges, or SMB network down. |
| **3** | `NETWORK_RETRY_EXHAUSTED` | Network stream completely failed. The configured `max_retries` download limits were exhausted without response. |
| **4** | `CONVERSION_CRASH` | Data transform exception. `Spire.XLS` could not process the workbook or target sheet `"Слияние"` is missing. |

---

## Error & Exception Handling

All structural and systemic exception stacks are processed by the Log4j2 pipeline layout and written directly to the system standard error stream (`System.err`) and `logs/common.log`. This ensures that background process logging structures instantly catch and flag analytical exceptions without polluting output reporting logs.
