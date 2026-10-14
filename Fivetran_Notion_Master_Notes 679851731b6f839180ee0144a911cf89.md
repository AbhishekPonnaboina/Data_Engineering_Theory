# Fivetran_Notion_Master_Notes

# Fivetran Technical Foundations — Master Notes

> **Purpose:** Learn Fivetran from fundamentals to advanced operations, then revise quickly before knowledge-transfer (KT) sessions and project discussions.
> 
> 
> **Source:** `FivetranTechnicalFoundations.txt` supplied for this study session.
> 
> **Important scope note:** Toyota Motor North America examples below are **hypothetical learning scenarios**, not claims about Toyota’s actual systems or architecture. Your confirmed project context includes Cognizant, Toyota Motor North America, AWS, Databricks, Spark and Scala. Fivetran was separately mentioned as something you may work with; confirm the actual source systems, destinations, configurations, ownership and runbooks with your team.
> 
> **Freshness note:** Product features, supported connectors, deployment options, pricing, plan restrictions and technical behaviors can change. The source includes 2023 company statistics and several plan-specific details. Treat those as notes from the supplied material, not guaranteed current terms; verify current documentation before making production or billing decisions.
> 

---

# 1. The whole topic in one go

## The 60-second recap

Fivetran is a managed data-movement platform. It connects a **source** (where data originates) to a **destination** (where data is delivered), using a configured **connection**. It automates much of extraction, loading, normalization, incremental synchronization, and connector maintenance.

The source file mainly describes an **ELT** approach:

**Extract → Load → Transform**

- **Extract:** Read data from an application, database, event source or files.
- **Load:** Deliver data to a destination such as a warehouse or lake.
- **Transform:** Apply business rules after loading, using a suitable transformation tool or compute engine.

A typical, hypothetical Toyota data flow could be:

```
Dealership service database / sales application / files
                         |
                         v
                 Fivetran connector
                         |
             Extract + normalize + sync
                         |
                         v
          Raw/landing tables in destination
                         |
              Validate and transform
          (Databricks/Spark/SQL, as chosen)
                         |
                         v
             Curated analytics datasets
                         |
                         v
           Reports / applications / AI use
```

This is an example architecture only. It does **not** confirm that Toyota uses this exact pipeline or that all these technologies are combined in your project.

## Remember these distinctions

| Term | Meaning |
| --- | --- |
| Source | The system where data starts |
| Destination | The system where data is loaded |
| Connector | The reusable integration type, e.g. a PostgreSQL connector |
| Connection | One configured pipeline instance linking a source to a destination |
| Sync | A run that extracts/processes/loads data |
| Historical/initial sync | The first load of existing data |
| Incremental sync | A sync of new or changed data rather than a full reload |
| CDC | Change Data Capture: a method for detecting inserts, updates and deletes |
| ELT | Extract, Load, Transform — transform after loading |
| Schema drift | Source tables/columns/types change over time |
| Checkpoint/cursor | Progress information that helps a sync resume or continue correctly |
| MAR | Monthly Active Rows, a Fivetran usage/billing metric described in the source |
| Soft delete | Keep the row but mark it deleted |
| History mode / SCD Type 2 | Preserve successive versions of a row over time |
| RBAC | Role-Based Access Control: permissions assigned through roles |

## What Fivetran automates vs. what engineers still own

**Fivetran commonly handles (connector-dependent):**

- Extracting source data and loading it into the destination.
- Repeated synchronization and tracking progress.
- Type mapping, normalization and some deduplication.
- Detecting/handling many schema changes.
- Sync status, logs and alerts.

**Engineers still need to handle:**

- Correct source/destination access and secure network configuration.
- Choosing the right connector, tables, columns, sync schedule and settings.
- Data quality checks, reconciliation and business logic.
- Destination capacity, downstream transformations and data models.
- Monitoring, incident response, permissions, governance and cost.
- Confirming connector-specific behavior instead of assuming every source works the same way.

**Memory click:** *Fivetran moves and maintains the data pipeline; engineers make sure the data is secure, correct, useful and affordable.*

---

# 2. Company overview and why managed ingestion matters

The supplied notes say Fivetran was founded in **2012** through Y Combinator and aims to reduce pipeline maintenance by automating data movement.

The file lists these **2023 figures**:

- 700+ pre-built connectors
- 6,600+ customers, including Fortune 500 companies
- 99.9% platform uptime
- 3M+ syncs daily

These are historical figures from the source; do not present them as current without checking.

## Problems with hand-built pipelines

- **Brittle custom code:** API, authentication or endpoint changes can break scripts.
- **Schema drift:** New/renamed/removed columns may require manual changes.
- **Maintenance tax:** Engineers spend time repairing pipelines instead of improving analytics.
- **Scaling and observability gaps:** Multiple sources, missed data and weak monitoring become harder to manage.

### Everyday analogy

Hand-built pipelines are like personally driving every package from every shop to every warehouse. A managed service is like using a logistics provider that handles much of the collection and delivery process. You still choose what to ship, where it goes, and how to check that the shipment is useful.

### Hypothetical Toyota example

Suppose an analytics team receives service records from several dealership systems. Custom scripts might fail when an API changes or a source adds a field. A configured Fivetran connection may reduce connector-maintenance work, but the team still needs to check whether the loaded service records are complete and whether downstream models understand the new field.

## Four value pillars in the source

1. **High-performance pipelines:** Reliable, timely delivery.
2. **Managed automation:** Less manual connector maintenance.
3. **Extensibility:** SDKs and APIs for custom requirements.
4. **Security and governance:** Controls intended for enterprise data protection.

---

# 3. ELT and the Fivetran sync loop

## ELT: Extract, Load, Transform

| Stage | What happens | Typical output |
| --- | --- | --- |
| Extract | Read data from the source using a supported method such as an API, file read or CDC | Source records/change events |
| Load | Write the data to the configured destination | Destination tables/files |
| Transform | Apply business rules after loading | Cleaned, joined, aggregated or curated datasets |

The source describes ELT as preserving raw data so teams can re-model it later without always re-extracting from the source. The exact retention and replay options depend on the platform, connector and destination configuration.

### Everyday analogy

A kitchen receives raw ingredients first, stores them safely, and then prepares different meals. ELT similarly loads source data first and shapes it for different uses afterward.

### Hypothetical Toyota example

A hypothetical service dataset lands with repair codes, timestamps and vehicle identifiers. A later transformation may standardize repair codes, reject invalid dates and calculate monthly service counts. Fivetran’s ingestion should not be confused with that business-specific transformation.

## The sync loop

```
1. CONNECT
   Authenticate and discover available source objects
        |
        v
2. EXTRACT
   Read the initial data or new/changed records
        |
        v
3. NORMALIZE / PROCESS
   Map data types and apply connector-specific handling
        |
        v
4. LOAD
   Write rows and schema to the destination
        |
        v
5. CHECK PROGRESS
   Track cursors/checkpoints and resume where supported
        |
        v
6. REPEAT
   Run again on the configured schedule or trigger
```

The source describes cursors and checkpoints as mechanisms for tracking progress and resuming interrupted syncs. Exact behavior varies by connector and table characteristics.

**Memory click:** *Connect → Extract → Normalize → Load → Track progress → Repeat.*

---

# 4. Deployment models and shared responsibility

## SaaS vs. Hybrid

| Model | Source-file description | What to remember |
| --- | --- | --- |
| SaaS | Fivetran-managed cloud service; some plans allow geography/provider/region selection | Easier managed setup; verify the selected processing location and plan |
| Hybrid | Uses a self-hosted agent inside the customer’s network | Intended for environments with specific network, residency or compliance requirements |

The source describes Hybrid as using outbound-only mutual Transport Layer Security (mTLS), without requiring open inbound ports, and says only metadata/logs reach the control plane. It also contains broad statements about data staying within the customer’s perimeter. **Do not assume these statements apply to every connector or Hybrid architecture.** Verify the exact data path, agent type, deployment model and current product documentation with the security/network team.

## Shared responsibility

**Fivetran responsibilities described in the source**

- Automated extraction, normalization and pipeline reliability.
- Handling many schema changes and migrations.

**Customer/engineering responsibilities described in the source**

- Granting read access to the source and write/create access at the destination.
- Sizing and operating destination compute.
- Post-load SQL models, business logic and internal access controls.

### Hypothetical Toyota example

If a dealership-data sync fails because the source account lost read permission, an engineer may need to coordinate access restoration. If ingestion succeeds but a monthly repair-rate report is wrong, the issue may instead be in transformation logic or the business definition.

**Memory click:** *The service moves the data; the customer controls access, destination readiness, meaning and downstream use.*

---

# 5. Connectors, connections and source categories

## Connector vs. connection

- **Connector = the type of integration.** Example: PostgreSQL connector.
- **Connection = the configured instance.** It identifies the specific source, destination, configuration and sync behavior.
- One connector type can be used by multiple connections.

### Everyday analogy

A connector is like a model of delivery truck. A connection is one specific delivery route configured for a particular pickup point and destination.

### Hypothetical Toyota example

A PostgreSQL connector could support separate configured connections for two hypothetical business databases. Those are separate connection instances, even if they use the same connector type.

## Source categories in the file

| Category | Examples listed in source |
| --- | --- |
| Applications | Salesforce, Google Ads, Facebook Ads |
| Databases | Oracle, SQL Server, PostgreSQL |
| Events | Apache Kafka, Azure Event Hubs, webhooks |
| Files | Amazon S3, Azure Blob, Google Cloud Storage, Google Sheets |
| Functions | Cloud-function sources for custom pulls |
| Logs | Fivetran Platform Connector for account metadata/logs |

The file describes a catalog of 700+ connectors and a Connector SDK for custom connectors using Python. Treat connector availability as subject to current support status.

## SaaS application connector flow

1. **Connect:** Authenticate using OAuth where available or a source-specific method.
2. **Discover:** During initial sync, discover accessible standard and custom objects.
3. **Map:** Map column types and handle unsupported types as the connector allows.
4. **Load:** Create destination tables and load the initial dataset.
5. **Sync:** Apply new/changed records and schema updates as supported.

The source describes SaaS application connectors as pull-based and notes that they do not perform business aggregation as part of basic replication.

## Schema selection and deleted rows

- Default selection may sync all accessible objects, depending on connector/configuration.
- Tables and columns can often be included/excluded through schema controls, subject to connector/API limits.
- The source says primary keys cannot be blocked or hashed because they are needed for tracking and deduplication.
- **Soft delete:** Mark a deleted record using `_fivetran_deleted`.
- **History mode:** Keep historical versions using Slowly Changing Dimension (SCD) Type 2-style behavior.

**Memory click:** *Connector is the recipe; connection is the configured pipeline.*

---

# 6. Database connectors and Change Data Capture (CDC)

## What CDC means

**Change Data Capture** detects changes made to source data—usually inserts, updates and deletes—so the destination can be kept up to date without repeatedly copying every row.

### Everyday analogy

Instead of rereading an entire notebook every day, you check the pages that changed since yesterday.

### Hypothetical Toyota example

A hypothetical service database receives new repair orders and updates existing orders. A CDC-based pipeline can replicate the changes to the destination. Downstream logic must still check that the replicated data satisfies reporting requirements.

## Initial sync vs. incremental replication

- **Initial/historical sync:** Reads existing source history to establish the destination baseline.
- **Incremental sync:** Reads new or changed data based on connector-specific change tracking.
- **Log-based CDC:** Reads database change logs or transaction logs when supported.
- **Query-based capture:** Uses source queries or metadata to identify changes when the native log-based option is unavailable.

## Replication methods listed in the source

| Source database/service | Method listed in the file |
| --- | --- |
| MySQL / MariaDB | Binary Log (native CDC) |
| PostgreSQL | Logical replication / `pgoutput`, or query-based `xmin` / `ctid` methods |
| SQL Server | CDC / Change Tracking |
| Databricks | Change Data Feed |
| MongoDB / Cosmos DB / DocumentDB | Change Streams |
| Amazon DynamoDB | DynamoDB Streams |
| BigQuery / Snowflake | Time Travel |
| Oracle / SQL Server / Db2 for i / SAP | High-Volume Agent for the listed use cases |

**Important:** This is the mapping in the supplied notes, not a guarantee that every option is currently available for every connector, plan or configuration. Check the current connector-specific setup guide.

## Database access and production safety

The source describes remote agentless database connections using a dedicated source user and potentially a production database or read replica. Networking options listed include direct connection, SSH/reverse SSH, VPN, AWS PrivateLink, Azure Private Link, Google Private Service Connect (PSC), and Proxy Agent.

Good-citizen principles from the file:

- Prefer low-impact source queries and avoid unnecessary production contention.
- Exclude unneeded schemas, tables and columns.
- Monitor connection status and sync state.
- Keep source change logs available long enough for the sync frequency; the source recommends 48+ hours.
- The file describes continuous replication as the intended use, not a one-time migration mechanism.

### PostgreSQL notes in the source

- It says Teleport Sync and XMIN are sunset for PostgreSQL and replaced by query-based capture using `xmin` / `ctid` where appropriate.
- Existing Teleport connections are described as continuing to run.
- Logical replication with `pgoutput` is listed as the log-based option.
- The file says Teleport on other databases uses read-only connections and in-memory row-hash comparisons without requiring transaction logs.

Treat sunset/deprecation details as time-sensitive and verify them before changing a live connector.

## High-Volume Agent (HVA)

The source describes HVA as an on-premises agent near the source that reads transaction logs for certain enterprise systems such as Oracle, SQL Server, SAP ECC and Db2 for i.

Source-described flow:

1. Install an agent/service on or near the source host.
2. Capture log changes asynchronously.
3. Compress and encrypt the payload.
4. Transmit over a secure TCP/IP connection; the source lists port 4343 as the default.
5. Deliver the data through the configured Fivetran pipeline.

| Standard remote database connector | HVA |
| --- | --- |
| Agentless remote CDC | Agent installed near source |
| No source-host software installation in the standard setup | Requires installation |
| Described as suitable for small-to-medium volumes | Described for high-volume, low-latency enterprise pipelines |

Verify current supported systems, ports, architecture and security requirements before implementation.

**Memory click:** *Initial sync builds the baseline; CDC keeps that baseline current.*

---

# 7. Event connectors and file ingestion

## Event connectors

The file describes event connectors for messaging systems, event buses and webhooks, including high event volumes.

Source-described behavior:

- New events are generally appended to tables.
- Unparseable events may be skipped with an alert.
- Deletions are not propagated because events represent historical occurrences.
- Schema drift is not propagated in the same way as database connectors; untracked metrics may become `NULL`, and tables may remain while no longer receiving updates.
- XML payloads may land in a text column when configured as `Message Type = Text`.

**Practical implication:** Do not assume event connectors behave like mutable database tables. Validate event-specific behavior.

## File connectors

The source lists file sources including:

- Cloud storage: Amazon S3, Azure Blob, Google Cloud Storage and S3-compatible storage.
- Collaboration/transfer: Box, Dropbox, Google Drive, OneDrive, SharePoint, FTP/SFTP, email and Google Sheets.

Listed file formats:

- CSV/TSV, JSON, Avro, Parquet, Excel, Google Sheets and XML.
- Compression listed: TAR, ZIP, BZip2 and GZip; the source says compression is not supported in Magic Folder mode.
- Encodings listed: UTF-8, UTF-16 and UTF-32.

The file describes these connectors as pull-based and notes that parsing settings can be configured per connection for formats such as CSV, JSON and Excel.

## Magic Folder vs. Merge mode

| Feature | Magic Folder mode | Merge mode |
| --- | --- | --- |
| Mapping | One file to one destination table | File-name patterns map files into shared tables |
| Incremental behavior | Source says no incremental sync; re-import based on last-modified detection | Source says incremental sync is supported |
| System columns | `_line` | `_file`, `_line`, `_modified` |
| Primary key | Automatically assigned `_line` | Custom primary key, file + line, or append mode |
| Multiple files to one schema | Source says no | Source says yes |
| Use case in source | Simple, low-volume ad-hoc folders | High-volume, recurring incremental feeds |

### Hypothetical Toyota example

A one-off file of service-location reference data might suit a simple folder import. A recurring set of daily service files that must merge into shared destination tables may need a different mode. The correct choice depends on file naming, keys, update behavior and connector support.

---

# 8. Sync scheduling, history, re-syncs and checkpoints

## Scheduling options listed in the source

- **Fixed interval:** Default listed as every six hours; described as available on all plans.
- **Cron:** Custom time-based scheduling; the source describes it as Private Preview with Enterprise/Business Critical restrictions and unavailable on Hybrid.
- **Manual/API-triggered:** Triggered through the Fivetran REST API; described as available on all plans for custom orchestration.

These availability and plan details may have changed. Check current product documentation and your account’s available settings.

## Historical, incremental and priority-first sync

- **Historical sync:** Copies the selected existing source data to establish a baseline.
- **Incremental sync:** Processes new/changed data using progress cursors.
- **Priority-first sync:** The source describes recent data being loaded first (forward sync) so it becomes available earlier, while older history is backfilled in background runs.

## Re-sync types

- **Connection re-sync:** Re-fetches all source data for the connection and rebuilds destination state according to connector behavior.
- **Table-level re-sync:** Re-syncs one table rather than the entire connection.
- **Automatic re-sync:** The source says Fivetran may trigger one due to source schema changes or expired change logs.

A re-sync can be useful for recovery or data-integrity issues, but it can also increase source load and change processing volume. Follow your team’s runbook before starting one.

## Teleport Sync

The source describes Teleport Sync as comparing in-memory row-hash snapshots instead of reading change logs. It lists use cases such as no change-log access, selective table/column syncs, heavy logs, and invalidated logs after `TRUNCATE` or `RESTORE`. It also says the method is sunset for PostgreSQL but remains available on other databases. Verify current support.

## Checkpoints and primary keys

- A checkpoint marks progress so a failed sync can resume rather than restart from the beginning.
- The source says checkpoints apply to historical and incremental syncs.
- The source notes that tables without a primary key cannot checkpoint and may need to restart.

Connector-specific limitations matter; do not generalize this behavior without checking the relevant connector.

**Memory click:** *A checkpoint is a bookmark, not the data itself.*

---

# 9. Schema naming, schema drift and governance

## Naming conventions

The source compares two modes:

| Topic | Fivetran naming | Source naming |
| --- | --- | --- |
| Names | Normalized, typically lowercase with ASCII letters, numbers and underscores | Preserves source names, including UTF-8 names |
| Database renaming | Source says it may prepend the connection name | No renaming, according to the source |
| Setup timing | Listed for supported connections | Must be selected for new database connections before initial sync |
| Compatibility | Source says broad support and Quickstart transformation compatibility | Source says support is limited to a subset and incompatible with Quickstart transformations |
| Change later | Locked after initial sync | Locked after initial sync |

The source also says the destination schema name is configured at setup and permanently locked after the initial sync. Treat this as a configuration warning and verify the current connector/destination behavior.

## Schema drift: what happens when the source changes?

| Source change | Behavior described in the file |
| --- | --- |
| Add a column/table | Automatically created in destination |
| Widen a type, e.g. `INT` to `BIGINT` | Widening can propagate |
| Narrow a type | Does not propagate according to source |
| Rename a column/table | Handled as add + delete; old object may remain |
| Remove a column/table | Data may remain in destination; a re-sync may be triggered |

**Why it matters:** Downstream SQL and Spark code can break if a field is renamed, its type changes, or an unexpected field appears.

### Hypothetical Toyota example

If a hypothetical service source renames `repair_code` to `service_code`, the destination may end up with a new column while the old one remains. A Databricks transformation that still reads only `repair_code` may fail or produce incomplete results.

## Schema governance controls

- **Allow All:** Most permissive; permits new schemas, tables and columns according to the source’s description.
- **Allow Columns:** Middle ground; permits new columns on existing tables but blocks new schemas and configurable tables.
- **Block All:** Most restrictive; blocks new schemas, configurable tables and new columns.

Exact controls and scope vary by connector. Choose based on data governance requirements and downstream compatibility.

## Row filtering

The source says row filters append a `WHERE` clause to source `SELECT` queries and lists:

- Types: integer/serial, date/timestamp and string.
- Operators: `=`, `!=`, `<`, `>`, `<=`, `>=`, `IS NULL`, `IS NOT NULL`, plus string `Contains` and `Starts With`.
- Supported connectors listed: most database connectors, including MySQL, PostgreSQL, MariaDB, SQL Server and SAP HANA.
- Limits in the source: up to three conditions per filter and one filter per table.
- Scope in the source: initial syncs and re-syncs only, not incremental syncs.

These are source-file details to validate against the current connector.

## Normalization

The file describes Fivetran normalization as reducing redundancy and mapping data into a canonical schema, approximately Third Normal Form (3NF), for sources that need it. It notes that:

- Database sources may already have normalized structures.
- Application/API data may need normalization.
- First-level JSON attributes may be promoted to typed columns; deeply nested data may remain as raw JSON.

Normalization is not the same as business transformation. It does not automatically know every Toyota-specific business rule.

---

# 10. Destinations, tables and data types

## Destination basics

A destination is the target system where Fivetran loads data. The source lists warehouses, databases, streaming systems and managed data lakes.

Destinations listed in the source:

- **Cloud warehouses:** Snowflake, BigQuery, Redshift, Azure Synapse and Databricks.
- **Databases:** PostgreSQL, SQL Server, MySQL (listed as Beta in the source) and Oracle (listed as Beta).
- **Streaming:** Apache Kafka.
- **Managed lake:** Managed Data Lake Service using S3, ADLS or GCS.
- **Other/partner-built options:** MotherDuck, ClickHouse Cloud, SingleStore, SurrealDB, Materialize, Teradata and Convex; Milvus is described as partner-built/private preview.

This is a source inventory, not a current support guarantee.

## Base tables: important safety rule

The source says:

- Fivetran creates and manages base tables.
- One base table cannot receive data from multiple connections.
- Do **not** hand-edit Fivetran-managed base tables because later syncs can overwrite changes.
- Put custom business logic in downstream views or transformation models.
- Destination-specific performance tuning such as sort keys, distribution keys and indexing may be possible.

### Hypothetical Toyota example

Keep an ingested service table as the replicated source representation. Create a separate curated view/table for standardizing repair codes and calculating service KPIs. Do not put business-rule edits directly into a table managed by the sync process.

## Data types and type promotion

The source lists these generalized types:

| Category | Types in the source |
| --- | --- |
| Boolean | `BOOLEAN` |
| Integers | `SHORT`, `INT`, `LONG` |
| Decimal/floating point | `BIGDECIMAL`, `FLOAT`, `DOUBLE` |
| Date/time | `LOCALDATE`, `LOCALDATETIME`, `INSTANT` |
| Text/semi-structured | `STRING`, `JSON`, `XML` |
| Binary | `BINARY` |
- **Explicit inference:** The source provides the type.
- **Implicit inference:** Type is inferred from values, such as `"9"` → integer, `"9.90"` → double, `"t"`/`"f"` → Boolean in the source’s examples.
- If a type cannot be inferred, a column may be skipped/not created, according to the source.
- The source’s widening hierarchy is `SHORT → INT → LONG → STRING`.
- On mixed types, the source gives examples such as `INT + DOUBLE → DOUBLE`; a value that cannot safely fit may result in a wider type such as `STRING`.

## Destination length edge cases

The source describes:

- Oversized `STRING`: truncated to the destination’s maximum length.
- Oversized `JSON`: replaced with a `{"FIVETRAN_ERROR": "..."}` object.
- Oversized `XML`: replaced with an XML error message.
- Oversized `BINARY`: replaced with `NULL` plus a dashboard warning.

These are important data-quality risks; actual behavior depends on destination and current connector behavior. Validate data after loading, especially where truncation or conversion can affect business meaning.

---

# 11. Managed Data Lake Service (MDLS) and Open Data Infrastructure (ODI)

## MDLS: what the source describes

Managed Data Lake Service (MDLS) is described as turning cloud object storage into a warehouse-like, queryable data foundation.

Source-file features:

- Data lands in the customer’s own S3, Azure Data Lake Storage (ADLS) or Google Cloud Storage bucket.
- Fivetran manages data movement but, according to the file, does not hold the customer’s stored data in its own storage.
- Data is written as Parquet using open table formats such as Apache Iceberg or Delta Lake.
- It supports ACID transactions, schema evolution, automated CDC/normalization/deduplication and file compaction.
- Data is registered in a catalog and can be queried by compatible engines.
- Catalogs listed include Fivetran Catalog (built on Apache Polaris), AWS Glue, Databricks Unity, BigLake and Snowflake.
- The file says MDLS is SaaS-only and does not support Fivetran in-destination ELT transformations.

Verify current product scope, supported table formats, catalog integrations and restrictions before relying on them.

## Open Data Infrastructure (ODI)

ODI in this file means **Open Data Infrastructure**, an architectural approach focused on open standards, customer control and reducing vendor lock-in. It is not the same as a specific integration product.

Three pillars in the source:

1. **Standards-based movement:** Ingest and transform using open standards and portable formats.
2. **Open data-lake foundation:** Store data in open formats such as Iceberg or Delta Lake, separating storage from compute.
3. **Unified consumption:** Reuse semantics, governance and activation across analytics, BI, ELT and AI.

## ODI flow in the source

```
Sources
  |
  v
Fivetran connectors / ingestion
  |
  v
Open table formats in managed data lake
(Iceberg / Delta Lake)
  |
  v
Normalization + CDC + deduplication + compaction
  |
  v
Catalog + governance + lineage
  |
  v
BI / ELT / activation / AI consumers
```

## AI-related features listed in the file

- **AI Columns in Activations:** Use large language models (LLMs) to enrich, classify or summarize data during operational activation.
- **Dataset SQL generation:** Natural-language prompts draft SQL for review/execution.
- **Bring Your Own Key (BYOK):** Use customer-managed model keys with providers listed in the source, including Anthropic, Google Gemini and OpenAI.
- **Privacy controls:** The source says data is not used for external model training and that administrators can enable/disable AI features at account level.

These claims and product capabilities are time-sensitive. Verify current contractual terms, privacy documentation and feature availability before using them with sensitive data.

**Memory click:** *ODI is the open-data approach; MDLS is a specific managed data-lake offering described by the source.*

---

# 12. Soft deletes, history mode and delete detection

## Soft delete vs. history mode

| Behavior | Soft delete (default, as described) | History mode / SCD Type 2 |
| --- | --- | --- |
| Main goal | Represent the current row state | Preserve successive versions |
| Updates | Overwrite current row state | Append a new version and close the prior active version |
| Deletes | Flag with `_fivetran_deleted` | Mark the active version inactive and set its end timestamp |
| Row count | Typically one current row per primary key | Multiple historical versions can exist |

The source describes history mode using `_fivetran_active` and an end timestamp that may initially be `9999-12-31`. Exact column names and behavior depend on connector/settings.

## How deletions are detected

- **Direct detection:** Source logs or API events explicitly report a delete.
- **Inferred detection:** Compare source state during a full re-sync with destination state when the source method does not emit delete events.

The source notes that PostgreSQL query-based `xmin` capture cannot detect deletes in the same way as WAL/log-based capture. Validate the chosen replication method’s delete semantics.

### Hypothetical Toyota example

If a hypothetical repair order is removed from the source, a soft-delete destination may retain its row with a deletion flag. A report that ignores the flag could count it incorrectly. A history-mode table may preserve the row’s earlier active version for audit or historical reporting.

**Memory click:** *Soft delete tells you a row was deleted; history mode tells you how the row changed over time.*

---

# 13. Transformations and Quickstart data models

## In-destination transformations

The source describes:

- **Quickstart data models:** Pre-built, no-code transformations maintained through Fivetran dbt packages, intended to provide reporting models for selected connectors.
- **Bring Your Own dbt:** Connect Fivetran-hosted dbt Core or third-party orchestration such as dbt Cloud and Coalesce to customer dbt repositories.

The file says models may include:

- Single-source models: cleaned and test-validated staging/transformation tables for one connector.
- Multi-source rollups: harmonized datasets across connectors, such as advertising or commerce reporting.

## Transformation billing details in the source

- Billing metric: successful monthly model runs, not Monthly Active Rows (MAR), for Quickstart data models and Fivetran-hosted dbt Core.
- First 5,000 successful model runs per month described as free.
- The file says Fivetran Platform Quickstart models do not consume usage credits.
- Model runs are measured separately from connector MAR.
- Transformations are described as unavailable on MDLS.

**Treat all pricing and billing figures as time-sensitive source notes.** Verify the current pricing page and your contract before estimating costs.

## Ingestion vs. transformation

| Ingestion | Transformation |
| --- | --- |
| Moves source data into a destination | Changes data structure or meaning for use |
| Often connector-driven | Usually business-rule/model-driven |
| Example: replicate service records | Example: standardize repair codes and calculate monthly counts |

### Hypothetical Toyota example

Fivetran may replicate hypothetical repair records into a destination. A downstream SQL, Databricks or Spark transformation might standardize codes, handle invalid dates and produce a curated dataset. Which tool is actually used depends on your team’s architecture.

---

# 14. Observability, monitoring and troubleshooting

## Four observability mechanisms in the source

1. **Dashboard and sync history:** Status, duration and data volume across Extract, Process and Load.
2. **Alerts:** Errors and warnings shown on the connection page with remediation guidance.
3. **Notifications:** Email and integrations listed for Slack, Microsoft Teams, PagerDuty, Opsgenie and ServiceNow.
4. **Logging:** Structured events via the Platform Connector or an external logging service.
5. **System status:** The source lists `status.fivetran.com` for platform-wide health.

## Reading sync history

The source says the Status page can show pipeline status, duration and volume over time, including a split across:

- **Extract:** Reading from the source.
- **Process:** Processing/normalization work.
- **Load:** Writing to the destination.

Look for the stage whose duration or volume changed unexpectedly. A long Extract stage may point toward source/network/query behavior; a long Load stage may point toward destination performance or capacity. These are diagnostic clues, not proof by themselves.

## Errors vs. warnings

| Errors | Warnings |
| --- | --- |
| Blocking according to the source | Non-blocking according to the source |
| Sync execution halts until resolved | Sync continues but needs review |
| Examples: lost source connection/permissions, destination misconfiguration/quota, repeated failures | Examples: excluded objects without access, file permission issues, change-tracking gaps |

The file mentions three consecutive failed syncs as one condition that can trigger an error. Confirm the current alert rules for your connector.

## Practical troubleshooting sequence

For a hypothetical Toyota service-data connection:

1. **Check status:** Is the connection running, paused, delayed or failing?
2. **Read the exact error/warning:** Do not guess from the red/yellow indicator alone.
3. **Check platform health:** Determine whether the issue is wider than one connection.
4. **Check source access:** Credentials, permissions, API limits, source availability and network path.
5. **Check destination readiness:** Credentials, write permissions, quota/capacity and schema constraints.
6. **Compare sync stages:** Extract, Process and Load durations/volumes.
7. **Check schema changes:** Added, renamed or type-changed columns may affect downstream code.
8. **Check freshness and row counts:** Did data arrive late, stop, duplicate or disappear?
9. **Review CDC/log retention:** Were required source logs lost or expired?
10. **Follow the runbook:** Re-sync only when justified and authorized.
11. **Validate recovery:** Confirm the next successful sync and downstream data quality.
12. **Document root cause:** Symptom, cause, fix, validation and prevention.

### Example SQL checks

These are **illustrative**. Replace table/column names with the actual destination schema and use syntax supported by your destination.

```sql
-- Check whether the newest records are arriving.
SELECT MAX(updated_at) AS latest_source_update
FROM service_records;
```

```sql
-- Find possible duplicate business keys.
SELECT service_record_id, COUNT(*) AS row_count
FROM service_records
GROUP BY service_record_id
HAVING COUNT(*) > 1;
```

```sql
-- Check for missing vehicle identifiers.
SELECT COUNT(*) AS missing_vehicle_ids
FROM service_records
WHERE vehicle_id IS NULL;
```

If the source/destination uses Fivetran system columns, inspect them only after confirming the actual columns and semantics for that connector.

**Memory click:** *Status → Error → Source → Network → Destination → Schema → Data checks → Recovery → Validate → Document.*

---

# 15. Monthly Active Rows (MAR) and cost optimization

## MAR definition in the source

**Monthly Active Rows (MAR)** is described as the number of distinct primary keys synced from source to destination during a calendar month.

Source-file rules:

- A primary key is counted once per calendar month, regardless of how many times it is updated/synced.
- Counts are separate by account, destination, connection and table.
- The same row replicated through two connections can count twice.
- For keyless tables, the source says Fivetran generates a synthetic hashed key from selected columns.

## Free vs. paid MAR examples in the source

**Listed as free:**

- Syncs during a new connection’s 14-day trial.
- Initial/historical baseline syncs.
- Certain re-syncs due to added columns or no-change scenarios (source notes post-March 2025).
- Source delete capture itself.

**Listed as paid:**

- Incremental inserts/updates/deletes after the trial.
- Adding or re-enabling a table.
- Re-adding rows to the destination after deletion.
- History-mode rows.
- Changes to non-key columns in keyless tables when the synthetic hash changes.
- Some rollback/priority-first cases involving prior-month or newly introduced data.
- Replicating the same source data through multiple connections/destinations.

The source describes a **$5/month connection base charge** for certain usage conditions, a per-connection volume curve, and older contract terms for some contracts signed before March 1, 2025. These are not reliable current quotes—check your plan, current pricing and contract.

## Common MAR spikes listed in the source

- Start-of-month activity after the distinct-row count resets.
- New tables/columns expanding synced data.
- Pausing and resuming a connection after changes accumulated.
- Deleting and recreating connections.
- Resuming without an appropriate re-sync.
- Trial expiration.

## Cost optimization levers in the source

- Adjust sync frequency to business latency needs; the file says cadence itself does not increase MAR.
- Consolidate usage per connection where architecturally appropriate.
- Exclude unneeded schemas/tables.
- Exclude high-churn columns on keyless tables if suitable.
- Investigate unexpected MAR changes before recreating connections or changing sync configuration.

### Hypothetical Toyota example

A hypothetical service pipeline starts syncing an additional table of repair events. Even if the existing connection remains healthy, usage may change because the new table introduces rows that count under the current pricing rules. Confirm the billing definition before taking action.

**Memory click:** *Sync frequency and billed rows are different concepts—check the current MAR rules, not just how often the pipeline runs.*

---

# 16. Data residency, security and access control

## Data residency and processing location

The source lists these plan differences:

- Free/Standard: processing geography only, described as Google Cloud only.
- Enterprise: geography plus cloud provider (GCP, AWS or Azure).
- Business Critical: geography, provider and specific region.

Geographies listed include US, Canada, EU, UK, Australia, Singapore, India, Japan, Indonesia and the Middle East.

The source also lists retention/purge statements:

- Destination data purged in under eight hours.
- Temporary staging data purged in under 24 hours.
- Cached data from blocking/outages auto-purges after 30 days.

These statements are product-specific and potentially contract-dependent. Verify current retention terms. The file says processing location is set per destination and inherited by its connections, and that network safelisting should match the chosen geography/region.

## Column blocking and hashing

### Column blocking

- Prevents selected columns/tables from being newly synced.
- Can support data minimization and privacy requirements.
- The source says primary keys cannot be blocked.
- Blocking a column does not reduce MAR according to the source.
- Existing values already in the destination remain until manually removed; blocking stops incoming new values.

### Column hashing

- The source describes one-way SHA-256 hashing, using a recipe of value + salt, UTF-8 encoding, SHA-256 and Base64 output.
- It says a destination-specific salt allows matching values to produce consistent hashes across connections in the same destination.
- Account administrators may access the salt in security settings, according to the source.
- Primary keys cannot be hashed; the file says hashing is supported on connectors except Magic Folder and requires re-sync to apply to historical records.

**Security warning:** Hashing is not automatically equivalent to anonymization. Confirm approved use, threat model and policy with the security/privacy team. Never expose sensitive source values or salts in logs or notes.

## Role-Based Access Control (RBAC)

Scopes described in the source:

- **Account scope:** Users, billing and account-wide settings; example roles include Account Administrator and Account Reviewer.
- **Destination scope:** Destination and its connections; examples include Destination Administrator and Destination Creator.
- **Connection scope:** Individual connection configuration and sync operations; examples include Connection Administrator and Create Connection.
- Enterprise/Business Critical are described as supporting custom roles and teams, with permissions cascading through the hierarchy.

## Single sign-on and organizations

The source lists:

- **SAML 2.0 Single Sign-On (SSO):** Microsoft Entra ID, Google Workspace, Okta, OneLogin, PingOne and RSA.
- **Just-in-Time (JIT) provisioning:** Creates user accounts after first successful login.
- **Session control:** Automatic logout after 24 hours of inactivity in the source, customizable.
- **Organizations:** Business Critical feature for governing multiple accounts, aggregating usage, enforcing global security settings and consolidating billing.

Verify current plan and configuration details.

## Diagnostic access and support

The source says:

- Fivetran staff have no customer-data access by default.
- Support access requires explicit administrator approval.
- Debug environments are isolated and subject to automatic deletion/access-expiration windows listed in the source.
- The file makes a stronger claim for Hybrid/self-hosted setups that support cannot access data under any circumstance.

Treat these as claims from the supplied notes; verify current security and support documentation and your organization’s contract before relying on them.

**Memory click:** *Least privilege, data minimization, correct region, and explicit approval are core governance habits.*

---

# 17. Networking and connection methods

The source lists these connection approaches:

| Method | Description in the source | Main consideration |
| --- | --- | --- |
| Direct connection | Public internet to a safelisted endpoint, encrypted with TLS 1.2+ | Safelist the correct Fivetran IPs for the provider/region |
| Forward SSH tunnel | Route through a customer-managed jump server/bastion | Database ports can remain internal |
| Reverse SSH tunnel | Customer environment initiates a connection toward Fivetran | No inbound ports required, according to the source |
| VPN tunnel | Site-to-site IPsec tunnel | Strong isolation but more setup and maintenance |
| Proxy Agent | Outbound-only agent; source lists WebSocket over TLS 1.3 | Avoids opening inbound ports |
| Private Link | AWS/Azure/Google private connectivity | Source says provider and region must match; plan restrictions apply |
| Hybrid deployment | Agent/pipeline deployed within customer network | Validate the exact data path and deployment-specific security model |

### Everyday analogy

Networking is choosing how a courier can reach a secure building: public entrance with an approved list, a guarded side entrance, a private tunnel, or an approved on-site delivery agent.

### Hypothetical Toyota example

If a hypothetical source database is private and cannot be exposed publicly, the network/security team may evaluate a tunnel, private connectivity or agent-based design. The engineer should not open ports or change safelists without approved network guidance.

**Memory click:** *Choose the network path based on the source location, security rules, supported connector and approved architecture.*

---

# 18. Complete troubleshooting playbook

Use this checklist when a sync fails, is delayed or produces unexpected data.

## A. Sync failure

- [ ]  Record connection name, source, destination and time of failure.
- [ ]  Read the full error text and recent sync history.
- [ ]  Check platform status.
- [ ]  Confirm credentials and source/destination permissions.
- [ ]  Check network, IP safelisting, tunnel/agent and firewall status.
- [ ]  Check API limits, source availability, destination quota and capacity.
- [ ]  Identify whether Extract, Process or Load is the failing/slow stage.
- [ ]  Apply the approved fix and confirm the next successful sync.

## B. Data is missing or stale

- [ ]  Check last successful sync and latest record timestamp.
- [ ]  Confirm table/column selection and row filters.
- [ ]  Check source permissions and whether the source object still exists.
- [ ]  Check incremental cursor/log retention and CDC configuration.
- [ ]  Check schema drift, renamed columns and type changes.
- [ ]  Compare source and destination counts using the same filters/time window.
- [ ]  Determine whether a table-level or connection re-sync is justified.
- [ ]  Validate downstream models after recovery.

## C. Duplicate records

- [ ]  Identify the true business key/primary key.
- [ ]  Check whether the source key is stable and unique.
- [ ]  Inspect connector-specific primary-key and deduplication behavior.
- [ ]  Check whether the same source data is being loaded through multiple connections.
- [ ]  Check downstream joins and transformation logic.
- [ ]  Do not delete production data until the cause and safe remediation are confirmed.

## D. Unexpected schema change

- [ ]  Identify whether a column/table was added, renamed, removed or type-changed.
- [ ]  Check destination schema and system metadata.
- [ ]  Find downstream SQL/Spark jobs that reference the old schema.
- [ ]  Confirm whether the connector performed add/delete handling or triggered a re-sync.
- [ ]  Update transformations with versioned, reviewed code.
- [ ]  Test and validate before production release.

## E. Unexpected MAR/cost increase

- [ ]  Compare current usage with prior months.
- [ ]  Check new/re-enabled tables and added connections/destinations.
- [ ]  Check keyless tables and high-churn columns.
- [ ]  Check pause/resume, re-sync and trial status.
- [ ]  Read the current contract/pricing rules before changing configuration.

## Root-cause template

```
Incident:
Connection / source / destination:
Start time and duration:
Observed symptom:
Exact error or warning:
Impact on downstream data:
Evidence checked:
Root cause:
Fix applied:
Validation performed:
Preventive action:
Owner / follow-up:
```

---

# 19. Hypothetical Toyota end-to-end walkthrough

**Scenario (fictional):** An analytics team wants a reliable dataset of dealership service records for monthly service reporting.

## Stage 1 — Source

A hypothetical dealership service application stores repair orders, vehicle identifiers, service dates and repair codes.

- **Input:** Application/database records.
- **Risk:** Missing vehicle IDs, invalid dates, permission changes or schema updates.
- **Validation:** Confirm required source objects, keys and access.

## Stage 2 — Fivetran connection

An engineer configures the supported connector, authenticates, chooses the destination and selects the data to sync.

- **Input:** Source access and connection settings.
- **Output:** Replicated tables at the destination.
- **Risk:** Authentication, network, schema or sync failure.
- **Validation:** Review initial sync, row counts, freshness and warnings.

## Stage 3 — Raw/landing data

The destination contains the replicated records and connector metadata/system columns where applicable.

- **Purpose:** Keep an accessible representation of source data.
- **Risk:** Stale data, deletes, unexpected types, renamed columns or duplicates.
- **Validation:** Check primary keys, timestamps, nulls and delete semantics.

## Stage 4 — Transformation (tool depends on architecture)

A downstream process might use SQL, Databricks or Spark to standardize repair codes, validate dates and create monthly aggregates.

- **Input:** Replicated records.
- **Output:** Curated dataset.
- **Risk:** Incorrect business rules, bad joins, schema mismatch or failed compute job.
- **Validation:** Tests, reconciliation and comparison with approved business definitions.

## Stage 5 — Analytics or application

A report or application consumes the curated dataset.

- **Input:** Curated tables/views.
- **Output:** Metrics or business-facing data.
- **Risk:** Misleading metrics if data is incomplete or business definitions differ.
- **Validation:** Freshness, totals, key metrics and business-owner sign-off.

## The end-to-end distinction

Fivetran ingestion success does **not** automatically prove the final report is correct. A successful sync proves only that the connector’s work completed according to its configured behavior. Data quality, business rules and downstream jobs still need validation.

---

# 20. Glossary

| Term | Simple meaning |
| --- | --- |
| API | Application Programming Interface; a defined way for software to communicate |
| ACID | Atomicity, Consistency, Isolation and Durability; transaction guarantees |
| ADLS | Azure Data Lake Storage |
| ASCII | A character encoding covering a basic set of characters |
| Backfill | Load older historical data after more recent data is available |
| Bastion / jump server | A controlled server used to reach a private network |
| BYOK | Bring Your Own Key; use a customer-managed key for a supported service |
| CDC | Change Data Capture |
| Checkpoint | Saved progress marker used for recovery/resumption |
| Connector | Reusable integration type for a source |
| Connection | Configured instance of an integration pipeline |
| Cursor | State/position used to determine which data to read next |
| Destination | Target system for loaded data |
| ELT | Extract, Load, Transform |
| ETL | Extract, Transform, Load; transform before loading to the final destination |
| GCP | Google Cloud Platform |
| HVA | High-Volume Agent |
| Iceberg | Apache Iceberg, an open table format |
| Incremental sync | Sync only new/changed data as supported |
| Initial sync | First load of existing data |
| JIT | Just-in-Time provisioning |
| JSON | JavaScript Object Notation, a structured data format |
| LLM | Large Language Model |
| MAR | Monthly Active Rows |
| MDLS | Managed Data Lake Service |
| mTLS | Mutual Transport Layer Security; both sides authenticate |
| Normalization | Restructure data to reduce redundancy and standardize representation |
| ODI | Open Data Infrastructure in this source |
| OLTP | Online Transaction Processing; systems that handle operational transactions |
| Parquet | Column-oriented file format commonly used for analytics |
| PSC | Private Service Connect (Google Cloud) |
| RBAC | Role-Based Access Control |
| Re-sync | Re-read data to rebuild or repair destination state |
| S3 | Amazon Simple Storage Service |
| SCD Type 2 | Slowly Changing Dimension technique that preserves historical versions |
| Schema | Structure of data: tables, columns and types |
| Schema drift | Change in source structure over time |
| SaaS | Software as a Service |
| SAML | Security Assertion Markup Language, used for identity federation/SSO |
| SFTP | Secure File Transfer Protocol |
| SSO | Single Sign-On |
| Sync | A run that reads, processes and loads source data |
| TLS | Transport Layer Security; protects data in transit |
| WAL | Write-Ahead Log, used by PostgreSQL for database changes |
| Webhook | HTTP-based mechanism for sending an event to a configured endpoint |

---

# 21. Knowledge-transfer and interview questions

Try answering without looking at the notes. Use the answers to find what you need to revise.

## Fundamentals

1. What problem does Fivetran solve?
2. What is the difference between ETL and ELT?
3. What is the difference between a connector and a connection?
4. What is a destination?
5. What does a successful sync prove—and what does it not prove?

## Syncs and CDC

1. What is the difference between an initial sync and an incremental sync?
2. What is CDC, and why can it reduce the need for full-table reads?
3. What is the purpose of a cursor or checkpoint?
4. What is the difference between soft delete and history mode?
5. Why must CDC behavior be checked per connector?

## Schema and data quality

1. What is schema drift?
2. How might a renamed source column affect downstream Spark code?
3. Why should you avoid manually editing Fivetran-managed base tables?
4. How would you check for missing IDs and duplicate business keys?
5. What should you validate after a re-sync?

## Operations and security

1. How would you troubleshoot a failed sync?
2. What is the difference between a blocking error and a warning?
3. What can Extract/Process/Load timing tell you?
4. Why do permissions, log retention and network settings matter?
5. What is RBAC, and why is least privilege important?

## Cost and architecture

1. What is MAR according to the supplied notes?
2. Why might adding a table affect usage?
3. What is the difference between a file Magic Folder and Merge mode?
4. What is MDLS, and how does it relate to open table formats?
5. In a hypothetical Toyota pipeline, which checks belong to ingestion and which belong to downstream transformation?

## Scenario practice

**Scenario 1:** The sync is green, but yesterday’s service records are missing from a report. What do you check first?

**Scenario 2:** A source field is renamed and a Databricks job fails. How do you find the cause and prevent recurrence?

**Scenario 3:** A destination table contains deleted rows marked with `_fivetran_deleted`. Why might a report still count them?

**Scenario 4:** A connection is failing after a credentials rotation. What evidence do you collect before changing anything?

**Scenario 5:** MAR unexpectedly increases. What usage/configuration changes do you investigate before recreating the connection?

**Scenario 6:** A source table has no primary key. What questions do you ask about deduplication, checkpointing and connector behavior?

---

# 22. Memory notes — quick revision

- **Fivetran:** Managed data movement; reduces custom connector maintenance.
- **ELT:** Extract → Load → Transform.
- **Connector vs. connection:** Integration type vs. configured pipeline instance.
- **Initial vs. incremental:** Baseline load vs. new/changed data.
- **CDC:** Detect source changes; exact method depends on source connector.
- **Checkpoint:** A bookmark for sync progress.
- **Schema drift:** Source structure changes; inspect downstream impact.
- **Base tables:** Treat as Fivetran-managed; place custom logic downstream.
- **Soft delete vs. history mode:** Mark deleted current state vs. preserve versions over time.
- **Monitoring:** Status → exact error → Extract/Process/Load → source/network/destination/schema → data validation.
- **MAR:** Billing/usage metric in the source; check current contract and pricing.
- **Security:** Least privilege, minimum required data, approved network path and correct region.
- **Toyota example:** A hypothetical service-record pipeline can ingest successfully yet still need data-quality tests and downstream business-rule validation.

## Final memory click

**“Connect it, sync it, monitor it, validate it, then trust it.”**

---

# 23. What to confirm with your project team

These are questions, not assumptions about Toyota’s actual architecture:

- Which source systems and connector types are in scope?
- Which destination is used for the ingested data?
- Which tables/columns are included, excluded or filtered?
- What is the configured sync schedule and freshness requirement?
- Which CDC/replication method is used, and how are deletes handled?
- Who owns credentials, network access and destination permissions?
- Where are sync logs, alerts and operational runbooks?
- Are transformations performed in Databricks/Spark, SQL, another tool, or more than one place?
- What data-quality checks and reconciliation totals are expected?
- What is the approved procedure for table-level re-syncs and connection re-syncs?
- Which team owns incidents and downstream failures?
- Which current Fivetran plan, billing terms and security controls apply?

**Project rule:** Confirm architecture and operational behavior from project documentation and your team. Use these notes to learn the concepts; do not treat hypothetical examples or product notes as proof of the actual Toyota implementation.