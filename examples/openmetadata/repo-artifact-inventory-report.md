# Repository Artifact Inventory Report — OpenMetadata

| Field | Value |
|-------|-------|
| **Repository** | [OpenMetadata](https://github.com/open-metadata/OpenMetadata) |
| **Analyzed** | 2026-06-17 |
| **Analyzer** | Universal Repository Artifact Inventory Agent |
| **Scope** | Full repository (excludes `target/`, `node_modules/`, generated build output) |
| **Repo shape** | Polyglot Maven monorepo (12 modules) + Python ingestion + Airflow APIs + React UI |

---

## Executive Summary

OpenMetadata is an all-in-one metadata management platform combining a **Dropwizard/Jersey Java API server**, a **React/Vite frontend**, a **Python ingestion framework**, **Airflow REST APIs**, and a **Kubernetes operator**. The backend exposes 118 JAX-RS resources auto-registered via `CollectionRegistry`, with JDBI3 repositories for 100+ entity types. Domain models are defined as **367 JSON Schema entity definitions** in `openmetadata-spec`, codegen'd to Java. The architecture is a **modular monolith** with an **event-driven notification subsystem** (Disruptor + Quartz consumers/publishers).

---

## 1. Technology Stack

| Dimension | Detected Value | Evidence | Confidence |
|-----------|----------------|----------|------------|
| Languages | Java, Python, TypeScript | `pom.xml` modules; `ingestion/pyproject.toml`; UI `package.json` | High |
| Backend framework | Dropwizard 5 + Jersey (JAX-RS) | `OpenMetadataApplication extends Application`; `environment.jersey().register(...)` | High |
| Frontend framework | React + Vite | `index.tsx`: `createRoot(container).render(<AppRoot />)`; `"vite"` in scripts | High |
| Python ingestion | setuptools + Airflow provider | `pyproject.toml`: `metadata = metadata.cmd:metadata` | High |
| Airflow APIs | Flask Blueprint REST | `get_blueprint()` in `openmetadata_managed_apis/api/app.py` | High |
| K8s operator | Java Operator SDK | `OMJobOperatorApplication` uses `io.javaoperatorsdk.operator.Operator` | High |
| Build tool | Maven multi-module | Root `pom.xml` `<modules>` (12 modules) | High |
| Package managers | Maven, Yarn, pip | `pom.xml`, `yarn.lock`, `pyproject.toml` | High |
| Persistence | JDBI3 + SQL | `CollectionDAO`, `EntityDAO`, `*Repository extends EntityRepository` | High |
| Search | OpenSearch / Elasticsearch | `SearchRepository.java`; shaded-deps modules | High |
| Messaging / events | Disruptor + Quartz | `EventPublisher` interface; `EventPubSub`; `AuditLogConsumer implements Job` | High |
| Architecture | Modular monolith + event-driven | Single primary deployable + auxiliary apps; change-event pipeline | High |

---

## 2. Repository Structure

```
OpenMetadata/
├── pom.xml                          # Maven parent (12 modules)
├── conf/                            # Runtime YAML configs
├── ingestion/                       # Python metadata ingestion framework
├── openmetadata-airflow-apis/       # Flask REST for Airflow integration
├── openmetadata-service/            # Core Dropwizard API server
│   └── src/main/java/.../
│       ├── resources/               # JAX-RS controllers (118 *Resource.java)
│       ├── jdbi3/                   # Repositories + DAOs
│       ├── events/                  # Event publishers, handlers, schedulers
│       └── apps/                    # Native applications, change-event bundles
├── openmetadata-spec/               # JSON Schema → generated Java models
├── openmetadata-ui/                 # React SPA (Vite)
├── openmetadata-sdk/                # Java SDK entity wrappers
├── openmetadata-mcp/                # MCP protocol server
├── openmetadata-k8s-operator/       # OMJob Kubernetes operator
└── openmetadata-clients/            # Generated Java HTTP clients
```

---

## 3. Architecture Overview

| Layer | Location | Role |
|-------|----------|------|
| HTTP API | `resources/*Resource.java` | JAX-RS handlers; registered via `CollectionRegistry` |
| Application logic | `jdbi3/*Repository.java`, `*Service.java` | Entity CRUD, business rules |
| Persistence | `jdbi3/*DAO.java`, `CollectionDAO` | JDBI SQL access |
| Schema / contracts | `openmetadata-spec/.../json/schema/` | Entity, API, type JSON schemas |
| Search | `search/SearchRepository.java` | OpenSearch/ES indexing |
| Events | `events/`, `apps/bundles/changeEvent/` | Change events → publishers/consumers |
| Ingestion | `ingestion/src/metadata/` | Connector workflows (Python) |
| UI | `openmetadata-ui/.../ui/src/` | React SPA |

**Wiring proof:** `CollectionRegistry.registerResources()` calls `environment.jersey().register(resource)` for each `@Collection`-annotated resource class.

**Event flow:** DB change events → `EventPubSub` (Disruptor) → `EventPublisher` implementations (Slack, email, Teams) + `AuditLogConsumer` (Quartz job).

---

## 4. Main Entry Points

| File Path | Evidence | Confidence |
|-----------|----------|------------|
| `openmetadata-service/.../OpenMetadataApplication.java` | `main` → Dropwizard bootstrap | High |
| `openmetadata-k8s-operator/.../OMJobOperatorApplication.java` | `main` → Java Operator SDK | High |
| `openmetadata-ui/.../ui/src/index.tsx` | `createRoot(container).render(<AppRoot />)` | High |
| `ingestion/pyproject.toml` | `metadata = metadata.cmd:metadata` CLI entry | High |
| `ingestion/operators/docker/main.py` | Docker operator entry script | High |
| `openmetadata-airflow-apis/.../api/app.py` | Flask `get_blueprint()` REST entry | High |

---

## 5. Architecture Summary

| Artifact Type | Verified | Inferred | Total |
|---------------|----------|----------|-------|
| Controllers / Routes | 115 | 3 | 118 |
| Services | 12 | 91 | 103 |
| Interfaces | 2 | 131 | 133 |
| Repositories | 110 | 4 | 114 |
| Models (entity schemas) | 367 | 0 | 367 |
| DTOs (API schemas) | 169 | 0 | 169 |
| Type schemas | 102 | 0 | 102 |
| Jobs | 7 | 0 | 7 |
| Workers | 4 | 0 | 4 |
| Consumers | 4 | 2 | 6 |
| Producers (Publishers) | 11 | 0 | 11 |
| Configurations | 7 | 0 | 7 |
| Utilities | 267 | 0 | 267 |
| Tests | 2,810 | 0 | 2,810 |
| Main Entry Points | 6 | 0 | 6 |

---

## Detailed Artifact Inventory

The sections below list **every discovered artifact** with file path, evidence, and confidence. Findings are grouped by artifact type and split into **Verified** and **Inferred**.

**Scope:** `/Users/chiragmogha/Downloads/repo-inventory-practice/OpenMetadata` (excludes `target/`, `node_modules/`, generated build output)
**Repo shape:** Polyglot Maven monorepo (12 modules) + Python ingestion + Airflow APIs

## Controllers / Routes
### Verified Findings
Count: 114
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ChangeSummaryResource.java | `@Path("/v1/changeSummary")` on `class ChangeSummaryResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/activity/ActivityResource.java | `@Path("/v1/activity")` on `class ActivityResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/AIApplicationResource.java | `@Path("/v1/aiApplications")` on `class AIApplicationResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/AIGovernancePolicyResource.java | `@Path("/v1/aiGovernancePolicies")` on `class AIGovernancePolicyResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/AgentExecutionResource.java | `@Path("/v1/agentExecutions")` on `class AgentExecutionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/LLMModelResource.java | `@Path("/v1/llmModels")` on `class LLMModelResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/McpExecutionResource.java | `@Path("/v1/mcpExecutions")` on `class McpExecutionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/McpServerResource.java | `@Path("/v1/mcpServers")` on `class McpServerResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/ai/PromptTemplateResource.java | `@Path("/v1/promptTemplates")` on `class PromptTemplateResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/analytics/ReportDataResource.java | `@Path("/v1/analytics/dataInsights/data")` on `class ReportDataResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/analytics/WebAnalyticEventResource.java | `@Path("/v1/analytics/web/events")` on `class WebAnalyticEventResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/apis/APICollectionResource.java | `@Path("/v1/apiCollections")` on `class APICollectionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/apis/APIEndpointResource.java | `@Path("/v1/apiEndpoints")` on `class APIEndpointResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/apps/AppMarketPlaceResource.java | `@Path("/v1/apps/marketplace")` on `class AppMarketPlaceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/apps/AppResource.java | `@Path("/v1/apps")` on `class AppResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/attachments/AttachmentResource.java | `@Path("/v1/attachments")` on `class AttachmentResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/audit/AuditLogResource.java | `@Path("/v1/audit/logs")` on `class AuditLogResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/automations/WorkflowResource.java | `@Path("/v1/automations/workflows")` on `class WorkflowResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/bots/BotResource.java | `@Path("/v1/bots")` on `class BotResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/charts/ChartResource.java | `@Path("/v1/charts")` on `class ChartResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/columns/ColumnResource.java | `@Path("/v1/columns")` on `class ColumnResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/context/ContextMemoryResource.java | `@Path("/v1/contextCenter/memories")` on `class ContextMemoryResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dashboards/DashboardResource.java | `@Path("/v1/dashboards")` on `class DashboardResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/data/DataContractResource.java | `@Path("/v1/dataContracts")` on `class DataContractResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/databases/DatabaseResource.java | `@Path("/v1/databases")` on `class DatabaseResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/databases/DatabaseSchemaResource.java | `@Path("/v1/databaseSchemas")` on `class DatabaseSchemaResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/databases/StoredProcedureResource.java | `@Path("/v1/storedProcedures")` on `class StoredProcedureResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/databases/TableResource.java | `@Path("/v1/tables")` on `class TableResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/datainsight/DataInsightChartResource.java | `@Path("/v1/analytics/dataInsights/charts")` on `class DataInsightChartResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/datainsight/system/DataInsightSystemChartResource.java | `@Path("/v1/analytics/dataInsights/system/charts")` on `class DataInsig`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/datamodels/DashboardDataModelResource.java | `@Path("/v1/dashboard/datamodels")` on `class DashboardDataModelResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/docstore/DocStoreResource.java | `@Path("/v1/docStore")` on `class DocStoreResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/domains/DataProductResource.java | `@Path("/v1/dataProducts")` on `class DataProductResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/domains/DomainResource.java | `@Path("/v1/domains")` on `class DomainResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestCaseDimensionResultResource.java | `@Path("/v1/dataQuality/testCases/dimensionResults")` on `class TestCaseDimensionResultResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestCaseResolutionStatusResource.java | `@Path("/v1/dataQuality/testCases/testCaseIncidentStatus")` on `class TestCaseResolutionStatusResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestCaseResource.java | `@Path("/v1/dataQuality/testCases")` on `class TestCaseResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestCaseResultResource.java | `@Path("/v1/dataQuality/testCases/testCaseResults")` on `class TestCaseResultResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestDefinitionResource.java | `@Path("/v1/dataQuality/testDefinitions")` on `class TestDefinitionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/dqtests/TestSuiteResource.java | `@Path("/v1/dataQuality/testSuites")` on `class TestSuiteResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drive/ContextFileResource.java | `@Path("/v1/contextCenter/drive/files")` on `class ContextFileResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drive/FolderResource.java | `@Path("/v1/contextCenter/drive/folders")` on `class FolderResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drives/DirectoryResource.java | `@Path("/v1/drives/directories")` on `class DirectoryResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drives/FileResource.java | `@Path("/v1/drives/files")` on `class FileResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drives/SpreadsheetResource.java | `@Path("/v1/drives/spreadsheets")` on `class SpreadsheetResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/drives/WorksheetResource.java | `@Path("/v1/drives/worksheets")` on `class WorksheetResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/entityProfiles/EntityProfileResource.java | `@Path("/v1/entity/profiles")` on `class EntityProfileResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/events/EventResource.java | `@Path("/v1/events")` on `class EventResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/events/NotificationTemplateResource.java | `@Path("/v1/notificationTemplates")` on `class NotificationTemplateResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/events/subscription/EventSubscriptionResource.java | `@Path("/v1/events/subscriptions")` on `class EventSubscriptionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/feeds/AnnouncementResource.java | `@Path("/v1/announcements")` on `class AnnouncementResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/feeds/FeedResource.java | `@Path("/v1/feed")` on `class FeedResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/feeds/TaskFormSchemaResource.java | `@Path("/v1/taskFormSchemas")` on `class TaskFormSchemaResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/glossary/GlossaryResource.java | `@Path("/v1/glossaries")` on `class GlossaryResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/glossary/GlossaryTermResource.java | `@Path("/v1/glossaryTerms")` on `class GlossaryTermResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/governance/IntakeFormResource.java | `@Path("/v1/governance/intakeForms")` on `class IntakeFormResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/governance/WorkflowDefinitionResource.java | `@Path("/v1/governance/workflowDefinitions")` on `class WorkflowDefinitionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/governance/WorkflowInstanceResource.java | `@Path("/v1/governance/workflowInstances")` on `class WorkflowInstance`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/governance/WorkflowInstanceStateResource.java | `@Path("/v1/governance/workflowInstanceStates")` on `class WorkflowInstanceStateResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/knowledge/KnowledgePageResource.java | `@Path("/v1/contextCenter/pages")` on `class KnowledgePageResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/kpi/KpiResource.java | `@Path("/v1/kpi")` on `class KpiResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/learning/LearningResourceResource.java | `@Path("/v1/learning/resources")` on `class LearningResourceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/limits/LimitsResource.java | `@Path("/v1/limits")` on `class LimitsResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/lineage/LineageResource.java | `@Path("/v1/lineage")` on `class LineageResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/lineage/OpenLineageResource.java | `@Path("/v1/openlineage")` on `class OpenLineageResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/mcp/McpUsageResource.java | `@Path("/v1/mcp/usage")` on `class McpUsageResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/mcpclient/McpClientResource.java | `@Path("/v1/mcp-client")` on `class McpClientResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/metrics/MetricResource.java | `@Path("/v1/metrics")` on `class MetricResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/mlmodels/MlModelResource.java | `@Path("/v1/mlmodels")` on `class MlModelResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/permissions/PermissionsResource.java | `@Path("/v1/permissions")` on `class PermissionsResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/pipelines/PipelineResource.java | `@Path("/v1/pipelines")` on `class PipelineResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/policies/PolicyResource.java | `@Path("/v1/policies")` on `class PolicyResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/query/QueryCostResource.java | `@Path("/v1/queryCostRecord")` on `class QueryCostResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/query/QueryResource.java | `@Path("/v1/queries")` on `class QueryResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/rdf/RdfResource.java | `@Path("/v1/rdf")` on `class RdfResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/rdf/RdfSqlResource.java | `@Path("/v1/rdf/sql")` on `class RdfSqlResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/reports/ReportResource.java | `@Path("/v1/reports")` on `class ReportResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/scim/ScimResource.java | `@Path("/v1/scim")` on `class ScimResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/search/SearchReindexResource.java | `@Path("/v1/search/reindex")` on `class SearchReindexResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/search/SearchResource.java | `@Path("/v1/search")` on `class SearchResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/search/VectorSearchResource.java | `@Path("/v1/search/vector")` on `class VectorSearchResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/searchindex/SearchIndexResource.java | `@Path("/v1/searchIndexes")` on `class SearchIndexResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/apiservices/APIServiceResource.java | `@Path("/v1/services/apiServices")` on `class APIServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/connections/TestConnectionDefinitionResource.java | `@Path("/v1/services/testConnectionDefinitions")` on `class TestConnectionDefiniti`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/dashboard/DashboardServiceResource.java | `@Path("/v1/services/dashboardServices")` on `class DashboardServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/database/DatabaseServiceResource.java | `@Path("/v1/services/databaseServices")` on `class DatabaseServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/drive/DriveServiceResource.java | `@Path("/v1/services/driveServices")` on `class DriveServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/ingestionpipelines/IngestionPipelineResource.java | `@Path("/v1/services/ingestionPipelines")` on `class IngestionPipelineResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/llm/LLMServiceResource.java | `@Path("/v1/services/llmServices")` on `class LLMServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/mcp/McpServiceResource.java | `@Path("/v1/services/mcpServices")` on `class McpServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/messaging/MessagingServiceResource.java | `@Path("/v1/services/messagingServices")` on `class MessagingServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/metadata/MetadataServiceResource.java | `@Path("/v1/services/metadataServices")` on `class MetadataServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/mlmodel/MlModelServiceResource.java | `@Path("/v1/services/mlmodelServices")` on `class MlModelServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/pipeline/PipelineServiceResource.java | `@Path("/v1/services/pipelineServices")` on `class PipelineServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/searchIndexes/SearchServiceResource.java | `@Path("/v1/services/searchServices")` on `class SearchServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/security/SecurityServiceResource.java | `@Path("/v1/services/securityServices")` on `class SecurityServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/storage/StorageServiceResource.java | `@Path("/v1/services/storageServices")` on `class StorageServiceResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/storages/ContainerResource.java | `@Path("/v1/containers")` on `class ContainerResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/system/ConfigResource.java | `@Path("/v1/system/config")` on `class ConfigResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/system/DiagnosticsResource.java | `@Path("/v1/system/diagnostics")` on `class DiagnosticsResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/system/IndexResource.java | `@Path("/")` on `class IndexResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/system/SystemResource.java | `@Path("/v1/system")` on `class SystemResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/tags/ClassificationResource.java | `@Path("/v1/classifications")` on `class ClassificationResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/tags/TagResource.java | `@Path("/v1/tags")` on `class TagResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/tasks/TaskResource.java | `@Path("/v1/tasks")` on `class TaskResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/teams/PersonaResource.java | `@Path("/v1/personas")` on `class PersonaResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/teams/RoleResource.java | `@Path("/v1/roles")` on `class RoleResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/teams/TeamResource.java | `@Path("/v1/teams")` on `class TeamResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/testsupport/TestSupportSearchResource.java | `@Path("/v1/test-support/search")` on `class TestSupportSearchResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/topics/TopicResource.java | `@Path("/v1/topics")` on `class TopicResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/types/TypeResource.java | `@Path("/v1/metadata/types")` on `class TypeResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/usage/UsageResource.java | `@Path("/v1/usage")` on `class UsageResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/version/VersionResource.java | `@Path("/v1/system/version")` on `class VersionResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/swagger/SwaggerResource.java | `@Path("/")` on `class SwaggerResource`; registered via `CollectionRegistry.registerResources` → `environment.jersey().register(resource)` | High |

### Inferred Findings
Count: 4
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/resources/EntityResource.java | `class EntityResource` in resources package; no class-level `@Path` found (may inherit from base resource) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/EntityTimeSeriesResource.java | `class EntityTimeSeriesResource` in resources package; no class-level `@Path` found (may inherit from base resource) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/services/ServiceEntityResource.java | `class ServiceEntityResource` in resources package; no class-level `@Path` found (may inherit from base resource) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/resources/teams/UserResource.java | `class UserResource` in resources package; no class-level `@Path` found (may inherit from base resource) | Medium |

## Repositories
### Verified Findings
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/audit/AuditLogRepository.java | `class AuditLogRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AIApplicationRepository.java | `class AIApplicationRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AIGovernancePolicyRepository.java | `class AIGovernancePolicyRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/APICollectionRepository.java | `class APICollectionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/APIEndpointRepository.java | `class APIEndpointRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/APIServiceRepository.java | `class APIServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ActivityStreamRepository.java | `class ActivityStreamRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AgentExecutionRepository.java | `class AgentExecutionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AnnouncementRepository.java | `class AnnouncementRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AppMarketPlaceRepository.java | `class AppMarketPlaceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AppRepository.java | `class AppRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/AssetRepository.java | `class AssetRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/BotRepository.java | `class BotRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ChangeEventRepository.java | `class ChangeEventRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ChartRepository.java | `class ChartRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ClassificationRepository.java | `class ClassificationRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ColumnRepository.java | `class ColumnRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ContainerRepository.java | `class ContainerRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ContextFileContentRepository.java | `class ContextFileContentRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ContextFileRepository.java | `class ContextFileRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ContextMemoryRepository.java | `class ContextMemoryRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DashboardDataModelRepository.java | `class DashboardDataModelRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DashboardRepository.java | `class DashboardRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DashboardServiceRepository.java | `class DashboardServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DataContractRepository.java | `class DataContractRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DataInsightChartRepository.java | `class DataInsightChartRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DataInsightSystemChartRepository.java | `class DataInsightSystemChartRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DataProductRepository.java | `class DataProductRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DatabaseRepository.java | `class DatabaseRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DatabaseSchemaRepository.java | `class DatabaseSchemaRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DatabaseServiceRepository.java | `class DatabaseServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DirectoryRepository.java | `class DirectoryRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DocumentRepository.java | `class DocumentRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DomainRepository.java | `class DomainRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DriveServiceRepository.java | `class DriveServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EntityProfileRepository.java | `class EntityProfileRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EntityRelationshipRepository.java | `class EntityRelationshipRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EntityRepository.java | `class EntityRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EntityTimeSeriesRepository.java | `class EntityTimeSeriesRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EventSubscriptionRepository.java | `class EventSubscriptionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/FeedRepository.java | `class FeedRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/FileRepository.java | `class FileRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/FolderRepository.java | `class FolderRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/GlossaryRepository.java | `class GlossaryRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/GlossaryTermRepository.java | `class GlossaryTermRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/IngestionPipelineRepository.java | `class IngestionPipelineRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/IntakeFormRepository.java | `class IntakeFormRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/KnowledgePageRepository.java | `class KnowledgePageRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/KpiRepository.java | `class KpiRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/LLMModelRepository.java | `class LLMModelRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/LLMServiceRepository.java | `class LLMServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/LearningResourceRepository.java | `class LearningResourceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/LineageRepository.java | `class LineageRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/McpConversationRepository.java | `class McpConversationRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/McpExecutionRepository.java | `class McpExecutionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/McpMessageRepository.java | `class McpMessageRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/McpServerRepository.java | `class McpServerRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/McpServiceRepository.java | `class McpServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MessagingServiceRepository.java | `class MessagingServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MetadataServiceRepository.java | `class MetadataServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MetricRepository.java | `class MetricRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MlModelRepository.java | `class MlModelRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MlModelServiceRepository.java | `class MlModelServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/NotificationTemplateRepository.java | `class NotificationTemplateRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/PersonaRepository.java | `class PersonaRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/PipelineRepository.java | `class PipelineRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/PipelineServiceRepository.java | `class PipelineServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/PolicyRepository.java | `class PolicyRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/PromptTemplateRepository.java | `class PromptTemplateRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/QueryCostRepository.java | `class QueryCostRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/QueryRepository.java | `class QueryRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/RecognizerFeedbackRepository.java | `class RecognizerFeedbackRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ReportDataRepository.java | `class ReportDataRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ReportRepository.java | `class ReportRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/Repository.java | Base repository interface/class `Repository` in jdbi3 layer | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/RoleRepository.java | `class RoleRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SearchIndexRepository.java | `class SearchIndexRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SearchServiceRepository.java | `class SearchServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SecurityServiceRepository.java | `class SecurityServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/ServiceEntityRepository.java | `class ServiceEntityRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SessionRepository.java | `class SessionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SpreadsheetRepository.java | `class SpreadsheetRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/StorageServiceRepository.java | `class StorageServiceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/StoredProcedureRepository.java | `class StoredProcedureRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/SystemRepository.java | `class SystemRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TableRepository.java | `class TableRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TagRepository.java | `class TagRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TaskFormSchemaRepository.java | `class TaskFormSchemaRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TaskRepository.java | `class TaskRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TeamRepository.java | `class TeamRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestCaseDimensionResultRepository.java | `class TestCaseDimensionResultRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestCaseRepository.java | `class TestCaseRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestCaseResolutionStatusRepository.java | `class TestCaseResolutionStatusRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestCaseResultRepository.java | `class TestCaseResultRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestConnectionDefinitionRepository.java | `class TestConnectionDefinitionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestDefinitionRepository.java | `class TestDefinitionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TestSuiteRepository.java | `class TestSuiteRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TokenRepository.java | `class TokenRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TopicRepository.java | `class TopicRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/TypeRepository.java | `class TypeRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/UsageRepository.java | `class UsageRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/UserRepository.java | `class UserRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WebAnalyticEventRepository.java | `class WebAnalyticEventRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WorkflowDefinitionRepository.java | `class WorkflowDefinitionRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WorkflowInstanceRepository.java | `class WorkflowInstanceRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WorkflowInstanceStateRepository.java | `class WorkflowInstanceStateRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WorkflowRepository.java | `class WorkflowRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/WorksheetRepository.java | `class WorksheetRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/rdf/RdfRepository.java | `class RdfRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchRepository.java | `class SearchRepository extends EntityRepository`; JDBI3 persistence in `org.openmetadata.service.jdbi3` | High |

### Inferred Findings
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/repository/McpPendingAuthRequestRepository.java | `class McpPendingAuthRequestRepository`; repository naming in non-service module | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/repository/OAuthAuthorizationCodeRepository.java | `class OAuthAuthorizationCodeRepository`; repository naming in non-service module | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/repository/OAuthClientRepository.java | `class OAuthClientRepository`; repository naming in non-service module | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/repository/OAuthTokenRepository.java | `class OAuthTokenRepository`; repository naming in non-service module | Medium |

## Services
### Verified Findings
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/service/HealthCheckService.java | `class HealthCheckService`; used in `OMJobOperatorApplication` health check wiring | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/APIService.java | `class APIService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/DashboardService.java | `class DashboardService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/DatabaseService.java | `class DatabaseService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/MessagingService.java | `class MessagingService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/MetadataService.java | `class MetadataService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/MlModelService.java | `class MlModelService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/PipelineService.java | `class PipelineService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/SearchService.java | `class SearchService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/entities/StorageService.java | `class StorageService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/ai/AIApplicationService.java | `class AIApplicationService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/ai/LLMModelService.java | `class LLMModelService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/ai/McpServerService.java | `class McpServerService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/ai/PromptTemplateService.java | `class PromptTemplateService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/apiservice/APICollectionService.java | `class APICollectionService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/apiservice/APIEndpointService.java | `class APIEndpointService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/automations/WorkflowService.java | `class WorkflowService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/bots/BotService.java | `class BotService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/classification/ClassificationService.java | `class ClassificationService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/classification/TagService.java | `class TagService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/context/ContextMemoryService.java | `class ContextMemoryService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/ChartService.java | `class ChartService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/DashboardDataModelService.java | `class DashboardDataModelService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/DashboardService.java | `class DashboardService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/MetricService.java | `class MetricService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/MlModelService.java | `class MlModelService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/PipelineService.java | `class PipelineService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/QueryService.java | `class QueryService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/ReportService.java | `class ReportService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/SearchIndexService.java | `class SearchIndexService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/TableService.java | `class TableService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/dataassets/TopicService.java | `class TopicService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/databases/DatabaseSchemaService.java | `class DatabaseSchemaService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/databases/DatabaseService.java | `class DatabaseService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/databases/StoredProcedureService.java | `class StoredProcedureService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/datacontracts/DataContractService.java | `class DataContractService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/datainsight/DataInsightChartService.java | `class DataInsightChartService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/domains/DataProductService.java | `class DataProductService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/domains/DomainService.java | `class DomainService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/ContextFileService.java | `class ContextFileService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/DirectoryService.java | `class DirectoryService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/FileService.java | `class FileService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/FolderService.java | `class FolderService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/SpreadsheetService.java | `class SpreadsheetService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/drives/WorksheetService.java | `class WorksheetService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/events/ChangeEventService.java | `class ChangeEventService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/events/EventSubscriptionService.java | `class EventSubscriptionService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/events/NotificationTemplateService.java | `class NotificationTemplateService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/feed/AnnouncementService.java | `class AnnouncementService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/feed/FeedService.java | `class FeedService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/feed/TaskFormSchemaService.java | `class TaskFormSchemaService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/glossary/GlossaryService.java | `class GlossaryService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/glossary/GlossaryTermService.java | `class GlossaryTermService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/governance/AIGovernancePolicyService.java | `class AIGovernancePolicyService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/governance/WorkflowDefinitionService.java | `class WorkflowDefinitionService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/ingestion/IngestionPipelineService.java | `class IngestionPipelineService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/knowledge/PageService.java | `class PageService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/kpi/KpiService.java | `class KpiService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/learning/LearningResourceService.java | `class LearningResourceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/policies/PolicyService.java | `class PolicyService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/APIServiceService.java | `class APIServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/DashboardServiceService.java | `class DashboardServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/DatabaseServiceService.java | `class DatabaseServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/DriveServiceService.java | `class DriveServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/LLMServiceService.java | `class LLMServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/MessagingServiceService.java | `class MessagingServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/MetadataServiceService.java | `class MetadataServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/MlModelServiceService.java | `class MlModelServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/PipelineServiceService.java | `class PipelineServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/SearchServiceService.java | `class SearchServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/services/StorageServiceService.java | `class StorageServiceService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/storages/ContainerService.java | `class ContainerService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/storages/DirectoryService.java | `class DirectoryService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/storages/FileService.java | `class FileService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/storages/SpreadsheetService.java | `class SpreadsheetService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/storages/WorksheetService.java | `class WorksheetService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/tasks/TaskService.java | `class TaskService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/teams/PersonaService.java | `class PersonaService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/teams/RoleService.java | `class RoleService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/teams/TeamService.java | `class TeamService` in SDK/client module; API wrapper for entity type | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/services/teams/UserService.java | `class UserService` in SDK/client module; API wrapper for entity type | High |

### Inferred Findings
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/AssetService.java | `class AssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/AzureAssetService.java | `class AzureAssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/InMemoryAssetService.java | `class InMemoryAssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/NoOpAssetService.java | `class NoOpAssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/ObjectDeleteQueueService.java | `class ObjectDeleteQueueService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/QueuedDeleteAssetService.java | `class QueuedDeleteAssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/S3AssetService.java | `class S3AssetService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/context/ContextEntityPromptService.java | `class ContextEntityPromptService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/drive/ContextFileProcessingService.java | `class ContextFileProcessingService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mcpclient/McpClientService.java | `class McpClientService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/rdf/semantic/EmbeddingService.java | `class EmbeddingService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/rdf/sql2sparql/SqlToSparqlService.java | `class SqlToSparqlService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/scim/ScimProvisioningService.java | `class ScimProvisioningService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/scim/impl/DefaultScimProvisioningService.java | `class DefaultScimProvisioningService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/nlq/NLQService.java | `class NLQService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/nlq/NoOpNLQService.java | `class NoOpNLQService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/OpenSearchVectorService.java | `class OpenSearchVectorService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/VectorIndexService.java | `class VectorIndexService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/secrets/SecretsManagerUpdateService.java | `class SecretsManagerUpdateService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/policyevaluator/PermissionDebugService.java | `class PermissionDebugService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/session/SessionService.java | `class SessionService` in service module; business/integration logic (verify per-class wiring separately) | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AsyncService.java | `class AsyncService` in service module; business/integration logic (verify per-class wiring separately) | Medium |

## Interfaces
### Verified Findings (2)
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/events/EventPublisher.java | `public interface EventPublisher`; contract used across service layer | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/EntityInterface.java | `public interface EntityInterface`; contract used across service layer | High |

### Inferred Findings (131) — see full list
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-clients/openmetadata-java-client/src/main/java/org/openmetadata/client/api/ElasticSearchApi.java | `public interface ElasticSearchApi` | Medium |
| openmetadata-clients/openmetadata-java-client/src/main/java/org/openmetadata/client/security/interfaces/AuthenticationProvider.java | `public interface AuthenticationProvider` | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/auth/OAuthAuthorizationServerProvider.java | `public interface OAuthAuthorizationServerProvider` | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/prompts/McpPrompt.java | `public interface McpPrompt` | Medium |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/tools/McpTool.java | `public interface McpTool` | Medium |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/network/HttpClient.java | `public interface HttpClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/csv/CsvExportProgressCallback.java | `public interface CsvExportProgressCallback` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/csv/CsvImportProgressCallback.java | `public interface CsvImportProgressCallback` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/McpServerProvider.java | `public interface McpServerProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/NativeApplication.java | `public interface NativeApplication` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/Alert.java | `public interface Alert` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/Consumer.java | `public interface Consumer` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/Destination.java | `public interface Destination` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/search/DataInsightsSearchInterface.java | `public interface DataInsightsSearchInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/EnrichmentStep.java | `public interface EnrichmentStep` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/BulkSink.java | `public interface BulkSink` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/OrchestratorContext.java | `public interface OrchestratorContext` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/QuartzOrchestratorContext.java | `public interface StatusPusher` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingJobContext.java | `public interface ReindexingJobContext` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingProgressListener.java | `public interface ReindexingProgressListener` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedJobNotifier.java | `public interface DistributedJobNotifier` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/promotion/PromotionPolicy.java | `public interface PromotionPolicy` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/attachments/AssetService.java | `public interface AssetService` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/cache/CacheProvider.java | `public interface CacheProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/cache/Invalidatable.java | `public interface Invalidatable` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/clients/llm/LlmClient.java | `public interface LlmClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/clients/pipeline/config/types/WorkflowConfigTypeStrategy.java | `public interface WorkflowConfigTypeStrategy` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/dataInsight/DataInsightAggregatorInterface.java | `public interface DataInsightAggregatorInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/events/EventHandler.java | `public interface EventHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/events/lifecycle/EntityLifecycleEventHandler.java | `public interface EntityLifecycleEventHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/formatter/entity/EntityFormatter.java | `public interface EntityFormatter` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/formatter/field/FieldFormatter.java | `public interface FieldFormatter` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/elements/NodeInterface.java | `public interface NodeInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/elements/TriggerInterface.java | `public interface TriggerInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/SinkProvider.java | `public interface SinkProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/flowable/sql/SqlMapper.java | `public interface SqlMapper` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/DeletionLockDAO.java | `public interface DeletionLockDAO` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/EntityTimeSeriesDAO.java | `public interface EntityTimeSeriesDAO` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/IndexMappingVersionDAO.java | `public interface IndexMappingVersionDAO` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jdbi3/MigrationDAO.java | `public interface MigrationDAO` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jobs/JobDAO.java | `public interface JobDAO` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/jobs/JobHandler.java | `public interface JobHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/limits/Limits.java | `public interface Limits` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mapper/EntityMapper.java | `public interface EntityMapper` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mapper/EntityTimeSeriesMapper.java | `public interface EntityTimeSeriesMapper` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mapper/EntityUpdateMapper.java | `public interface EntityUpdateMapper` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mcpclient/ChatEventEmitter.java | `public interface ChatEventEmitter` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/mcpclient/ToolExecutor.java | `public interface ToolExecutor` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/api/MigrationProcessExtensionProvider.java | `public interface MigrationProcessExtensionProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/NotificationMessageEngine.java | `public interface NotificationMessageEngine` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/channels/ChannelRenderer.java | `public interface ChannelRenderer` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/channels/NotificationMessage.java | `public interface NotificationMessage` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/channels/TemplateFormatAdapter.java | `public interface TemplateFormatAdapter` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/recipients/downstream/DownstreamHandler.java | `public interface DownstreamHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/recipients/downstream/EntityLineageResolver.java | `public interface EntityLineageResolver` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/recipients/strategy/RecipientResolutionStrategy.java | `public interface RecipientResolutionStrategy` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/template/NotificationTemplateProcessor.java | `public interface NotificationTemplateProcessor` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/template/handlebars/HandlebarsHelper.java | `public interface HandlebarsHelper` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/notifications/template/testing/MockChangeEventRegistry.java | `public interface EntityFixtureBuilder` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/rdf/semantic/EmbeddingService.java | `public interface EmbeddingProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/rdf/storage/RdfStorageInterface.java | `public interface RdfStorageInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/scim/ScimProvisioningService.java | `public interface ScimProvisioningService` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/AggregationManagementClient.java | `public interface AggregationManagementClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/ColumnAggregator.java | `public interface ColumnAggregator` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/DataInsightAggregatorClient.java | `public interface DataInsightAggregatorClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/EntityManagementClient.java | `public interface EntityManagementClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/GenericClient.java | `public interface GenericClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/IndexManagementClient.java | `public interface IndexManagementClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/InheritedFieldEntitySearch.java | `public interface InheritedFieldEntitySearch` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/RecreateIndexHandler.java | `public interface RecreateIndexHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchClient.java | `public interface SearchClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchManagementClient.java | `public interface SearchManagementClient` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchRepositoryProvider.java | `public interface SearchRepositoryProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchRetryUtil.java | `public interface IOOperation` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchSourceBuilderFactory.java | `public interface SearchSourceBuilderFactory` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/elasticsearch/aggregations/ElasticAggregations.java | `public interface ElasticAggregations` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/elasticsearch/dataInsightAggregators/ElasticSearchDynamicChartAggregatorInterface.java | `public interface ElasticSearchDynamicChartAggregatorInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/indexes/ColumnIndex.java | `public interface ColumnIndex` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/indexes/DataAssetIndex.java | `public interface DataAssetIndex` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/indexes/LineageIndex.java | `public interface LineageIndex` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/indexes/ServiceBackedIndex.java | `public interface ServiceBackedIndex` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/indexes/TaggableIndex.java | `public interface TaggableIndex` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/lineage/LineageGraphCache.java | `public interface LineageGraphCache` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/lineage/LineageGraphExecutor.java | `public interface LineageGraphExecutor` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/lineage/LineageGraphStrategy.java | `public interface LineageGraphStrategy` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/lineage/LineageProgressTracker.java | `public interface LineageProgressTracker` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/nlq/NLQService.java | `public interface NLQService` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/opensearch/aggregations/OpenAggregations.java | `public interface OpenAggregations` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/opensearch/dataInsightAggregator/OpenSearchDynamicChartAggregatorInterface.java | `public interface OpenSearchDynamicChartAggregatorInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/queries/OMQueryBuilder.java | `public interface OMQueryBuilder` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/queries/QueryBuilderFactory.java | `public interface QueryBuilderFactory` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/VectorBodyTextContributor.java | `public interface VectorBodyTextContributor` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/VectorIndexService.java | `public interface VectorIndexService` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/AuthServeletHandler.java | `public interface AuthServeletHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/Authorizer.java | `public interface Authorizer` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/auth/AuthenticatorHandler.java | `public interface AuthenticatorHandler` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/auth/SecurityConfigurationManager.java | `public interface ConfigurationChangeListener` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/policyevaluator/ResourceContextInterface.java | `public interface ResourceContextInterface` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/session/SessionStore.java | `public interface SessionStore` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LambdaExceptionUtil.java | `public interface ConsumerWithExceptions` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ListWithOffsetFunction.java | `public interface ListWithOffsetFunction` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/branding/MessageBrandingProvider.java | `public interface MessageBrandingProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/AutoTuner.java | `public interface AutoTuner` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/Diagnostic.java | `public interface Diagnostic` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/TemplateProvider.java | `public interface TemplateProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProvider.java | `public interface DatabaseAuthenticationProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/RelationshipValidator.java | `public interface RelationshipValidator` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/ResourcePathProvider.java | `public interface ResourcePathProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/EmailEnvelopeResourcePathProvider.java | `public interface EmailEnvelopeResourcePathProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/NotificationTemplateResourcePathProvider.java | `public interface NotificationTemplateResourcePathProvider` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/workflows/interfaces/Processor.java | `public interface Processor` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/workflows/interfaces/Sink.java | `public interface Sink` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/workflows/interfaces/Source.java | `public interface Source` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/workflows/interfaces/Stats.java | `public interface Stats` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/AppRuntime.java | `public interface AppRuntime` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/BulkAssetsRequestInterface.java | `public interface BulkAssetsRequestInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/ColumnsEntityInterface.java | `public interface ColumnsEntityInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/CreateEntity.java | `public interface CreateEntity` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/DataInsightInterface.java | `public interface DataInsightInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/DocStoreEntityInterface.java | `public interface DocStoreEntityInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/EntityTimeSeriesInterface.java | `public interface EntityTimeSeriesInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/EnumInterface.java | `public interface EnumInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/FieldInterface.java | `public interface FieldInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/NamedEntityInterface.java | `public interface NamedEntityInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/ServiceConnectionEntityInterface.java | `public interface ServiceConnectionEntityInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/ServiceEntityInterface.java | `public interface ServiceEntityInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/SubscriptionAction.java | `public interface SubscriptionAction` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/TokenInterface.java | `public interface TokenInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/schema/governance/workflows/elements/WorkflowTriggerInterface.java | `public interface WorkflowTriggerInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/sdk/PipelineServiceClientInterface.java | `public interface PipelineServiceClientInterface` | Medium |
| openmetadata-spec/src/main/java/org/openmetadata/service/logstorage/LogStorageInterface.java | `public interface LogStorageInterface` | Medium |

## Models (Entity JSON Schemas)
Count: 367
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-spec/src/main/resources/json/schema/entity/activity/activityEvent.json | JSON Schema entity definition: `ActivityEvent` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/activity/activityStreamConfig.json | JSON Schema entity definition: `ActivityStreamConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/agentExecution.json | JSON Schema entity definition: `AgentExecution` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/aiApplication.json | JSON Schema entity definition: `AIApplication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/aiGovernancePolicy.json | JSON Schema entity definition: `AIGovernancePolicy` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/llmModel.json | JSON Schema entity definition: `LLMModel` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/mcpExecution.json | JSON Schema entity definition: `McpExecution` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/mcpServer.json | JSON Schema entity definition: `McpServer` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/ai/promptTemplate.json | JSON Schema entity definition: `PromptTemplate` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/app.json | JSON Schema entity definition: `App` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/appExtension.json | JSON Schema entity definition: `AppExtension` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/appRunRecord.json | JSON Schema entity definition: `AppRunRecord` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/applicationConfig.json | JSON Schema entity definition: `ApplicationConfigModel` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addCustomProperties.json | JSON Schema entity definition: `AddCustomPropertiesAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addDataProductAction.json | JSON Schema entity definition: `AddDataProductAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addDescriptionAction.json | JSON Schema entity definition: `AddDescriptionAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addDomainAction.json | JSON Schema entity definition: `AddDomainAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addOwnerAction.json | JSON Schema entity definition: `AddOwnerAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addTagsAction.json | JSON Schema entity definition: `AddTagsAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addTermsAction.json | JSON Schema entity definition: `AddTermsAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addTestCaseAction.json | JSON Schema entity definition: `AddTestCaseAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/addTierAction.json | JSON Schema entity definition: `AddTierAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/lineagePropagationAction.json | JSON Schema entity definition: `LineagePropagationAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/mlTaggingAction.json | JSON Schema entity definition: `MLTaggingAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/propagationStopConfig.json | JSON Schema entity definition: `PropagationStopConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeCustomPropertiesAction.json | JSON Schema entity definition: `RemoveCustomPropertiesAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeDataProductAction.json | JSON Schema entity definition: `RemoveDataProductAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeDescriptionAction.json | JSON Schema entity definition: `RemoveDescriptionAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeDomainAction.json | JSON Schema entity definition: `RemoveDomainAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeOwnerAction.json | JSON Schema entity definition: `RemoveOwnerAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeTagsAction.json | JSON Schema entity definition: `RemoveTagsAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeTermsAction.json | JSON Schema entity definition: `RemoveTermsAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeTestCaseAction.json | JSON Schema entity definition: `RemoveTestCaseAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automator/removeTierAction.json | JSON Schema entity definition: `RemoveTierAction` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/automatorAppConfig.json | JSON Schema entity definition: `AutomatorAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/collateAIAppConfig.json | JSON Schema entity definition: `CollateAIAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/collateAIQualityAgentAppConfig.json | JSON Schema entity definition: `CollateAIQualityAgentAppConfig.json` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/collateAITierAgentAppConfig.json | JSON Schema entity definition: `CollateAITierAgentAppConfig.json` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterAppConfig.json | JSON Schema entity definition: `MetadataExporterAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterConnectors/bigQueryConnection.json | JSON Schema entity definition: `BigQueryConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterConnectors/databricksConnection.json | JSON Schema entity definition: `DatabricksConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterConnectors/redshiftConnection.json | JSON Schema entity definition: `RedshiftConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterConnectors/snowflakeConnection.json | JSON Schema entity definition: `SnowflakeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/external/metadataExporterConnectors/trinoConnection.json | JSON Schema entity definition: `TrinoConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/autoPilotAppConfig.json | JSON Schema entity definition: `AutoPilotAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/cacheWarmupAppConfig.json | JSON Schema entity definition: `CacheWarmupApp` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/dataInsightsAppConfig.json | JSON Schema entity definition: `DataInsightsAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/dataInsightsReportAppConfig.json | JSON Schema entity definition: `DataInsightsReportAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/dataRetentionConfiguration.json | JSON Schema entity definition: `Retention Configuration` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/helloPipelinesConfiguration.json | JSON Schema entity definition: `Hello Pipelines App Configuration` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/mcpChatAppConfig.json | JSON Schema entity definition: `McpChatAppConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/rdfIndexingAppConfig.json | JSON Schema entity definition: `RdfIndexingApp` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/internal/searchIndexingAppConfig.json | JSON Schema entity definition: `SearchIndexingApp` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/private/external/collateAIAppPrivateConfig.json | JSON Schema entity definition: `CollateAIAppPrivateConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/private/internal/collateAITierAgentAppPrivateConfig.json | JSON Schema entity definition: `CollateAITierAgentAppPrivateConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/configuration/private/limits.json | JSON Schema entity definition: `AppLimitsConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/createAppRequest.json | JSON Schema entity definition: `CreateAppRequest` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/jobStatus.json | JSON Schema entity definition: `JobRun` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/liveExecutionContext.json | JSON Schema entity definition: `JobRun` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/marketplace/appMarketPlaceDefinition.json | JSON Schema entity definition: `AppMarketPlaceDefinition` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/marketplace/createAppMarketPlaceDefinitionReq.json | JSON Schema entity definition: `CreateAppMarketPlaceDefinitionRequest` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/mcp/mcpToolCallUsage.json | JSON Schema entity definition: `McpToolCallUsage` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/applications/scheduledExecutionContext.json | JSON Schema entity definition: `JobRun` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/automations/queryRunnerRequest.json | JSON Schema entity definition: `QueryRunnerRequest` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/automations/response/queryRunnerResponse.json | JSON Schema entity definition: `QueryRunnerResponse` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/automations/testServiceConnection.json | JSON Schema entity definition: `TestServiceConnectionRequest` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/automations/testSparkEngineConnection.json | JSON Schema entity definition: `TestSparkEngineConnectionRequest` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/automations/workflow.json | JSON Schema entity definition: `Workflow` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/bot.json | JSON Schema entity definition: `Bot` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/chat/content/chatContentType.json | JSON Schema entity definition: `ChatContentType` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/chat/content/messageBlock.json | JSON Schema entity definition: `MessageBlock` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/chat/mcpConversation.json | JSON Schema entity definition: `McpConversation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/chat/mcpMessage.json | JSON Schema entity definition: `McpMessage` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/classification/classification.json | JSON Schema entity definition: `Classification` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/classification/tag.json | JSON Schema entity definition: `Tag` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/column/dashboardDataModelColumn.json | JSON Schema entity definition: `Dashboard Data Model Column Type` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/column/tableColumn.json | JSON Schema entity definition: `Table Column Type` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/context/contextMemory.json | JSON Schema entity definition: `ContextMemory` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/apiCollection.json | JSON Schema entity definition: `APICollection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/apiEndpoint.json | JSON Schema entity definition: `APIEndpoint` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/article.json | JSON Schema entity definition: `Article` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/chart.json | JSON Schema entity definition: `Chart` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/container.json | JSON Schema entity definition: `Container` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/contextFile.json | JSON Schema entity definition: `ContextFile` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/contextFileContent.json | JSON Schema entity definition: `ContextFileContent` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/dashboard.json | JSON Schema entity definition: `Dashboard` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/dashboardDataModel.json | JSON Schema entity definition: `DashboardDataModel` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/dataContract.json | JSON Schema entity definition: `DataContract` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/database.json | JSON Schema entity definition: `Database` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/databaseSchema.json | JSON Schema entity definition: `Database Schema` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/directory.json | JSON Schema entity definition: `Directory` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/file.json | JSON Schema entity definition: `File` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/folder.json | JSON Schema entity definition: `Folder` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/glossary.json | JSON Schema entity definition: `Glossary` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/glossaryTerm.json | JSON Schema entity definition: `GlossaryTerm` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/metric.json | JSON Schema entity definition: `Metric` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/mlmodel.json | JSON Schema entity definition: `MlModel` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/page.json | JSON Schema entity definition: `Page` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/pageHierarchy.json | JSON Schema entity definition: `Page Hierarchy` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/pipeline.json | JSON Schema entity definition: `Pipeline` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/query.json | JSON Schema entity definition: `Query` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/queryCostRecord.json | JSON Schema entity definition: `QueryCostRecord` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/queryCostSearchResult.json | JSON Schema entity definition: `QueryCostSearchResult` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/quickLink.json | JSON Schema entity definition: `QuickLink` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/report.json | JSON Schema entity definition: `Report` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/searchIndex.json | JSON Schema entity definition: `SearchIndex` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/spreadsheet.json | JSON Schema entity definition: `Spreadsheet` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/storedProcedure.json | JSON Schema entity definition: `StoredProcedure` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/table.json | JSON Schema entity definition: `Table` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/topic.json | JSON Schema entity definition: `Topic` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/data/worksheet.json | JSON Schema entity definition: `Worksheet` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/contractValidation.json | JSON Schema entity definition: `ContractValidation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/dataContractResult.json | JSON Schema entity definition: `DataContractResult` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/odcs/odcsDataContract.json | JSON Schema entity definition: `ODCSDataContract` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/qualityValidation.json | JSON Schema entity definition: `QualityValidation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/schemaValidation.json | JSON Schema entity definition: `SchemaValidation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/semanticsValidation.json | JSON Schema entity definition: `SemanticsValidation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/datacontract/slaValidation.json | JSON Schema entity definition: `SlaValidation` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/docStore/document.json | JSON Schema entity definition: `Document` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/domains/dataProduct.json | JSON Schema entity definition: `DataProduct` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/domains/domain.json | JSON Schema entity definition: `Domain` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/domains/odps/odpsDataProduct.json | JSON Schema entity definition: `ODPSDataProduct` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/events/authentication/webhookBearerAuth.json | JSON Schema entity definition: `BearerAuth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/events/authentication/webhookNoAuth.json | JSON Schema entity definition: `WebhookNoAuth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/events/authentication/webhookOAuth2Config.json | JSON Schema entity definition: `WebhookOAuth2Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/events/notificationTemplate.json | JSON Schema entity definition: `NotificationTemplate` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/events/webhook.json | JSON Schema entity definition: `Webhook` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/announcement.json | JSON Schema entity definition: `Announcement` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/assets.json | JSON Schema entity definition: `AssetsFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/customProperty.json | JSON Schema entity definition: `CustomPropertyFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/description.json | JSON Schema entity definition: `DescriptionFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/domain.json | JSON Schema entity definition: `DomainFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/entityInfo.json | JSON Schema entity definition: `EntityInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/owner.json | JSON Schema entity definition: `OwnerFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/suggestion.json | JSON Schema entity definition: `Suggestion` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/tag.json | JSON Schema entity definition: `TagFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/taskFormSchema.json | JSON Schema entity definition: `TaskFormSchema` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/testCaseResult.json | JSON Schema entity definition: `TestCaseResultFeedInfo` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/feed/thread.json | JSON Schema entity definition: `Thread` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/learning/learningResource.json | JSON Schema entity definition: `LearningResource` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/policies/accessControl/resourceDescriptor.json | JSON Schema entity definition: `ResourceDescriptor` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/policies/accessControl/resourcePermission.json | JSON Schema entity definition: `ResourcePermission` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/policies/accessControl/rule.json | JSON Schema entity definition: `Rule` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/policies/filters.json | JSON Schema entity definition: `Filters` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/policies/policy.json | JSON Schema entity definition: `Policy` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/apiService.json | JSON Schema entity definition: `Api Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/api/openAPISchemaFilePath.json | JSON Schema entity definition: `OpenAPISchemaFilePath` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/api/openAPISchemaS3.json | JSON Schema entity definition: `OpenAPISchemaS3` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/api/openAPISchemaURL.json | JSON Schema entity definition: `OpenAPISchemaURL` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/api/restConnection.json | JSON Schema entity definition: `RestConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/common/sslCertPaths.json | JSON Schema entity definition: `SSL Certificates By Path` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/common/sslCertValues.json | JSON Schema entity definition: `SSL Certificates By Values` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/common/sslConfig.json | JSON Schema entity definition: `SSL Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/connectionBasicType.json | JSON Schema entity definition: `ConnectionType` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/customDashboardConnection.json | JSON Schema entity definition: `CustomDashboardConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/domoDashboardConnection.json | JSON Schema entity definition: `DomoDashboardConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/grafanaConnection.json | JSON Schema entity definition: `GrafanaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/hexConnection.json | JSON Schema entity definition: `HexConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/lightdashConnection.json | JSON Schema entity definition: `LightdashConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/lookerConnection.json | JSON Schema entity definition: `LookerConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/metabaseConnection.json | JSON Schema entity definition: `MetabaseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/microStrategyConnection.json | JSON Schema entity definition: `MicroStrategyConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/modeConnection.json | JSON Schema entity definition: `ModeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerBIConnection.json | JSON Schema entity definition: `PowerBIConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerBIReportServerConnection.json | JSON Schema entity definition: `PowerBIReportServerConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerbi/azureConfig.json | JSON Schema entity definition: `AzureConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerbi/bucketDetails.json | JSON Schema entity definition: `Bucket Details` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerbi/gcsConfig.json | JSON Schema entity definition: `GCSConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/powerbi/s3Config.json | JSON Schema entity definition: `S3Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/qlikCloudConnection.json | JSON Schema entity definition: `QlikCloudConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/qlikSenseConnection.json | JSON Schema entity definition: `QlikSenseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/quickSightConnection.json | JSON Schema entity definition: `QuickSightConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/redashConnection.json | JSON Schema entity definition: `RedashConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/sapS4HanaConnection.json | JSON Schema entity definition: `SapS4HanaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/sigmaConnection.json | JSON Schema entity definition: `SigmaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/ssrsConnection.json | JSON Schema entity definition: `SsrsConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/supersetConnection.json | JSON Schema entity definition: `SupersetConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/tableauConnection.json | JSON Schema entity definition: `TableauConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/dashboard/thoughtSpotConnection.json | JSON Schema entity definition: `ThoughtSpotConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/athenaConnection.json | JSON Schema entity definition: `AthenaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/azureSQLConnection.json | JSON Schema entity definition: `AzureSQLConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/bigQueryConnection.json | JSON Schema entity definition: `BigQueryConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/bigTableConnection.json | JSON Schema entity definition: `BigTableConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/burstIQConnection.json | JSON Schema entity definition: `BurstIQConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/cassandra/cloudConfig.json | JSON Schema entity definition: `Cloud Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/cassandraConnection.json | JSON Schema entity definition: `CassandraConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/clickhouseConnection.json | JSON Schema entity definition: `ClickhouseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/cockroachConnection.json | JSON Schema entity definition: `CockroachConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/azureConfig.json | JSON Schema entity definition: `Azure Configuration Source` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/basicAuth.json | JSON Schema entity definition: `Basic Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/gcpCloudSqlConfig.json | JSON Schema entity definition: `GCP CloudSQL Configuration Source` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/iamAuthConfig.json | JSON Schema entity definition: `IAM Auth Configuration Source` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/jwtAuth.json | JSON Schema entity definition: `JWT Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/common/noConfigAuthenticationTypes.json | JSON Schema entity definition: `No Config Authentication Types` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/couchbaseConnection.json | JSON Schema entity definition: `Couchbase Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/customDatabaseConnection.json | JSON Schema entity definition: `CustomDatabaseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/databricks/azureAdSetup.json | JSON Schema entity definition: `Azure AD Setup` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/databricks/databricksOAuth.json | JSON Schema entity definition: `Databricks OAuth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/databricks/personalAccessToken.json | JSON Schema entity definition: `Personal Access Token` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/databricksConnection.json | JSON Schema entity definition: `DatabricksConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/datalake/azureConfig.json | JSON Schema entity definition: `AzureConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/datalake/gcsConfig.json | JSON Schema entity definition: `GCSConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/datalake/s3Config.json | JSON Schema entity definition: `S3Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/datalakeConnection.json | JSON Schema entity definition: `DatalakeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/db2Connection.json | JSON Schema entity definition: `Db2Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/deltaLakeConnection.json | JSON Schema entity definition: `DeltaLakeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/deltalake/metastoreConfig.json | JSON Schema entity definition: `MetastoreConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/deltalake/storageConfig.json | JSON Schema entity definition: `StorageConfig` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/domoDatabaseConnection.json | JSON Schema entity definition: `DomoDatabaseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/dorisConnection.json | JSON Schema entity definition: `DorisConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/dremio/cloudAuth.json | JSON Schema entity definition: `Dremio Cloud Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/dremio/softwareAuth.json | JSON Schema entity definition: `Dremio Software Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/dremioConnection.json | JSON Schema entity definition: `DremioConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/druidConnection.json | JSON Schema entity definition: `DruidConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/dynamoDBConnection.json | JSON Schema entity definition: `DynamoDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/epicConnection.json | JSON Schema entity definition: `EpicConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/exasolConnection.json | JSON Schema entity definition: `ExasolConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/glueConnection.json | JSON Schema entity definition: `GlueConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/greenplumConnection.json | JSON Schema entity definition: `GreenplumConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/hiveConnection.json | JSON Schema entity definition: `HiveConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/impalaConnection.json | JSON Schema entity definition: `ImpalaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/informixConnection.json | JSON Schema entity definition: `InformixConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/iometeConnection.json | JSON Schema entity definition: `IometeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/mariaDBConnection.json | JSON Schema entity definition: `MariaDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/microsoftAccessConnection.json | JSON Schema entity definition: `MicrosoftAccessConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/microsoftFabricConnection.json | JSON Schema entity definition: `MicrosoftFabricConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/mongoDBConnection.json | JSON Schema entity definition: `MongoDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/mssqlConnection.json | JSON Schema entity definition: `MssqlConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/myDbConnection.json | JSON Schema entity definition: `MyDbConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/mysqlConnection.json | JSON Schema entity definition: `MysqlConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/oracleConnection.json | JSON Schema entity definition: `OracleConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/pinotDBConnection.json | JSON Schema entity definition: `PinotDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/postgresConnection.json | JSON Schema entity definition: `PostgresConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/prestoConnection.json | JSON Schema entity definition: `PrestoConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/questdbConnection.json | JSON Schema entity definition: `QuestDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/redshiftConnection.json | JSON Schema entity definition: `RedshiftConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/salesforceConnection.json | JSON Schema entity definition: `SalesforceConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sapErpConnection.json | JSON Schema entity definition: `SapErpConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sapHana/sapHanaHDBConnection.json | JSON Schema entity definition: `SapHanaHDBConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sapHana/sapHanaSQLConnection.json | JSON Schema entity definition: `SapHanaSQLConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sapHanaConnection.json | JSON Schema entity definition: `SapHanaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sapSuccessFactorsConnection.json | JSON Schema entity definition: `SapSuccessFactorsConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sasConnection.json | JSON Schema entity definition: `SASConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/serviceNowConnection.json | JSON Schema entity definition: `ServiceNowConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/singleStoreConnection.json | JSON Schema entity definition: `SingleStoreConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/snowflakeConnection.json | JSON Schema entity definition: `SnowflakeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/sqliteConnection.json | JSON Schema entity definition: `SQLiteConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/ssasConnection.json | JSON Schema entity definition: `SSASConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/starrocksConnection.json | JSON Schema entity definition: `StarRocksConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/synapseConnection.json | JSON Schema entity definition: `SynapseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/teradataConnection.json | JSON Schema entity definition: `TeradataConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/timescaleConnection.json | JSON Schema entity definition: `TimescaleConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/trinoConnection.json | JSON Schema entity definition: `TrinoConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/unityCatalogConnection.json | JSON Schema entity definition: `UnityCatalogConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/database/verticaConnection.json | JSON Schema entity definition: `VerticaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/drive/customDriveConnection.json | JSON Schema entity definition: `CustomDriveConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/drive/googleDriveConnection.json | JSON Schema entity definition: `GoogleDriveConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/drive/sftpConnection.json | JSON Schema entity definition: `SftpConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/drive/sharePointConnection.json | JSON Schema entity definition: `SharePointConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/anthropicConnection.json | JSON Schema entity definition: `AnthropicConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/azureOpenAIConnection.json | JSON Schema entity definition: `AzureOpenAIConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/bedrockConnection.json | JSON Schema entity definition: `BedrockConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/customLLMConnection.json | JSON Schema entity definition: `CustomLLMConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/huggingFaceConnection.json | JSON Schema entity definition: `HuggingFaceConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/ollamaConnection.json | JSON Schema entity definition: `OllamaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/openAIConnection.json | JSON Schema entity definition: `OpenAIConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/llm/vertexAIConnection.json | JSON Schema entity definition: `VertexAIConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mcp/mcpConnection.json | JSON Schema entity definition: `McpConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/customMessagingConnection.json | JSON Schema entity definition: `CustomMessagingConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/kafkaConnection.json | JSON Schema entity definition: `KafkaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/kinesisConnection.json | JSON Schema entity definition: `KinesisConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/pubSubConnection.json | JSON Schema entity definition: `PubSubConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/pulsarConnection.json | JSON Schema entity definition: `PulsarConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/redpandaConnection.json | JSON Schema entity definition: `RedpandaConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/messaging/saslMechanismType.json | JSON Schema entity definition: `SaslMechanismType` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/alationConnection.json | JSON Schema entity definition: `AlationConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/alationSinkConnection.json | JSON Schema entity definition: `AlationSinkConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/amundsenConnection.json | JSON Schema entity definition: `AmundsenConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/atlasConnection.json | JSON Schema entity definition: `AtlasConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/collibraConnection.json | JSON Schema entity definition: `CollibraConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/metadataESConnection.json | JSON Schema entity definition: `MetadataESConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/metadata/openMetadataConnection.json | JSON Schema entity definition: `OpenMetadataConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mlmodel/customMlModelConnection.json | JSON Schema entity definition: `CustomMlModelConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mlmodel/mlflowConnection.json | JSON Schema entity definition: `MlflowConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mlmodel/sageMakerConnection.json | JSON Schema entity definition: `SageMakerConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mlmodel/sklearnConnection.json | JSON Schema entity definition: `SklearnConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/mlmodel/vertexaiConnection.json | JSON Schema entity definition: `VertexAIConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/airbyte/basicAuth.json | JSON Schema entity definition: `Basic Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/airbyte/oauthClientAuth.json | JSON Schema entity definition: `OAuth 2.0 Client Credentials Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/airbyteConnection.json | JSON Schema entity definition: `AirbyteConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/airflowConnection.json | JSON Schema entity definition: `AirflowConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/backendConnection.json | JSON Schema entity definition: `BackendConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/customPipelineConnection.json | JSON Schema entity definition: `CustomPipelineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/dagsterConnection.json | JSON Schema entity definition: `DagsterConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/databricksPipelineConnection.json | JSON Schema entity definition: `DatabricksPipelineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/datafactoryConnection.json | JSON Schema entity definition: `DataFactoryConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/dbtCloudConnection.json | JSON Schema entity definition: `DBTCloudConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/domoPipelineConnection.json | JSON Schema entity definition: `DomoPipelineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/fivetranConnection.json | JSON Schema entity definition: `FivetranConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/flinkConnection.json | JSON Schema entity definition: `FlinkConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/gluePipelineConnection.json | JSON Schema entity definition: `GluePipelineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/kafkaConnectConnection.json | JSON Schema entity definition: `KafkaConnectConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/kinesisFirehoseConnection.json | JSON Schema entity definition: `KinesisFirehoseConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/matillion/matillionDPC.json | JSON Schema entity definition: `Matillion DPC Auth Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/matillion/matillionETL.json | JSON Schema entity definition: `Matillion ETL Auth Config` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/matillionConnection.json | JSON Schema entity definition: `MatillionConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/microsoftFabricPipelineConnection.json | JSON Schema entity definition: `MicrosoftFabricPipelineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/mulesoftConnection.json | JSON Schema entity definition: `MulesoftConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/nifi/basicAuth.json | JSON Schema entity definition: `Nifi Basic Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/nifi/clientCertificateAuth.json | JSON Schema entity definition: `Nifi Client Certificate Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/nifiConnection.json | JSON Schema entity definition: `NifiConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/openLineageConnection.json | JSON Schema entity definition: `OpenLineageConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/snowplowConnection.json | JSON Schema entity definition: `SnowplowConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/sparkConnection.json | JSON Schema entity definition: `SparkConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/splineConnection.json | JSON Schema entity definition: `SplineConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/ssisConnection.json | JSON Schema entity definition: `SSISConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/stitchConnection.json | JSON Schema entity definition: `StitchConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/pipeline/wherescapeConnection.json | JSON Schema entity definition: `WherescapeConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/search/customSearchConnection.json | JSON Schema entity definition: `CustomSearchConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/search/elasticSearch/apiAuth.json | JSON Schema entity definition: `API Key Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/search/elasticSearch/basicAuth.json | JSON Schema entity definition: `Basic Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/search/elasticSearchConnection.json | JSON Schema entity definition: `ElasticSearch Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/search/openSearchConnection.json | JSON Schema entity definition: `OpenSearchConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/security/ranger/basicAuth.json | JSON Schema entity definition: `Ranger Basic Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/security/rangerConnection.json | JSON Schema entity definition: `RangerConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/serviceConnection.json | JSON Schema entity definition: `Service Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/storage/adlsConnection.json | JSON Schema entity definition: `ADLS Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/storage/customStorageConnection.json | JSON Schema entity definition: `CustomStorageConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/storage/gcsConnection.json | JSON Schema entity definition: `GCS Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/storage/s3Connection.json | JSON Schema entity definition: `S3 Connection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/testConnectionDefinition.json | JSON Schema entity definition: `TestConnectionDefinition` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/connections/testConnectionResult.json | JSON Schema entity definition: `TestConnectionResult` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/dashboardService.json | JSON Schema entity definition: `Dashboard Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/databaseService.json | JSON Schema entity definition: `Database Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/driveService.json | JSON Schema entity definition: `Drive Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/ingestionPipeline.json | JSON Schema entity definition: `IngestionPipeline` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/operationMetrics.json | JSON Schema entity definition: `OperationMetrics` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/operationMetricsBatch.json | JSON Schema entity definition: `OperationMetricsBatch` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/pipelineServiceClientResponse.json | JSON Schema entity definition: `PipelineServiceClientResponse` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/progressUpdate.json | JSON Schema entity definition: `ProgressUpdate` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/reverseIngestionResponse.json | JSON Schema entity definition: `ReverseIngestionResponse` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/ingestionPipelines/status.json | JSON Schema entity definition: `IngestionStatusModel` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/llmService.json | JSON Schema entity definition: `LLMService` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/mcpService.json | JSON Schema entity definition: `McpService` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/messagingService.json | JSON Schema entity definition: `Messaging Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/metadataService.json | JSON Schema entity definition: `Metadata Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/mlmodelService.json | JSON Schema entity definition: `MlModelService` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/pipelineService.json | JSON Schema entity definition: `Pipeline Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/searchService.json | JSON Schema entity definition: `Search Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/securityService.json | JSON Schema entity definition: `Security Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/serviceType.json | JSON Schema entity definition: `Service Type` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/services/storageService.json | JSON Schema entity definition: `Storage Service` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/tasks/task.json | JSON Schema entity definition: `Task` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/teams/persona.json | JSON Schema entity definition: `Persona` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/teams/role.json | JSON Schema entity definition: `Role` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/teams/team.json | JSON Schema entity definition: `Team` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/teams/teamHierarchy.json | JSON Schema entity definition: `Team Hierarchy` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/teams/user.json | JSON Schema entity definition: `User` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/type.json | JSON Schema entity definition: `Type` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/airflowRestApiConnection.json | JSON Schema entity definition: `AirflowRestApiConnection` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/common/accessTokenConfig.json | JSON Schema entity definition: `Access Token` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/common/basicAuthConfig.json | JSON Schema entity definition: `Basic Auth` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/common/gcpCredentialsConfig.json | JSON Schema entity definition: `GCP Service Account` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/common/mwaaAuthConfig.json | JSON Schema entity definition: `MWAA Authentication` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/entitiesCount.json | JSON Schema entity definition: `Entities Count` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/servicesCount.json | JSON Schema entity definition: `Services Count` → generates Java POJO via spec build | High |
| openmetadata-spec/src/main/resources/json/schema/entity/utils/supersetApiConnection.json | JSON Schema entity definition: `SupersetApiConnection` → generates Java POJO via spec build | High |

## DTOs (API JSON Schemas)
Count: 169
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-spec/src/main/resources/json/schema/api/addGlossaryToAssetsRequest.json | JSON Schema API request/response: `AddGlossaryToAssetsRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/addTagToAssetsRequest.json | JSON Schema API request/response: `AddTagToAssetsRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createAIApplication.json | JSON Schema API request/response: `CreateAIApplicationRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createAIGovernancePolicy.json | JSON Schema API request/response: `CreateAIGovernancePolicyRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createAgentExecution.json | JSON Schema API request/response: `CreateAgentExecutionRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createLLMModel.json | JSON Schema API request/response: `CreateLLMModelRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createMcpServer.json | JSON Schema API request/response: `CreateMcpServerRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/ai/createPromptTemplate.json | JSON Schema API request/response: `CreatePromptTemplateRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/analytics/createWebAnalyticEvent.json | JSON Schema API request/response: `CreateWebAnalyticEvent` | High |
| openmetadata-spec/src/main/resources/json/schema/api/attachments/createAsset.json | JSON Schema API request/response: `CreateAssetRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/automations/createWorkflow.json | JSON Schema API request/response: `CreateWorkflowRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/bulkAssets.json | JSON Schema API request/response: `BulkAssets` | High |
| openmetadata-spec/src/main/resources/json/schema/api/bulkEntityPatch.json | JSON Schema API request/response: `BulkEntityPatch` | High |
| openmetadata-spec/src/main/resources/json/schema/api/chat/createMcpConversation.json | JSON Schema API request/response: `CreateMcpConversation` | High |
| openmetadata-spec/src/main/resources/json/schema/api/chat/createMcpMessage.json | JSON Schema API request/response: `CreateMcpMessage` | High |
| openmetadata-spec/src/main/resources/json/schema/api/classification/createClassification.json | JSON Schema API request/response: `CreateClassificationRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/classification/createTag.json | JSON Schema API request/response: `CreateTagRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/classification/createTagWithRecognizers.json | JSON Schema API request/response: `CreateTagWithRecognizersRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/classification/loadTags.json | JSON Schema API request/response: `loadTags` | High |
| openmetadata-spec/src/main/resources/json/schema/api/configuration/rdfConfiguration.json | JSON Schema API request/response: `RdfConfiguration` | High |
| openmetadata-spec/src/main/resources/json/schema/api/context/createContextMemory.json | JSON Schema API request/response: `CreateContextMemory` | High |
| openmetadata-spec/src/main/resources/json/schema/api/createBot.json | JSON Schema API request/response: `createBot` | High |
| openmetadata-spec/src/main/resources/json/schema/api/createEventPublisherJob.json | JSON Schema API request/response: `CreateEventPublisherJob` | High |
| openmetadata-spec/src/main/resources/json/schema/api/createType.json | JSON Schema API request/response: `createType` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkColumnUpdatePreview.json | JSON Schema API request/response: `BulkColumnUpdatePreview` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkColumnUpdateRequest.json | JSON Schema API request/response: `BulkColumnUpdateRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkCreateTable.json | JSON Schema API request/response: `BulkCreateTable` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkUpdateContainer.json | JSON Schema API request/response: `BulkUpdateContainer` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkUpdateDashboard.json | JSON Schema API request/response: `BulkUpdateDashboard` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkUpdateTable.json | JSON Schema API request/response: `BulkUpdateTable` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/bulkUpdateTopic.json | JSON Schema API request/response: `BulkUpdateTopic` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/columnGridResponse.json | JSON Schema API request/response: `ColumnGridResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/columnGroupUpdateRequest.json | JSON Schema API request/response: `ColumnGroupUpdateRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createAPICollection.json | JSON Schema API request/response: `CreateAPICollectionRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createAPIEndpoint.json | JSON Schema API request/response: `CreateAPIEndpointRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createChart.json | JSON Schema API request/response: `CreateChartRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createContainer.json | JSON Schema API request/response: `CreateContainerRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createContextFile.json | JSON Schema API request/response: `CreateContextFile` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createCustomProperty.json | JSON Schema API request/response: `CreateCustomPropertyRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDashboard.json | JSON Schema API request/response: `CreateDashboardRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDashboardDataModel.json | JSON Schema API request/response: `CreateDashboardDataModelRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDataContract.json | JSON Schema API request/response: `CreateDataContractRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDatabase.json | JSON Schema API request/response: `CreateDatabaseRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDatabaseSchema.json | JSON Schema API request/response: `CreateDatabaseSchemaRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createDirectory.json | JSON Schema API request/response: `CreateDirectoryRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createEntityProfile.json | JSON Schema API request/response: `CreateEntityProfileRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createFile.json | JSON Schema API request/response: `CreateFileRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createFolder.json | JSON Schema API request/response: `CreateFolder` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createGlossary.json | JSON Schema API request/response: `CreateGlossaryRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createGlossaryTerm.json | JSON Schema API request/response: `CreateGlossaryTermRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createMetric.json | JSON Schema API request/response: `CreateMetricRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createMlModel.json | JSON Schema API request/response: `CreateMlModelRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createPage.json | JSON Schema API request/response: `CreatePage` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createPipeline.json | JSON Schema API request/response: `CreatePipelineRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createQuery.json | JSON Schema API request/response: `CreateQueryRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createQueryCostRecord.json | JSON Schema API request/response: `CreateQueryCostRecordRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createSearchIndex.json | JSON Schema API request/response: `CreateSearchIndexRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createSpreadsheet.json | JSON Schema API request/response: `CreateSpreadsheetRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createStoredProcedure.json | JSON Schema API request/response: `CreateStoredProcedureRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createTable.json | JSON Schema API request/response: `CreateTableRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createTableProfile.json | JSON Schema API request/response: `CreateTableProfileRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createTopic.json | JSON Schema API request/response: `CreateTopicRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/createWorksheet.json | JSON Schema API request/response: `CreateWorksheetRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/groupedColumnsResponse.json | JSON Schema API request/response: `GroupedColumnsResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/loadGlossary.json | JSON Schema API request/response: `loadGlossary` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/moveContextFileRequest.json | JSON Schema API request/response: `MoveContextFileRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/restoreEntity.json | JSON Schema API request/response: `restoreEntity` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/searchColumnsRequest.json | JSON Schema API request/response: `SearchColumnsRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/data/updateColumn.json | JSON Schema API request/response: `UpdateColumn` | High |
| openmetadata-spec/src/main/resources/json/schema/api/dataInsight/createDataInsightChart.json | JSON Schema API request/response: `CreateDataInsightChart` | High |
| openmetadata-spec/src/main/resources/json/schema/api/dataInsight/custom/createDataInsightCustomChart.json | JSON Schema API request/response: `CreateDataInsightCustomChart` | High |
| openmetadata-spec/src/main/resources/json/schema/api/dataInsight/kpi/createKpiRequest.json | JSON Schema API request/response: `CreateKpiRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/docStore/createDocument.json | JSON Schema API request/response: `CreateDocumentRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/domains/createDataProduct.json | JSON Schema API request/response: `CreateDataProductRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/domains/createDomain.json | JSON Schema API request/response: `CreateDomainRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/domains/dataProductPortsView.json | JSON Schema API request/response: `DataProductPortsView` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/entityRelationshipDirection.json | JSON Schema API request/response: `EntityRelationshipDirection` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/esEntityRelationshipData.json | JSON Schema API request/response: `EsEntityRelationshipData` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/relationshipRef.json | JSON Schema API request/response: `RelationshipRef` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/searchEntityRelationshipRequest.json | JSON Schema API request/response: `SearchEntityRelationshipRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/searchEntityRelationshipResult.json | JSON Schema API request/response: `SearchEntityRelationshipResult` | High |
| openmetadata-spec/src/main/resources/json/schema/api/entityRelationship/searchSchemaEntityRelationshipResult.json | JSON Schema API request/response: `SearchSchemaEntityRelationshipResult` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/createNotificationTemplate.json | JSON Schema API request/response: `CreateNotificationTemplate` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/notificationTemplateRenderRequest.json | JSON Schema API request/response: `NotificationTemplateRenderRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/notificationTemplateRenderResponse.json | JSON Schema API request/response: `NotificationTemplateRenderResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/notificationTemplateSendRequest.json | JSON Schema API request/response: `NotificationTemplateSendRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/notificationTemplateValidationRequest.json | JSON Schema API request/response: `NotificationTemplateValidationRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/events/notificationTemplateValidationResponse.json | JSON Schema API request/response: `NotificationTemplateValidationResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/closeTask.json | JSON Schema API request/response: `CloseTaskRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/createAnnouncement.json | JSON Schema API request/response: `CreateAnnouncementRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/createPost.json | JSON Schema API request/response: `CreatePostRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/createSuggestion.json | JSON Schema API request/response: `CreateSuggestionRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/createThread.json | JSON Schema API request/response: `CreateThreadRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/resolveTask.json | JSON Schema API request/response: `ResolveTaskRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/feed/threadCount.json | JSON Schema API request/response: `Count of threads related to an entity` | High |
| openmetadata-spec/src/main/resources/json/schema/api/governance/createIntakeForm.json | JSON Schema API request/response: `CreateIntakeFormRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/governance/createWorkflowDefinition.json | JSON Schema API request/response: `CreateWorkflowDefinitionRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/governance/createWorkflowInstanceState.json | JSON Schema API request/response: `CreateWorkflowInstanceStateRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/handlebarsHelpers.json | JSON Schema API request/response: `Handlebars Helpers List` | High |
| openmetadata-spec/src/main/resources/json/schema/api/learning/createLearningResource.json | JSON Schema API request/response: `CreateLearningResourceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/addLineage.json | JSON Schema API request/response: `AddLineageRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/entityCountLineageRequest.json | JSON Schema API request/response: `EntityCountLineageRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/esLineageData.json | JSON Schema API request/response: `EsLineageData` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/hydrateLineageRequest.json | JSON Schema API request/response: `HydrateLineageRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/hydrateLineageResponse.json | JSON Schema API request/response: `HydrateLineageResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/lineageDirection.json | JSON Schema API request/response: `LineageDirection` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/lineagePaginationInfo.json | JSON Schema API request/response: `LineagePaginationInfo` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/nodeInformation.json | JSON Schema API request/response: `NodeInformation` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/openlineage/openLineageBatchRequest.json | JSON Schema API request/response: `OpenLineageBatchRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/openlineage/openLineageDataset.json | JSON Schema API request/response: `OpenLineageDataset` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/openlineage/openLineageFacets.json | JSON Schema API request/response: `OpenLineageFacets` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/openlineage/openLineageResponse.json | JSON Schema API request/response: `OpenLineageResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/openlineage/openLineageRunEvent.json | JSON Schema API request/response: `OpenLineageRunEvent` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/searchLineageRequest.json | JSON Schema API request/response: `SearchLineageRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/lineage/searchLineageResult.json | JSON Schema API request/response: `SearchLineageResult` | High |
| openmetadata-spec/src/main/resources/json/schema/api/mcp/mcpSearchResponse.json | JSON Schema API request/response: `MCP Search Response` | High |
| openmetadata-spec/src/main/resources/json/schema/api/mcp/mcpToolDefinition.json | JSON Schema API request/response: `MCP Tool Definition` | High |
| openmetadata-spec/src/main/resources/json/schema/api/openMetadataServerVersion.json | JSON Schema API request/response: `OpenMetadataServerVersion` | High |
| openmetadata-spec/src/main/resources/json/schema/api/policies/createPolicy.json | JSON Schema API request/response: `CreatePolicyRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/rdf/sparqlQuery.json | JSON Schema API request/response: `SparqlQuery` | High |
| openmetadata-spec/src/main/resources/json/schema/api/rdf/sparqlResponse.json | JSON Schema API request/response: `SparqlResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/scim/scimGroup.json | JSON Schema API request/response: `ScimGroup` | High |
| openmetadata-spec/src/main/resources/json/schema/api/scim/scimPatchOp.json | JSON Schema API request/response: `ScimPatchOp` | High |
| openmetadata-spec/src/main/resources/json/schema/api/scim/scimUser.json | JSON Schema API request/response: `ScimUser` | High |
| openmetadata-spec/src/main/resources/json/schema/api/search/orphanCleanupResponse.json | JSON Schema API request/response: `OrphanCleanupResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/search/previewSearchRequest.json | JSON Schema API request/response: `PreviewSearchRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/search/searchStats.json | JSON Schema API request/response: `SearchStatsResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createApiService.json | JSON Schema API request/response: `CreateApiServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createDashboardService.json | JSON Schema API request/response: `CreateDashboardServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createDatabaseService.json | JSON Schema API request/response: `CreateDatabaseServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createDriveService.json | JSON Schema API request/response: `CreateDriveServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createLLMService.json | JSON Schema API request/response: `CreateLLMServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createMcpService.json | JSON Schema API request/response: `CreateMcpServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createMessagingService.json | JSON Schema API request/response: `CreateMessagingServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createMetadataService.json | JSON Schema API request/response: `CreateMetadataServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createMlModelService.json | JSON Schema API request/response: `CreateMlModelServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createPipelineService.json | JSON Schema API request/response: `CreatePipelineServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createSearchService.json | JSON Schema API request/response: `CreateSearchServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createSecurityService.json | JSON Schema API request/response: `CreateSecurityServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/createStorageService.json | JSON Schema API request/response: `CreateStorageServiceRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/services/ingestionPipelines/createIngestionPipeline.json | JSON Schema API request/response: `CreateIngestionPipelineRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/setOwner.json | JSON Schema API request/response: `SetOwnershipRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tasks/bulkTaskOperation.json | JSON Schema API request/response: `BulkTaskOperationRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tasks/createTask.json | JSON Schema API request/response: `CreateTaskRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tasks/createTaskComment.json | JSON Schema API request/response: `CreateTaskCommentRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tasks/resolveTask.json | JSON Schema API request/response: `ResolveTaskRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tasks/taskCount.json | JSON Schema API request/response: `Count of tasks` | High |
| openmetadata-spec/src/main/resources/json/schema/api/teams/createPersona.json | JSON Schema API request/response: `CreatePersonaRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/teams/createRole.json | JSON Schema API request/response: `CreateRoleRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/teams/createTeam.json | JSON Schema API request/response: `CreateTeamRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/teams/createUser.json | JSON Schema API request/response: `CreateUserRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/bundleSuiteBulkAddRequest.json | JSON Schema API request/response: `BundleSuiteBulkAddRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/bundleSuiteBulkAddRequestBulkAll.json | JSON Schema API request/response: `BundleSuiteBulkAddRequestBulkAll` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/bundleSuiteBulkAddRequestBulkByIds.json | JSON Schema API request/response: `BundleSuiteBulkAddRequestBulkByIds` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createCustomMetric.json | JSON Schema API request/response: `CreateCustomMetricRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createLogicalTestCases.json | JSON Schema API request/response: `CreateLogicalTestCases` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createTestCase.json | JSON Schema API request/response: `CreateTestCaseRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createTestCaseResolutionStatus.json | JSON Schema API request/response: `CreateTestCaseResolutionStatus` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createTestCaseResult.json | JSON Schema API request/response: `CreateTestCaseResult` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createTestDefinition.json | JSON Schema API request/response: `CreateTestDefinitionRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/createTestSuite.json | JSON Schema API request/response: `CreateTestSuiteRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/tests/moveGlossaryTermRequest.json | JSON Schema API request/response: `MoveGlossaryTermRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/validateGlossaryTagsRequest.json | JSON Schema API request/response: `ValidateGlossaryTagsRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/api/voteRequest.json | JSON Schema API request/response: `Query Vote` | High |
| openmetadata-spec/src/main/resources/json/schema/events/api/createEventSubscription.json | JSON Schema API request/response: `CreateEventSubscription` | High |
| openmetadata-spec/src/main/resources/json/schema/events/api/eventSubscriptionDiagnosticInfo.json | JSON Schema API request/response: `Event Subscription Diagnostic Info` | High |
| openmetadata-spec/src/main/resources/json/schema/events/api/eventsRecord.json | JSON Schema API request/response: `Event Subscription Events Record` | High |
| openmetadata-spec/src/main/resources/json/schema/events/api/testEventSubscriptionDestination.json | JSON Schema API request/response: `Test EventSubscription Request` | High |
| openmetadata-spec/src/main/resources/json/schema/events/api/typedEvent.json | JSON Schema API request/response: `Typed Event` | High |

## Entities / Types (Type JSON Schemas)
Count: 102
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/aggregatedUnusedAssetsCount.json | JSON Schema type definition: `AggregatedUnusedAssetsCount` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/aggregatedUnusedAssetsSize.json | JSON Schema type definition: `AggregatedUnusedAssetsSize` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/aggregatedUsedVsUnusedAssetsCount.json | JSON Schema type definition: `AggregatedUsedVsUnusedAssetsCount` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/aggregatedUsedVsUnusedAssetsSize.json | JSON Schema type definition: `AggregatedUsedVsUnusedAssetsSize` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/dailyActiveUsers.json | JSON Schema type definition: `DailyActiveUsers` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/mostActiveUsers.json | JSON Schema type definition: `MostActiveUsers` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/mostViewedEntities.json | JSON Schema type definition: `MostViewedEntities` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/pageViewsByEntities.json | JSON Schema type definition: `PageViewsByEntities` | High |
| openmetadata-spec/src/main/resources/json/schema/dataInsight/type/unusedAssets.json | JSON Schema type definition: `UnusedAssets` | High |
| openmetadata-spec/src/main/resources/json/schema/type/aiCompliance.json | JSON Schema type definition: `AICompliance` | High |
| openmetadata-spec/src/main/resources/json/schema/type/apiSchema.json | JSON Schema type definition: `APISchema` | High |
| openmetadata-spec/src/main/resources/json/schema/type/assetCertification.json | JSON Schema type definition: `AssetCertification` | High |
| openmetadata-spec/src/main/resources/json/schema/type/auditLog.json | JSON Schema type definition: `AuditLog` | High |
| openmetadata-spec/src/main/resources/json/schema/type/basic.json | JSON Schema type definition: `Basic` | High |
| openmetadata-spec/src/main/resources/json/schema/type/bulkDeleteStaleRequest.json | JSON Schema type definition: `BulkDeleteStaleRequest` | High |
| openmetadata-spec/src/main/resources/json/schema/type/bulkOperationResult.json | JSON Schema type definition: `BulkOperationResult` | High |
| openmetadata-spec/src/main/resources/json/schema/type/bulkTaskOperationResult.json | JSON Schema type definition: `BulkTaskOperationResult` | High |
| openmetadata-spec/src/main/resources/json/schema/type/changeEvent.json | JSON Schema type definition: `ChangeEvent` | High |
| openmetadata-spec/src/main/resources/json/schema/type/changeEventType.json | JSON Schema type definition: `EventType` | High |
| openmetadata-spec/src/main/resources/json/schema/type/changeSummaryMap.json | JSON Schema type definition: `Change Summary` | High |
| openmetadata-spec/src/main/resources/json/schema/type/classificationLanguages.json | JSON Schema type definition: `ClassificationLanguage` | High |
| openmetadata-spec/src/main/resources/json/schema/type/collectionDescriptor.json | JSON Schema type definition: `CollectionDescriptor` | High |
| openmetadata-spec/src/main/resources/json/schema/type/contextRecognizer.json | JSON Schema type definition: `ContextRecognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/contractExecutionStatus.json | JSON Schema type definition: `ContractExecutionStatus` | High |
| openmetadata-spec/src/main/resources/json/schema/type/csvDocumentation.json | JSON Schema type definition: `csvDocumentation` | High |
| openmetadata-spec/src/main/resources/json/schema/type/csvErrorType.json | JSON Schema type definition: `csvErrorType` | High |
| openmetadata-spec/src/main/resources/json/schema/type/csvFile.json | JSON Schema type definition: `csvFile` | High |
| openmetadata-spec/src/main/resources/json/schema/type/csvImportResult.json | JSON Schema type definition: `csvImportResult` | High |
| openmetadata-spec/src/main/resources/json/schema/type/customProperties/complexTypes.json | JSON Schema type definition: `https://open-metadata.org/schema/type/customProperties/complexTypes.json` | High |
| openmetadata-spec/src/main/resources/json/schema/type/customProperties/enumConfig.json | JSON Schema type definition: `EnumConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/customProperties/tableConfig.json | JSON Schema type definition: `TableConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/customProperty.json | JSON Schema type definition: `CustomProperty` | High |
| openmetadata-spec/src/main/resources/json/schema/type/customRecognizer.json | JSON Schema type definition: `CustomRecognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/dailyCount.json | JSON Schema type definition: `Daily count of some measurement` | High |
| openmetadata-spec/src/main/resources/json/schema/type/dataAccessRequestPayload.json | JSON Schema type definition: `DataAccessRequestPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/databaseConnectionConfig.json | JSON Schema type definition: `DatabaseConnectionConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/descriptionUpdatePayload.json | JSON Schema type definition: `DescriptionUpdatePayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/domainUpdatePayload.json | JSON Schema type definition: `DomainUpdatePayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/dynamicSamplingConfig.json | JSON Schema type definition: `DynamicSamplingConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityHierarchy.json | JSON Schema type definition: `EntityHierarchy` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityHistory.json | JSON Schema type definition: `Entity Version History` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityLineage.json | JSON Schema type definition: `Entity Lineage` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityProfile.json | JSON Schema type definition: `EntityProfile` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityReference.json | JSON Schema type definition: `Entity Reference` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityReferenceList.json | JSON Schema type definition: `Entity Reference List` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityRelationship.json | JSON Schema type definition: `EntityRelationship` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityRelationship/nodeInformation.json | JSON Schema type definition: `NodeInformation` | High |
| openmetadata-spec/src/main/resources/json/schema/type/entityUsage.json | JSON Schema type definition: `EntityUsage` | High |
| openmetadata-spec/src/main/resources/json/schema/type/exactTermsRecognizer.json | JSON Schema type definition: `ExactTermsRecognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/filterPattern.json | JSON Schema type definition: `FilterPatternModel` | High |
| openmetadata-spec/src/main/resources/json/schema/type/function.json | JSON Schema type definition: `function` | High |
| openmetadata-spec/src/main/resources/json/schema/type/genericTaskPayload.json | JSON Schema type definition: `GenericTaskPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/glossaryApprovalPayload.json | JSON Schema type definition: `GlossaryApprovalPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/incidentResolutionPayload.json | JSON Schema type definition: `IncidentResolutionPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/include.json | JSON Schema type definition: `Include` | High |
| openmetadata-spec/src/main/resources/json/schema/type/jdbcConnection.json | JSON Schema type definition: `JDBC connection` | High |
| openmetadata-spec/src/main/resources/json/schema/type/layerPaging.json | JSON Schema type definition: `LayerPaging` | High |
| openmetadata-spec/src/main/resources/json/schema/type/lifeCycle.json | JSON Schema type definition: `Life Cycle` | High |
| openmetadata-spec/src/main/resources/json/schema/type/ownerConfig.json | JSON Schema type definition: `Owner Configuration` | High |
| openmetadata-spec/src/main/resources/json/schema/type/ownershipUpdatePayload.json | JSON Schema type definition: `OwnershipUpdatePayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/paging.json | JSON Schema type definition: `Paging` | High |
| openmetadata-spec/src/main/resources/json/schema/type/patternRecognizer.json | JSON Schema type definition: `PatternRecognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/personaPreferences.json | JSON Schema type definition: `PersonaPreferences` | High |
| openmetadata-spec/src/main/resources/json/schema/type/piiEntity.json | JSON Schema type definition: `PIIEntity` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineExecutionTrend.json | JSON Schema type definition: `Pipeline Execution Trend` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineExecutionTrendList.json | JSON Schema type definition: `Pipeline Execution Trend List` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineMetrics.json | JSON Schema type definition: `Pipeline Metrics` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineObservability.json | JSON Schema type definition: `Pipeline Observability` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineObservabilityResponse.json | JSON Schema type definition: `PipelineObservabilityResponse` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineRuntimeTrend.json | JSON Schema type definition: `Pipeline Runtime Trend` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineRuntimeTrendList.json | JSON Schema type definition: `Pipeline Runtime Trend List` | High |
| openmetadata-spec/src/main/resources/json/schema/type/pipelineSummary.json | JSON Schema type definition: `Pipeline Summary` | High |
| openmetadata-spec/src/main/resources/json/schema/type/predefinedRecognizer.json | JSON Schema type definition: `PredefinedRecognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/profile.json | JSON Schema type definition: `Profile` | High |
| openmetadata-spec/src/main/resources/json/schema/type/queryParserData.json | JSON Schema type definition: `Query Parser Data` | High |
| openmetadata-spec/src/main/resources/json/schema/type/reaction.json | JSON Schema type definition: `Reaction` | High |
| openmetadata-spec/src/main/resources/json/schema/type/recognizer.json | JSON Schema type definition: `Recognizer` | High |
| openmetadata-spec/src/main/resources/json/schema/type/recognizerFeedback.json | JSON Schema type definition: `RecognizerFeedback` | High |
| openmetadata-spec/src/main/resources/json/schema/type/recognizerMetadata.json | JSON Schema type definition: `RecognizerMetadata` | High |
| openmetadata-spec/src/main/resources/json/schema/type/recognizers/patterns.json | JSON Schema type definition: `Pattern` | High |
| openmetadata-spec/src/main/resources/json/schema/type/recognizers/regexFlags.json | JSON Schema type definition: `RegexFlags` | High |
| openmetadata-spec/src/main/resources/json/schema/type/regexMode.json | JSON Schema type definition: `RegexMode` | High |
| openmetadata-spec/src/main/resources/json/schema/type/reviewPayload.json | JSON Schema type definition: `ReviewPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/samplingConfig.json | JSON Schema type definition: `SamplingConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/schedule.json | JSON Schema type definition: `Schedule` | High |
| openmetadata-spec/src/main/resources/json/schema/type/schema.json | JSON Schema type definition: `Topic` | High |
| openmetadata-spec/src/main/resources/json/schema/type/staticSamplingConfig.json | JSON Schema type definition: `StaticSamplingConfig` | High |
| openmetadata-spec/src/main/resources/json/schema/type/status.json | JSON Schema type definition: `EntityStatus` | High |
| openmetadata-spec/src/main/resources/json/schema/type/suggestionPayload.json | JSON Schema type definition: `SuggestionPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tableQuery.json | JSON Schema type definition: `Table Queries` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tableUsageCount.json | JSON Schema type definition: `Table Usage Count` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tagLabel.json | JSON Schema type definition: `TagLabel` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tagLabelMetadata.json | JSON Schema type definition: `TagLabelMetadata` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tagLabelRecognizerMetadata.json | JSON Schema type definition: `TagLabelRecognizerMetadata` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tagUpdatePayload.json | JSON Schema type definition: `TagUpdatePayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/termRelation.json | JSON Schema type definition: `TermRelation` | High |
| openmetadata-spec/src/main/resources/json/schema/type/testCaseResolutionPayload.json | JSON Schema type definition: `TestCaseResolutionPayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/tierUpdatePayload.json | JSON Schema type definition: `TierUpdatePayload` | High |
| openmetadata-spec/src/main/resources/json/schema/type/usageDetails.json | JSON Schema type definition: `UsageDetails` | High |
| openmetadata-spec/src/main/resources/json/schema/type/usageRequest.json | JSON Schema type definition: `Usage Request` | High |
| openmetadata-spec/src/main/resources/json/schema/type/votes.json | JSON Schema type definition: `Votes` | High |
| openmetadata-spec/src/main/resources/json/schema/type/workflowTriggerFields.json | JSON Schema type definition: `WorkflowTriggerFields` | High |

## Jobs
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/jobs/OAuthTokenCleanupJob.java | `class OAuthTokenCleanupJob` implements Quartz/K8s job contract | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfIndexJob.java | `class RdfIndexJob` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/SearchIndexJob.java | `class SearchIndexJob` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/clients/pipeline/k8s/CronOMJob.java | `class CronOMJob` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/clients/pipeline/k8s/OMJob.java | `class OMJob` | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/events/scheduled/DatabseAndSearchServiceStatusJob.java | `class DatabseAndSearchServiceStatusJob` implements Quartz/K8s job contract | High |

## Workers
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfPartitionWorker.java | `class RdfPartitionWorker`; background partition/retry worker | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/PartitionWorker.java | `class PartitionWorker`; background partition/retry worker | High |
| openmetadata-service/src/main/java/org/openmetadata/service/jobs/GenericBackgroundWorker.java | `class GenericBackgroundWorker`; background partition/retry worker | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/SearchIndexRetryWorker.java | `class SearchIndexRetryWorker`; background partition/retry worker | High |

## Consumers
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/AbstractEventConsumer.java | `class AbstractEventConsumer`; event/message consumption role | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/Consumer.java | `class Consumer`; event/message consumption role | High |
| openmetadata-service/src/main/java/org/openmetadata/service/audit/AuditLogConsumer.java | `class AuditLogConsumer implements org.quartz.Job`; consumes change events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/WorkflowEventConsumer.java | `class WorkflowEventConsumer`; event/message consumption role | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/search/opensearch/SafeResponseConsumer.java | `class SafeResponseConsumer`; event/message consumption role | Medium |
| openmetadata-service/src/main/java/org/openmetadata/service/security/saml/SamlAssertionConsumerServlet.java | `class SamlAssertionConsumerServlet`; event/message consumption role | Medium |

## Producers
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/AlertPublisher.java | `class AlertPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/email/EmailPublisher.java | `class EmailPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/feed/ActivityStreamPublisher.java | `class ActivityStreamPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/gchat/GChatPublisher.java | `class GChatPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/generic/GenericPublisher.java | `class GenericPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/msteams/MSTeamsPublisher.java | `class MSTeamsPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/changeEvent/slack/SlackEventPublisher.java | `class SlackEventPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/audit/AuditLogEventPublisher.java | `class AuditLogEventPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/events/AbstractEventPublisher.java | `class AbstractEventPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/events/EventPublisher.java | `class EventPublisher` publishes change/alert/notification events | High |
| openmetadata-service/src/main/java/org/openmetadata/service/monitoring/EventMonitorPublisher.java | `class EventMonitorPublisher` publishes change/alert/notification events | High |

## Configurations
| File Path | Evidence | Confidence |
|---|---|---|
| conf/openmetadata.yaml | Primary Dropwizard server config; referenced at runtime bootstrap | High |
| conf/openmetadata-h2-test.yaml | H2 test configuration | High |
| conf/openmetadata-s3-logs.yaml | S3 logging configuration variant | High |
| conf/operations.yaml | Operations mode configuration | High |
| openmetadata-service/src/main/java/org/openmetadata/service/OpenMetadataApplicationConfig.java | `class OpenMetadataApplicationConfig extends Configuration`; Dropwizard config binding | High |
| openmetadata-k8s-operator/src/main/resources/application.yaml | K8s operator application config | High |
| docker/openmetadata.yaml | Docker deployment configuration | High |

## Utilities
### Verified Findings (by package directory)
| File Path | Evidence | Confidence |
|---|---|---|
| common/src/main/java/org/openmetadata/annotations/utils/ | 1 utility source files; e.g. `common/src/main/java/org/openmetadata/annotations/utils/AnnotationChecker.java` | High |
| common/src/main/java/org/openmetadata/common/utils/ | 1 utility source files; e.g. `common/src/main/java/org/openmetadata/common/utils/CommonUtil.java` | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/ | 4 utility source files; e.g. `openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/LabelBuilder.java` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/ | 2 utility source files; e.g. `openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/UriUtils.java` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/ | 2 utility source files; e.g. `openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/McpParams.java` | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/ | 4 utility source files; e.g. `openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/SdkClients.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/utils/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/utils/TimestampUtils.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/ | 2 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/FeedMessage.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/ | 2 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/ChangePreviewUtils.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/ | 4 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/SearchSettingsMergeUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V112/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V112/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V114/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V114/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V117/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V117/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v110/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v110/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1100/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1100/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1102/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1102/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1104/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1104/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1105/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1105/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v111/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v111/MigrationUtilV111.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1110/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1110/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1112/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1112/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1114/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1114/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1115/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1115/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1120/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1120/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v11210/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v11210/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1122/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1122/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1125/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1125/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1126/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1126/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1129/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1129/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1130/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1130/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v120/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v120/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v130/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v130/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v131/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v131/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v132/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v132/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v140/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v140/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v141/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v141/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v150/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v150/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v153/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v153/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v155/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v155/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v157/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v157/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v159/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v159/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v160/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v160/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v170/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v170/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v171/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v171/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v172/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v172/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v180/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v180/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v184/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v184/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v190/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v190/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1910/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1910/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1911/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1911/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v196/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v196/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v199/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v199/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v200/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v200/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v201/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v201/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v210/ | 1 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v210/MigrationUtil.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/ | 3 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/TextChunkManager.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ | 74 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/RestoreEntityMessage.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/branding/ | 3 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/branding/DefaultMessageBrandingProvider.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/ | 18 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/PostgresAutoTuner.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/ | 4 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/email/TemplateProvider.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/ | 2 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/IncidentSeverityClassifierInterface.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/ | 13 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProviderException.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/ | 3 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/RelationshipValidator.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/ | 2 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/ResourcePathResolver.java` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/ | 4 utility source files; e.g. `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/EmailEnvelopeResourcePathProvider.java` | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/utils/ | 4 utility source files; e.g. `openmetadata-spec/src/main/java/org/openmetadata/schema/utils/ResultList.java` | High |

### File-level enumeration
| File Path | Evidence | Confidence |
|---|---|---|
| common/src/main/java/org/openmetadata/annotations/utils/AnnotationChecker.java | Utility module under `common/src/main/java/org/openmetadata/annotations/utils/` | High |
| common/src/main/java/org/openmetadata/common/utils/CommonUtil.java | Utility module under `common/src/main/java/org/openmetadata/common/utils/` | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/EnvVarUtils.java | Utility module under `openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/` | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/HashUtils.java | Utility module under `openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/` | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/KubernetesNameBuilder.java | Utility module under `openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/` | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/LabelBuilder.java | Utility module under `openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/util/` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/ClientCredentialsExtractor.java | Utility module under `openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/UriUtils.java | Utility module under `openmetadata-mcp/src/main/java/org/openmetadata/mcp/server/auth/util/` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/McpParams.java | Utility module under `openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/` | High |
| openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/McpResponseTrim.java | Utility module under `openmetadata-mcp/src/main/java/org/openmetadata/mcp/util/` | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/RestClient.java | Utility module under `openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/` | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/SdkClients.java | Utility module under `openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/` | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/TestNamespace.java | Utility module under `openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/` | High |
| openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/TestNamespaceExtension.java | Utility module under `openmetadata-sdk/src/main/java/org/openmetadata/sdk/test/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/utils/TimestampUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/apps/bundles/insights/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/FeedMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/FormatterUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/formatter/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/ChangePreviewUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/FieldChangeValueExtractor.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/governance/workflows/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/FlywayMigrationFile.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/LongStatementsUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/MigrationFile.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/SearchSettingsMergeUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V112/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V112/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V114/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V114/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V117/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/V117/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v110/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v110/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1100/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1100/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1102/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1102/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1104/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1104/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1105/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1105/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v111/MigrationUtilV111.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v111/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1110/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1110/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1112/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1112/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1114/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1114/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1115/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1115/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1120/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1120/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v11210/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v11210/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1122/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1122/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1125/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1125/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1126/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1126/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1129/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1129/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1130/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1130/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v120/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v120/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v130/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v130/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v131/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v131/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v132/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v132/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v140/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v140/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v141/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v141/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v150/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v150/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v153/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v153/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v155/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v155/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v157/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v157/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v159/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v159/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v160/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v160/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v170/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v170/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v171/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v171/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v172/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v172/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v180/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v180/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v184/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v184/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v190/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v190/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1910/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1910/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1911/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v1911/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v196/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v196/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v199/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v199/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v200/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v200/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v201/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v201/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v210/MigrationUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/migration/utils/v210/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/AvailableEntityTypes.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/DTOs.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/TextChunkManager.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/search/vector/utils/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ActivityStreamPartitionManager.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AppMarketPlaceUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AsciiTable.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AsyncService.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AuthenticationMechanismBuilder.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AwsCredentialsUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/AzureTokenProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/BulkAssetsOperationMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/BulkAssetsOperationResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/CSVExportMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/CSVExportResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/CSVImportMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/CSVImportResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/CustomParameterNameProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/DIContainer.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/DataInsightFormulaEvaluator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/DeleteEntityMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/DeleteEntityResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/DescriptionSanitizer.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityETag.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityFieldUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityHierarchyList.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityRelationshipCleanup.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityRelationshipCleanupUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/EntityWithType.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/FeedUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/FieldPathUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/FlowableCleanup.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/FullyQualifiedName.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/IngestionPipelineBuilder.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/IntakeFormValidator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/JsonPatchUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LambdaExceptionUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LdapUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LikeEscape.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LineageGraphExplorer.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/LineageUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ListWithOffsetFunction.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/MoveGlossaryTermMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/MoveGlossaryTermResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/OAuth2TokenManager.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ODCSConverter.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ODPSConverter.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/OMMicroMeterBundler.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/OpenMetadataConnectionBuilder.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/OpenMetadataOperations.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/OrphanTestCaseCleanup.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ParallelStreamUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/PasswordUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/PipelineStatusUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/QueryRunnerMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ReflectionUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/Registry.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ReindexingProgressMonitor.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/RequestEntityCache.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/RestUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/RestoreEntityMessage.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/RestoreEntityResponse.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/SSLUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/SchemaFieldExtractor.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/SearchUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ServiceHierarchyCleanup.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/SubscriptionUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/TableUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/TagUsageCleanup.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/TokenUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/URLValidator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/UserUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/Utilities.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ValidationErrorBuilder.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ValidationHttpUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/ValidatorUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/WebsocketNotificationHandler.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/branding/DefaultMessageBrandingProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/branding/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/branding/MessageBrandingProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/branding/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/branding/MessageBrandingResolver.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/branding/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/Action.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/AutoTuner.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/DbTuneDiagnosis.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/DbTuneReport.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/DbTuneResult.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/Diagnostic.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/DiagnosticCategory.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/Finding.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/MysqlAutoTuner.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/MysqlDiagnostic.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/MysqlTuningCatalog.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/PostgresAutoTuner.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/PostgresDiagnostic.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/PostgresTuningCatalog.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/ServerParamCheck.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/Severity.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/TableRecommendation.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/TableStats.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/dbtune/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/DefaultTemplateProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/email/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/EmailUtil.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/email/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/TemplateConstants.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/email/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/email/TemplateProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/email/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/IncidentSeverityClassifierInterface.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/LogisticRegressionIncidentSeverityClassifier.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/incidentSeverityClassifier/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/AwsRdsDatabaseAuthenticationProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/AzureDatabaseAuthenticationProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/BindConcat.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/BindFQN.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/BindJsonContains.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/BindListFQN.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/BindUUID.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProviderException.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProviderFactory.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/JdbiUtils.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/JsonContainsFilterFactory.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/OMSqlLogger.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/jdbi/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/DefaultRelationshipValidator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/LineageRelationshipValidator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/RelationshipValidator.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/relationshipcleanup/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/ResourcePathProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/ResourcePathResolver.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/DefaultEmailEnvelopeResourcePathProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/DefaultNotificationTemplateResourcePathProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/EmailEnvelopeResourcePathProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/` | High |
| openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/NotificationTemplateResourcePathProvider.java | Utility module under `openmetadata-service/src/main/java/org/openmetadata/service/util/resourcepath/providers/` | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/utils/EntityInterfaceUtil.java | Utility module under `openmetadata-spec/src/main/java/org/openmetadata/schema/utils/` | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/utils/JsonUtils.java | Utility module under `openmetadata-spec/src/main/java/org/openmetadata/schema/utils/` | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/utils/ResultList.java | Utility module under `openmetadata-spec/src/main/java/org/openmetadata/schema/utils/` | High |
| openmetadata-spec/src/main/java/org/openmetadata/schema/utils/VersionUtils.java | Utility module under `openmetadata-spec/src/main/java/org/openmetadata/schema/utils/` | High |

## Tests
### Summary by module
| Module | Count | Evidence | Confidence |
|---|---|---|---|
| common | 2 | 2 test files under module `common` | High |
| ingestion | 945 | 945 test files under module `ingestion` | High |
| openmetadata-clients | 1 | 1 test files under module `openmetadata-clients` | High |
| openmetadata-integration-tests | 1 | 1 test files under module `openmetadata-integration-tests` | High |
| openmetadata-k8s-operator | 8 | 8 test files under module `openmetadata-k8s-operator` | High |
| openmetadata-mcp | 39 | 39 test files under module `openmetadata-mcp` | High |
| openmetadata-sdk | 27 | 27 test files under module `openmetadata-sdk` | High |
| openmetadata-service | 495 | 495 test files under module `openmetadata-service` | High |
| openmetadata-ui | 1292 | 1292 test files under module `openmetadata-ui` | High |

### Full enumeration
| File Path | Evidence | Confidence |
|---|---|---|
| common/src/test/java/org/openmetadata/annotations/utils/AnnotationCheckerTest.java | JUnit test class `AnnotationCheckerTest.java` | High |
| common/src/test/java/org/openmetadata/common/utils/CommonUtilTest.java | JUnit test class `CommonUtilTest.java` | High |
| ingestion/src/metadata/sdk/data_quality/tests/__init__.py | Python test `__init__.py` | High |
| ingestion/src/metadata/sdk/data_quality/tests/base_tests.py | Python test `base_tests.py` | High |
| ingestion/src/metadata/sdk/data_quality/tests/column_tests.py | Python test `column_tests.py` | High |
| ingestion/src/metadata/sdk/data_quality/tests/table_tests.py | Python test `table_tests.py` | High |
| ingestion/tests/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/cli_e2e/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/cli_e2e/base/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/cli_e2e/base/config_builders/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/cli_e2e/base/config_builders/builders.py | Python test `builders.py` | High |
| ingestion/tests/cli_e2e/base/e2e_types.py | Python test `e2e_types.py` | High |
| ingestion/tests/cli_e2e/base/test_cli.py | Python test `test_cli.py` | High |
| ingestion/tests/cli_e2e/base/test_cli_dashboard.py | Python test `test_cli_dashboard.py` | High |
| ingestion/tests/cli_e2e/base/test_cli_db.py | Python test `test_cli_db.py` | High |
| ingestion/tests/cli_e2e/base/test_cli_dbt.py | Python test `test_cli_dbt.py` | High |
| ingestion/tests/cli_e2e/common/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/cli_e2e/common/test_cli_dashboard.py | Python test `test_cli_dashboard.py` | High |
| ingestion/tests/cli_e2e/common/test_cli_db.py | Python test `test_cli_db.py` | High |
| ingestion/tests/cli_e2e/common_e2e_sqa_mixins.py | Python test `common_e2e_sqa_mixins.py` | High |
| ingestion/tests/cli_e2e/test_cli_athena.py | Python test `test_cli_athena.py` | High |
| ingestion/tests/cli_e2e/test_cli_bigquery.py | Python test `test_cli_bigquery.py` | High |
| ingestion/tests/cli_e2e/test_cli_bigquery_multiple_project.py | Python test `test_cli_bigquery_multiple_project.py` | High |
| ingestion/tests/cli_e2e/test_cli_datalake_s3.py | Python test `test_cli_datalake_s3.py` | High |
| ingestion/tests/cli_e2e/test_cli_dbt_redshift.py | Python test `test_cli_dbt_redshift.py` | High |
| ingestion/tests/cli_e2e/test_cli_exasol.py | Python test `test_cli_exasol.py` | High |
| ingestion/tests/cli_e2e/test_cli_hive.py | Python test `test_cli_hive.py` | High |
| ingestion/tests/cli_e2e/test_cli_metabase.py | Python test `test_cli_metabase.py` | High |
| ingestion/tests/cli_e2e/test_cli_mssql.py | Python test `test_cli_mssql.py` | High |
| ingestion/tests/cli_e2e/test_cli_mysql.py | Python test `test_cli_mysql.py` | High |
| ingestion/tests/cli_e2e/test_cli_oracle.py | Python test `test_cli_oracle.py` | High |
| ingestion/tests/cli_e2e/test_cli_postgres.py | Python test `test_cli_postgres.py` | High |
| ingestion/tests/cli_e2e/test_cli_powerbi.py | Python test `test_cli_powerbi.py` | High |
| ingestion/tests/cli_e2e/test_cli_quicksight.py | Python test `test_cli_quicksight.py` | High |
| ingestion/tests/cli_e2e/test_cli_redash.py | Python test `test_cli_redash.py` | High |
| ingestion/tests/cli_e2e/test_cli_redshift.py | Python test `test_cli_redshift.py` | High |
| ingestion/tests/cli_e2e/test_cli_snowflake.py | Python test `test_cli_snowflake.py` | High |
| ingestion/tests/cli_e2e/test_cli_tableau.py | Python test `test_cli_tableau.py` | High |
| ingestion/tests/cli_e2e/test_cli_vertica.py | Python test `test_cli_vertica.py` | High |
| ingestion/tests/integration/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/airflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/airflow/test_airflow_api_connection.py | Python test `test_airflow_api_connection.py` | High |
| ingestion/tests/integration/airflow/test_airflow_lineage.py | Python test `test_airflow_lineage.py` | High |
| ingestion/tests/integration/airflow/test_dags/lineage_etl.py | Python test `lineage_etl.py` | High |
| ingestion/tests/integration/airflow/test_dags/ol_lineage_etl.py | Python test `ol_lineage_etl.py` | High |
| ingestion/tests/integration/airflow/test_dags/sample_branching.py | Python test `sample_branching.py` | High |
| ingestion/tests/integration/airflow/test_dags/sample_etl.py | Python test `sample_etl.py` | High |
| ingestion/tests/integration/airflow/test_lineage_runner.py | Python test `test_lineage_runner.py` | High |
| ingestion/tests/integration/airflow/test_openlineage_lineage.py | Python test `test_openlineage_lineage.py` | High |
| ingestion/tests/integration/airflow/test_status_callback.py | Python test `test_status_callback.py` | High |
| ingestion/tests/integration/alationsink/test_alationsink.py | Python test `test_alationsink.py` | High |
| ingestion/tests/integration/amundsen/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/amundsen/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/atlas/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/atlas/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/auto_classification/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/auto_classification/containers/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/auto_classification/containers/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/auto_classification/containers/test_container_classification.py | Python test `test_container_classification.py` | High |
| ingestion/tests/integration/auto_classification/databases/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/auto_classification/databases/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/auto_classification/databases/test_azuresql_temporal_table.py | Python test `test_azuresql_temporal_table.py` | High |
| ingestion/tests/integration/auto_classification/databases/test_global_sample_data_config.py | Python test `test_global_sample_data_config.py` | High |
| ingestion/tests/integration/auto_classification/databases/test_tag_processor.py | Python test `test_tag_processor.py` | High |
| ingestion/tests/integration/automations/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/automations/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/automations/test_connection_automation.py | Python test `test_connection_automation.py` | High |
| ingestion/tests/integration/cassandra/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/cassandra/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/cassandra/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/cockroach/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/cockroach/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/cockroach/test_classifier.py | Python test `test_classifier.py` | High |
| ingestion/tests/integration/cockroach/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/connections/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/connections/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/connections/test_mysql_connection.py | Python test `test_mysql_connection.py` | High |
| ingestion/tests/integration/connections/test_ssrs_connection.py | Python test `test_ssrs_connection.py` | High |
| ingestion/tests/integration/containers.py | Python test `containers.py` | High |
| ingestion/tests/integration/data_quality/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/data_quality/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/data_quality/test_data_diff.py | Python test `test_data_diff.py` | High |
| ingestion/tests/integration/data_quality/test_data_quality.py | Python test `test_data_quality.py` | High |
| ingestion/tests/integration/data_quality/test_failed_row_samples.py | Python test `test_failed_row_samples.py` | High |
| ingestion/tests/integration/datalake/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/datalake/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/datalake/test_data_quality.py | Python test `test_data_quality.py` | High |
| ingestion/tests/integration/datalake/test_datalake_profiler_e2e.py | Python test `test_datalake_profiler_e2e.py` | High |
| ingestion/tests/integration/datalake/test_datalake_profiler_sampling.py | Python test `test_datalake_profiler_sampling.py` | High |
| ingestion/tests/integration/datalake/test_ingestion.py | Python test `test_ingestion.py` | High |
| ingestion/tests/integration/datalake/test_rule_library_pandas.py | Python test `test_rule_library_pandas.py` | High |
| ingestion/tests/integration/datalake/test_table_rule_library_pandas.py | Python test `test_table_rule_library_pandas.py` | High |
| ingestion/tests/integration/fivetran/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/fivetran/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/fivetran/test_fivetran_client.py | Python test `test_fivetran_client.py` | High |
| ingestion/tests/integration/great_expectations/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/great_expectations/test_great_expectation_integration.py | Python test `test_great_expectation_integration.py` | High |
| ingestion/tests/integration/great_expectations/test_great_expectation_integration_1xx.py | Python test `test_great_expectation_integration_1xx.py` | High |
| ingestion/tests/integration/integration_base.py | Python test `integration_base.py` | High |
| ingestion/tests/integration/kafka/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/kafka/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/kafka/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/lineage/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/lineage/e2e/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/lineage/e2e/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/lineage/e2e/helpers.py | Python test `helpers.py` | High |
| ingestion/tests/integration/lineage/e2e/test_query_lineage.py | Python test `test_query_lineage.py` | High |
| ingestion/tests/integration/lineage/e2e/test_stored_procedure_lineage.py | Python test `test_stored_procedure_lineage.py` | High |
| ingestion/tests/integration/lineage/e2e/test_view_lineage.py | Python test `test_view_lineage.py` | High |
| ingestion/tests/integration/mcp/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/mcp/test_mcp_integration.py | Python test `test_mcp_integration.py` | High |
| ingestion/tests/integration/mongodb/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/mongodb/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/mysql/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/mysql/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/mysql/test_classifier.py | Python test `test_classifier.py` | High |
| ingestion/tests/integration/mysql/test_column_order.py | Python test `test_column_order.py` | High |
| ingestion/tests/integration/mysql/test_data_quality.py | Python test `test_data_quality.py` | High |
| ingestion/tests/integration/mysql/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/mysql/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/integration/mysql/test_profiler_sampling.py | Python test `test_profiler_sampling.py` | High |
| ingestion/tests/integration/mysql/test_rule_library_sql_expression.py | Python test `test_rule_library_sql_expression.py` | High |
| ingestion/tests/integration/mysql/test_table_rule_library_sql_expression.py | Python test `test_table_rule_library_sql_expression.py` | High |
| ingestion/tests/integration/ometa/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/ometa/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/ometa/test_ometa_app_api.py | Python test `test_ometa_app_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_bot_rbac.py | Python test `test_ometa_bot_rbac.py` | High |
| ingestion/tests/integration/ometa/test_ometa_chart_api.py | Python test `test_ometa_chart_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_connection_definition_api.py | Python test `test_ometa_connection_definition_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_custom_properties_api.py | Python test `test_ometa_custom_properties_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_dashboard_api.py | Python test `test_ometa_dashboard_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_data_contract_api.py | Python test `test_ometa_data_contract_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_database_api.py | Python test `test_ometa_database_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_database_service_api.py | Python test `test_ometa_database_service_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_domains_api.py | Python test `test_ometa_domains_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_es_api.py | Python test `test_ometa_es_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_from_env.py | Python test `test_ometa_from_env.py` | High |
| ingestion/tests/integration/ometa/test_ometa_glossary.py | Python test `test_ometa_glossary.py` | High |
| ingestion/tests/integration/ometa/test_ometa_ingestion_pipeline.py | Python test `test_ometa_ingestion_pipeline.py` | High |
| ingestion/tests/integration/ometa/test_ometa_life_cycle_api.py | Python test `test_ometa_life_cycle_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_lineage_api.py | Python test `test_ometa_lineage_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_mlmodel_api.py | Python test `test_ometa_mlmodel_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_patch.py | Python test `test_ometa_patch.py` | High |
| ingestion/tests/integration/ometa/test_ometa_pipeline_api.py | Python test `test_ometa_pipeline_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_rest_api.py | Python test `test_ometa_rest_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_role_policy_api.py | Python test `test_ometa_role_policy_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_secrets_manager.py | Python test `test_ometa_secrets_manager.py` | High |
| ingestion/tests/integration/ometa/test_ometa_server_api.py | Python test `test_ometa_server_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_service_api.py | Python test `test_ometa_service_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_storage_api.py | Python test `test_ometa_storage_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_subscription_api.py | Python test `test_ometa_subscription_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_suggestion_api.py | Python test `test_ometa_suggestion_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_table_api.py | Python test `test_ometa_table_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_tags_mixin.py | Python test `test_ometa_tags_mixin.py` | High |
| ingestion/tests/integration/ometa/test_ometa_test_suite.py | Python test `test_ometa_test_suite.py` | High |
| ingestion/tests/integration/ometa/test_ometa_topic_api.py | Python test `test_ometa_topic_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_topology_patch.py | Python test `test_ometa_topology_patch.py` | High |
| ingestion/tests/integration/ometa/test_ometa_topology_restore.py | Python test `test_ometa_topology_restore.py` | High |
| ingestion/tests/integration/ometa/test_ometa_user_api.py | Python test `test_ometa_user_api.py` | High |
| ingestion/tests/integration/ometa/test_ometa_workflow_api.py | Python test `test_ometa_workflow_api.py` | High |
| ingestion/tests/integration/orm_profiler/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/orm_profiler/system/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/orm_profiler/system/test_bigquery_system_metrics.py | Python test `test_bigquery_system_metrics.py` | High |
| ingestion/tests/integration/orm_profiler/system/test_redshift_system_metrics.py | Python test `test_redshift_system_metrics.py` | High |
| ingestion/tests/integration/orm_profiler/system/test_snowflake_system_metrics.py | Python test `test_snowflake_system_metrics.py` | High |
| ingestion/tests/integration/orm_profiler/test_converter.py | Python test `test_converter.py` | High |
| ingestion/tests/integration/orm_profiler/test_orm_profiler_e2e.py | Python test `test_orm_profiler_e2e.py` | High |
| ingestion/tests/integration/orm_profiler/test_pii_processor.py | Python test `test_pii_processor.py` | High |
| ingestion/tests/integration/postgres/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/postgres/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/postgres/test_data_quality.py | Python test `test_data_quality.py` | High |
| ingestion/tests/integration/postgres/test_lineage.py | Python test `test_lineage.py` | High |
| ingestion/tests/integration/postgres/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/postgres/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/integration/postgres/test_rule_library_sql_expression.py | Python test `test_rule_library_sql_expression.py` | High |
| ingestion/tests/integration/postgres/test_table_metric_computer.py | Python test `test_table_metric_computer.py` | High |
| ingestion/tests/integration/postgres/test_table_rule_library_sql_expression.py | Python test `test_table_rule_library_sql_expression.py` | High |
| ingestion/tests/integration/postgres/test_usage.py | Python test `test_usage.py` | High |
| ingestion/tests/integration/powerbi/test_powerbi_file_client.py | Python test `test_powerbi_file_client.py` | High |
| ingestion/tests/integration/profiler/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/profiler/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/profiler/test_dynamic_sampling.py | Python test `test_dynamic_sampling.py` | High |
| ingestion/tests/integration/profiler/test_dynamic_sampling_mssql.py | Python test `test_dynamic_sampling_mssql.py` | High |
| ingestion/tests/integration/profiler/test_dynamodb.py | Python test `test_dynamodb.py` | High |
| ingestion/tests/integration/profiler/test_median_mariadb.py | Python test `test_median_mariadb.py` | High |
| ingestion/tests/integration/profiler/test_median_mysql.py | Python test `test_median_mysql.py` | High |
| ingestion/tests/integration/profiler/test_median_singlestore.py | Python test `test_median_singlestore.py` | High |
| ingestion/tests/integration/profiler/test_nosql_profiler.py | Python test `test_nosql_profiler.py` | High |
| ingestion/tests/integration/profiler/test_sqa_profiler.py | Python test `test_sqa_profiler.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_clickhouse.py | Python test `test_table_metric_computer_clickhouse.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_cockroach.py | Python test `test_table_metric_computer_cockroach.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_hive.py | Python test `test_table_metric_computer_hive.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_mssql.py | Python test `test_table_metric_computer_mssql.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_mysql.py | Python test `test_table_metric_computer_mysql.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_postgres.py | Python test `test_table_metric_computer_postgres.py` | High |
| ingestion/tests/integration/profiler/test_table_metric_computer_timescale.py | Python test `test_table_metric_computer_timescale.py` | High |
| ingestion/tests/integration/s3/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/s3/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/s3/test_s3_manifest_wildcards.py | Python test `test_s3_manifest_wildcards.py` | High |
| ingestion/tests/integration/s3/test_s3_storage.py | Python test `test_s3_storage.py` | High |
| ingestion/tests/integration/sas/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sas/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/sdk/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sdk/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/sdk/data_quality/dataframes/test_dataframe_validator.py | Python test `test_dataframe_validator.py` | High |
| ingestion/tests/integration/sdk/test_dq_as_code_integration.py | Python test `test_dq_as_code_integration.py` | High |
| ingestion/tests/integration/sdk/test_sdk_integration.py | Python test `test_sdk_integration.py` | High |
| ingestion/tests/integration/sftp/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sftp/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/sftp/test_sftp_ingestion.py | Python test `test_sftp_ingestion.py` | High |
| ingestion/tests/integration/sources/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sources/database/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sources/database/delta_lake/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sources/database/delta_lake/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/sources/database/delta_lake/test_deltalake_storage.py | Python test `test_deltalake_storage.py` | High |
| ingestion/tests/integration/sources/mlmodels/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sources/mlmodels/mlflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sources/mlmodels/mlflow/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/sources/mlmodels/mlflow/test_mlflow.py | Python test `test_mlflow.py` | High |
| ingestion/tests/integration/sql_server/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/sql_server/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/sql_server/test_lineage.py | Python test `test_lineage.py` | High |
| ingestion/tests/integration/sql_server/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/sql_server/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/integration/sql_server/test_usage.py | Python test `test_usage.py` | High |
| ingestion/tests/integration/ssrs/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/ssrs/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/ssrs/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/superset/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/superset/test_superset.py | Python test `test_superset.py` | High |
| ingestion/tests/integration/test_suite/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/test_suite/test_e2e_workflow.py | Python test `test_e2e_workflow.py` | High |
| ingestion/tests/integration/test_suite/test_registry_names_match_test_definition.py | Python test `test_registry_names_match_test_definition.py` | High |
| ingestion/tests/integration/test_suite/test_workflow.py | Python test `test_workflow.py` | High |
| ingestion/tests/integration/trino/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/trino/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/trino/test_classifier.py | Python test `test_classifier.py` | High |
| ingestion/tests/integration/trino/test_data_quality.py | Python test `test_data_quality.py` | High |
| ingestion/tests/integration/trino/test_metadata.py | Python test `test_metadata.py` | High |
| ingestion/tests/integration/trino/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/integration/trino/test_profiler_sampling.py | Python test `test_profiler_sampling.py` | High |
| ingestion/tests/integration/trino/test_table_metric_computer_trino.py | Python test `test_table_metric_computer_trino.py` | High |
| ingestion/tests/integration/usage/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/usage/test_sample_usage.py | Python test `test_sample_usage.py` | High |
| ingestion/tests/integration/workflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/integration/workflow/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/integration/workflow/test_workflow.py | Python test `test_workflow.py` | High |
| ingestion/tests/load/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/load/test_load.py | Python test `test_load.py` | High |
| ingestion/tests/load/test_resources/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/load/test_resources/all_resources.py | Python test `all_resources.py` | High |
| ingestion/tests/load/test_resources/tasks/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/load/test_resources/tasks/test_case_result_tasks.py | Python test `test_case_result_tasks.py` | High |
| ingestion/tests/load/test_resources/tasks/test_case_tasks.py | Python test `test_case_tasks.py` | High |
| ingestion/tests/load/utils.py | Python test `utils.py` | High |
| ingestion/tests/unit/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/airflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/airflow/test_airflow_metadata.py | Python test `test_airflow_metadata.py` | High |
| ingestion/tests/unit/airflow/test_lineage_parser.py | Python test `test_lineage_parser.py` | High |
| ingestion/tests/unit/bulksink/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/bulksink/test_metadata_usage.py | Python test `test_metadata_usage.py` | High |
| ingestion/tests/unit/clients/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/clients/test_aws_client.py | Python test `test_aws_client.py` | High |
| ingestion/tests/unit/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/connections/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/connections/test_iam_token_refresh.py | Python test `test_iam_token_refresh.py` | High |
| ingestion/tests/unit/connections/test_test_connections.py | Python test `test_test_connections.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_base_connection.py | Python test `test_base_connection.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_check.py | Python test `test_check.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_classifier.py | Python test `test_classifier.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_database_checks.py | Python test `test_database_checks.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_mapper.py | Python test `test_mapper.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_patch_serialization.py | Python test `test_patch_serialization.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_records.py | Python test `test_records.py` | High |
| ingestion/tests/unit/core/connections/test_connection/test_runner.py | Python test `test_runner.py` | High |
| ingestion/tests/unit/data_quality/validations/test_failed_sample_mixin.py | Python test `test_failed_sample_mixin.py` | High |
| ingestion/tests/unit/diagnostics/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/diagnostics/test_db_introspect.py | Python test `test_db_introspect.py` | High |
| ingestion/tests/unit/diagnostics/test_heartbeat.py | Python test `test_heartbeat.py` | High |
| ingestion/tests/unit/diagnostics/test_http_introspect.py | Python test `test_http_introspect.py` | High |
| ingestion/tests/unit/diagnostics/test_lifecycle.py | Python test `test_lifecycle.py` | High |
| ingestion/tests/unit/diagnostics/test_memory.py | Python test `test_memory.py` | High |
| ingestion/tests/unit/diagnostics/test_monitor.py | Python test `test_monitor.py` | High |
| ingestion/tests/unit/diagnostics/test_registry.py | Python test `test_registry.py` | High |
| ingestion/tests/unit/diagnostics/test_signals.py | Python test `test_signals.py` | High |
| ingestion/tests/unit/diagnostics/test_stage_progress.py | Python test `test_stage_progress.py` | High |
| ingestion/tests/unit/diagnostics/test_summary.py | Python test `test_summary.py` | High |
| ingestion/tests/unit/diagnostics/test_time_accounting.py | Python test `test_time_accounting.py` | High |
| ingestion/tests/unit/diagnostics/test_tripwire.py | Python test `test_tripwire.py` | High |
| ingestion/tests/unit/diagnostics/test_watchdog.py | Python test `test_watchdog.py` | High |
| ingestion/tests/unit/domain/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/domain/tags/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/domain/tags/test_canonicalizer.py | Python test `test_canonicalizer.py` | High |
| ingestion/tests/unit/domain/tags/test_registry.py | Python test `test_registry.py` | High |
| ingestion/tests/unit/great_expectations/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/great_expectations/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/great_expectations/test_ometa_validation_action.py | Python test `test_ometa_validation_action.py` | High |
| ingestion/tests/unit/great_expectations/test_ometa_validation_action1xx.py | Python test `test_ometa_validation_action1xx.py` | High |
| ingestion/tests/unit/lineage/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/lineage/masker/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/lineage/masker/helpers.py | Python test `helpers.py` | High |
| ingestion/tests/unit/lineage/masker/test_query_masker.py | Python test `test_query_masker.py` | High |
| ingestion/tests/unit/lineage/masker/test_query_masker_dialect_specific.py | Python test `test_query_masker_dialect_specific.py` | High |
| ingestion/tests/unit/lineage/masker/test_query_masker_ordinal_preservation.py | Python test `test_query_masker_ordinal_preservation.py` | High |
| ingestion/tests/unit/lineage/queries/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/lineage/queries/helpers.py | Python test `helpers.py` | High |
| ingestion/tests/unit/lineage/queries/test_complex_query_patterns.py | Python test `test_complex_query_patterns.py` | High |
| ingestion/tests/unit/lineage/queries/test_specific_dialect_queries.py | Python test `test_specific_dialect_queries.py` | High |
| ingestion/tests/unit/lineage/test_cross_database_lineage_sql.py | Python test `test_cross_database_lineage_sql.py` | High |
| ingestion/tests/unit/lineage/test_databricks_lineage.py | Python test `test_databricks_lineage.py` | High |
| ingestion/tests/unit/lineage/test_lineage_processors.py | Python test `test_lineage_processors.py` | High |
| ingestion/tests/unit/lineage/test_lineage_source.py | Python test `test_lineage_source.py` | High |
| ingestion/tests/unit/lineage/test_lineage_workflow_filter_pattern.py | Python test `test_lineage_workflow_filter_pattern.py` | High |
| ingestion/tests/unit/lineage/test_pgspider_lineage_unit.py | Python test `test_pgspider_lineage_unit.py` | High |
| ingestion/tests/unit/lineage/test_sql_lineage.py | Python test `test_sql_lineage.py` | High |
| ingestion/tests/unit/lineage/test_stored_procedure_lineage.py | Python test `test_stored_procedure_lineage.py` | High |
| ingestion/tests/unit/lineage/test_temp_table_lineage.py | Python test `test_temp_table_lineage.py` | High |
| ingestion/tests/unit/metadata/cli/resources/profiler_workflow.py | Python test `profiler_workflow.py` | High |
| ingestion/tests/unit/metadata/common/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/metadata/common/test_ingest_file_load.py | Python test `test_ingest_file_load.py` | High |
| ingestion/tests/unit/metadata/data_quality/test_data_diff.py | Python test `test_data_diff.py` | High |
| ingestion/tests/unit/metadata/data_quality/test_table_diff_param_setter.py | Python test `test_table_diff_param_setter.py` | High |
| ingestion/tests/unit/metadata/ingestion/connections/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/metadata/ingestion/models/test_patch_request.py | Python test `test_patch_request.py` | High |
| ingestion/tests/unit/metadata/ingestion/models/test_table_constraints.py | Python test `test_table_constraints.py` | High |
| ingestion/tests/unit/metadata/ingestion/ometa/test_rest_client_headers.py | Python test `test_rest_client_headers.py` | High |
| ingestion/tests/unit/metadata/ingestion/ometa/test_sse_client.py | Python test `test_sse_client.py` | High |
| ingestion/tests/unit/metadata/ingestion/ometa/test_table_mixin.py | Python test `test_table_mixin.py` | High |
| ingestion/tests/unit/metadata/ingestion/ometa/test_task_announcement_feed_mixins.py | Python test `test_task_announcement_feed_mixins.py` | High |
| ingestion/tests/unit/metadata/ingestion/ometa/test_user_mixin.py | Python test `test_user_mixin.py` | High |
| ingestion/tests/unit/metadata/ingestion/source/database/snowflake/profiler/test_system_metrics.py | Python test `test_system_metrics.py` | High |
| ingestion/tests/unit/metadata/pii/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/metadata/pii/test_classification_manager.py | Python test `test_classification_manager.py` | High |
| ingestion/tests/unit/metadata/pii/test_conflict_resolver.py | Python test `test_conflict_resolver.py` | High |
| ingestion/tests/unit/metadata/pii/test_language_filtering.py | Python test `test_language_filtering.py` | High |
| ingestion/tests/unit/metadata/pii/test_pii_models.py | Python test `test_pii_models.py` | High |
| ingestion/tests/unit/metadata/pii/test_presidio_utils.py | Python test `test_presidio_utils.py` | High |
| ingestion/tests/unit/metadata/pii/test_tag_analyzer_any_language.py | Python test `test_tag_analyzer_any_language.py` | High |
| ingestion/tests/unit/metadata/pii/test_tag_processor.py | Python test `test_tag_processor.py` | High |
| ingestion/tests/unit/metadata/pii/test_tag_processor_integration.py | Python test `test_tag_processor_integration.py` | High |
| ingestion/tests/unit/metadata/pii/test_tag_scoring.py | Python test `test_tag_scoring.py` | High |
| ingestion/tests/unit/metadata/profiler/api/test_models.py | Python test `test_models.py` | High |
| ingestion/tests/unit/metadata/utils/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/metadata/utils/dependency_injector/test_dependency_injector.py | Python test `test_dependency_injector.py` | High |
| ingestion/tests/unit/metadata/utils/secrets/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/metadata/utils/secrets/test_kubernetes_secrets_manager.py | Python test `test_kubernetes_secrets_manager.py` | High |
| ingestion/tests/unit/metadata/utils/secrets/test_secrets_manager_factory.py | Python test `test_secrets_manager_factory.py` | High |
| ingestion/tests/unit/metadata/utils/test_class_helper.py | Python test `test_class_helper.py` | High |
| ingestion/tests/unit/metadata/utils/test_entity_link.py | Python test `test_entity_link.py` | High |
| ingestion/tests/unit/metadata/utils/test_lru_cache.py | Python test `test_lru_cache.py` | High |
| ingestion/tests/unit/metadata/utils/test_operation_metrics.py | Python test `test_operation_metrics.py` | High |
| ingestion/tests/unit/metadata/utils/test_progress_tracker.py | Python test `test_progress_tracker.py` | High |
| ingestion/tests/unit/metadata/utils/test_time_utils.py | Python test `test_time_utils.py` | High |
| ingestion/tests/unit/models/test_custom_basemodel_validation.py | Python test `test_custom_basemodel_validation.py` | High |
| ingestion/tests/unit/models/test_custom_pydantic.py | Python test `test_custom_pydantic.py` | High |
| ingestion/tests/unit/observability/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/observability/data_quality/processor/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/processor/test_test_case_runner.py | Python test `test_test_case_runner.py` | High |
| ingestion/tests/unit/observability/data_quality/source/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/source/test_test_suite.py | Python test `test_test_suite.py` | High |
| ingestion/tests/unit/observability/data_quality/test_column_value_to_at_location.py | Python test `test_column_value_to_at_location.py` | High |
| ingestion/tests/unit/observability/data_quality/test_impact_scoring.py | Python test `test_impact_scoring.py` | High |
| ingestion/tests/unit/observability/data_quality/test_metric_registry.py | Python test `test_metric_registry.py` | High |
| ingestion/tests/unit/observability/data_quality/test_validations_databases.py | Python test `test_validations_databases.py` | High |
| ingestion/tests/unit/observability/data_quality/test_validations_datalake.py | Python test `test_validations_datalake.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/runtime_param_setter/test_base_diff_params_setter.py | Python test `test_base_diff_params_setter.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/runtime_param_setter/test_table_diff_params_setter.py | Python test `test_table_diff_params_setter.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/table/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/table/sqlalchemy/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/table/sqlalchemy/test_table_diff.py | Python test `test_table_diff.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_base_handler.py | Python test `test_base_handler.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_passed_failed_row_calculation.py | Python test `test_passed_failed_row_calculation.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_row_count_logic_with_total.py | Python test `test_row_count_logic_with_total.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_rule_library_sql_expression_validator.py | Python test `test_rule_library_sql_expression_validator.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_table_custom_sql_query.py | Python test `test_table_custom_sql_query.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_table_custom_sql_query_consent.py | Python test `test_table_custom_sql_query_consent.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_table_custom_sql_query_row_counts.py | Python test `test_table_custom_sql_query_row_counts.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_utils.py | Python test `test_utils.py` | High |
| ingestion/tests/unit/observability/data_quality/validations/test_zero_threshold_edge_cases.py | Python test `test_zero_threshold_edge_cases.py` | High |
| ingestion/tests/unit/observability/profiler/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/observability/profiler/custom_types/test_custom_hex_byte_string.py | Python test `test_custom_hex_byte_string.py` | High |
| ingestion/tests/unit/observability/profiler/custom_types/test_custom_types.py | Python test `test_custom_types.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_burstiq_profiler_interface.py | Python test `test_burstiq_profiler_interface.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_custom_metrics.py | Python test `test_custom_metrics.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_datalake_metrics.py | Python test `test_datalake_metrics.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_profiler_interface.py | Python test `test_profiler_interface.py` | High |
| ingestion/tests/unit/observability/profiler/pandas/test_sample.py | Python test `test_sample.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/athena/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/athena/test_visit_column.py | Python test `test_visit_column.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/azuresql/test_azuresql_sampling.py | Python test `test_azuresql_sampling.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/bigquery/test_bigquery_sampling.py | Python test `test_bigquery_sampling.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/bigquery/test_map_struct.py | Python test `test_map_struct.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/databricks/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/databricks/test_visit_column.py | Python test `test_visit_column.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/mssql/test_mssql_sampling.py | Python test `test_mssql_sampling.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/mysql/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/mysql/test_mysql_median.py | Python test `test_mysql_median.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/postgres/test_postgres_sampling.py | Python test `test_postgres_sampling.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/snowflake/test_query_log.py | Python test `test_query_log.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/snowflake/test_sampling_method.py | Python test `test_sampling_method.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/snowflake/test_snowflake_sampling.py | Python test `test_snowflake_sampling.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_inherited_metrics/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_inherited_metrics/test_metric_signatures.py | Python test `test_metric_signatures.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_metrics.py | Python test `test_metrics.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_profiler.py | Python test `test_profiler.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_runner.py | Python test `test_runner.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_sample.py | Python test `test_sample.py` | High |
| ingestion/tests/unit/observability/profiler/sqlalchemy/test_sqa_profiler_interface.py | Python test `test_sqa_profiler_interface.py` | High |
| ingestion/tests/unit/observability/profiler/test_container_fetcher.py | Python test `test_container_fetcher.py` | High |
| ingestion/tests/unit/observability/profiler/test_converter.py | Python test `test_converter.py` | High |
| ingestion/tests/unit/observability/profiler/test_entity_fetcher.py | Python test `test_entity_fetcher.py` | High |
| ingestion/tests/unit/observability/profiler/test_nosql_profiler_processor_status.py | Python test `test_nosql_profiler_processor_status.py` | High |
| ingestion/tests/unit/observability/profiler/test_profiler_interface.py | Python test `test_profiler_interface.py` | High |
| ingestion/tests/unit/observability/profiler/test_profiler_models.py | Python test `test_profiler_models.py` | High |
| ingestion/tests/unit/observability/profiler/test_profiler_partitions.py | Python test `test_profiler_partitions.py` | High |
| ingestion/tests/unit/observability/profiler/test_profiler_processor_status.py | Python test `test_profiler_processor_status.py` | High |
| ingestion/tests/unit/observability/profiler/test_sampling_config.py | Python test `test_sampling_config.py` | High |
| ingestion/tests/unit/observability/profiler/test_table_metric_computer.py | Python test `test_table_metric_computer.py` | High |
| ingestion/tests/unit/observability/profiler/test_utils.py | Python test `test_utils.py` | High |
| ingestion/tests/unit/observability/profiler/test_workflow.py | Python test `test_workflow.py` | High |
| ingestion/tests/unit/pii/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/pii/algorithms/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/pii/algorithms/conftest.py | Python test `conftest.py` | High |
| ingestion/tests/unit/pii/algorithms/data/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/pii/algorithms/data/pii_samples.py | Python test `pii_samples.py` | High |
| ingestion/tests/unit/pii/algorithms/test_classifiers.py | Python test `test_classifiers.py` | High |
| ingestion/tests/unit/pii/algorithms/test_feature_extraction.py | Python test `test_feature_extraction.py` | High |
| ingestion/tests/unit/pii/algorithms/test_preprocessing.py | Python test `test_preprocessing.py` | High |
| ingestion/tests/unit/pii/algorithms/test_presidio_recognizer_factory.py | Python test `test_presidio_recognizer_factory.py` | High |
| ingestion/tests/unit/pii/algorithms/test_presidio_utils.py | Python test `test_presidio_utils.py` | High |
| ingestion/tests/unit/pii/test_cases/azuresql_temporal_table.py | Python test `azuresql_temporal_table.py` | High |
| ingestion/tests/unit/pii/test_cases/credit_cards.py | Python test `credit_cards.py` | High |
| ingestion/tests/unit/pii/test_cases/customers_sensitive.py | Python test `customers_sensitive.py` | High |
| ingestion/tests/unit/pii/test_cases/demo_meetup_dbt_jaffle_customers.py | Python test `demo_meetup_dbt_jaffle_customers.py` | High |
| ingestion/tests/unit/pii/test_cases/demo_meetup_dbt_jaffle_orders.py | Python test `demo_meetup_dbt_jaffle_orders.py` | High |
| ingestion/tests/unit/pii/test_cases/timestamps_milliseconds_and_versions.py | Python test `timestamps_milliseconds_and_versions.py` | High |
| ingestion/tests/unit/pii/test_cases/timestamps_seconds_and_nhs_number.py | Python test `timestamps_seconds_and_nhs_number.py` | High |
| ingestion/tests/unit/pii/test_cases/users.py | Python test `users.py` | High |
| ingestion/tests/unit/pii/test_column_name_scanner.py | Python test `test_column_name_scanner.py` | High |
| ingestion/tests/unit/pii/test_ner_scanner.py | Python test `test_ner_scanner.py` | High |
| ingestion/tests/unit/pii/test_pii_sensitive.py | Python test `test_pii_sensitive.py` | High |
| ingestion/tests/unit/pii/test_processor.py | Python test `test_processor.py` | High |
| ingestion/tests/unit/readers/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/readers/test_avro_reader.py | Python test `test_avro_reader.py` | High |
| ingestion/tests/unit/readers/test_credentials.py | Python test `test_credentials.py` | High |
| ingestion/tests/unit/readers/test_df_reader.py | Python test `test_df_reader.py` | High |
| ingestion/tests/unit/readers/test_dsv_reader.py | Python test `test_dsv_reader.py` | High |
| ingestion/tests/unit/readers/test_github_reader.py | Python test `test_github_reader.py` | High |
| ingestion/tests/unit/readers/test_json_reader.py | Python test `test_json_reader.py` | High |
| ingestion/tests/unit/readers/test_parquet_azure_reader.py | Python test `test_parquet_azure_reader.py` | High |
| ingestion/tests/unit/readers/test_parquet_batches.py | Python test `test_parquet_batches.py` | High |
| ingestion/tests/unit/readers/test_parquet_reader.py | Python test `test_parquet_reader.py` | High |
| ingestion/tests/unit/readers/test_s3_n_plus_one.py | Python test `test_s3_n_plus_one.py` | High |
| ingestion/tests/unit/readers/test_s3_reader_credentials.py | Python test `test_s3_reader_credentials.py` | High |
| ingestion/tests/unit/sampler/pandas/test_pandas_truncation.py | Python test `test_pandas_truncation.py` | High |
| ingestion/tests/unit/sampler/sqlalchemy/test_sqlalchemy_truncation.py | Python test `test_sqlalchemy_truncation.py` | High |
| ingestion/tests/unit/sampler/sqlalchemy/test_timescale_sampler.py | Python test `test_timescale_sampler.py` | High |
| ingestion/tests/unit/sampler/sqlalchemy/test_unitycatalog_sampler.py | Python test `test_unitycatalog_sampler.py` | High |
| ingestion/tests/unit/sampler/test_container_sampler_processor.py | Python test `test_container_sampler_processor.py` | High |
| ingestion/tests/unit/sampler/test_sample_config.py | Python test `test_sample_config.py` | High |
| ingestion/tests/unit/sampler/test_sampler_100_pct.py | Python test `test_sampler_100_pct.py` | High |
| ingestion/tests/unit/sampler/test_sampler_interface.py | Python test `test_sampler_interface.py` | High |
| ingestion/tests/unit/sdk/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/sdk/data_quality/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/sdk/data_quality/test_dataframe_validator.py | Python test `test_dataframe_validator.py` | High |
| ingestion/tests/unit/sdk/data_quality/test_dq_runner.py | Python test `test_dq_runner.py` | High |
| ingestion/tests/unit/sdk/data_quality/test_result_capturing_processor.py | Python test `test_result_capturing_processor.py` | High |
| ingestion/tests/unit/sdk/data_quality/test_validation_results.py | Python test `test_validation_results.py` | High |
| ingestion/tests/unit/sdk/data_quality/test_workflow_config_builder.py | Python test `test_workflow_config_builder.py` | High |
| ingestion/tests/unit/sdk/test_api_collection_entity.py | Python test `test_api_collection_entity.py` | High |
| ingestion/tests/unit/sdk/test_api_endpoint_entity.py | Python test `test_api_endpoint_entity.py` | High |
| ingestion/tests/unit/sdk/test_base_entity.py | Python test `test_base_entity.py` | High |
| ingestion/tests/unit/sdk/test_chart_entity.py | Python test `test_chart_entity.py` | High |
| ingestion/tests/unit/sdk/test_classification_entity.py | Python test `test_classification_entity.py` | High |
| ingestion/tests/unit/sdk/test_config.py | Python test `test_config.py` | High |
| ingestion/tests/unit/sdk/test_container_entity.py | Python test `test_container_entity.py` | High |
| ingestion/tests/unit/sdk/test_csv_mixin.py | Python test `test_csv_mixin.py` | High |
| ingestion/tests/unit/sdk/test_csv_operations.py | Python test `test_csv_operations.py` | High |
| ingestion/tests/unit/sdk/test_custom_properties.py | Python test `test_custom_properties.py` | High |
| ingestion/tests/unit/sdk/test_dashboard_data_model_entity.py | Python test `test_dashboard_data_model_entity.py` | High |
| ingestion/tests/unit/sdk/test_dashboard_entity.py | Python test `test_dashboard_entity.py` | High |
| ingestion/tests/unit/sdk/test_data_product_entity.py | Python test `test_data_product_entity.py` | High |
| ingestion/tests/unit/sdk/test_database_entity.py | Python test `test_database_entity.py` | High |
| ingestion/tests/unit/sdk/test_database_schema_entity.py | Python test `test_database_schema_entity.py` | High |
| ingestion/tests/unit/sdk/test_domain_entity.py | Python test `test_domain_entity.py` | High |
| ingestion/tests/unit/sdk/test_glossary_entity.py | Python test `test_glossary_entity.py` | High |
| ingestion/tests/unit/sdk/test_improved_entities.py | Python test `test_improved_entities.py` | High |
| ingestion/tests/unit/sdk/test_metric_entity.py | Python test `test_metric_entity.py` | High |
| ingestion/tests/unit/sdk/test_pipeline_entity.py | Python test `test_pipeline_entity.py` | High |
| ingestion/tests/unit/sdk/test_query_entity.py | Python test `test_query_entity.py` | High |
| ingestion/tests/unit/sdk/test_restore_async.py | Python test `test_restore_async.py` | High |
| ingestion/tests/unit/sdk/test_sdk_apis.py | Python test `test_sdk_apis.py` | High |
| ingestion/tests/unit/sdk/test_sdk_entities.py | Python test `test_sdk_entities.py` | High |
| ingestion/tests/unit/sdk/test_sdk_fluent_api.py | Python test `test_sdk_fluent_api.py` | High |
| ingestion/tests/unit/sdk/test_sdk_plural_entities.py | Python test `test_sdk_plural_entities.py` | High |
| ingestion/tests/unit/sdk/test_search_index_entity.py | Python test `test_search_index_entity.py` | High |
| ingestion/tests/unit/sdk/test_stored_procedure_entity.py | Python test `test_stored_procedure_entity.py` | High |
| ingestion/tests/unit/sdk/test_table_entity.py | Python test `test_table_entity.py` | High |
| ingestion/tests/unit/sdk/test_tag_entity.py | Python test `test_tag_entity.py` | High |
| ingestion/tests/unit/sdk/test_team_entity.py | Python test `test_team_entity.py` | High |
| ingestion/tests/unit/sdk/test_to_entity_reference.py | Python test `test_to_entity_reference.py` | High |
| ingestion/tests/unit/sdk/test_types.py | Python test `test_types.py` | High |
| ingestion/tests/unit/sdk/test_user_entity.py | Python test `test_user_entity.py` | High |
| ingestion/tests/unit/source/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/api/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/api/rest/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/api/rest/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/qlikcloud/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/qlikcloud/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/qliksense/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/qliksense/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/quicksight/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/quicksight/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/redash/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/redash/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/sigma/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/sigma/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/ssrs/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/ssrs/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/superset/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/superset/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/dashboard/tableau/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/dashboard/tableau/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/athena/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/athena/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/azuresql/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/azuresql/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/bigtable/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/bigtable/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/burstiq/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/burstiq/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/cassandra/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/cassandra/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/clickhouse/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/clickhouse/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/cockroach/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/cockroach/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/couchbase/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/couchbase/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/db2/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/db2/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/deltalake/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/deltalake/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/domodatabase/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/domodatabase/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/doris/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/doris/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/druid/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/druid/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/dynamodb/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/dynamodb/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/exasol/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/exasol/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/glue/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/glue/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/greenplum/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/greenplum/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/hive/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/hive/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/impala/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/impala/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/iomete/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/iomete/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/mariadb/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/mariadb/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/mongodb/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/mongodb/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/mssql/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/mssql/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/mysql/test_mysql_connection.py | Python test `test_mysql_connection.py` | High |
| ingestion/tests/unit/source/database/pinotdb/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/pinotdb/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/postgres/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/postgres/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/presto/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/presto/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/redshift/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/redshift/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/salesforce/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/salesforce/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/saperp/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/saperp/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/saphana/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/saphana/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/sas/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/sas/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/singlestore/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/singlestore/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/sqlite/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/sqlite/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/starrocks/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/starrocks/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/teradata/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/teradata/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/test_json_schema_extractor.py | Python test `test_json_schema_extractor.py` | High |
| ingestion/tests/unit/source/database/test_mysql_cloudsql.py | Python test `test_mysql_cloudsql.py` | High |
| ingestion/tests/unit/source/database/test_mysql_iam.py | Python test `test_mysql_iam.py` | High |
| ingestion/tests/unit/source/database/timescale/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/timescale/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/trino/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/trino/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/trino/test_lineage.py | Python test `test_lineage.py` | High |
| ingestion/tests/unit/source/database/unitycatalog/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/unitycatalog/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/database/vertica/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/database/vertica/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/drive/googledrive/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/drive/googledrive/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/drive/sftp/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/drive/sftp/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/mcp/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/mcp/test_mcp_client.py | Python test `test_mcp_client.py` | High |
| ingestion/tests/unit/source/mcp/test_mcp_connection.py | Python test `test_mcp_connection.py` | High |
| ingestion/tests/unit/source/mcp/test_mcp_metadata.py | Python test `test_mcp_metadata.py` | High |
| ingestion/tests/unit/source/messaging/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/messaging/kafka/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/messaging/kafka/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/messaging/kinesis/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/messaging/kinesis/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/messaging/pubsub/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/messaging/pubsub/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/messaging/redpanda/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/messaging/redpanda/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/messaging/test_common_broker_source.py | Python test `test_common_broker_source.py` | High |
| ingestion/tests/unit/source/messaging/test_global_sample_data_config.py | Python test `test_global_sample_data_config.py` | High |
| ingestion/tests/unit/source/messaging/test_pubsub.py | Python test `test_pubsub.py` | High |
| ingestion/tests/unit/source/metadata/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/metadata/alationsink/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/metadata/alationsink/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/metadata/amundsen/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/metadata/amundsen/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/metadata/atlas/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/metadata/atlas/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/mlmodel/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/mlmodel/mlflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/mlmodel/mlflow/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/mlmodel/sagemaker/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/mlmodel/sagemaker/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/airbyte/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/airbyte/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/airflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/airflow/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/dagster/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/dagster/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/databrickspipeline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/databrickspipeline/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/dbtcloud/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/dbtcloud/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/domopipeline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/domopipeline/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/fivetran/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/fivetran/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/flink/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/flink/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/gluepipeline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/gluepipeline/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/kafkaconnect/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/kafkaconnect/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/nifi/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/nifi/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/openlineage/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/openlineage/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/spline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/pipeline/spline/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/pipeline/test_kafkaconnect.py | Python test `test_kafkaconnect.py` | High |
| ingestion/tests/unit/source/search/elasticsearch/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/search/elasticsearch/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/search/opensearch/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/search/opensearch/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/storage/gcs/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/storage/gcs/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/source/storage/s3/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/source/storage/s3/test_connection.py | Python test `test_connection.py` | High |
| ingestion/tests/unit/test_avro_parser.py | Python test `test_avro_parser.py` | High |
| ingestion/tests/unit/test_azure_credentials.py | Python test `test_azure_credentials.py` | High |
| ingestion/tests/unit/test_build_connection_url.py | Python test `test_build_connection_url.py` | High |
| ingestion/tests/unit/test_column_type_parser.py | Python test `test_column_type_parser.py` | High |
| ingestion/tests/unit/test_config.py | Python test `test_config.py` | High |
| ingestion/tests/unit/test_connection_builders.py | Python test `test_connection_builders.py` | High |
| ingestion/tests/unit/test_credentials.py | Python test `test_credentials.py` | High |
| ingestion/tests/unit/test_data_insight_chart_imports.py | Python test `test_data_insight_chart_imports.py` | High |
| ingestion/tests/unit/test_datatypes.py | Python test `test_datatypes.py` | High |
| ingestion/tests/unit/test_db_utils.py | Python test `test_db_utils.py` | High |
| ingestion/tests/unit/test_dbt.py | Python test `test_dbt.py` | High |
| ingestion/tests/unit/test_dbt_http_config.py | Python test `test_dbt_http_config.py` | High |
| ingestion/tests/unit/test_dbt_ingest.py | Python test `test_dbt_ingest.py` | High |
| ingestion/tests/unit/test_dbt_test_definition.py | Python test `test_dbt_test_definition.py` | High |
| ingestion/tests/unit/test_entity_link.py | Python test `test_entity_link.py` | High |
| ingestion/tests/unit/test_exit_handler.py | Python test `test_exit_handler.py` | High |
| ingestion/tests/unit/test_filter_pattern.py | Python test `test_filter_pattern.py` | High |
| ingestion/tests/unit/test_fqn.py | Python test `test_fqn.py` | High |
| ingestion/tests/unit/test_group_entities_by_type_ordering.py | Python test `test_group_entities_by_type_ordering.py` | High |
| ingestion/tests/unit/test_handle_partitions.py | Python test `test_handle_partitions.py` | High |
| ingestion/tests/unit/test_helpers.py | Python test `test_helpers.py` | High |
| ingestion/tests/unit/test_importer.py | Python test `test_importer.py` | High |
| ingestion/tests/unit/test_incremental_extraction.py | Python test `test_incremental_extraction.py` | High |
| ingestion/tests/unit/test_json_schema_parser.py | Python test `test_json_schema_parser.py` | High |
| ingestion/tests/unit/test_lineage_empty_result.py | Python test `test_lineage_empty_result.py` | High |
| ingestion/tests/unit/test_logger.py | Python test `test_logger.py` | High |
| ingestion/tests/unit/test_mf4_reader.py | Python test `test_mf4_reader.py` | High |
| ingestion/tests/unit/test_ometa_client_resilience.py | Python test `test_ometa_client_resilience.py` | High |
| ingestion/tests/unit/test_ometa_endpoints.py | Python test `test_ometa_endpoints.py` | High |
| ingestion/tests/unit/test_ometa_http_adapter.py | Python test `test_ometa_http_adapter.py` | High |
| ingestion/tests/unit/test_ometa_mlmodel.py | Python test `test_ometa_mlmodel.py` | High |
| ingestion/tests/unit/test_ometa_restore.py | Python test `test_ometa_restore.py` | High |
| ingestion/tests/unit/test_ometa_to_dataframe.py | Python test `test_ometa_to_dataframe.py` | High |
| ingestion/tests/unit/test_ometa_utils.py | Python test `test_ometa_utils.py` | High |
| ingestion/tests/unit/test_owner_config.py | Python test `test_owner_config.py` | High |
| ingestion/tests/unit/test_owner_utils.py | Python test `test_owner_utils.py` | High |
| ingestion/tests/unit/test_parser_connection_class.py | Python test `test_parser_connection_class.py` | High |
| ingestion/tests/unit/test_parser_connection_fallback.py | Python test `test_parser_connection_fallback.py` | High |
| ingestion/tests/unit/test_parser_connection_module.py | Python test `test_parser_connection_module.py` | High |
| ingestion/tests/unit/test_partition.py | Python test `test_partition.py` | High |
| ingestion/tests/unit/test_path_pattern.py | Python test `test_path_pattern.py` | High |
| ingestion/tests/unit/test_powerbi_filter_query.py | Python test `test_powerbi_filter_query.py` | High |
| ingestion/tests/unit/test_powerbi_table_measures.py | Python test `test_powerbi_table_measures.py` | High |
| ingestion/tests/unit/test_protobuf_parser.py | Python test `test_protobuf_parser.py` | High |
| ingestion/tests/unit/test_pydantic_v2.py | Python test `test_pydantic_v2.py` | High |
| ingestion/tests/unit/test_query_parser.py | Python test `test_query_parser.py` | High |
| ingestion/tests/unit/test_root_model_defaults.py | Python test `test_root_model_defaults.py` | High |
| ingestion/tests/unit/test_scaffold.py | Python test `test_scaffold.py` | High |
| ingestion/tests/unit/test_sink_barrier.py | Python test `test_sink_barrier.py` | High |
| ingestion/tests/unit/test_sink_buffer_on_flush_failure.py | Python test `test_sink_buffer_on_flush_failure.py` | High |
| ingestion/tests/unit/test_sink_bulk_conflict_warning.py | Python test `test_sink_bulk_conflict_warning.py` | High |
| ingestion/tests/unit/test_sink_deduplication.py | Python test `test_sink_deduplication.py` | High |
| ingestion/tests/unit/test_sink_empty_tag_validation.py | Python test `test_sink_empty_tag_validation.py` | High |
| ingestion/tests/unit/test_source_connection.py | Python test `test_source_connection.py` | High |
| ingestion/tests/unit/test_source_parsing.py | Python test `test_source_parsing.py` | High |
| ingestion/tests/unit/test_source_url.py | Python test `test_source_url.py` | High |
| ingestion/tests/unit/test_ssl_manager.py | Python test `test_ssl_manager.py` | High |
| ingestion/tests/unit/test_status.py | Python test `test_status.py` | High |
| ingestion/tests/unit/test_time_utils.py | Python test `test_time_utils.py` | High |
| ingestion/tests/unit/test_trino_complex_types.py | Python test `test_trino_complex_types.py` | High |
| ingestion/tests/unit/test_trino_connection_ssl_verify.py | Python test `test_trino_connection_ssl_verify.py` | High |
| ingestion/tests/unit/test_ttl_cache.py | Python test `test_ttl_cache.py` | High |
| ingestion/tests/unit/test_usage_filter.py | Python test `test_usage_filter.py` | High |
| ingestion/tests/unit/test_usage_log.py | Python test `test_usage_log.py` | High |
| ingestion/tests/unit/test_user_agent.py | Python test `test_user_agent.py` | High |
| ingestion/tests/unit/test_version.py | Python test `test_version.py` | High |
| ingestion/tests/unit/test_workflow_parse.py | Python test `test_workflow_parse.py` | High |
| ingestion/tests/unit/test_workflow_parse_example_config.py | Python test `test_workflow_parse_example_config.py` | High |
| ingestion/tests/unit/topology/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/api/test_openapi_parser.py | Python test `test_openapi_parser.py` | High |
| ingestion/tests/unit/topology/api/test_rest.py | Python test `test_rest.py` | High |
| ingestion/tests/unit/topology/dashboard/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/dashboard/_test_superset.py | Python test `_test_superset.py` | High |
| ingestion/tests/unit/topology/dashboard/fixtures/grafana_fixtures.py | Python test `grafana_fixtures.py` | High |
| ingestion/tests/unit/topology/dashboard/test_domodashboard.py | Python test `test_domodashboard.py` | High |
| ingestion/tests/unit/topology/dashboard/test_grafana.py | Python test `test_grafana.py` | High |
| ingestion/tests/unit/topology/dashboard/test_grafana_client.py | Python test `test_grafana_client.py` | High |
| ingestion/tests/unit/topology/dashboard/test_grafana_simple.py | Python test `test_grafana_simple.py` | High |
| ingestion/tests/unit/topology/dashboard/test_hex.py | Python test `test_hex.py` | High |
| ingestion/tests/unit/topology/dashboard/test_hex_client.py | Python test `test_hex_client.py` | High |
| ingestion/tests/unit/topology/dashboard/test_hex_ingestion_flow.py | Python test `test_hex_ingestion_flow.py` | High |
| ingestion/tests/unit/topology/dashboard/test_hex_lineage.py | Python test `test_hex_lineage.py` | High |
| ingestion/tests/unit/topology/dashboard/test_lightdash_client.py | Python test `test_lightdash_client.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker.py | Python test `test_looker.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_chart_lineage.py | Python test `test_looker_chart_lineage.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_extends_lineage.py | Python test `test_looker_extends_lineage.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_lkml_parser.py | Python test `test_looker_lkml_parser.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_local_repo.py | Python test `test_looker_local_repo.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_multi_repo.py | Python test `test_looker_multi_repo.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_standalone_views.py | Python test `test_looker_standalone_views.py` | High |
| ingestion/tests/unit/topology/dashboard/test_looker_utils.py | Python test `test_looker_utils.py` | High |
| ingestion/tests/unit/topology/dashboard/test_lookml_bitbucket_reader.py | Python test `test_lookml_bitbucket_reader.py` | High |
| ingestion/tests/unit/topology/dashboard/test_lookml_github_reader.py | Python test `test_lookml_github_reader.py` | High |
| ingestion/tests/unit/topology/dashboard/test_lookml_gitlab_reader.py | Python test `test_lookml_gitlab_reader.py` | High |
| ingestion/tests/unit/topology/dashboard/test_metabase.py | Python test `test_metabase.py` | High |
| ingestion/tests/unit/topology/dashboard/test_microstrategy.py | Python test `test_microstrategy.py` | High |
| ingestion/tests/unit/topology/dashboard/test_powerbi.py | Python test `test_powerbi.py` | High |
| ingestion/tests/unit/topology/dashboard/test_powerbi_resilience.py | Python test `test_powerbi_resilience.py` | High |
| ingestion/tests/unit/topology/dashboard/test_powerbi_workspace_state.py | Python test `test_powerbi_workspace_state.py` | High |
| ingestion/tests/unit/topology/dashboard/test_qlikcloud.py | Python test `test_qlikcloud.py` | High |
| ingestion/tests/unit/topology/dashboard/test_qliksense.py | Python test `test_qliksense.py` | High |
| ingestion/tests/unit/topology/dashboard/test_quicksight.py | Python test `test_quicksight.py` | High |
| ingestion/tests/unit/topology/dashboard/test_sigma.py | Python test `test_sigma.py` | High |
| ingestion/tests/unit/topology/dashboard/test_sigma_client.py | Python test `test_sigma_client.py` | High |
| ingestion/tests/unit/topology/dashboard/test_ssrs.py | Python test `test_ssrs.py` | High |
| ingestion/tests/unit/topology/dashboard/test_ssrs_rdl_parser.py | Python test `test_ssrs_rdl_parser.py` | High |
| ingestion/tests/unit/topology/dashboard/test_tableau.py | Python test `test_tableau.py` | High |
| ingestion/tests/unit/topology/dashboard/test_tableau_client.py | Python test `test_tableau_client.py` | High |
| ingestion/tests/unit/topology/database/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/database/test_athena.py | Python test `test_athena.py` | High |
| ingestion/tests/unit/topology/database/test_athena_utils.py | Python test `test_athena_utils.py` | High |
| ingestion/tests/unit/topology/database/test_bigquery.py | Python test `test_bigquery.py` | High |
| ingestion/tests/unit/topology/database/test_bigquery_incremental_table_processor.py | Python test `test_bigquery_incremental_table_processor.py` | High |
| ingestion/tests/unit/topology/database/test_bigtable.py | Python test `test_bigtable.py` | High |
| ingestion/tests/unit/topology/database/test_burstiq_client.py | Python test `test_burstiq_client.py` | High |
| ingestion/tests/unit/topology/database/test_burstiq_connection.py | Python test `test_burstiq_connection.py` | High |
| ingestion/tests/unit/topology/database/test_burstiq_lineage.py | Python test `test_burstiq_lineage.py` | High |
| ingestion/tests/unit/topology/database/test_burstiq_metadata.py | Python test `test_burstiq_metadata.py` | High |
| ingestion/tests/unit/topology/database/test_burstiq_sampler.py | Python test `test_burstiq_sampler.py` | High |
| ingestion/tests/unit/topology/database/test_cassandra.py | Python test `test_cassandra.py` | High |
| ingestion/tests/unit/topology/database/test_clickhouse_utils.py | Python test `test_clickhouse_utils.py` | High |
| ingestion/tests/unit/topology/database/test_cockroach.py | Python test `test_cockroach.py` | High |
| ingestion/tests/unit/topology/database/test_common_db_source.py | Python test `test_common_db_source.py` | High |
| ingestion/tests/unit/topology/database/test_couchbase.py | Python test `test_couchbase.py` | High |
| ingestion/tests/unit/topology/database/test_databricks.py | Python test `test_databricks.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_get_columns.py | Python test `test_databricks_get_columns.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_log_filters.py | Python test `test_databricks_log_filters.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_migration.py | Python test `test_databricks_migration.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_nested_comments.py | Python test `test_databricks_nested_comments.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_ordinal_position.py | Python test `test_databricks_ordinal_position.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_ownership.py | Python test `test_databricks_ownership.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_query_reduction.py | Python test `test_databricks_query_reduction.py` | High |
| ingestion/tests/unit/topology/database/test_databricks_valueless_tags.py | Python test `test_databricks_valueless_tags.py` | High |
| ingestion/tests/unit/topology/database/test_datalake.py | Python test `test_datalake.py` | High |
| ingestion/tests/unit/topology/database/test_datalake_azure_blob_client.py | Python test `test_datalake_azure_blob_client.py` | High |
| ingestion/tests/unit/topology/database/test_datalake_gcs_client.py | Python test `test_datalake_gcs_client.py` | High |
| ingestion/tests/unit/topology/database/test_datalake_s3_client.py | Python test `test_datalake_s3_client.py` | High |
| ingestion/tests/unit/topology/database/test_db2.py | Python test `test_db2.py` | High |
| ingestion/tests/unit/topology/database/test_deltalake.py | Python test `test_deltalake.py` | High |
| ingestion/tests/unit/topology/database/test_domodatabase.py | Python test `test_domodatabase.py` | High |
| ingestion/tests/unit/topology/database/test_doris.py | Python test `test_doris.py` | High |
| ingestion/tests/unit/topology/database/test_exasol.py | Python test `test_exasol.py` | High |
| ingestion/tests/unit/topology/database/test_filter_invalid_constraints.py | Python test `test_filter_invalid_constraints.py` | High |
| ingestion/tests/unit/topology/database/test_glue.py | Python test `test_glue.py` | High |
| ingestion/tests/unit/topology/database/test_greenplum.py | Python test `test_greenplum.py` | High |
| ingestion/tests/unit/topology/database/test_hive.py | Python test `test_hive.py` | High |
| ingestion/tests/unit/topology/database/test_hive_metastore_mysql_dialect.py | Python test `test_hive_metastore_mysql_dialect.py` | High |
| ingestion/tests/unit/topology/database/test_hive_metastore_postgres_dialect.py | Python test `test_hive_metastore_postgres_dialect.py` | High |
| ingestion/tests/unit/topology/database/test_iomete.py | Python test `test_iomete.py` | High |
| ingestion/tests/unit/topology/database/test_mariadb.py | Python test `test_mariadb.py` | High |
| ingestion/tests/unit/topology/database/test_mongodb.py | Python test `test_mongodb.py` | High |
| ingestion/tests/unit/topology/database/test_mssql.py | Python test `test_mssql.py` | High |
| ingestion/tests/unit/topology/database/test_mysql.py | Python test `test_mysql.py` | High |
| ingestion/tests/unit/topology/database/test_mysql_query_parser.py | Python test `test_mysql_query_parser.py` | High |
| ingestion/tests/unit/topology/database/test_oracle.py | Python test `test_oracle.py` | High |
| ingestion/tests/unit/topology/database/test_pinotdb.py | Python test `test_pinotdb.py` | High |
| ingestion/tests/unit/topology/database/test_postgres.py | Python test `test_postgres.py` | High |
| ingestion/tests/unit/topology/database/test_presto.py | Python test `test_presto.py` | High |
| ingestion/tests/unit/topology/database/test_questdb.py | Python test `test_questdb.py` | High |
| ingestion/tests/unit/topology/database/test_redshift.py | Python test `test_redshift.py` | High |
| ingestion/tests/unit/topology/database/test_redshift_connection.py | Python test `test_redshift_connection.py` | High |
| ingestion/tests/unit/topology/database/test_redshift_incremental_table_processor.py | Python test `test_redshift_incremental_table_processor.py` | High |
| ingestion/tests/unit/topology/database/test_redshift_ordinal_position.py | Python test `test_redshift_ordinal_position.py` | High |
| ingestion/tests/unit/topology/database/test_redshift_serverless.py | Python test `test_redshift_serverless.py` | High |
| ingestion/tests/unit/topology/database/test_redshift_utils.py | Python test `test_redshift_utils.py` | High |
| ingestion/tests/unit/topology/database/test_salesforce.py | Python test `test_salesforce.py` | High |
| ingestion/tests/unit/topology/database/test_sap_hana.py | Python test `test_sap_hana.py` | High |
| ingestion/tests/unit/topology/database/test_saperp.py | Python test `test_saperp.py` | High |
| ingestion/tests/unit/topology/database/test_sas.py | Python test `test_sas.py` | High |
| ingestion/tests/unit/topology/database/test_snowflake.py | Python test `test_snowflake.py` | High |
| ingestion/tests/unit/topology/database/test_snowflake_access_history_lineage.py | Python test `test_snowflake_access_history_lineage.py` | High |
| ingestion/tests/unit/topology/database/test_snowflake_ordinal_position.py | Python test `test_snowflake_ordinal_position.py` | High |
| ingestion/tests/unit/topology/database/test_snowflake_schema_columns_lru.py | Python test `test_snowflake_schema_columns_lru.py` | High |
| ingestion/tests/unit/topology/database/test_snowflake_table_type_cache_pollution.py | Python test `test_snowflake_table_type_cache_pollution.py` | High |
| ingestion/tests/unit/topology/database/test_starrocks.py | Python test `test_starrocks.py` | High |
| ingestion/tests/unit/topology/database/test_teradata.py | Python test `test_teradata.py` | High |
| ingestion/tests/unit/topology/database/test_timescale.py | Python test `test_timescale.py` | High |
| ingestion/tests/unit/topology/database/test_trino_metadata.py | Python test `test_trino_metadata.py` | High |
| ingestion/tests/unit/topology/database/test_unity_catalog.py | Python test `test_unity_catalog.py` | High |
| ingestion/tests/unit/topology/database/test_unity_catalog_connection.py | Python test `test_unity_catalog_connection.py` | High |
| ingestion/tests/unit/topology/database/test_unity_catalog_lineage.py | Python test `test_unity_catalog_lineage.py` | High |
| ingestion/tests/unit/topology/database/test_unitycatalog_error_isolation.py | Python test `test_unitycatalog_error_isolation.py` | High |
| ingestion/tests/unit/topology/database/test_unitycatalog_incremental.py | Python test `test_unitycatalog_incremental.py` | High |
| ingestion/tests/unit/topology/database/test_unitycatalog_ordinal_position.py | Python test `test_unitycatalog_ordinal_position.py` | High |
| ingestion/tests/unit/topology/database/test_unitycatalog_valueless_tags.py | Python test `test_unitycatalog_valueless_tags.py` | High |
| ingestion/tests/unit/topology/database/test_vertica_type_mapping.py | Python test `test_vertica_type_mapping.py` | High |
| ingestion/tests/unit/topology/drive/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/drive/test_googledrive.py | Python test `test_googledrive.py` | High |
| ingestion/tests/unit/topology/drive/test_sftp.py | Python test `test_sftp.py` | High |
| ingestion/tests/unit/topology/mlmodel/test_sagemaker.py | Python test `test_sagemaker.py` | High |
| ingestion/tests/unit/topology/pipeline/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airbyte.py | Python test `test_airbyte.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airbyte_client.py | Python test `test_airbyte_client.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airflow.py | Python test `test_airflow.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airflow_connection.py | Python test `test_airflow_connection.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airflow_mwaa_client.py | Python test `test_airflow_mwaa_client.py` | High |
| ingestion/tests/unit/topology/pipeline/test_airflowapi.py | Python test `test_airflowapi.py` | High |
| ingestion/tests/unit/topology/pipeline/test_dagster.py | Python test `test_dagster.py` | High |
| ingestion/tests/unit/topology/pipeline/test_databricks_kafka_lineage.py | Python test `test_databricks_kafka_lineage.py` | High |
| ingestion/tests/unit/topology/pipeline/test_databricks_kafka_parser.py | Python test `test_databricks_kafka_parser.py` | High |
| ingestion/tests/unit/topology/pipeline/test_databricks_pipeline.py | Python test `test_databricks_pipeline.py` | High |
| ingestion/tests/unit/topology/pipeline/test_dbtcloud.py | Python test `test_dbtcloud.py` | High |
| ingestion/tests/unit/topology/pipeline/test_domopipeline.py | Python test `test_domopipeline.py` | High |
| ingestion/tests/unit/topology/pipeline/test_fivetran.py | Python test `test_fivetran.py` | High |
| ingestion/tests/unit/topology/pipeline/test_flink.py | Python test `test_flink.py` | High |
| ingestion/tests/unit/topology/pipeline/test_glue_script_parser.py | Python test `test_glue_script_parser.py` | High |
| ingestion/tests/unit/topology/pipeline/test_gluepipeline.py | Python test `test_gluepipeline.py` | High |
| ingestion/tests/unit/topology/pipeline/test_kafkaconnect.py | Python test `test_kafkaconnect.py` | High |
| ingestion/tests/unit/topology/pipeline/test_kafkaconnect_service_discovery.py | Python test `test_kafkaconnect_service_discovery.py` | High |
| ingestion/tests/unit/topology/pipeline/test_nifi.py | Python test `test_nifi.py` | High |
| ingestion/tests/unit/topology/pipeline/test_openlineage.py | Python test `test_openlineage.py` | High |
| ingestion/tests/unit/topology/pipeline/test_openlineage_ownership.py | Python test `test_openlineage_ownership.py` | High |
| ingestion/tests/unit/topology/pipeline/test_service_resolver.py | Python test `test_service_resolver.py` | High |
| ingestion/tests/unit/topology/pipeline/test_spline.py | Python test `test_spline.py` | High |
| ingestion/tests/unit/topology/search/test_elasticsearch.py | Python test `test_elasticsearch.py` | High |
| ingestion/tests/unit/topology/search/test_opensearch.py | Python test `test_opensearch.py` | High |
| ingestion/tests/unit/topology/storage/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/topology/storage/test_archive.py | Python test `test_archive.py` | High |
| ingestion/tests/unit/topology/storage/test_gcs_archive.py | Python test `test_gcs_archive.py` | High |
| ingestion/tests/unit/topology/storage/test_gcs_connection.py | Python test `test_gcs_connection.py` | High |
| ingestion/tests/unit/topology/storage/test_gcs_storage.py | Python test `test_gcs_storage.py` | High |
| ingestion/tests/unit/topology/storage/test_gcs_unstructured.py | Python test `test_gcs_unstructured.py` | High |
| ingestion/tests/unit/topology/storage/test_manifest_wildcards.py | Python test `test_manifest_wildcards.py` | High |
| ingestion/tests/unit/topology/storage/test_s3_archive.py | Python test `test_s3_archive.py` | High |
| ingestion/tests/unit/topology/storage/test_s3_methods.py | Python test `test_s3_methods.py` | High |
| ingestion/tests/unit/topology/storage/test_s3_storage.py | Python test `test_s3_storage.py` | High |
| ingestion/tests/unit/topology/test_common_db_source_isolation.py | Python test `test_common_db_source_isolation.py` | High |
| ingestion/tests/unit/topology/test_context.py | Python test `test_context.py` | High |
| ingestion/tests/unit/topology/test_context_manager.py | Python test `test_context_manager.py` | High |
| ingestion/tests/unit/topology/test_delete_stale.py | Python test `test_delete_stale.py` | High |
| ingestion/tests/unit/topology/test_queue.py | Python test `test_queue.py` | High |
| ingestion/tests/unit/topology/test_runner.py | Python test `test_runner.py` | High |
| ingestion/tests/unit/topology/test_sqa_utils.py | Python test `test_sqa_utils.py` | High |
| ingestion/tests/unit/utils/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/utils/test_collaborative_super.py | Python test `test_collaborative_super.py` | High |
| ingestion/tests/unit/utils/test_credentials.py | Python test `test_credentials.py` | High |
| ingestion/tests/unit/utils/test_datalake.py | Python test `test_datalake.py` | High |
| ingestion/tests/unit/utils/test_deprecation.py | Python test `test_deprecation.py` | High |
| ingestion/tests/unit/utils/test_fqn_special_chars.py | Python test `test_fqn_special_chars.py` | High |
| ingestion/tests/unit/utils/test_helpers.py | Python test `test_helpers.py` | High |
| ingestion/tests/unit/utils/test_logger.py | Python test `test_logger.py` | High |
| ingestion/tests/unit/utils/test_memory_limit.py | Python test `test_memory_limit.py` | High |
| ingestion/tests/unit/utils/test_service_spec.py | Python test `test_service_spec.py` | High |
| ingestion/tests/unit/utils/test_source_hash.py | Python test `test_source_hash.py` | High |
| ingestion/tests/unit/utils/test_status_warning_handler.py | Python test `test_status_warning_handler.py` | High |
| ingestion/tests/unit/utils/test_stored_procedures.py | Python test `test_stored_procedures.py` | High |
| ingestion/tests/unit/utils/test_streamable_logger.py | Python test `test_streamable_logger.py` | High |
| ingestion/tests/unit/utils/test_tag_utils.py | Python test `test_tag_utils.py` | High |
| ingestion/tests/unit/workflow/__init__.py | Python test `__init__.py` | High |
| ingestion/tests/unit/workflow/test_application_workflow.py | Python test `test_application_workflow.py` | High |
| ingestion/tests/unit/workflow/test_base_workflow.py | Python test `test_base_workflow.py` | High |
| ingestion/tests/unit/workflow/test_context_manager.py | Python test `test_context_manager.py` | High |
| ingestion/tests/unit/workflow/test_deprecated_workflow_functions.py | Python test `test_deprecated_workflow_functions.py` | High |
| ingestion/tests/utils/docker_service_builders/abstract_test_container.py | Python test `abstract_test_container.py` | High |
| ingestion/tests/utils/docker_service_builders/database_container/database_test_container.py | Python test `database_test_container.py` | High |
| ingestion/tests/utils/docker_service_builders/database_container/mysql_test_container.py | Python test `mysql_test_container.py` | High |
| ingestion/tests/utils/docker_service_builders/database_container/oracle_test_container.py | Python test `oracle_test_container.py` | High |
| ingestion/tests/utils/docker_service_builders/database_container/postgres_test_container.py | Python test `postgres_test_container.py` | High |
| ingestion/tests/utils/docker_service_builders/test_container_builder.py | Python test `test_container_builder.py` | High |
| ingestion/tests/utils/sqa.py | Python test `sqa.py` | High |
| openmetadata-airflow-apis/tests/unit/ingestion_pipeline/__init__.py | Python test `__init__.py` | High |
| openmetadata-airflow-apis/tests/unit/ingestion_pipeline/test_build_dag.py | Python test `test_build_dag.py` | High |
| openmetadata-airflow-apis/tests/unit/ingestion_pipeline/test_deploy.py | Python test `test_deploy.py` | High |
| openmetadata-airflow-apis/tests/unit/ingestion_pipeline/test_workflow_creation.py | Python test `test_workflow_creation.py` | High |
| openmetadata-clients/openmetadata-java-client/src/test/java/org/openmetadata/client/security/OpenMetadataAuthenticationProviderTest.java | JUnit test class `OpenMetadataAuthenticationProviderTest.java` | High |
| openmetadata-integration-tests/src/test/java/org/openmetadata/it/tests/repositories/RecognizerFeedbackRepositoryTest.java | JUnit test class `RecognizerFeedbackRepositoryTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/controller/OMJobReconcilerTest.java | JUnit test class `OMJobReconcilerTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/service/PodManagerTest.java | JUnit test class `PodManagerTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/unit/CRDSchemaValidationTest.java | JUnit test class `CRDSchemaValidationTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/unit/CronOMJobReconcilerTest.java | JUnit test class `CronOMJobReconcilerTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/unit/KubernetesNameBuilderTest.java | JUnit test class `KubernetesNameBuilderTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/unit/LabelBuilderTest.java | JUnit test class `LabelBuilderTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/unit/OMJobPhaseTest.java | JUnit test class `OMJobPhaseTest.java` | High |
| openmetadata-k8s-operator/src/test/java/org/openmetadata/operator/util/EnvVarUtilsTest.java | JUnit test class `EnvVarUtilsTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/AuthEnrichedMcpContextExtractorTest.java | JUnit test class `AuthEnrichedMcpContextExtractorTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/McpImpersonationTest.java | JUnit test class `McpImpersonationTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/McpSdkUpgradeTest.java | JUnit test class `McpSdkUpgradeTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/auth/OAuthClientMetadataTest.java | JUnit test class `OAuthClientMetadataTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/handlers/AuthorizationHandlerTest.java | JUnit test class `AuthorizationHandlerTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/handlers/McpCallbackServletTest.java | JUnit test class `McpCallbackServletTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/handlers/RegistrationHandlerTest.java | JUnit test class `RegistrationHandlerTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/handlers/RevocationHandlerTest.java | JUnit test class `RevocationHandlerTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/jobs/OAuthTokenCleanupJobTest.java | JUnit test class `OAuthTokenCleanupJobTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/middleware/ClientAuthenticatorTest.java | JUnit test class `ClientAuthenticatorTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/model/AuthorizationErrorResponseTest.java | JUnit test class `AuthorizationErrorResponseTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/repository/OAuthTokenRepositoryTest.java | JUnit test class `OAuthTokenRepositoryTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/util/ClientCredentialsExtractorTest.java | JUnit test class `ClientCredentialsExtractorTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/util/UriUtilsTest.java | JUnit test class `UriUtilsTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/auth/validators/IdTokenValidatorTest.java | JUnit test class `IdTokenValidatorTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/server/transport/HttpServletStatelessServerTransportTest.java | JUnit test class `HttpServletStatelessServerTransportTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateClassificationToolTest.java | JUnit test class `CreateClassificationToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateDataProductToolTest.java | JUnit test class `CreateDataProductToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateDomainToolTest.java | JUnit test class `CreateDomainToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateMetricToolTest.java | JUnit test class `CreateMetricToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateTagToolTest.java | JUnit test class `CreateTagToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/CreateTestCaseToolTest.java | JUnit test class `CreateTestCaseToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/DefaultToolContextTest.java | JUnit test class `DefaultToolContextTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/GetCompanyContextToolTest.java | JUnit test class `GetCompanyContextToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/GetEntityToolTest.java | JUnit test class `GetEntityToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/GetLineageToolTest.java | JUnit test class `GetLineageToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/GlossaryTermToolTest.java | JUnit test class `GlossaryTermToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/GlossaryToolTest.java | JUnit test class `GlossaryToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/McpChangeEventUtilTest.java | JUnit test class `McpChangeEventUtilTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/McpResponseUtilsTest.java | JUnit test class `McpResponseUtilsTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/PatchEntityToolTest.java | JUnit test class `PatchEntityToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/RootCauseAnalysisToolTest.java | JUnit test class `RootCauseAnalysisToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/SearchCompanyContextToolTest.java | JUnit test class `SearchCompanyContextToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/SearchMetadataAggregationTest.java | JUnit test class `SearchMetadataAggregationTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/SearchMetadataToolTest.java | JUnit test class `SearchMetadataToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/tools/SemanticSearchToolTest.java | JUnit test class `SemanticSearchToolTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/usage/McpUsageRecorderTest.java | JUnit test class `McpUsageRecorderTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/util/McpParamsTest.java | JUnit test class `McpParamsTest.java` | High |
| openmetadata-mcp/src/test/java/org/openmetadata/mcp/util/McpResponseTrimTest.java | JUnit test class `McpResponseTrimTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/BaseSDKTest.java | JUnit test class `BaseSDKTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/EntityReferencesTest.java | JUnit test class `EntityReferencesTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/OMStorageServicesInitTest.java | JUnit test class `OMStorageServicesInitTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/api/LineageFluentAPITest.java | JUnit test class `LineageFluentAPITest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/ClassificationMockTest.java | JUnit test class `ClassificationMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/ContainerMockTest.java | JUnit test class `ContainerMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/DashboardMockTest.java | JUnit test class `DashboardMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/DatabaseMockTest.java | JUnit test class `DatabaseMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/DatabaseServiceMockTest.java | JUnit test class `DatabaseServiceMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/DomainMockTest.java | JUnit test class `DomainMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/GlossaryMockTest.java | JUnit test class `GlossaryMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/MlModelMockTest.java | JUnit test class `MlModelMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/PipelineMockTest.java | JUnit test class `PipelineMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/QueryMockTest.java | JUnit test class `QueryMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/SearchIndexMockTest.java | JUnit test class `SearchIndexMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/TableEntityOperationsTest.java | JUnit test class `TableEntityOperationsTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/TablesFluentAPITest.java | JUnit test class `TablesFluentAPITest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/TeamMockTest.java | JUnit test class `TeamMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/TestCaseMockTest.java | JUnit test class `TestCaseMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/TopicMockTest.java | JUnit test class `TopicMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/entities/UserMockTest.java | JUnit test class `UserMockTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/fluent/ContainersFluentAPITest.java | JUnit test class `ContainersFluentAPITest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/fluent/CustomPropertiesTest.java | JUnit test class `CustomPropertiesTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/fluent/DashboardsFluentAPITest.java | JUnit test class `DashboardsFluentAPITest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/fluent/RestoreFluentAPITest.java | JUnit test class `RestoreFluentAPITest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/resources/BaseResourceTest.java | JUnit test class `BaseResourceTest.java` | High |
| openmetadata-sdk/src/test/java/org/openmetadata/sdk/services/EntityServiceBaseTest.java | JUnit test class `EntityServiceBaseTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/csv/CsvParseTest.java | JUnit test class `CsvParseTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/csv/CsvUtilTest.java | JUnit test class `CsvUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/csv/EntityCsvTest.java | JUnit test class `EntityCsvTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/schema/utils/VersionUtilsTest.java | JUnit test class `VersionUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/EntityLinkGrammarTest.java | JUnit test class `EntityLinkGrammarTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/EnumBackwardCompatibilityTest.java | JUnit test class `EnumBackwardCompatibilityTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/OpenMetadataServerHealthCheckTest.java | JUnit test class `OpenMetadataServerHealthCheckTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/TypeRegistryTest.java | JUnit test class `TypeRegistryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/AbstractNativeApplicationTest.java | JUnit test class `AbstractNativeApplicationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/cache/CacheWarmupAppConfigParseTest.java | JUnit test class `CacheWarmupAppConfigParseTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/changeEvent/AbstractEventConsumerTest.java | JUnit test class `AbstractEventConsumerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/changeEvent/AlertPublisherTest.java | JUnit test class `AlertPublisherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/changeEvent/feed/ActivityStreamPublisherTest.java | JUnit test class `ActivityStreamPublisherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/DataInsightsEntityEnricherProcessorTest.java | JUnit test class `DataInsightsEntityEnricherProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/EnrichmentPipelineTest.java | JUnit test class `EnrichmentPipelineTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/OwnerResolverTest.java | JUnit test class `OwnerResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/SnapshotMaterializerTest.java | JUnit test class `SnapshotMaterializerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/VersionResolverTest.java | JUnit test class `VersionResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/insights/workflows/dataAssets/processors/enricher/steps/TierStepTest.java | JUnit test class `TierStepTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/RdfBatchProcessorTest.java | JUnit test class `RdfBatchProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/RdfIndexAppTest.java | JUnit test class `RdfIndexAppTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/DistributedRdfIndexCoordinatorTest.java | JUnit test class `DistributedRdfIndexCoordinatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/DistributedRdfIndexExecutorTest.java | JUnit test class `DistributedRdfIndexExecutorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfDistributedJobParticipantTest.java | JUnit test class `RdfDistributedJobParticipantTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfEntityCompletionTrackerTest.java | JUnit test class `RdfEntityCompletionTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfPartitionWorkerTest.java | JUnit test class `RdfPartitionWorkerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/rdf/distributed/RdfPollingJobNotifierTest.java | JUnit test class `RdfPollingJobNotifierTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/BulkCircuitBreakerTest.java | JUnit test class `BulkCircuitBreakerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/CompositeProgressListenerTest.java | JUnit test class `CompositeProgressListenerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/DistributedIndexingStrategyTest.java | JUnit test class `DistributedIndexingStrategyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/DistributedReindexFinalizerTest.java | JUnit test class `DistributedReindexFinalizerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ElasticSearchBulkProcessorTest.java | JUnit test class `ElasticSearchBulkProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ElasticSearchBulkSinkBehaviorTest.java | JUnit test class `ElasticSearchBulkSinkBehaviorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ElasticSearchBulkSinkSimpleTest.java | JUnit test class `ElasticSearchBulkSinkSimpleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ElasticSearchIndexSinkTest.java | JUnit test class `ElasticSearchIndexSinkTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/EntityPriorityTest.java | JUnit test class `EntityPriorityTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/IndexingFailureRecorderTest.java | JUnit test class `IndexingFailureRecorderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OpenSearchBulkProcessorTest.java | JUnit test class `OpenSearchBulkProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OpenSearchBulkSinkBehaviorTest.java | JUnit test class `OpenSearchBulkSinkBehaviorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OpenSearchBulkSinkSimpleTest.java | JUnit test class `OpenSearchBulkSinkSimpleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OpenSearchBulkSinkStatsTest.java | JUnit test class `OpenSearchBulkSinkStatsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OpenSearchIndexSinkTest.java | JUnit test class `OpenSearchIndexSinkTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/OrphanedIndexCleanerTest.java | JUnit test class `OrphanedIndexCleanerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/QuartzJobContextTest.java | JUnit test class `QuartzJobContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/QuartzOrchestratorContextTest.java | JUnit test class `QuartzOrchestratorContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingConfigurationTest.java | JUnit test class `ReindexingConfigurationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingConfigurationTimeSeriesTest.java | JUnit test class `ReindexingConfigurationTimeSeriesTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingJobLoggerTest.java | JUnit test class `ReindexingJobLoggerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingMetricsTest.java | JUnit test class `ReindexingMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/ReindexingOrchestratorTest.java | JUnit test class `ReindexingOrchestratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/SearchIndexAppConfigSanitizerTest.java | JUnit test class `SearchIndexAppConfigSanitizerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/SearchIndexAppTest.java | JUnit test class `SearchIndexAppTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/SearchIndexClusterValidatorTest.java | JUnit test class `SearchIndexClusterValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/SearchIndexMetricsTest.java | JUnit test class `SearchIndexMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/SlackWebApiClientTest.java | JUnit test class `SlackWebApiClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/StatsReconcilerTest.java | JUnit test class `StatsReconcilerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/StatsThreadSafetyTest.java | JUnit test class `StatsThreadSafetyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedJobContextTest.java | JUnit test class `DistributedJobContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedJobNotifierFactoryTest.java | JUnit test class `DistributedJobNotifierFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedJobParticipantTest.java | JUnit test class `DistributedJobParticipantTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedJobStatsAggregatorTest.java | JUnit test class `DistributedJobStatsAggregatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedSearchIndexCoordinatorTest.java | JUnit test class `DistributedSearchIndexCoordinatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/DistributedSearchIndexExecutorTest.java | JUnit test class `DistributedSearchIndexExecutorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/EntityCompletionTrackerTest.java | JUnit test class `EntityCompletionTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/JobRecoveryManagerTest.java | JUnit test class `JobRecoveryManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/JobRecoveryOrphanDetectionTest.java | JUnit test class `JobRecoveryOrphanDetectionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/OrphanJobMonitorTest.java | JUnit test class `OrphanJobMonitorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/PartitionCalculatorTest.java | JUnit test class `PartitionCalculatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/PartitionWorkerTest.java | JUnit test class `PartitionWorkerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/PollingJobNotifierTest.java | JUnit test class `PollingJobNotifierTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/SearchIndexJobTest.java | JUnit test class `SearchIndexJobTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/SearchIndexPartitionTest.java | JUnit test class `SearchIndexPartitionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/distributed/ServerIdentityResolverTest.java | JUnit test class `ServerIdentityResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/listeners/LoggingProgressListenerTest.java | JUnit test class `LoggingProgressListenerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/listeners/QuartzProgressListenerTest.java | JUnit test class `QuartzProgressListenerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/listeners/SlackProgressListenerTest.java | JUnit test class `SlackProgressListenerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/promotion/RatioPromotionPolicyTest.java | JUnit test class `RatioPromotionPolicyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/stats/EntityStatsTrackerTest.java | JUnit test class `EntityStatsTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/stats/JobStatsManagerTest.java | JUnit test class `JobStatsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/stats/StageCounterTest.java | JUnit test class `StageCounterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/bundles/searchIndex/stats/StageStatsTrackerTest.java | JUnit test class `StageStatsTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/logging/AppRunLogAppenderTest.java | JUnit test class `AppRunLogAppenderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/logging/RunLogBufferTest.java | JUnit test class `RunLogBufferTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/scheduler/AppSchedulerStopTest.java | JUnit test class `AppSchedulerStopTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/scheduler/AppSchedulerTest.java | JUnit test class `AppSchedulerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/apps/scheduler/OmAppJobListenerTest.java | JUnit test class `OmAppJobListenerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/audit/AuditLogConsumerTest.java | JUnit test class `AuditLogConsumerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/audit/AuditLogRepositoryTest.java | JUnit test class `AuditLogRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/cache/BundleWarmupBatcherTest.java | JUnit test class `BundleWarmupBatcherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/cache/EntityCacheBypassTest.java | JUnit test class `EntityCacheBypassTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/cache/ListCountCacheTest.java | JUnit test class `ListCountCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/cache/RedisCacheProviderStateMachineTest.java | JUnit test class `RedisCacheProviderStateMachineTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/airflow/AirflowRESTClientTest.java | JUnit test class `AirflowRESTClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/config/WorkflowConfigBuilderTest.java | JUnit test class `WorkflowConfigBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/config/WorkflowConfigTest.java | JUnit test class `WorkflowConfigTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/CronOMJobSerializationTest.java | JUnit test class `CronOMJobSerializationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/EnvVarEndToEndTest.java | JUnit test class `EnvVarEndToEndTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/IngestionLogHandlerTest.java | JUnit test class `IngestionLogHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/K8sJobUtilsTest.java | JUnit test class `K8sJobUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/K8sPipelineClientConfigTest.java | JUnit test class `K8sPipelineClientConfigTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/K8sPipelineClientTest.java | JUnit test class `K8sPipelineClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/clients/pipeline/k8s/OMJobSerializationTest.java | JUnit test class `OMJobSerializationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/config/LoggingConfigurationYamlTest.java | JUnit test class `LoggingConfigurationYamlTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/context/ContextEntityPromptServiceTest.java | JUnit test class `ContextEntityPromptServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/context/DefaultContextEntityPromptLoaderTest.java | JUnit test class `DefaultContextEntityPromptLoaderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/drive/ContextFileProcessingServiceTest.java | JUnit test class `ContextFileProcessingServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/drive/ContextFileTextExtractorTest.java | JUnit test class `ContextFileTextExtractorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/drive/ContextMemoryExtractorTest.java | JUnit test class `ContextMemoryExtractorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/lifecycle/EntityLifecycleEventDispatcherTest.java | JUnit test class `EntityLifecycleEventDispatcherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/lifecycle/OrderedLaneExecutorTest.java | JUnit test class `OrderedLaneExecutorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/lifecycle/handlers/SearchIndexHandlerTest.java | JUnit test class `SearchIndexHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/scheduled/EventSubscriptionSchedulerClusteringTest.java | JUnit test class `EventSubscriptionSchedulerClusteringTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/scheduled/EventSubscriptionSchedulerTest.java | JUnit test class `EventSubscriptionSchedulerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/subscription/AlertUtilTest.java | JUnit test class `AlertUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/events/subscription/AlertsRuleEvaluatorTaskMentionsTest.java | JUnit test class `AlertsRuleEvaluatorTaskMentionsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/exception/CatalogExceptionMessageTest.java | JUnit test class `CatalogExceptionMessageTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/fernet/FernetEncryptWebhookOAuth2Test.java | JUnit test class `FernetEncryptWebhookOAuth2Test.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/fernet/FernetKeyCacheTest.java | JUnit test class `FernetKeyCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/decorators/EmailMessageDecoratorTest.java | JUnit test class `EmailMessageDecoratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/decorators/MessageDecoratorTest.java | JUnit test class `MessageDecoratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/decorators/PlatformMessageDecoratorTest.java | JUnit test class `PlatformMessageDecoratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/decorators/RichPlatformMessageDecoratorTest.java | JUnit test class `RichPlatformMessageDecoratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/decorators/SlackMessageDecoratorTest.java | JUnit test class `SlackMessageDecoratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/entity/IngestionPipelineFormatterTest.java | JUnit test class `IngestionPipelineFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/entity/PipelineFormatterTest.java | JUnit test class `PipelineFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/field/DefaultFieldFormatterTest.java | JUnit test class `DefaultFieldFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/field/OwnerFormatterTest.java | JUnit test class `OwnerFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/field/TagFormatterTest.java | JUnit test class `TagFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/formatter/util/FormatterUtilTest.java | JUnit test class `FormatterUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/IdempotentDdlDataSourceTest.java | JUnit test class `IdempotentDdlDataSourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/IdempotentDdlStatementTest.java | JUnit test class `IdempotentDdlStatementTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/WorkflowEventConsumerTest.java | JUnit test class `WorkflowEventConsumerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/WorkflowHandlerSchemaUpdateTest.java | JUnit test class `WorkflowHandlerSchemaUpdateTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/NodeFactoryRegistryTest.java | JUnit test class `NodeFactoryRegistryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/NodeFactoryTest.java | JUnit test class `NodeFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/TriggerFactoryTest.java | JUnit test class `TriggerFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/createAndRunIngestionPipeline/RunIngestionPipelineImplTest.java | JUnit test class `RunIngestionPipelineImplTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/SinkProviderRegistryTest.java | JUnit test class `SinkProviderRegistryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/SinkResultTest.java | JUnit test class `SinkResultTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/SinkTaskDelegateTest.java | JUnit test class `SinkTaskDelegateTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/SinkTaskTest.java | JUnit test class `SinkTaskTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/automatedTask/sink/WebhookSinkProviderTest.java | JUnit test class `WebhookSinkProviderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/userTask/CreateTaskTest.java | JUnit test class `CreateTaskTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/userTask/impl/ApprovalTaskCompletionValidatorTest.java | JUnit test class `ApprovalTaskCompletionValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/nodes/userTask/impl/SetApprovalAssigneesImplTest.java | JUnit test class `SetApprovalAssigneesImplTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/triggers/NoOpTriggerTest.java | JUnit test class `NoOpTriggerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/triggers/PeriodicBatchEntityTriggerTest.java | JUnit test class `PeriodicBatchEntityTriggerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/elements/triggers/impl/FilterEntityImplTest.java | JUnit test class `FilterEntityImplTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/flowable/builders/IntermediateCatchEventBuilderTest.java | JUnit test class `IntermediateCatchEventBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/governance/workflows/util/ChangePreviewUtilsTest.java | JUnit test class `ChangePreviewUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ActivityStreamRepositoryTest.java | JUnit test class `ActivityStreamRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/BoundedListFilterTest.java | JUnit test class `BoundedListFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/BulkExecutorTest.java | JUnit test class `BulkExecutorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ChangeSummarizerTest.java | JUnit test class `ChangeSummarizerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/CollectionDAOMcpTest.java | JUnit test class `CollectionDAOMcpTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ContainerRepositoryParentValidationTest.java | JUnit test class `ContainerRepositoryParentValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/DataContractFieldSupportTest.java | JUnit test class `DataContractFieldSupportTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityCacheMemoryTest.java | JUnit test class `EntityCacheMemoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityRelationshipPerformanceTest.java | JUnit test class `EntityRelationshipPerformanceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityRepositoryBulkFieldsTest.java | JUnit test class `EntityRepositoryBulkFieldsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityRepositoryCertificationTest.java | JUnit test class `EntityRepositoryCertificationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityRepositoryRestoreTest.java | JUnit test class `EntityRepositoryRestoreTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EntityTimeSeriesRepositoryPaginationTest.java | JUnit test class `EntityTimeSeriesRepositoryPaginationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/EventSubscriptionDaoDedupTest.java | JUnit test class `EventSubscriptionDaoDedupTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ImpersonationCleanupFilterTest.java | JUnit test class `ImpersonationCleanupFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/InListChunkingTest.java | JUnit test class `InListChunkingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/IngestionPipelineRepositoryTest.java | JUnit test class `IngestionPipelineRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/IngestionPipelineStatusParserTest.java | JUnit test class `IngestionPipelineStatusParserTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/JsonContainsFilterFactoryTest.java | JUnit test class `JsonContainsFilterFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/LineageRepositoryTest.java | JUnit test class `LineageRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ListFilterTest.java | JUnit test class `ListFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/McpExecutionRepositoryTest.java | JUnit test class `McpExecutionRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/McpServerRepositoryTest.java | JUnit test class `McpServerRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/McpServiceRepositoryTest.java | JUnit test class `McpServiceRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/N1QueryFixValidationTest.java | JUnit test class `N1QueryFixValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/PolicyRepositoryTest.java | JUnit test class `PolicyRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ReadBundleTest.java | JUnit test class `ReadBundleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ReadPlannerTest.java | JUnit test class `ReadPlannerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SearchListFilterTest.java | JUnit test class `SearchListFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/ShouldCompareFieldNamesTest.java | JUnit test class `ShouldCompareFieldNamesTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SystemRepositoryEmbeddingsValidationTest.java | JUnit test class `SystemRepositoryEmbeddingsValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SystemRepositoryLdapConfigTest.java | JUnit test class `SystemRepositoryLdapConfigTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SystemRepositoryMissingIndexesTest.java | JUnit test class `SystemRepositoryMissingIndexesTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SystemRepositoryObjectStorageValidationTest.java | JUnit test class `SystemRepositoryObjectStorageValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/SystemRepositoryReindexStatusTest.java | JUnit test class `SystemRepositoryReindexStatusTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/TagRepositoryUnitTest.java | JUnit test class `TagRepositoryUnitTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/TestCaseRepositoryTest.java | JUnit test class `TestCaseRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/TestCaseResolutionStatusRepositoryTest.java | JUnit test class `TestCaseResolutionStatusRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/UsageDetailsMapperTest.java | JUnit test class `UsageDetailsMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/UserRepositoryUnitTest.java | JUnit test class `UserRepositoryUnitTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/WorkflowDefinitionGraphValidationTest.java | JUnit test class `WorkflowDefinitionGraphValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/jdbi3/locator/ConnectionAwareAnnotationSqlLocatorTest.java | JUnit test class `ConnectionAwareAnnotationSqlLocatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/llm/LLMClientHolderTest.java | JUnit test class `LLMClientHolderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/llm/LLMCompletionClientFactoryTest.java | JUnit test class `LLMCompletionClientFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/llm/LLMCompletionClientTest.java | JUnit test class `LLMCompletionClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/logging/SwitchableLayoutFactoryTest.java | JUnit test class `SwitchableLayoutFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/logstorage/LogStorageFactoryTest.java | JUnit test class `LogStorageFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/logstorage/LogStorageTest.java | JUnit test class `LogStorageTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/logstorage/S3LogStorageTest.java | JUnit test class `S3LogStorageTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/MigrationWorkflowReprocessingTest.java | JUnit test class `MigrationWorkflowReprocessingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/api/MigrationProcessImplTest.java | JUnit test class `MigrationProcessImplTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/api/MigrationWorkflowTest.java | JUnit test class `MigrationWorkflowTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/mysql/v1125/MigrationTest.java | JUnit test class `MigrationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/postgres/v1125/MigrationTest.java | JUnit test class `MigrationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/SearchSettingsMergeUtilTest.java | JUnit test class `SearchSettingsMergeUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1100/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1104/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1105/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1112/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1114/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1120/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v11210/FileExtensionAggregationScrubTest.java | JUnit test class `FileExtensionAggregationScrubTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v11210/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1125/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1126/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1129/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1130/FileExtensionAggregationScrubTest.java | JUnit test class `FileExtensionAggregationScrubTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1130/FlattenedChildrenScrubTest.java | JUnit test class `FlattenedChildrenScrubTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v1130/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v160/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v200/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/migration/utils/v201/MigrationUtilTest.java | JUnit test class `MigrationUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/CloudWatchEventMonitorTest.java | JUnit test class `CloudWatchEventMonitorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/EventMonitorFactoryTest.java | JUnit test class `EventMonitorFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/EventMonitorPublisherTest.java | JUnit test class `EventMonitorPublisherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/IngestionProgressTrackerTest.java | JUnit test class `IngestionProgressTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/LatencyPhaseFilterTest.java | JUnit test class `LatencyPhaseFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/MetricsDebugTest.java | JUnit test class `MetricsDebugTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/MetricsErrorHandlingTest.java | JUnit test class `MetricsErrorHandlingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/MicrometerBundleTest.java | JUnit test class `MicrometerBundleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/PatchOperationMetricsTest.java | JUnit test class `PatchOperationMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/RequestLatencyContextTest.java | JUnit test class `RequestLatencyContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/RequestLatencyTrackingSimpleTest.java | JUnit test class `RequestLatencyTrackingSimpleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/RequestMetricsFilterTest.java | JUnit test class `RequestMetricsFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/SimpleMetricsTest.java | JUnit test class `SimpleMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/StreamableLogsMetricsTest.java | JUnit test class `StreamableLogsMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/monitoring/UserMetricsServletTest.java | JUnit test class `UserMetricsServletTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/NotificationChannelUtilitiesTest.java | JUnit test class `NotificationChannelUtilitiesTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/gchat/GChatCardAssemblerTest.java | JUnit test class `GChatCardAssemblerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/gchat/GChatMarkdownFormatterTest.java | JUnit test class `GChatMarkdownFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/slack/SlackBlockAssemblerTest.java | JUnit test class `SlackBlockAssemblerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/slack/SlackMarkdownFormatterTest.java | JUnit test class `SlackMarkdownFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/teams/TeamsCardAssemblerTest.java | JUnit test class `TeamsCardAssemblerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/channels/teams/TeamsMarkdownFormatterTest.java | JUnit test class `TeamsMarkdownFormatterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/template/TaskEntityUpdatedTemplateTest.java | JUnit test class `TaskEntityUpdatedTemplateTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/template/handlebars/helpers/BuildEntityUrlHelperTest.java | JUnit test class `BuildEntityUrlHelperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/template/handlebars/helpers/NotificationTemplateHelperAdvancedTest.java | JUnit test class `NotificationTemplateHelperAdvancedTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/notifications/template/handlebars/helpers/NotificationTemplateHelperCoreTest.java | JUnit test class `NotificationTemplateHelperCoreTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/openlineage/OpenLineageEntityResolverTest.java | JUnit test class `OpenLineageEntityResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/openlineage/OpenLineageMapperTest.java | JUnit test class `OpenLineageMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/pipelineService/PipelineServiceClientTest.java | JUnit test class `PipelineServiceClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfFqnEscapingTest.java | JUnit test class `RdfFqnEscapingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfInferenceConfigurationTest.java | JUnit test class `RdfInferenceConfigurationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfJsonLdContextTest.java | JUnit test class `RdfJsonLdContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfLineageDetailsTest.java | JUnit test class `RdfLineageDetailsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfParserHelpersTest.java | JUnit test class `RdfParserHelpersTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfPredicatePartitionTest.java | JUnit test class `RdfPredicatePartitionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfPropertyMapperTest.java | JUnit test class `RdfPropertyMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfRelationPredicateValidationTest.java | JUnit test class `RdfRelationPredicateValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfRepositoryBulkChunkingTest.java | JUnit test class `RdfRepositoryBulkChunkingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfStorageIdempotencyTest.java | JUnit test class `RdfStorageIdempotencyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfUpdaterTest.java | JUnit test class `RdfUpdaterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/RdfUtilsTest.java | JUnit test class `RdfUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/sql2sparql/SparqlBuilderNestedFieldsTest.java | JUnit test class `SparqlBuilderNestedFieldsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/sql2sparql/SqlToSparqlTranslatorTest.java | JUnit test class `SqlToSparqlTranslatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rdf/storage/JenaFusekiStorageTest.java | JUnit test class `JenaFusekiStorageTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/ai/McpExecutionResourceTest.java | JUnit test class `McpExecutionResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/ai/McpServerMapperTest.java | JUnit test class `McpServerMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/ai/McpServerResourceTest.java | JUnit test class `McpServerResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/analytics/WebAnalyticEventResourceTest.java | JUnit test class `WebAnalyticEventResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/apps/AppResourceRetryQueueTest.java | JUnit test class `AppResourceRetryQueueTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/apps/ConfigurationReaderTest.java | JUnit test class `ConfigurationReaderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/automations/WorkflowResourceDecryptTest.java | JUnit test class `WorkflowResourceDecryptTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/context/ContextMemoryStatusTransitionTest.java | JUnit test class `ContextMemoryStatusTransitionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/context/ContextMemoryVisibilityTest.java | JUnit test class `ContextMemoryVisibilityTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/drive/ContextFileResourceTest.java | JUnit test class `ContextFileResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/drive/ContextFileUploadSupportTest.java | JUnit test class `ContextFileUploadSupportTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/drive/DriveMapperTest.java | JUnit test class `DriveMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/feeds/MessageParserTest.java | JUnit test class `MessageParserTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/filters/ETagResponseFilterTest.java | JUnit test class `ETagResponseFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/mcp/McpUsageResourceTest.java | JUnit test class `McpUsageResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/rdf/RdfResourceTest.java | JUnit test class `RdfResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/services/mcp/McpServiceMapperTest.java | JUnit test class `McpServiceMapperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/services/mcp/McpServiceResourceTest.java | JUnit test class `McpServiceResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/settings/SettingsCacheDefaultCaseTest.java | JUnit test class `SettingsCacheDefaultCaseTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/system/IndexResourceTest.java | JUnit test class `IndexResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/system/SearchSettingsHandlerTest.java | JUnit test class `SearchSettingsHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/tags/TagLabelUtilTest.java | JUnit test class `TagLabelUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/resources/teams/TeamResourceTest.java | JUnit test class `TeamResourceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/rules/RuleEngineTest.java | JUnit test class `RuleEngineTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/AwsCredentialsProviderTest.java | JUnit test class `AwsCredentialsProviderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/ColumnAggregatorTest.java | JUnit test class `ColumnAggregatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/ColumnFilterMatcherTest.java | JUnit test class `ColumnFilterMatcherTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/ColumnMetadataCacheTest.java | JUnit test class `ColumnMetadataCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/ColumnMetadataGrouperTest.java | JUnit test class `ColumnMetadataGrouperTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/CustomPropertySearchTest.java | JUnit test class `CustomPropertySearchTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/DataProductIndexMappingTest.java | JUnit test class `DataProductIndexMappingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/DefaultInheritedFieldEntitySearchTest.java | JUnit test class `DefaultInheritedFieldEntitySearchTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/DefaultRecreateHandlerTest.java | JUnit test class `DefaultRecreateHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/FuzzySearchClauseTest.java | JUnit test class `FuzzySearchClauseTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/IndexMappingNestedFieldConsistencyTest.java | JUnit test class `IndexMappingNestedFieldConsistencyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/IndexMappingVersionTrackerTest.java | JUnit test class `IndexMappingVersionTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/LineagePathPreserverTest.java | JUnit test class `LineagePathPreserverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/QueryFilterBuilderTest.java | JUnit test class `QueryFilterBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/QueryFilterParserTest.java | JUnit test class `QueryFilterParserTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchAggregationTest.java | JUnit test class `SearchAggregationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchClientConfigurationTest.java | JUnit test class `SearchClientConfigurationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchClientTagScriptSeparationTest.java | JUnit test class `SearchClientTagScriptSeparationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchClusterMetricsTest.java | JUnit test class `SearchClusterMetricsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchFieldResolutionTest.java | JUnit test class `SearchFieldResolutionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchHostParsingTest.java | JUnit test class `SearchHostParsingTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchIndexFactoryTest.java | JUnit test class `SearchIndexFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchIndexReindexFieldsParityTest.java | JUnit test class `SearchIndexReindexFieldsParityTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchIndexUtilsTest.java | JUnit test class `SearchIndexUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchRepositoryBehaviorTest.java | JUnit test class `SearchRepositoryBehaviorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchRepositoryTest.java | JUnit test class `SearchRepositoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchResultCsvExporterTest.java | JUnit test class `SearchResultCsvExporterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchSourceBuilderFactoryTest.java | JUnit test class `SearchSourceBuilderFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SearchUtilsTest.java | JUnit test class `SearchUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/SigV4RequestSigningInterceptorTest.java | JUnit test class `SigV4RequestSigningInterceptorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/capability/EntityIndexCapabilityRegistryTest.java | JUnit test class `EntityIndexCapabilityRegistryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/ESLineageGraphBuilderTest.java | JUnit test class `ESLineageGraphBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/ElasticSearchEntitiesProcessorTest.java | JUnit test class `ElasticSearchEntitiesProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/ElasticSearchIndexManagerTest.java | JUnit test class `ElasticSearchIndexManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/ElasticSearchIndexSinkTest.java | JUnit test class `ElasticSearchIndexSinkTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/EsUtilsTest.java | JUnit test class `EsUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/elasticsearch/dataInsightAggregators/ElasticSearchLineChartAggregatorTest.java | JUnit test class `ElasticSearchLineChartAggregatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/AddUpdateLineageScriptTest.java | JUnit test class `AddUpdateLineageScriptTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/BuildSearchIndexDocTest.java | JUnit test class `BuildSearchIndexDocTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/ContextFileIndexTest.java | JUnit test class `ContextFileIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/ContextMemoryIndexTest.java | JUnit test class `ContextMemoryIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/CustomIndexTest.java | JUnit test class `CustomIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/DataAssetIndexTest.java | JUnit test class `DataAssetIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/DocBuildContextTest.java | JUnit test class `DocBuildContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/EntitySpecificFieldsTest.java | JUnit test class `EntitySpecificFieldsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/EntityTypeNameTest.java | JUnit test class `EntityTypeNameTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/FolderIndexTest.java | JUnit test class `FolderIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/GlossaryTermIndexTest.java | JUnit test class `GlossaryTermIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/LineageIndexTest.java | JUnit test class `LineageIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/McpExecutionIndexTest.java | JUnit test class `McpExecutionIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/McpServerIndexTest.java | JUnit test class `McpServerIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/McpServiceIndexTest.java | JUnit test class `McpServiceIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/MergeChildTagsTest.java | JUnit test class `MergeChildTagsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/MigratedIndexTest.java | JUnit test class `MigratedIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/PageIndexTest.java | JUnit test class `PageIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/PopulateCommonFieldsTest.java | JUnit test class `PopulateCommonFieldsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/SearchDocFieldValidationTest.java | JUnit test class `SearchDocFieldValidationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/SearchIndexPrefetchTest.java | JUnit test class `SearchIndexPrefetchTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/SearchIndexTest.java | JUnit test class `SearchIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/ServiceBackedIndexTest.java | JUnit test class `ServiceBackedIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/TaggableIndexTest.java | JUnit test class `TaggableIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/TestCaseIndexTest.java | JUnit test class `TestCaseIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/indexes/TestSuiteIndexTest.java | JUnit test class `TestSuiteIndexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/AbstractLineageGraphBuilderTest.java | JUnit test class `AbstractLineageGraphBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/GuavaLineageGraphCacheTest.java | JUnit test class `GuavaLineageGraphCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/LineageCacheKeyTest.java | JUnit test class `LineageCacheKeyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/LineageGraphConfigurationTest.java | JUnit test class `LineageGraphConfigurationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/LineageStrategySelectorTest.java | JUnit test class `LineageStrategySelectorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/lineage/SmallGraphStrategyTest.java | JUnit test class `SmallGraphStrategyTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OSLineageGraphBuilderTest.java | JUnit test class `OSLineageGraphBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OpenSearchClientTransportTest.java | JUnit test class `OpenSearchClientTransportTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OpenSearchEntitiesProcessorTest.java | JUnit test class `OpenSearchEntitiesProcessorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OpenSearchIndexManagerTest.java | JUnit test class `OpenSearchIndexManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OpenSearchIndexSinkTest.java | JUnit test class `OpenSearchIndexSinkTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/OsUtilsTest.java | JUnit test class `OsUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/opensearch/dataInsightAggregator/OpenSearchLineChartAggregatorTest.java | JUnit test class `OpenSearchLineChartAggregatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/scripts/SoftDeleteScriptTest.java | JUnit test class `SoftDeleteScriptTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/security/ElasticSearchRBACConditionEvaluatorTest.java | JUnit test class `ElasticSearchRBACConditionEvaluatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/security/OpenSearchRBACConditionEvaluatorTest.java | JUnit test class `OpenSearchRBACConditionEvaluatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/validation/IndexMappingValidatorTest.java | JUnit test class `IndexMappingValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/ContextMemoryBodyTextContributorTest.java | JUnit test class `ContextMemoryBodyTextContributorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/EmbeddingClientTest.java | JUnit test class `EmbeddingClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/EmbeddingModelChangeDetectionTest.java | JUnit test class `EmbeddingModelChangeDetectionTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/OpenSearchVectorServiceTest.java | JUnit test class `OpenSearchVectorServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/PageBodyTextContributorTest.java | JUnit test class `PageBodyTextContributorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/TestEntityBodyTextContributorTest.java | JUnit test class `TestEntityBodyTextContributorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/VectorDocBuilderTest.java | JUnit test class `VectorDocBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/VectorEmbeddingHandlerTest.java | JUnit test class `VectorEmbeddingHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/VectorSearchQueryBuilderTest.java | JUnit test class `VectorSearchQueryBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/client/GoogleEmbeddingClientTest.java | JUnit test class `GoogleEmbeddingClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/client/OpenAIEmbeddingClientTest.java | JUnit test class `OpenAIEmbeddingClientTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/search/vector/utils/TextChunkManagerTest.java | JUnit test class `TextChunkManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AWSBasedSecretsManagerTest.java | JUnit test class `AWSBasedSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AWSSSMSecretsManagerLocalStackTest.java | JUnit test class `AWSSSMSecretsManagerLocalStackTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AWSSSMSecretsManagerTest.java | JUnit test class `AWSSSMSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AWSSecretsManagerLocalStackTest.java | JUnit test class `AWSSecretsManagerLocalStackTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AWSSecretsManagerTest.java | JUnit test class `AWSSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/AzureKVSecretsManagerTest.java | JUnit test class `AzureKVSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/DBSecretsManagerTest.java | JUnit test class `DBSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/ExternalSecretsManagerTest.java | JUnit test class `ExternalSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/ExternalSecretsManagerUnitTest.java | JUnit test class `ExternalSecretsManagerUnitTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/GCPSecretsManagerTest.java | JUnit test class `GCPSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/KubernetesSecretsManagerTest.java | JUnit test class `KubernetesSecretsManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/SecretsManagerFactoryTest.java | JUnit test class `SecretsManagerFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/SecretsManagerLifecycleTest.java | JUnit test class `SecretsManagerLifecycleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/SecretsManagerRateLimiterTest.java | JUnit test class `SecretsManagerRateLimiterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/converter/AirflowRestApiConnectionClassConverterTest.java | JUnit test class `AirflowRestApiConnectionClassConverterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/converter/ClassConverterFactoryTest.java | JUnit test class `ClassConverterFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/converter/MatillionConnectionClassConverterTest.java | JUnit test class `MatillionConnectionClassConverterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/masker/EntityMaskerFactoryTest.java | JUnit test class `EntityMaskerFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/secrets/masker/PasswordEntityMaskerTest.java | JUnit test class `PasswordEntityMaskerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/AuthCallbackServletTest.java | JUnit test class `AuthCallbackServletTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/AuthLogoutServletTest.java | JUnit test class `AuthLogoutServletTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/AuthenticationCodeFlowHandlerTest.java | JUnit test class `AuthenticationCodeFlowHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/CspNonceHandlerTest.java | JUnit test class `CspNonceHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/DefaultAuthorizerTest.java | JUnit test class `DefaultAuthorizerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/JWTTokenGeneratorTest.java | JUnit test class `JWTTokenGeneratorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/JwtFilterTest.java | JUnit test class `JwtFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/MultiUrlJwkProviderTest.java | JUnit test class `MultiUrlJwkProviderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/SecurityUtilTest.java | JUnit test class `SecurityUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/BasicAuthServletHandlerTest.java | JUnit test class `BasicAuthServletHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/LdapAuthServletHandlerTest.java | JUnit test class `LdapAuthServletHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/SamlAuthServletHandlerTest.java | JUnit test class `SamlAuthServletHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/SecurityConfigurationManagerTest.java | JUnit test class `SecurityConfigurationManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/UnifiedAuthTest.java | JUnit test class `UnifiedAuthTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/UserActivityTrackerTest.java | JUnit test class `UserActivityTrackerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/Auth0ValidatorTest.java | JUnit test class `Auth0ValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/AzureAuthValidatorTest.java | JUnit test class `AzureAuthValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/CognitoValidatorTest.java | JUnit test class `CognitoValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/CustomOidcValidatorTest.java | JUnit test class `CustomOidcValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/GoogleAuthValidatorTest.java | JUnit test class `GoogleAuthValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/OidcDiscoveryValidatorTest.java | JUnit test class `OidcDiscoveryValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/OktaAuthValidatorTest.java | JUnit test class `OktaAuthValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/auth/validator/SamlValidatorTest.java | JUnit test class `SamlValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/mask/PIIMaskerTest.java | JUnit test class `PIIMaskerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/CachedPermissionEvaluationTest.java | JUnit test class `CachedPermissionEvaluationTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/CompiledRuleTest.java | JUnit test class `CompiledRuleTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/DomainAccessTest.java | JUnit test class `DomainAccessTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/ExpressionValidatorTest.java | JUnit test class `ExpressionValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/McpExecutionContextTest.java | JUnit test class `McpExecutionContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/PermissionDebugServiceTest.java | JUnit test class `PermissionDebugServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/PolicyConditionUpdaterTest.java | JUnit test class `PolicyConditionUpdaterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/PolicyEvaluatorTest.java | JUnit test class `PolicyEvaluatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/RuleEvaluatorTest.java | JUnit test class `RuleEvaluatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/SubjectCacheTest.java | JUnit test class `SubjectCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/SubjectContextTest.java | JUnit test class `SubjectContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/policyevaluator/TaskResourceContextTest.java | JUnit test class `TaskResourceContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/saml/MockSamlIdpTest.java | JUnit test class `MockSamlIdpTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/session/RedisSessionStoreTest.java | JUnit test class `RedisSessionStoreTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/session/SessionCookieUtilTest.java | JUnit test class `SessionCookieUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/session/SessionServiceTest.java | JUnit test class `SessionServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/session/SessionStoreContractTest.java | JUnit test class `SessionStoreContractTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/security/session/SessionTimeoutResolverTest.java | JUnit test class `SessionTimeoutResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/socket/OpenMetadataAssetServletTest.java | JUnit test class `OpenMetadataAssetServletTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/socket/SocketAddressFilterTest.java | JUnit test class `SocketAddressFilterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/socket/WebSocketManagerTest.java | JUnit test class `WebSocketManagerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/tasks/DataAccessRequestWorkflowTest.java | JUnit test class `DataAccessRequestWorkflowTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/tasks/TaskFormExecutionResolverTest.java | JUnit test class `TaskFormExecutionResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/tasks/TaskFormSchemaValidatorTest.java | JUnit test class `TaskFormSchemaValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/tasks/TaskWorkflowHandlerTest.java | JUnit test class `TaskWorkflowHandlerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/tasks/TaskWorkflowLifecycleResolverTest.java | JUnit test class `TaskWorkflowLifecycleResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/AsciiTableTest.java | JUnit test class `AsciiTableTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/AsyncServiceTest.java | JUnit test class `AsyncServiceTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/AwsCredentialsUtilTest.java | JUnit test class `AwsCredentialsUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/DescriptionSanitizerTest.java | JUnit test class `DescriptionSanitizerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/EntityETagTest.java | JUnit test class `EntityETagTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/EntityFieldUtilsTest.java | JUnit test class `EntityFieldUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/EntityUtilTest.java | JUnit test class `EntityUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/FeedUtilsTest.java | JUnit test class `FeedUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/FieldPathUtilsTest.java | JUnit test class `FieldPathUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/FullyQualifiedNameTest.java | JUnit test class `FullyQualifiedNameTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/IntakeFormValidatorTest.java | JUnit test class `IntakeFormValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/JsonPatchUtilsTest.java | JUnit test class `JsonPatchUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/JsonUtilsTest.java | JUnit test class `JsonUtilsTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/LambdaExceptionUtilTest.java | JUnit test class `LambdaExceptionUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/LikeEscapeTest.java | JUnit test class `LikeEscapeTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/LineageUtilTest.java | JUnit test class `LineageUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/LoginAttemptCacheTest.java | JUnit test class `LoginAttemptCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/ODCSConverterTest.java | JUnit test class `ODCSConverterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/ODPSConverterTest.java | JUnit test class `ODPSConverterTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/OpenMetadataConnectionBuilderTest.java | JUnit test class `OpenMetadataConnectionBuilderTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/OpenMetadataOperationsSmartReindexTest.java | JUnit test class `OpenMetadataOperationsSmartReindexTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/OrphanTestCaseCleanupTest.java | JUnit test class `OrphanTestCaseCleanupTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/PasswordUtilTest.java | JUnit test class `PasswordUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/RequestEntityCacheTest.java | JUnit test class `RequestEntityCacheTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/RestUtilTest.java | JUnit test class `RestUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/SchemaFieldExtractorTest.java | JUnit test class `SchemaFieldExtractorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/SubscriptionUtilTest.java | JUnit test class `SubscriptionUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/TableUtilTest.java | JUnit test class `TableUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/URLValidatorTest.java | JUnit test class `URLValidatorTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/UserUtilTest.java | JUnit test class `UserUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/UtilitiesTest.java | JUnit test class `UtilitiesTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/ValidationHttpUtilTest.java | JUnit test class `ValidationHttpUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/ValidatorUtilTest.java | JUnit test class `ValidatorUtilTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/branding/MessageBrandingResolverTest.java | JUnit test class `MessageBrandingResolverTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/dbtune/DbTuneReportTest.java | JUnit test class `DbTuneReportTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/dbtune/DiagnosticReportTest.java | JUnit test class `DiagnosticReportTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/dbtune/MysqlAutoTunerTest.java | JUnit test class `MysqlAutoTunerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/dbtune/PostgresAutoTunerTest.java | JUnit test class `PostgresAutoTunerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/jdbi/DatabaseAuthenticationProviderFactoryTest.java | JUnit test class `DatabaseAuthenticationProviderFactoryTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/util/jdbi/OMSqlLoggerTest.java | JUnit test class `OMSqlLoggerTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/workflows/searchIndex/PaginatedEntityTimeSeriesSourceStaleRelationshipTest.java | JUnit test class `PaginatedEntityTimeSeriesSourceStaleRelationshipTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/workflows/searchIndex/ReindexingUtilDocBuildContextTest.java | JUnit test class `ReindexingUtilDocBuildContextTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/workflows/searchIndex/ReindexingUtilStaleRelationshipTest.java | JUnit test class `ReindexingUtilStaleRelationshipTest.java` | High |
| openmetadata-service/src/test/java/org/openmetadata/service/workflows/searchIndex/ReindexingUtilTest.java | JUnit test class `ReindexingUtilTest.java` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Auth/SSOAuthentication.spec.ts | Frontend test `SSOAuthentication.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Auth/SSOLogin.spec.ts | Frontend test `SSOLogin.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Auth/SSORenewal.spec.ts | Frontend test `SSORenewal.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ActivityAPI.spec.ts | Frontend test `ActivityAPI.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ActivityFeed.spec.ts | Frontend test `ActivityFeed.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ActivityFeedTabBadge.spec.ts | Frontend test `ActivityFeedTabBadge.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ActivityStream.spec.ts | Frontend test `ActivityStream.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/AdvancedSearch.spec.ts | Frontend test `AdvancedSearch.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/AdvancedSearchSuggestions.spec.ts | Frontend test `AdvancedSearchSuggestions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Announcements/AnnouncementEntity.spec.ts | Frontend test `AnnouncementEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/AutoPilot.spec.ts | Frontend test `AutoPilot.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/BulkEditEntity.spec.ts | Frontend test `BulkEditEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/BulkImport.spec.ts | Frontend test `BulkImport.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/BulkImportWithDotInName.spec.ts | Frontend test `BulkImportWithDotInName.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CertificationDropdown.spec.ts | Frontend test `CertificationDropdown.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ChangeSummaryBadge.spec.ts | Frontend test `ChangeSummaryBadge.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ColumnBulkOperations.spec.ts | Frontend test `ColumnBulkOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ColumnSorting.spec.ts | Frontend test `ColumnSorting.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Container.spec.ts | Frontend test `Container.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ContextCenter.spec.ts | Frontend test `ContextCenter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CronValidations.spec.ts | Frontend test `CronValidations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CuratedAssets.spec.ts | Frontend test `CuratedAssets.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CustomMetric.spec.ts | Frontend test `CustomMetric.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CustomizeDetailPage.spec.ts | Frontend test `CustomizeDetailPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/CustomizeNavigationNewItems.spec.ts | Frontend test `CustomizeNavigationNewItems.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Dashboards.spec.ts | Frontend test `Dashboards.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataAssetRulesDisabled.spec.ts | Frontend test `DataAssetRulesDisabled.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataAssetRulesEnabled.spec.ts | Frontend test `DataAssetRulesEnabled.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataProductDomainMigration.spec.ts | Frontend test `DataProductDomainMigration.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataProductPersonaCustomization.spec.ts | Frontend test `DataProductPersonaCustomization.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataProductRename.spec.ts | Frontend test `DataProductRename.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataProductRenameConsolidation.spec.ts | Frontend test `DataProductRenameConsolidation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/AddTestCaseNewFlow.spec.ts | Frontend test `AddTestCaseNewFlow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/BundleSuiteBulkOperations.spec.ts | Frontend test `BundleSuiteBulkOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/CertificationFilter.spec.ts | Frontend test `CertificationFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/ColumnLevelTests.spec.ts | Frontend test `ColumnLevelTests.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/DataObservabilityGovernanceTab.spec.ts | Frontend test `DataObservabilityGovernanceTab.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/DataQuality.spec.ts | Frontend test `DataQuality.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/DataQualityDashboard.spec.ts | Frontend test `DataQualityDashboard.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/DataQualityPermissions.spec.ts | Frontend test `DataQualityPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/Dimensionality.spec.ts | Frontend test `Dimensionality.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/IncidentManagerAfterSoftDelete.spec.ts | Frontend test `IncidentManagerAfterSoftDelete.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/IncidentManagerDateFilter.spec.ts | Frontend test `IncidentManagerDateFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/Profiler.spec.ts | Frontend test `Profiler.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/ProfilerIngestionForm.spec.ts | Frontend test `ProfilerIngestionForm.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TableLevelTests.spec.ts | Frontend test `TableLevelTests.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestCaseImportExportBasic.spec.ts | Frontend test `TestCaseImportExportBasic.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestCaseImportExportE2eFlow.spec.ts | Frontend test `TestCaseImportExportE2eFlow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestCaseIncidentPermissions.spec.ts | Frontend test `TestCaseIncidentPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestCaseResultPermissions.spec.ts | Frontend test `TestCaseResultPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestCaseStatusAfterReindex.spec.ts | Frontend test `TestCaseStatusAfterReindex.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestDefinitionFilters.spec.ts | Frontend test `TestDefinitionFilters.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestDefinitionPermissions.spec.ts | Frontend test `TestDefinitionPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestLibrary.spec.ts | Frontend test `TestLibrary.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestSuiteListAfterReindex.spec.ts | Frontend test `TestSuiteListAfterReindex.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DataQuality/TestSuiteSummaryAfterReindex.spec.ts | Frontend test `TestSuiteSummaryAfterReindex.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DescriptionSuggestion.spec.ts | Frontend test `DescriptionSuggestion.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DomainFilterQueryFilter.spec.ts | Frontend test `DomainFilterQueryFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/DomainTierCertificationVoting.spec.ts | Frontend test `DomainTierCertificationVoting.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/EntityRenameConsolidation.spec.ts | Frontend test `EntityRenameConsolidation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/EntityRightCollapsablePanel.spec.ts | Frontend test `EntityRightCollapsablePanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/EntitySummaryPanel.spec.ts | Frontend test `EntitySummaryPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ExploreQuickFilters.spec.ts | Frontend test `ExploreQuickFilters.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ExploreSortOrderFilter.spec.ts | Frontend test `ExploreSortOrderFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/FailedTestCaseSampleData.spec.ts | Frontend test `FailedTestCaseSampleData.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/GlobalPageSize.spec.ts | Frontend test `GlobalPageSize.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/GlobalSearchSuggestions.spec.ts | Frontend test `GlobalSearchSuggestions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryAdvancedOperations.spec.ts | Frontend test `GlossaryAdvancedOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryAssets.spec.ts | Frontend test `GlossaryAssets.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryBulkOperations.spec.ts | Frontend test `GlossaryBulkOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryCRUDOperations.spec.ts | Frontend test `GlossaryCRUDOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryExpandAllWithStatusFilter.spec.ts | Frontend test `GlossaryExpandAllWithStatusFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryHierarchy.spec.ts | Frontend test `GlossaryHierarchy.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryMiscOperations.spec.ts | Frontend test `GlossaryMiscOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryMutualExclusivity.spec.ts | Frontend test `GlossaryMutualExclusivity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryNavigation.spec.ts | Frontend test `GlossaryNavigation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryP2Tests.spec.ts | Frontend test `GlossaryP2Tests.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryP3Tests.spec.ts | Frontend test `GlossaryP3Tests.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryPagination.spec.ts | Frontend test `GlossaryPagination.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryPersonaCustomization.spec.ts | Frontend test `GlossaryPersonaCustomization.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryRelationsGraph.spec.ts | Frontend test `GlossaryRelationsGraph.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryRelationsGraphPerf.spec.ts | Frontend test `GlossaryRelationsGraphPerf.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryRemoveOperations.spec.ts | Frontend test `GlossaryRemoveOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryStatusFilterLargeDataset.spec.ts | Frontend test `GlossaryStatusFilterLargeDataset.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryStatusFilterNestedTerms.spec.ts | Frontend test `GlossaryStatusFilterNestedTerms.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryTermDetails.spec.ts | Frontend test `GlossaryTermDetails.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryTermRelatedTerms.spec.ts | Frontend test `GlossaryTermRelatedTerms.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryTermRelationsGraph.spec.ts | Frontend test `GlossaryTermRelationsGraph.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryTermRelationsGraphNested.spec.ts | Frontend test `GlossaryTermRelationsGraphNested.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryVoting.spec.ts | Frontend test `GlossaryVoting.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/GlossaryWorkflow.spec.ts | Frontend test `GlossaryWorkflow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/LargeGlossaryPerformance.spec.ts | Frontend test `LargeGlossaryPerformance.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Glossary/MUIGlossaryMutualExclusivity.spec.ts | Frontend test `MUIGlossaryMutualExclusivity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/ImpactAnalysis.spec.ts | Frontend test `ImpactAnalysis.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/IncidentManager.spec.ts | Frontend test `IncidentManager.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/KnowledgeCenter.spec.ts | Frontend test `KnowledgeCenter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/KnowledgeCenterList.spec.ts | Frontend test `KnowledgeCenterList.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/KnowledgeCenterRoleBasedAccess.spec.ts | Frontend test `KnowledgeCenterRoleBasedAccess.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/KnowledgeCenterTextEditor.spec.ts | Frontend test `KnowledgeCenterTextEditor.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/KnowledgeGraph.spec.ts | Frontend test `KnowledgeGraph.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LandingPageWidgets/DataAssetsWidget.spec.ts | Frontend test `DataAssetsWidget.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LandingPageWidgets/DomainDataProductsWidgets.spec.ts | Frontend test `DomainDataProductsWidgets.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LandingPageWidgets/DomainWidgetFilter.spec.ts | Frontend test `DomainWidgetFilter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LandingPageWidgets/FollowingWidget.spec.ts | Frontend test `FollowingWidget.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LanguageOverride.spec.ts | Frontend test `LanguageOverride.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/LineagePipelineAnnotator.spec.ts | Frontend test `LineagePipelineAnnotator.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Markdown.spec.ts | Frontend test `Markdown.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/McpChat.spec.ts | Frontend test `McpChat.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/MetricCustomUnitFlow.spec.ts | Frontend test `MetricCustomUnitFlow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/MultipleRename.spec.ts | Frontend test `MultipleRename.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/MutuallyExclusiveColumnTags.spec.ts | Frontend test `MutuallyExclusiveColumnTags.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/NavigationBlocker.spec.ts | Frontend test `NavigationBlocker.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/NestedColumnsExpandCollapse.spec.ts | Frontend test `NestedColumnsExpandCollapse.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OnlineUsers.spec.ts | Frontend test `OnlineUsers.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorer.spec.ts | Frontend test `OntologyExplorer.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerCardinality.spec.ts | Frontend test `OntologyExplorerCardinality.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerE2E.spec.ts | Frontend test `OntologyExplorerE2E.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerFilters.spec.ts | Frontend test `OntologyExplorerFilters.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerIntegration.spec.ts | Frontend test `OntologyExplorerIntegration.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerInteractions.spec.ts | Frontend test `OntologyExplorerInteractions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerIsolatedToggle.spec.ts | Frontend test `OntologyExplorerIsolatedToggle.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/OntologyExplorerRdf.spec.ts | Frontend test `OntologyExplorerRdf.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Pagination.spec.ts | Frontend test `Pagination.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permission.spec.ts | Frontend test `Permission.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permissions/DataProductPermissions.spec.ts | Frontend test `DataProductPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permissions/DomainPermissions.spec.ts | Frontend test `DomainPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permissions/EntityPermissions.spec.ts | Frontend test `EntityPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permissions/GlossaryPermissions.spec.ts | Frontend test `GlossaryPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Permissions/ServiceEntityPermissions.spec.ts | Frontend test `ServiceEntityPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/QueryEntity.spec.ts | Frontend test `QueryEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/RTL.spec.ts | Frontend test `RTL.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/RecentlyViewed.spec.ts | Frontend test `RecentlyViewed.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/RestoreEntityInheritedFields.spec.ts | Frontend test `RestoreEntityInheritedFields.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SSOConfiguration.spec.ts | Frontend test `SSOConfiguration.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SampleDataDomainDataProduct.spec.ts | Frontend test `SampleDataDomainDataProduct.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SampleDataTableOperations.spec.ts | Frontend test `SampleDataTableOperations.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SchemaDefinition.spec.ts | Frontend test `SchemaDefinition.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SchemaSearch.spec.ts | Frontend test `SchemaSearch.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchExport.spec.ts | Frontend test `SearchExport.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchIndexNestedColumns.spec.ts | Frontend test `SearchIndexNestedColumns.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/ApiEndpoint.spec.ts | Frontend test `ApiEndpoint.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Container.spec.ts | Frontend test `Container.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Dashboard.spec.ts | Frontend test `Dashboard.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Database.spec.ts | Frontend test `Database.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/DatabaseSchema.spec.ts | Frontend test `DatabaseSchema.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/DomainRenamePrefixCascade.spec.ts | Frontend test `DomainRenamePrefixCascade.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/ExploreFilterSeparation.spec.ts | Frontend test `ExploreFilterSeparation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/GlossaryRenameCascade.spec.ts | Frontend test `GlossaryRenameCascade.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/GlossaryRenamePrefixCascade.spec.ts | Frontend test `GlossaryRenamePrefixCascade.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Metric.spec.ts | Frontend test `Metric.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/MlModel.spec.ts | Frontend test `MlModel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Pipeline.spec.ts | Frontend test `Pipeline.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/StoredProcedure.spec.ts | Frontend test `StoredProcedure.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SearchSeparation/Topic.spec.ts | Frontend test `Topic.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SettingsNavigationPage.spec.ts | Frontend test `SettingsNavigationPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/SystemCertificationTags.spec.ts | Frontend test `SystemCertificationTags.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Table.spec.ts | Frontend test `Table.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TableConstraint.spec.ts | Frontend test `TableConstraint.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TableSearch.spec.ts | Frontend test `TableSearch.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TableSorting.spec.ts | Frontend test `TableSorting.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TagsSuggestion.spec.ts | Frontend test `TagsSuggestion.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks.spec.ts | Frontend test `Tasks.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/ActivityFeed.spec.ts | Frontend test `ActivityFeed.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/DomainFiltering.spec.ts | Frontend test `DomainFiltering.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskAllEntities.spec.ts | Frontend test `TaskAllEntities.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskAssigneeManagement.spec.ts | Frontend test `TaskAssigneeManagement.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskComments.spec.ts | Frontend test `TaskComments.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskContainerEntity.spec.ts | Frontend test `TaskContainerEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskCreation.spec.ts | Frontend test `TaskCreation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskCustomFormWorkflow.spec.ts | Frontend test `TaskCustomFormWorkflow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskDashboardEntity.spec.ts | Frontend test `TaskDashboardEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskEntityResolution.spec.ts | Frontend test `TaskEntityResolution.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskNavigation.spec.ts | Frontend test `TaskNavigation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskNestedFields.spec.ts | Frontend test `TaskNestedFields.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskPermissions.spec.ts | Frontend test `TaskPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskPipelineEntity.spec.ts | Frontend test `TaskPipelineEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskResolution.spec.ts | Frontend test `TaskResolution.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskSuggestionAPIs.spec.ts | Frontend test `TaskSuggestionAPIs.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TaskTopicEntity.spec.ts | Frontend test `TaskTopicEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Tasks/TeamActivity.spec.ts | Frontend test `TeamActivity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TeamSubscriptions.spec.ts | Frontend test `TeamSubscriptions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TeamsDragAndDrop.spec.ts | Frontend test `TeamsDragAndDrop.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TeamsHierarchy.spec.ts | Frontend test `TeamsHierarchy.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TestSuiteMultiPipeline.spec.ts | Frontend test `TestSuiteMultiPipeline.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/TestSuitePipelineRedeploy.spec.ts | Frontend test `TestSuitePipelineRedeploy.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Topic.spec.ts | Frontend test `Topic.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/UserProfileOnlineStatus.spec.ts | Frontend test `UserProfileOnlineStatus.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Workflows/NoOpWorkflowNodeConfig.spec.ts | Frontend test `NoOpWorkflowNodeConfig.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Features/Workflows/WorkflowOssRestrictions.spec.ts | Frontend test `WorkflowOssRestrictions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/AddRoleAndAssignToUser.spec.ts | Frontend test `AddRoleAndAssignToUser.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ApiCollection.spec.ts | Frontend test `ApiCollection.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ApiDocs.spec.ts | Frontend test `ApiDocs.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ApiServiceRest.spec.ts | Frontend test `ApiServiceRest.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/AppBasic.spec.ts | Frontend test `AppBasic.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/Collect.spec.ts | Frontend test `Collect.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ConditionalPermissions.spec.ts | Frontend test `ConditionalPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/CustomizeLandingPage.spec.ts | Frontend test `CustomizeLandingPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/CustomizeWidgets.spec.ts | Frontend test `CustomizeWidgets.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ExploreAggregationCountsMatching.spec.ts | Frontend test `ExploreAggregationCountsMatching.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ExploreDiscovery.spec.ts | Frontend test `ExploreDiscovery.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/FrequentlyJoined.spec.ts | Frontend test `FrequentlyJoined.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/GlobalSearch.spec.ts | Frontend test `GlobalSearch.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/IngestionBot.spec.ts | Frontend test `IngestionBot.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/LineageSettings.spec.ts | Frontend test `LineageSettings.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/Metric.spec.ts | Frontend test `Metric.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/MetricSearch.spec.ts | Frontend test `MetricSearch.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/Navbar.spec.ts | Frontend test `Navbar.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/NestedChildrenUpdates.spec.ts | Frontend test `NestedChildrenUpdates.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/NotificationAlerts.spec.ts | Frontend test `NotificationAlerts.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ObservabilityAlerts.spec.ts | Frontend test `ObservabilityAlerts.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/PersonaDeletionUserProfile.spec.ts | Frontend test `PersonaDeletionUserProfile.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/PersonaFlow.spec.ts | Frontend test `PersonaFlow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/PlatformLineage.spec.ts | Frontend test `PlatformLineage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/SchemaTable.spec.ts | Frontend test `SchemaTable.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/SearchRBAC.spec.ts | Frontend test `SearchRBAC.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ServiceCreationPermissions.spec.ts | Frontend test `ServiceCreationPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ServiceDocPanel.spec.ts | Frontend test `ServiceDocPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/ServiceForm.spec.ts | Frontend test `ServiceForm.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/Tour.spec.ts | Frontend test `Tour.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Flow/UsersPagination.spec.ts | Frontend test `UsersPagination.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Http2/SmokeH2.spec.ts | Frontend test `SmokeH2.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/AppStopRunModal.spec.ts | Frontend test `AppStopRunModal.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/AuditLogs.spec.ts | Frontend test `AuditLogs.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Bots.spec.ts | Frontend test `Bots.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/CSVImportWithQuotesAndCommas.spec.ts | Frontend test `CSVImportWithQuotesAndCommas.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ClassificationConditionalRendering.spec.ts | Frontend test `ClassificationConditionalRendering.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/CustomProperties.spec.ts | Frontend test `CustomProperties.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/CustomThemeConfig.spec.ts | Frontend test `CustomThemeConfig.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataContractInheritance.spec.ts | Frontend test `DataContractInheritance.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataContracts.spec.ts | Frontend test `DataContracts.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataContractsSemanticRules.spec.ts | Frontend test `DataContractsSemanticRules.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataInsight.spec.ts | Frontend test `DataInsight.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataInsightReportApplication.spec.ts | Frontend test `DataInsightReportApplication.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataInsightSettings.spec.ts | Frontend test `DataInsightSettings.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataMarketplace.spec.ts | Frontend test `DataMarketplace.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataMarketplaceAnnouncements.spec.ts | Frontend test `DataMarketplaceAnnouncements.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataMarketplacePermissions.spec.ts | Frontend test `DataMarketplacePermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataProductAndSubdomains.spec.ts | Frontend test `DataProductAndSubdomains.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DataProducts.spec.ts | Frontend test `DataProducts.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DescriptionVisibility.spec.ts | Frontend test `DescriptionVisibility.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DomainAdvanced.spec.ts | Frontend test `DomainAdvanced.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DomainDataProductsRightPanel.spec.ts | Frontend test `DomainDataProductsRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/DomainUIInteractions.spec.ts | Frontend test `DomainUIInteractions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Domains.spec.ts | Frontend test `Domains.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Entity.spec.ts | Frontend test `Entity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/EntityDataConsumer.spec.ts | Frontend test `EntityDataConsumer.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/EntityDataSteward.spec.ts | Frontend test `EntityDataSteward.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ExplorePageRightPanel.spec.ts | Frontend test `ExplorePageRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ExplorePageRightPanel_KnowledgeCenter.spec.ts | Frontend test `ExplorePageRightPanel_KnowledgeCenter.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ExploreTree.spec.ts | Frontend test `ExploreTree.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Glossary.spec.ts | Frontend test `Glossary.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/GlossaryFormValidation.spec.ts | Frontend test `GlossaryFormValidation.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/GlossaryImportExport.spec.ts | Frontend test `GlossaryImportExport.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/GlossaryTermRightPanel.spec.ts | Frontend test `GlossaryTermRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/HealthCheck.spec.ts | Frontend test `HealthCheck.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/InputOutputPorts.spec.ts | Frontend test `InputOutputPorts.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/IntakeForm.spec.ts | Frontend test `IntakeForm.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/LearningResources.spec.ts | Frontend test `LearningResources.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/DataAssetLineage.spec.ts | Frontend test `DataAssetLineage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/LineageControls.spec.ts | Frontend test `LineageControls.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/LineageFilters.spec.ts | Frontend test `LineageFilters.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/LineageInteraction.spec.ts | Frontend test `LineageInteraction.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/LineageNodePagination.spec.ts | Frontend test `LineageNodePagination.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/LineageRightPanel.spec.ts | Frontend test `LineageRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Lineage/PlatformLineage.spec.ts | Frontend test `PlatformLineage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/LiveIndexingTab.spec.ts | Frontend test `LiveIndexingTab.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Login.spec.ts | Frontend test `Login.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/LoginConfiguration.spec.ts | Frontend test `LoginConfiguration.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/LogsViewer.spec.ts | Frontend test `LogsViewer.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ODCSImportExport.spec.ts | Frontend test `ODCSImportExport.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ODCSImportExportPermissions.spec.ts | Frontend test `ODCSImportExportPermissions.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/OmdURLConfiguration.spec.ts | Frontend test `OmdURLConfiguration.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/PipelineExecution.spec.ts | Frontend test `PipelineExecution.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Policies.spec.ts | Frontend test `Policies.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ProfilerConfigurationPage.spec.ts | Frontend test `ProfilerConfigurationPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Roles.spec.ts | Frontend test `Roles.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/SearchIndexApplication.spec.ts | Frontend test `SearchIndexApplication.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/SearchSettings.spec.ts | Frontend test `SearchSettings.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ServiceEntity.spec.ts | Frontend test `ServiceEntity.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/ServiceListing.spec.ts | Frontend test `ServiceListing.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/SubDomainPagination.spec.ts | Frontend test `SubDomainPagination.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Tag.spec.ts | Frontend test `Tag.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TagPageRightPanel.spec.ts | Frontend test `TagPageRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Tags.spec.ts | Frontend test `Tags.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TaskComments.spec.ts | Frontend test `TaskComments.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TaskFormSettings.spec.ts | Frontend test `TaskFormSettings.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Tasks.spec.ts | Frontend test `Tasks.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TasksUIFlow.spec.ts | Frontend test `TasksUIFlow.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TeamAssetsRightPanel.spec.ts | Frontend test `TeamAssetsRightPanel.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Teams.spec.ts | Frontend test `Teams.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TestSuite.spec.ts | Frontend test `TestSuite.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/TestSuiteDetailsPage.spec.ts | Frontend test `TestSuiteDetailsPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/UserCreationWithPersona.spec.ts | Frontend test `UserCreationWithPersona.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/UserDetails.spec.ts | Frontend test `UserDetails.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Pages/Users.spec.ts | Frontend test `Users.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/Search/SearchNightly.spec.ts | Frontend test `SearchNightly.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/VersionPages/ClassificationVersionPage.spec.ts | Frontend test `ClassificationVersionPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/VersionPages/EntityVersionPages.spec.ts | Frontend test `EntityVersionPages.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/VersionPages/GlossaryVersionPage.spec.ts | Frontend test `GlossaryVersionPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/VersionPages/ServiceEntityVersionPage.spec.ts | Frontend test `ServiceEntityVersionPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/VersionPages/TestCaseVersionPage.spec.ts | Frontend test `TestCaseVersionPage.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/nightly/AutoClassification.spec.ts | Frontend test `AutoClassification.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/playwright/e2e/nightly/ServiceIngestion.spec.ts | Frontend test `ServiceIngestion.spec.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/App.test.tsx | Frontend test `App.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/APIEndpoint/APIEndpointDetails/APIEndpointDetails.test.tsx | Frontend test `APIEndpointDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCard/ActivityFeedCard.test.tsx | Frontend test `ActivityFeedCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCard/FeedCardBody/FeedCardBody.test.tsx | Frontend test `FeedCardBody.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCard/FeedCardFooter/FeedCardFooter.test.tsx | Frontend test `FeedCardFooter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCard/FeedCardHeader/FeedCardHeader.test.tsx | Frontend test `FeedCardHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCard/PopoverContent.test.tsx | Frontend test `PopoverContent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardNew/CommentCard.test.tsx | Frontend test `CommentCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardBody/DescriptionFeed/ActivityDescriptionFeed.test.tsx | Frontend test `ActivityDescriptionFeed.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardBody/DescriptionFeed/DescriptionFeed.test.tsx | Frontend test `DescriptionFeed.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardBody/OwnerFeed/ActivityOwnersFeed.test.tsx | Frontend test `ActivityOwnersFeed.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardBody/TagsFeed/ActivityTagsFeed.test.tsx | Frontend test `ActivityTagsFeed.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardBody/TestCaseFeed/TestCaseFeed.test.tsx | Frontend test `TestCaseFeed.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardFooter/ActivityEventFooter.test.tsx | Frontend test `ActivityEventFooter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedCardV2/FeedCardHeader/FeedCardHeaderV2.test.tsx | Frontend test `FeedCardHeaderV2.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedEditor/ActivityFeedEditor.test.tsx | Frontend test `ActivityFeedEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedEditor/KeyHelp.test.tsx | Frontend test `KeyHelp.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedEditor/SendButton.test.tsx | Frontend test `SendButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedPanel/FeedPanelHeader.test.tsx | Frontend test `FeedPanelHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedProvider/ActivityFeedProvider.test.tsx | Frontend test `ActivityFeedProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityFeedTab/ActivityFeedTab.component.test.tsx | Frontend test `ActivityFeedTab.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityThreadPanel/ActivityThread.test.tsx | Frontend test `ActivityThread.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityThreadPanel/ActivityThreadList.test.tsx | Frontend test `ActivityThreadList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityThreadPanel/ActivityThreadPanel.test.tsx | Frontend test `ActivityThreadPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/ActivityThreadPanel/ActivityThreadPanelBody.test.tsx | Frontend test `ActivityThreadPanelBody.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/FeedEditor/FeedEditor.test.tsx | Frontend test `FeedEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/FeedListSeparator/FeedListSeparator.test.tsx | Frontend test `FeedListSeparator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/Reactions/Emoji.test.tsx | Frontend test `Emoji.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/Reactions/Reaction.test.tsx | Frontend test `Reaction.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/Reactions/Reactions.test.tsx | Frontend test `Reactions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/Shared/ActivityFeedActions.test.tsx | Frontend test `ActivityFeedActions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/TaskFeedCard/TaskFeedCard.component.test.tsx | Frontend test `TaskFeedCard.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/TaskFeedCard/TaskFeedCardFromTask.component.test.tsx | Frontend test `TaskFeedCardFromTask.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ActivityFeed/TaskFeedCard/TaskFeedCardNew.component.test.tsx | Frontend test `TaskFeedCardNew.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AlertBar/AlertBar.test.tsx | Frontend test `AlertBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/AlertDetails/AlertConfigDetails/AlertConfigDetails.test.tsx | Frontend test `AlertConfigDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/AlertDetails/AlertDiagnosticInfo/AlertDiagnosticInfoTab.test.tsx | Frontend test `AlertDiagnosticInfoTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/AlertDetails/AlertRecentEventsTab/AlertRecentEventsTab.test.tsx | Frontend test `AlertRecentEventsTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/AlertFormSourceItem/AlertFormSourceItem.test.tsx | Frontend test `AlertFormSourceItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/DestinationFormItem/DestinationFormItem.test.tsx | Frontend test `DestinationFormItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/DestinationFormItem/DestinationSelectItem/DestinationSelectItem.test.tsx | Frontend test `DestinationSelectItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/DestinationFormItem/TeamAndUserSelectItem/TeamAndUserSelectItem.test.tsx | Frontend test `TeamAndUserSelectItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/FQNListSelect/FQNListSelect.component.test.tsx | Frontend test `FQNListSelect.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/ObservabilityFormFiltersItem/ObservabilityFormFiltersItem.test.tsx | Frontend test `ObservabilityFormFiltersItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Alerts/ObservabilityFormTriggerItem/ObservabilityFormTriggerItem.test.tsx | Frontend test `ObservabilityFormTriggerItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Announcement/AnnouncementFeedCard.test.tsx | Frontend test `AnnouncementFeedCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Announcement/AnnouncementFeedCardBody.test.tsx | Frontend test `AnnouncementFeedCardBody.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Announcement/AnnouncementThreadBody.test.tsx | Frontend test `AnnouncementThreadBody.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Announcement/AnnouncementThreads.test.tsx | Frontend test `AnnouncementThreads.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppBar/Suggestions.test.tsx | Frontend test `Suggestions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/AppRouter.test.tsx | Frontend test `AppRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/ClassificationRouter.test.tsx | Frontend test `ClassificationRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/DomainRouter.test.tsx | Frontend test `DomainRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/EntityImportRouter.test.tsx | Frontend test `EntityImportRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/EntityRouter.test.tsx | Frontend test `EntityRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/GlossaryRouter/GlossaryRouter.test.tsx | Frontend test `GlossaryRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/GlossaryTermRouter/GlossaryTermRouter.test.tsx | Frontend test `GlossaryTermRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppRouter/SettingsRouter.test.tsx | Frontend test `SettingsRouter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AppTour/AppTour.test.tsx | Frontend test `AppTour.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AuditLog/AuditLogFilters.test.tsx | Frontend test `AuditLogFilters.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/AuditLog/AuditLogList.test.tsx | Frontend test `AuditLogList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppAuthenticators/Auth0Authenticator.test.tsx | Frontend test `Auth0Authenticator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppAuthenticators/BasicAuthAuthenticator.test.tsx | Frontend test `BasicAuthAuthenticator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppAuthenticators/GenericAuthenticator.test.tsx | Frontend test `GenericAuthenticator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppAuthenticators/MsalAuthenticator.test.tsx | Frontend test `MsalAuthenticator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppAuthenticators/OktaAuthenticator.test.tsx | Frontend test `OktaAuthenticator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AppCallbacks/Auth0Callback/Auth0Callback.test.tsx | Frontend test `Auth0Callback.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AuthProviders/AuthProvider.test.tsx | Frontend test `AuthProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Auth/AuthProviders/OktaAuthProvider.test.tsx | Frontend test `OktaAuthProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/BlockMenu/BlockMenu.test.tsx | Frontend test `BlockMenu.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/BubbleMenu/BubbleMenu.test.tsx | Frontend test `BubbleMenu.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/BlockAndDragDrop/BlockAndDragDrop.test.tsx | Frontend test `BlockAndDragDrop.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/BlockAndDragDrop/BlockAndDragHandle.test.ts | Frontend test `BlockAndDragHandle.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/BlockAndDragDrop/helpers.test.ts | Frontend test `helpers.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/Callout/Callout.test.ts | Frontend test `Callout.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/Callout/CalloutComponent.test.tsx | Frontend test `CalloutComponent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/File/AttachmentComponents/AttachmentPlaceholder.test.tsx | Frontend test `AttachmentPlaceholder.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/File/AttachmentComponents/FileAttachment.test.tsx | Frontend test `FileAttachment.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/File/AttachmentComponents/ImageAttachment.test.tsx | Frontend test `ImageAttachment.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/File/FileNode.test.ts | Frontend test `FileNode.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/diff-view.test.ts | Frontend test `diff-view.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/hashtag/HashList.test.tsx | Frontend test `HashList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/image/EmbedLinkElement/EmbedLinkElement.test.tsx | Frontend test `EmbedLinkElement.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/Extensions/mention/MentionList.test.tsx | Frontend test `MentionList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/TableMenu/TableMenu.test.tsx | Frontend test `TableMenu.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BlockEditor/hooks/useCustomEditor.test.ts | Frontend test `useCustomEditor.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/BulkEditEntity/BulkEditEntity.test.tsx | Frontend test `BulkEditEntity.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Chart/ChartDetails/ChartDetails.component.test.tsx | Frontend test `ChartDetails.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Classifications/ClassificationDetails/ClassificationDetails.test.tsx | Frontend test `ClassificationDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Container/ContainerChildren/ContainerChildren.test.tsx | Frontend test `ContainerChildren.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Container/ContainerDataModel/ContainerDataModel.test.tsx | Frontend test `ContainerDataModel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Container/ContainerVersion/ContainerVersion.test.tsx | Frontend test `ContainerVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/ArticleDetailHeader/ArticleDetailHeader.test.tsx | Frontend test `ArticleDetailHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/ContextCenterHeader/ContextCenterHeader.test.tsx | Frontend test `ContextCenterHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/CreateFolderModal/CreateFolderModal.test.tsx | Frontend test `CreateFolderModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/CreateMemoryModal/CreateMemoryModal.test.tsx | Frontend test `CreateMemoryModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/DocumentStatusBadge/DocumentStatusBadge.test.tsx | Frontend test `DocumentStatusBadge.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/DocumentsView/DocumentFolderView.test.tsx | Frontend test `DocumentFolderView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/DocumentsView/DocumentPreviewPanel.test.tsx | Frontend test `DocumentPreviewPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/DocumentsView/DocumentsView.test.tsx | Frontend test `DocumentsView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/MemoriesView/MemoriesView.test.tsx | Frontend test `MemoriesView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ContextCenter/UploadDocumentModal/UploadDocumentModal.test.tsx | Frontend test `UploadDocumentModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/CopyLinkButton/CopyLinkButton.test.tsx | Frontend test `CopyLinkButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Customization/GenericProvider/GenericProvider.test.tsx | Frontend test `GenericProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Customization/GenericTab/DynamicHeightWidget.test.tsx | Frontend test `DynamicHeightWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Customization/GenericTab/GenericTab.test.tsx | Frontend test `GenericTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Dashboard/DashboardDetails/DashboardDetails.component.test.tsx | Frontend test `DashboardDetails.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Dashboard/DashboardVersion/DashboardVersion.test.tsx | Frontend test `DashboardVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Dashboard/DataModel/DataModels/DataModelDetails.component.test.tsx | Frontend test `DataModelDetails.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataAssetSummaryPanelV1/DataAssetSummaryPanelV1.test.tsx | Frontend test `DataAssetSummaryPanelV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataAssets/CommonWidgets/CommonWidgets.test.tsx | Frontend test `CommonWidgets.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataAssets/DataAssetAsyncSelectList/DataAssetAsyncSelectList.test.tsx | Frontend test `DataAssetAsyncSelectList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataAssets/DataAssetsHeader/DataAssetsHeader.test.tsx | Frontend test `DataAssetsHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/AddDataContract/AddDataContract.test.tsx | Frontend test `AddDataContract.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractDetailFormTab/ContractDetailFormTab.test.tsx | Frontend test `ContractDetailFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractDetailTab/ContractDetail.test.tsx | Frontend test `ContractDetail.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractExecutionChart/ContractExecutionChart.test.tsx | Frontend test `ContractExecutionChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractExecutionChart/ContractExecutionChartTooltip.test.tsx | Frontend test `ContractExecutionChartTooltip.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractQualityCard/ContractQualityCard.test.tsx | Frontend test `ContractQualityCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractQualityFormTab/ContractQualityFormTab.test.tsx | Frontend test `ContractQualityFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSLACard/ContractSLA.test.tsx | Frontend test `ContractSLA.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSLAFormTab/ContractSLAFormTab.test.tsx | Frontend test `ContractSLAFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSchemaFormTab/ContractSchemaFormTab.test.tsx | Frontend test `ContractSchemaFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSchemaTable/ContractSchemaTable.test.tsx | Frontend test `ContractSchemaTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSecurity/ContractSecurityCard.test.tsx | Frontend test `ContractSecurityCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSecurityFormTab/ContractSecurityFormTab.test.tsx | Frontend test `ContractSecurityFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSemanticFormTab/ContractSemanticFormTab.test.tsx | Frontend test `ContractSemanticFormTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractSemantics/ContractSemantics.test.tsx | Frontend test `ContractSemantics.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractTab/ContractTab.test.tsx | Frontend test `ContractTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractViewSwitchTab/ContractViewSwitchTab.test.tsx | Frontend test `ContractViewSwitchTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ContractYaml/ContractYaml.test.tsx | Frontend test `ContractYaml.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataContract/ODCSImportModal/ODCSImportModal.test.tsx | Frontend test `ODCSImportModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/DataInsightProgressBar.test.tsx | Frontend test `DataInsightProgressBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/DataInsightSummary.test.tsx | Frontend test `DataInsightSummary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/EntitySummaryProgressBar.test.tsx | Frontend test `EntitySummaryProgressBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/KPIChart.test.tsx | Frontend test `KPIChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/KPILatestResultsV1.test.tsx | Frontend test `KPILatestResultsV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataInsight/TotalEntityInsightSummary.test.tsx | Frontend test `TotalEntityInsightSummary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataMarketplace/MarketplaceGreetingBanner/MarketplaceGreetingBanner.test.tsx | Frontend test `MarketplaceGreetingBanner.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataMarketplace/MarketplaceSearchBar/MarketplaceSearchBar.test.tsx | Frontend test `MarketplaceSearchBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataProducts/DataProductDomainWidget/DataProductDomainWidget.test.tsx | Frontend test `DataProductDomainWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataProducts/DataProductsDetailsPage/DataProductsDetailsPage.component.test.tsx | Frontend test `DataProductsDetailsPage.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataProducts/DataProductsPage/DataProductsPage.component.test.tsx | Frontend test `DataProductsPage.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataProducts/DataProductsSelectList/DataProductsSelectListV1.test.tsx | Frontend test `DataProductsSelectListV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddDataQualityTest/EditTestCaseModal.test.tsx | Frontend test `EditTestCaseModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddDataQualityTest/components/AddTestSuitePipeline.test.tsx | Frontend test `AddTestSuitePipeline.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddDataQualityTest/components/EditTestCaseModalV1.test.tsx | Frontend test `EditTestCaseModalV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddDataQualityTest/components/ParameterForm.test.tsx | Frontend test `ParameterForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddDataQualityTest/components/TestCaseFormV1.test.tsx | Frontend test `TestCaseFormV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddTestCaseList/AddTestCaseList.component.test.tsx | Frontend test `AddTestCaseList.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddTestCaseList/AddTestCaseListForm.utils.test.ts | Frontend test `AddTestCaseListForm.utils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/AddToBundleSuiteModal/AddToBundleSuiteModal.test.tsx | Frontend test `AddToBundleSuiteModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/BundleSuiteForm/BundleSuiteForm.test.tsx | Frontend test `BundleSuiteForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/DataAssetsCoveragePieChartWidget/DataAssetsCoveragePieChartWidget.test.tsx | Frontend test `DataAssetsCoveragePieChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/DataStatisticWidget/DataStatisticWidget.test.tsx | Frontend test `DataStatisticWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/EntityHealthStatusPieChartWidget/EntityHealthStatusPieChartWidget.test.tsx | Frontend test `EntityHealthStatusPieChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/IncidentTimeChartWidget/IncidentTimeChartWidget.test.tsx | Frontend test `IncidentTimeChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/IncidentTypeAreaChartWidget/IncidentTypeAreaChartWidget.test.tsx | Frontend test `IncidentTypeAreaChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/StatusByDimensionCardWidget/StatusByDimensionCardWidget.test.tsx | Frontend test `StatusByDimensionCardWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/StatusCardWidget/StatusCardWidget.test.tsx | Frontend test `StatusCardWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/TestCaseStatusAreaChartWidget/TestCaseStatusAreaChartWidget.test.tsx | Frontend test `TestCaseStatusAreaChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/ChartWidgets/TestCaseStatusPieChartWidget/TestCaseStatusPieChartWidget.test.tsx | Frontend test `TestCaseStatusPieChartWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/CustomMetricForm/CustomMetricForm.test.tsx | Frontend test `CustomMetricForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/DataQualityDashboard/DataQualityDashboard.test.tsx | Frontend test `DataQualityDashboard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/DimensionalityTab/DimensionalityHeatmap/DimensionalityHeatmap.component.test.tsx | Frontend test `DimensionalityHeatmap.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/DimensionalityTab/DimensionalityHeatmap/DimensionalityHeatmap.utils.test.ts | Frontend test `DimensionalityHeatmap.utils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/DimensionalityTab/DimensionalityHeatmap/HeatmapCellTooltip.test.tsx | Frontend test `HeatmapCellTooltip.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/DimensionalityTab/DimensionalityHeatmap/useScrollIndicator.test.ts | Frontend test `useScrollIndicator.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/FailedTestCaseSampleData/FailedTestCaseSampleData.test.tsx | Frontend test `FailedTestCaseSampleData.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/IncidentManagerPageHeader/IncidentManagerPageHeader.test.tsx | Frontend test `IncidentManagerPageHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/Severity/InlineSeverity.test.tsx | Frontend test `InlineSeverity.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/Severity/Severity.test.tsx | Frontend test `Severity.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/Severity/SeverityModal.test.tsx | Frontend test `SeverityModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/TestCaseIncidentTab/TestCaseIncidentTab.test.tsx | Frontend test `TestCaseIncidentTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/TestCaseResultTab/TestCaseResultTab.test.tsx | Frontend test `TestCaseResultTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/TestCaseResultTab/TestCaseResultTabClassBase.test.ts | Frontend test `TestCaseResultTabClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/TestCaseStatus/InlineTestCaseIncidentStatus.test.tsx | Frontend test `InlineTestCaseIncidentStatus.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/IncidentManager/TestCaseStatus/TestCaseIncidentManagerStatus.test.tsx | Frontend test `TestCaseIncidentManagerStatus.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/TestCaseStatusModal/TestCaseStatusModal.test.tsx | Frontend test `TestCaseStatusModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/TestCases/TestCases.test.tsx | Frontend test `TestCases.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/TestSuite/TestSuiteList/TestSuites.test.tsx | Frontend test `TestSuites.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DataQuality/TestSuite/TestSuitePipelineTab/TestSuitePipelineTab.test.tsx | Frontend test `TestSuitePipelineTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/ColumnDetailPanel/ColumnDetailPanel.test.tsx | Frontend test `ColumnDetailPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/ColumnDetailPanel/KeyProfileMetrics/KeyProfileMetrics.test.tsx | Frontend test `KeyProfileMetrics.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/ColumnDetailPanel/NestedColumnsSection.test.tsx | Frontend test `NestedColumnsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/DataObservability/DataObservabilityTab.test.tsx | Frontend test `DataObservabilityTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/DataObservability/TabFilters/TabFilters.test.tsx | Frontend test `TabFilters.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/DataQualityTab/DataQualityTab.test.tsx | Frontend test `DataQualityTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/ProfilerDetailsCard/ProfilerDetailsCard.test.tsx | Frontend test `ProfilerDetailsCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/ProfilerLatestValue/ProfilerLatestValue.test.tsx | Frontend test `ProfilerLatestValue.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/ProfilerSettings/CustomeRangeWidget.test.tsx | Frontend test `CustomeRangeWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/ProfilerStateWrapper/ProfilerStateWrapper.test.tsx | Frontend test `ProfilerStateWrapper.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/ColumnProfileTable/ColumnProfileTable.test.tsx | Frontend test `ColumnProfileTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/CustomMetricGraphs/CustomMetricGraphs.test.tsx | Frontend test `CustomMetricGraphs.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/NoProfilerBanner/NoProfilerBanner.test.tsx | Frontend test `NoProfilerBanner.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/ProfilerClassBase.test.ts | Frontend test `ProfilerClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/ProfilerProgressWidget/ProfilerProgressWidget.test.tsx | Frontend test `ProfilerProgressWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/ProfilerSettingsModal/ProfilerSettingsModal.test.tsx | Frontend test `ProfilerSettingsModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/QualityTab/QualityTab.test.tsx | Frontend test `QualityTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/SingleColumnProfile.test.tsx | Frontend test `SingleColumnProfile.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/TableProfilerChart/TableProfilerChart.test.tsx | Frontend test `TableProfilerChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TableProfiler/TableProfilerProvider.test.tsx | Frontend test `TableProfilerProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TestSummary/TestSummary.test.tsx | Frontend test `TestSummary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TestSummary/TestSummaryGraph.test.tsx | Frontend test `TestSummaryGraph.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/Profiler/TestSummaryCustomTooltip/TestSummaryCustomTooltip.test.tsx | Frontend test `TestSummaryCustomTooltip.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/RetentionPeriod/RetentionPeriod.component.test.tsx | Frontend test `RetentionPeriod.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SampleDataTable/RowData.test.tsx | Frontend test `RowData.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SampleDataTable/SampleDataTable.test.tsx | Frontend test `SampleDataTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SampleDataTable/SampleDataTable.utils.test.ts | Frontend test `SampleDataTable.utils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SampleDataWithMessages/SampleDataWithMessages.test.tsx | Frontend test `SampleDataWithMessages.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SchemaEditor/CodeEditor.test.tsx | Frontend test `CodeEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SchemaEditor/SchemaEditor.test.tsx | Frontend test `SchemaEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/SchemaTable/SchemaTable.test.tsx | Frontend test `SchemaTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/StoredProcedureVersion/StoredProcedureVersion.test.tsx | Frontend test `StoredProcedureVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableDataCardBody/TableDataCardBody.test.tsx | Frontend test `TableDataCardBody.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableDescription/TableDescription.test.tsx | Frontend test `TableDescription.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableQueries/QueryCard.test.tsx | Frontend test `QueryCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableQueries/QueryCardExtraOption/QueryCardExtraOption.test.tsx | Frontend test `QueryCardExtraOption.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableQueries/QueryUsedByOtherTable/QueryUsedByOtherTable.test.tsx | Frontend test `QueryUsedByOtherTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableQueries/TableQueries.test.tsx | Frontend test `TableQueries.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableQueries/TableQueryRightPanel/TableQueryRightPanel.test.tsx | Frontend test `TableQueryRightPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableTags/TableTags.test.tsx | Frontend test `TableTags.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Database/TableVersion/TableVersion.test.tsx | Frontend test `TableVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Domain/AddDomainForm/AddDomainForm.test.tsx | Frontend test `AddDomainForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Domain/DomainDetailPage/DomainDetailPage.test.tsx | Frontend test `DomainDetailPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Domain/DomainTabs/DocumentationTab/DocumentationTab.test.tsx | Frontend test `DocumentationTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Domain/DomainTypeSelectForm/DomainTypeSelectForm.test.tsx | Frontend test `DomainTypeSelectForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Domain/DomainVersion/DomainVersion.test.tsx | Frontend test `DomainVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Directory/DirectoryChildrenTable/DirectoryChildrenTable.test.tsx | Frontend test `DirectoryChildrenTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Directory/DirectoryDetails.test.tsx | Frontend test `DirectoryDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Directory/DirectoryVersion/DirectoryVersion.test.tsx | Frontend test `DirectoryVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/File/FileDetails.test.tsx | Frontend test `FileDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/File/FileVersion/FileVersion.test.tsx | Frontend test `FileVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/File/FilesTable/FilesTable.test.tsx | Frontend test `FilesTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Spreadsheet/SpreadsheetDetails.test.tsx | Frontend test `SpreadsheetDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Spreadsheet/SpreadsheetVersion/SpreadsheetVersion.test.tsx | Frontend test `SpreadsheetVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Spreadsheet/SpreadsheetsTable/SpreadsheetsTable.test.tsx | Frontend test `SpreadsheetsTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Spreadsheet/WorkflowsTable/WorkflowsTable.test.tsx | Frontend test `WorkflowsTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Worksheet/WorksheetColumnsTable/WorksheetColumnsTable.test.tsx | Frontend test `WorksheetColumnsTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Worksheet/WorksheetDetails.test.tsx | Frontend test `WorksheetDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/DriveService/Worksheet/WorksheetVersion/WorksheetVersion.test.tsx | Frontend test `WorksheetVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityExportModalProvider/EntityExportModalProvider.test.tsx | Frontend test `EntityExportModalProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityHeaderTitle/EntityHeaderTitle.test.tsx | Frontend test `EntityHeaderTitle.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityInfoDrawer/EdgeInfoDrawer.test.tsx | Frontend test `EdgeInfoDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/AppPipelineModel/AddPipeLineModal.test.tsx | Frontend test `AddPipeLineModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/CanvasButtonPopover.test.tsx | Frontend test `CanvasButtonPopover.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/CanvasEdgeRenderer.component.test.tsx | Frontend test `CanvasEdgeRenderer.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/CustomControls.test.tsx | Frontend test `CustomControls.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/CustomNode.utils.test.tsx | Frontend test `CustomNode.utils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/CustomNodeV1.test.tsx | Frontend test `CustomNodeV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/EdgeInteractionOverlay.component.test.tsx | Frontend test `EdgeInteractionOverlay.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/EntityLineageSidebar.component.test.tsx | Frontend test `EntityLineageSidebar.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LineageConfigModal.test.tsx | Frontend test `LineageConfigModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LineageControlButtons/LineageControlButtons.test.tsx | Frontend test `LineageControlButtons.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LineageLayers/LineageLayers.test.tsx | Frontend test `LineageLayers.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LineageNodeLabelV1.test.tsx | Frontend test `LineageNodeLabelV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LineageSearchSelect/LineageSearchSelect.test.tsx | Frontend test `LineageSearchSelect.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/LoadMoreNode/LoadMoreNode.test.tsx | Frontend test `LoadMoreNode.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/NodeChildren/NodeChildren.component.test.tsx | Frontend test `NodeChildren.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/NodeChildren/VirtualColumnList.component.test.tsx | Frontend test `VirtualColumnList.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/NodeSuggestions.test.tsx | Frontend test `NodeSuggestions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityLineage/TestSuiteSummaryWidget/TestSuiteSummaryWidget.test.tsx | Frontend test `TestSuiteSummaryWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityRightPanel/EntityRightPanel.test.tsx | Frontend test `EntityRightPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityRightPanel/EntityRightPanelVerticalNav.test.tsx | Frontend test `EntityRightPanelVerticalNav.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityStatusBadge/EntityStatusBadge.test.tsx | Frontend test `EntityStatusBadge.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityVersionTimeLine/BulkImportVersionSummary/BulkImportVersionSummary.test.tsx | Frontend test `BulkImportVersionSummary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/EntityVersionTimeLine/EntityVersionTimeline.test.tsx | Frontend test `EntityVersionTimeline.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/Task/TaskTab/TaskTabNew.component.test.tsx | Frontend test `TaskTabNew.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/Task/TaskTabIncidentManagerHeader/TaskTabIncidentManagerHeader.test.tsx | Frontend test `TaskTabIncidentManagerHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/VersionTable/VersionTable.test.tsx | Frontend test `VersionTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Entity/Voting/Voting.component.test.tsx | Frontend test `Voting.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/AdvanceSearchProvider/AdvanceSearchProvider.test.tsx | Frontend test `AdvanceSearchProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/ColumnSummaryList/ColumnsSummaryList.test.tsx | Frontend test `ColumnsSummaryList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/CommonEntitySummaryInfo/CommonEntitySummaryInfo.test.tsx | Frontend test `CommonEntitySummaryInfo.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/CustomPropertiesSection/CustomPropertiesSection.test.tsx | Frontend test `CustomPropertiesSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/DataQualityTab/DataQualityTab.test.tsx | Frontend test `DataQualityTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/EntitySummaryPanel.test.tsx | Frontend test `EntitySummaryPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/LineageTab/LineageTabContent.test.tsx | Frontend test `LineageTabContent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/SummaryList/SummaryList.test.tsx | Frontend test `SummaryList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/EntitySummaryPanel/SummaryList/SummaryListItems/SummaryListItems.test.tsx | Frontend test `SummaryListItems.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/ExploreQuickFilters.test.tsx | Frontend test `ExploreQuickFilters.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/ExploreTree/ExploreTree.test.tsx | Frontend test `ExploreTree.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Explore/SortingDropDown.test.tsx | Frontend test `SortingDropDown.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ExploreV1/ExploreSearchCard/ExploreSearchCard.test.tsx | Frontend test `ExploreSearchCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ExploreV1/ExploreV1.test.tsx | Frontend test `ExploreV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ExploreV1/IndexNotFoundBanner.test.tsx | Frontend test `IndexNotFoundBanner.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/AddGlossary/AddGlossary.test.tsx | Frontend test `AddGlossary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryDetails/GlossaryDetails.test.tsx | Frontend test `GlossaryDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryHeader/GlossaryHeader.test.tsx | Frontend test `GlossaryHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTermTab/GlossaryTermTab.test.tsx | Frontend test `GlossaryTermTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/GlossaryTermReferencesModal.test.tsx | Frontend test `GlossaryTermReferencesModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/GlossaryTermsV1.test.tsx | Frontend test `GlossaryTermsV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/AssetsTabs.test.tsx | Frontend test `AssetsTabs.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/GlossaryTermReferences.test.tsx | Frontend test `GlossaryTermReferences.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/GlossaryTermSynonyms.test.tsx | Frontend test `GlossaryTermSynonyms.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/RelatedTerms.test.tsx | Frontend test `RelatedTerms.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/TermsRowEditor.test.tsx | Frontend test `TermsRowEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryTerms/tabs/WorkFlowTab/WorkflowHistory.test.tsx | Frontend test `WorkflowHistory.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryUpdateConfirmationModal/GlossaryUpdateConfirmationModal.test.tsx | Frontend test `GlossaryUpdateConfirmationModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryV1.test.tsx | Frontend test `GlossaryV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Glossary/GlossaryVersion/GlossaryVersion.test.tsx | Frontend test `GlossaryVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/IncidentManager/IncidentManager.test.tsx | Frontend test `IncidentManager.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/BookMarkWidget/BookMarkWidget.test.tsx | Frontend test `BookMarkWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgeCard/KnowledgeCard.test.tsx | Frontend test `KnowledgeCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgeCenterLayout/KnowledgeCenterLayout.test.tsx | Frontend test `KnowledgeCenterLayout.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgeCenterLayout/SizeAwareElement/SizeAwareElement.test.tsx | Frontend test `SizeAwareElement.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgeCenterWidget/KnowledgeCenterWidget.test.tsx | Frontend test `KnowledgeCenterWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgeDetailPageHeader/KnowledgeDetailPageHeader.test.tsx | Frontend test `KnowledgeDetailPageHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgePageListRightPanel/KnowledgePageListRightPanel.test.tsx | Frontend test `KnowledgePageListRightPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgePageSummary/KnowledgePageSummary.test.tsx | Frontend test `KnowledgePageSummary.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgePageVersion/KnowledgePageVersion.test.tsx | Frontend test `KnowledgePageVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgePages/KnowledgePages.test.tsx | Frontend test `KnowledgePages.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/KnowledgePagesHierarchy/KnowledgePagesHierarchy.test.tsx | Frontend test `KnowledgePagesHierarchy.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/QuickLinkFormModal/QuickLinkFormModal.test.tsx | Frontend test `QuickLinkFormModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/RelatedDataAssets/RelatedDataAssets.test.tsx | Frontend test `RelatedDataAssets.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/RelatedDataAssets/RelatedDataAssetsForm.test.tsx | Frontend test `RelatedDataAssetsForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeCenter/TitleComponent/TitleComponent.test.tsx | Frontend test `TitleComponent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeGraph/GraphElements/CustomNode.test.tsx | Frontend test `CustomNode.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/KnowledgeGraph/KnowledgeGraph.test.tsx | Frontend test `KnowledgeGraph.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/LearningDrawer/LearningDrawer.test.tsx | Frontend test `LearningDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/LearningIcon/LearningIcon.test.tsx | Frontend test `LearningIcon.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/LearningResourceCard/LearningResourceCard.test.tsx | Frontend test `LearningResourceCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/ResourcePlayer/ArticleViewer.test.tsx | Frontend test `ArticleViewer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/ResourcePlayer/ResourcePlayerModal.test.tsx | Frontend test `ResourcePlayerModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/ResourcePlayer/StorylaneTour.test.tsx | Frontend test `StorylaneTour.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Learning/ResourcePlayer/VideoPlayer.test.tsx | Frontend test `VideoPlayer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Lineage/Edges/CanvasLayerWrapper/CanvasLayerWrapper.test.tsx | Frontend test `CanvasLayerWrapper.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Lineage/Lineage.test.tsx | Frontend test `Lineage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/LineageTable/LineageTable.test.tsx | Frontend test `LineageTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/LineageTable/useLineagetTableState.test.ts | Frontend test `useLineagetTableState.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Metric/MetricDetails/MetricDetails.test.tsx | Frontend test `MetricDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MlModel/MlModelDetail/MlModelDetail.component.test.tsx | Frontend test `MlModelDetail.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MlModel/MlModelDetail/MlModelFeaturesList.test.tsx | Frontend test `MlModelFeaturesList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MlModel/MlModelVersion/MlModelVersion.test.tsx | Frontend test `MlModelVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/AnnouncementModal/AddAnnouncementModal.test.tsx | Frontend test `AddAnnouncementModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/AnnouncementModal/EditAnnouncementModal.test.tsx | Frontend test `EditAnnouncementModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ChangeParentHierarchy/ChangeParentHierarchy.test.tsx | Frontend test `ChangeParentHierarchy.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ConfirmationModal/ConfirmationModal.test.tsx | Frontend test `ConfirmationModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/DeployIngestionLoaderModal/DeployIngestionLoaderModal.test.tsx | Frontend test `DeployIngestionLoaderModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/EntityDeleteModal/EntityDeleteModal.test.tsx | Frontend test `EntityDeleteModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/IconColorModal/IconColorModal.test.tsx | Frontend test `IconColorModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/IngestionRunDetailsModal/IngestionRunDetailsModal.test.tsx | Frontend test `IngestionRunDetailsModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/KillIngestionPipelineModal/KillIngestionModal.test.tsx | Frontend test `KillIngestionModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ModalWithFunctionEditor/ModalWithFunctionEditor.test.tsx | Frontend test `ModalWithFunctionEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ModalWithMarkdownEditor/ModalWithMarkdownEditor.test.tsx | Frontend test `ModalWithMarkdownEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ModalWithQueryEditor/ModalWithQueryEditor.test.tsx | Frontend test `ModalWithQueryEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/ProfileEditModal/ProfileEditModal.test.tsx | Frontend test `ProfileEditModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/SchemaModal/SchemaModal.test.tsx | Frontend test `SchemaModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/StopScheduleRun/StopScheduleRunModal.test.tsx | Frontend test `StopScheduleRunModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/StyleModal/StyleModal.test.tsx | Frontend test `StyleModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/TourEndModal/TourEndModal.test.tsx | Frontend test `TourEndModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/UnsavedChangesModal/UnsavedChangesModal.test.tsx | Frontend test `UnsavedChangesModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Modals/WhatsNewModal/WhatsNewAlert/WhatsNewAlert.test.tsx | Frontend test `WhatsNewAlert.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/AddWidgetModal/AddWidgetModal.test.tsx | Frontend test `AddWidgetModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/AddWidgetModal/AddWidgetTabContent.test.tsx | Frontend test `AddWidgetTabContent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/AllWidgetsContent/AllWidgetsContent.test.tsx | Frontend test `AllWidgetsContent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/CustomiseHomeModal/CustomiseHomeModal.test.tsx | Frontend test `CustomiseHomeModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/CustomiseLandingPageHeader/CustomiseLandingPageHeader.test.tsx | Frontend test `CustomiseLandingPageHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/CustomiseLandingPageHeader/CustomiseSearchBar.test.tsx | Frontend test `CustomiseSearchBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/CustomizablePageHeader/CustomizablePageHeader.test.tsx | Frontend test `CustomizablePageHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/CustomizeMyData/CustomizeMyData.test.tsx | Frontend test `CustomizeMyData.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/EmptyWidgetPlaceholder/EmptyWidgetPlaceholder.test.tsx | Frontend test `EmptyWidgetPlaceholder.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/EmptyWidgetPlaceholder/EmptyWidgetPlaceholderV1.test.tsx | Frontend test `EmptyWidgetPlaceholderV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/CustomizableComponents/WidgetCard/WidgetCard.test.tsx | Frontend test `WidgetCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/FeedWidget/FeedWidget.test.tsx | Frontend test `FeedWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/GithubStarCard/GithubStarCard.test.tsx | Frontend test `GithubStarCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/HeaderTheme/HeaderTheme.test.tsx | Frontend test `HeaderTheme.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/LeftSidebar/LeftSidebar.test.tsx | Frontend test `LeftSidebar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/LeftSidebar/LeftSidebarItem.test.tsx | Frontend test `LeftSidebarItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/MyDataWidget/MyDataWidget.test.tsx | Frontend test `MyDataWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Persona/PersonaDetailsCard/PersonaDetailsCard.test.tsx | Frontend test `PersonaDetailsCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/RightSidebar/AnnouncementsWidget.test.tsx | Frontend test `AnnouncementsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/RightSidebar/FollowingWidget.test.tsx | Frontend test `FollowingWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/WelcomeScreen/WelcomScreen.test.tsx | Frontend test `WelcomScreen.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/AnnouncementsWidgetV1/AnnouncementCardV1/AnnouncementCardV1.test.tsx | Frontend test `AnnouncementCardV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/AnnouncementsWidgetV1/AnnouncementsWidgetV1.test.tsx | Frontend test `AnnouncementsWidgetV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetEmptyState/WidgetEmptyState.test.tsx | Frontend test `WidgetEmptyState.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetFooter/WidgetFooter.test.tsx | Frontend test `WidgetFooter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetHeader/WidgetHeader.test.tsx | Frontend test `WidgetHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetMoreOptions/WidgetMoreOptions.test.tsx | Frontend test `WidgetMoreOptions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetSortFilter/WidgetSortFilter.test.tsx | Frontend test `WidgetSortFilter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/Common/WidgetWrapper/WidgetWrapper.test.tsx | Frontend test `WidgetWrapper.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/CuratedAssetsWidget/AdvancedAssetsFilterField/AdvancedAssetsFilterField.test.tsx | Frontend test `AdvancedAssetsFilterField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/CuratedAssetsWidget/CuratedAssetsModal/CuratedAssetsModal.test.tsx | Frontend test `CuratedAssetsModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/CuratedAssetsWidget/CuratedAssetsWidget.test.tsx | Frontend test `CuratedAssetsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/CuratedAssetsWidget/SelectAssetTypeField/SelectAssetTypeField.test.tsx | Frontend test `SelectAssetTypeField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/DataAssetsWidget/DataAssetCard/DataAssetCard.test.tsx | Frontend test `DataAssetCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/DataAssetsWidget/DataAssetWidget.test.tsx | Frontend test `DataAssetWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/DataProductsWidget/DataProductsWidget.test.tsx | Frontend test `DataProductsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/DomainsWidget/DomainsWidget.test.tsx | Frontend test `DomainsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/KPIWidget/KPIWidget.test.tsx | Frontend test `KPIWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/MyTaskWidget/MyTaskWidget.test.tsx | Frontend test `MyTaskWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/MyData/Widgets/TotalDataAssetsWidget/TotalDataAssetsWidget.test.tsx | Frontend test `TotalDataAssetsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/NavBar/NavBar.test.tsx | Frontend test `NavBar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/NotificationBox/NotificationBox.test.tsx | Frontend test `NotificationBox.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/NotificationBox/NotificationFeedCard.test.tsx | Frontend test `NotificationFeedCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/OntologyExplorer/hooks/useGraphData.test.ts | Frontend test `useGraphData.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/OntologyExplorer/utils/graphBuilders.test.ts | Frontend test `graphBuilders.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/OntologyExplorer/utils/hierarchyGraphBuilder.test.ts | Frontend test `hierarchyGraphBuilder.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/PageLayoutV1/PageLayoutV1.test.tsx | Frontend test `PageLayoutV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/Execution/Execution.component.test.tsx | Frontend test `Execution.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/Execution/ListView/ListViewTab.component.test.tsx | Frontend test `ListViewTab.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/PipelineDetails/PipelineDetails.test.tsx | Frontend test `PipelineDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/PipelineVersion/PipelineVersion.test.tsx | Frontend test `PipelineVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/TasksDAGView/TaskNode/TaskNode.test.tsx | Frontend test `TaskNode.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Pipeline/TasksDAGView/TasksDAGView.test.tsx | Frontend test `TasksDAGView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ProfileCard/ProfileSectionUserDetailsCard.test.tsx | Frontend test `ProfileSectionUserDetailsCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchDropdown/SearchDropdown.test.tsx | Frontend test `SearchDropdown.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/EntitySeachSettings/EntitySearchSettings.test.tsx | Frontend test `EntitySearchSettings.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/FieldConfiguration/FieldConfiguration.test.tsx | Frontend test `FieldConfiguration.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/FieldValueBoostList/FieldValueBoostList.test.tsx | Frontend test `FieldValueBoostList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/GlobalSettingsItem/GlobalSettingsItem.test.tsx | Frontend test `GlobalSettingsItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/SearchPreview/SearchPreview.test.tsx | Frontend test `SearchPreview.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/TermBoost/TermBoost.test.tsx | Frontend test `TermBoost.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchSettings/TermBoostList/TermBoostList.test.tsx | Frontend test `TermBoostList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SearchedData/SearchedData.test.tsx | Frontend test `SearchedData.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/ServiceInsights/PlatformInsightsWidget/PlatformInsightsWidget.test.tsx | Frontend test `PlatformInsightsWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Alerts/AlertsDetails/AlertDetails.test.tsx | Frontend test `AlertDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppDetails/AppDetails.test.tsx | Frontend test `AppDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppDetails/ApplicationsClassBase.test.ts | Frontend test `ApplicationsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppInstallVerifyCard/AppInstallVerifyCard.test.tsx | Frontend test `AppInstallVerifyCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppLogsViewer/AppLogsViewer.test.tsx | Frontend test `AppLogsViewer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppRunsHistory/AppRunsHistory.test.tsx | Frontend test `AppRunsHistory.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/AppSchedule/AppSchedule.test.tsx | Frontend test `AppSchedule.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/ApplicationCard/ApplicationCard.test.tsx | Frontend test `ApplicationCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/ApplicationConfiguration/ApplicationConfiguration.test.tsx | Frontend test `ApplicationConfiguration.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Applications/MarketPlaceAppDetails/MarketPlaceAppDetails.test.tsx | Frontend test `MarketPlaceAppDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Bot/BotDetails/AuthMechanismForm.test.tsx | Frontend test `AuthMechanismForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Bot/BotDetails/BotDetails.test.tsx | Frontend test `BotDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Bot/BotListV1/BotListV1.component.test.tsx | Frontend test `BotListV1.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/CustomProperty/AddCustomProperty/AddCustomProperty.test.tsx | Frontend test `AddCustomProperty.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/CustomProperty/CustomPropertyTable.test.tsx | Frontend test `CustomPropertyTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Email/EmailConfigForm/EmailConfigForm.test.tsx | Frontend test `EmailConfigForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Email/TestEmail/TestEmail.test.tsx | Frontend test `TestEmail.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/AddIngestion/AddIngestion.test.tsx | Frontend test `AddIngestion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/AddIngestion/Steps/ScheduleInterval.test.tsx | Frontend test `ScheduleInterval.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/AddService/Steps/ConfigureService.test.tsx | Frontend test `ConfigureService.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/AddService/Steps/SelectServiceType.test.tsx | Frontend test `SelectServiceType.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/AddIngestionButton.test.tsx | Frontend test `AddIngestionButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/Ingestion.test.tsx | Frontend test `Ingestion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionListTable/IngestionListTable.test.tsx | Frontend test `IngestionListTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionListTable/PipelineActions/PipelineActions.test.tsx | Frontend test `PipelineActions.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionListTable/PipelineActions/PipelineActionsDropdown.test.tsx | Frontend test `PipelineActionsDropdown.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionPipelineList/IngestinoPipelineList.test.tsx | Frontend test `IngestinoPipelineList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionRecentRun/IngestionRecentRun.test.tsx | Frontend test `IngestionRecentRun.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionStepper/IngestionStepper.test.tsx | Frontend test `IngestionStepper.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Ingestion/IngestionWorkflowForm/ProfileSampleConfigField.test.tsx | Frontend test `ProfileSampleConfigField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/ServiceConfig/ConnectionConfigForm.test.tsx | Frontend test `ConnectionConfigForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/ServiceConfig/EmbeddedConnectionConfigForm.test.tsx | Frontend test `EmbeddedConnectionConfigForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/ServiceConfig/FiltersConfigForm.test.tsx | Frontend test `FiltersConfigForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/ServiceConnectionDetails/ServiceConnectionDetails.test.tsx | Frontend test `ServiceConnectionDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Services/Services.test.tsx | Frontend test `Services.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/SettingItemCard/SettingsItemCard.test.tsx | Frontend test `SettingsItemCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/TeamHierarchy.test.tsx | Frontend test `TeamHierarchy.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/TeamsHeaderSection/TeamHierarchyNameCell.test.tsx | Frontend test `TeamHierarchyNameCell.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/TeamsHeaderSection/TeamsHeadingLabel.test.tsx | Frontend test `TeamsHeadingLabel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/TeamsHeaderSection/TeamsInfo.test.tsx | Frontend test `TeamsInfo.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/TeamsHeaderSection/TeamsSubscription.test.tsx | Frontend test `TeamsSubscription.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamDetails/UserTab/UserTab.test.tsx | Frontend test `UserTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamImportResult/TeamImportResult.test.tsx | Frontend test `TeamImportResult.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/TeamsSelectable/TeamsSelectable.test.tsx | Frontend test `TeamsSelectable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Team/UserImportResult/UserImportResult.test.tsx | Frontend test `UserImportResult.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/AccessTokenCard/AccessTokenCard.test.tsx | Frontend test `AccessTokenCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/AdminPermissionDebugger/AdminPermissionDebugger.test.tsx | Frontend test `AdminPermissionDebugger.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/ChangePasswordForm.test.tsx | Frontend test `ChangePasswordForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/CreateUser/CreateUser.test.tsx | Frontend test `CreateUser.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UserProfileIcon/UserProfileIcon.test.tsx | Frontend test `UserProfileIcon.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/Users.component.test.tsx | Frontend test `Users.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UsersProfile/UserProfileImage/UserProfileImage.test.tsx | Frontend test `UserProfileImage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UsersProfile/UserProfileInheritedRoles/UserProfileInheritedRoles.test.tsx | Frontend test `UserProfileInheritedRoles.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UsersProfile/UserProfileRoles/UserProfileRoles.test.tsx | Frontend test `UserProfileRoles.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UsersProfile/UserProfileTeams/UserProfileTeams.test.tsx | Frontend test `UserProfileTeams.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Settings/Users/UsersTab/UsersTab.test.tsx | Frontend test `UsersTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SettingsSso/ProviderSelector/ProviderSelector.test.tsx | Frontend test `ProviderSelector.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SettingsSso/SSOConfigurationForm/SSOConfigurationForm.test.tsx | Frontend test `SSOConfigurationForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SettingsSso/SSOConfigurationForm/SsoConfigurationFormArrayFieldTemplate.test.tsx | Frontend test `SsoConfigurationFormArrayFieldTemplate.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SettingsSso/SSOConfigurationForm/SsoRolesSelectField.test.tsx | Frontend test `SsoRolesSelectField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/SettingsSso/SettingsSso.test.tsx | Frontend test `SettingsSso.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Suggestions/SuggestionsAlert/SuggestionsAlert.test.tsx | Frontend test `SuggestionsAlert.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Suggestions/SuggestionsProvider/SuggestionsProvider.test.tsx | Frontend test `SuggestionsProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Suggestions/SuggestionsSlider/SuggestionsSlider.test.tsx | Frontend test `SuggestionsSlider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Tag/TagsContainerV2/TagsContainerV2.test.tsx | Frontend test `TagsContainerV2.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Tag/TagsSelectForm/TagsSelectForm.component.test.tsx | Frontend test `TagsSelectForm.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Tag/TagsV1/TagsV1.test.tsx | Frontend test `TagsV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Tag/TagsViewer/TagsViewer.test.tsx | Frontend test `TagsViewer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/TestLibrary/TestDefinitionForm/TestDefinitionForm.test.tsx | Frontend test `TestDefinitionForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/TestLibrary/TestDefinitionList/TestDefinitionList.test.tsx | Frontend test `TestDefinitionList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Topic/TopicDetails/TopicDetails.component.test.tsx | Frontend test `TopicDetails.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Topic/TopicSchema/TopicSchema.test.tsx | Frontend test `TopicSchema.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Topic/TopicVersion/TopicVersion.test.tsx | Frontend test `TopicVersion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/UploadFile/UploadFile.test.tsx | Frontend test `UploadFile.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/CardinalityDistributionChart.test.tsx | Frontend test `CardinalityDistributionChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/CustomAreaChart.test.tsx | Frontend test `CustomAreaChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/CustomBarChart.test.tsx | Frontend test `CustomBarChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/CustomPieChart.test.tsx | Frontend test `CustomPieChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/DataDistributionHistogram.test.tsx | Frontend test `DataDistributionHistogram.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/Visualisations/Chart/OperationDateBarChart.test.tsx | Frontend test `OperationDateBarChart.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WebAnalytics/WebAnalyticsProvider.test.tsx | Frontend test `WebAnalyticsProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/EmptyCanvasMessage.test.tsx | Frontend test `EmptyCanvasMessage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/WorkflowHeader.test.tsx | Frontend test `WorkflowHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/common/FormField.test.tsx | Frontend test `FormField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/forms/MetadataFormSection.test.tsx | Frontend test `MetadataFormSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/forms/SchemaBasedNodeForm.test.tsx | Frontend test `SchemaBasedNodeForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/forms/SinkTaskForm.test.tsx | Frontend test `SinkTaskForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/WorkflowBuilder/forms/WorkflowConfigFormV1.test.tsx | Frontend test `WorkflowConfigFormV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/WorkflowDefinitions/Workflows/Nodes/GatewayNode/GatewayNode.test.tsx | Frontend test `GatewayNode.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AirflowMessageBanner/AirflowMessageBanner.test.tsx | Frontend test `AirflowMessageBanner.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AsyncSelect/AsyncSelect.test.tsx | Frontend test `AsyncSelect.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AsyncSelectList/AsyncSelectList.test.tsx | Frontend test `AsyncSelectList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AsyncSelectList/TreeAsyncSelectList.test.tsx | Frontend test `TreeAsyncSelectList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AvatarCarousel/AvatarCarousel.test.tsx | Frontend test `AvatarCarousel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AvatarCarouselItem/AvatarCarouselItem.test.tsx | Frontend test `AvatarCarouselItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/AvatarComponent/Avatar.test.tsx | Frontend test `Avatar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Badge/Badge.test.tsx | Frontend test `Badge.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/BrandImage/BrandImage.test.tsx | Frontend test `BrandImage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Chip/Chip.test.tsx | Frontend test `Chip.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CollapseHeader/CollapseHeader.test.tsx | Frontend test `CollapseHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ColorPicker/ColorPicker.test.tsx | Frontend test `ColorPicker.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CopyLinkButton/CopyLinkButton.test.tsx | Frontend test `CopyLinkButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CopyToClipboardButton/CopyToClipboardButton.test.tsx | Frontend test `CopyToClipboardButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomPropertyTable/CustomPropertyTable.test.tsx | Frontend test `CustomPropertyTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomPropertyTable/PropertyInput.test.tsx | Frontend test `PropertyInput.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomPropertyTable/PropertyValue.test.tsx | Frontend test `PropertyValue.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomPropertyTable/TableTypeProperty/EditTableTypePropertyModal.test.tsx | Frontend test `EditTableTypePropertyModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomPropertyTable/TableTypeProperty/TableTypePropertyEditTable.test.tsx | Frontend test `TableTypePropertyEditTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/CustomUnitSelect/CustomUnitSelect.test.tsx | Frontend test `CustomUnitSelect.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DataProductsSection/DataProductsSection.test.tsx | Frontend test `DataProductsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DataQualitySection/DataQualityLegendItem.test.tsx | Frontend test `DataQualityLegendItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DataQualitySection/DataQualityProgressSegment.test.tsx | Frontend test `DataQualityProgressSegment.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DataQualitySection/DataQualitySection.test.tsx | Frontend test `DataQualitySection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DataQualitySection/DataQualityStatCard.test.tsx | Frontend test `DataQualityStatCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DatePickerMenu/DatePickerMenu.test.tsx | Frontend test `DatePickerMenu.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DeferredWidget/DeferredWidget.test.tsx | Frontend test `DeferredWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DeleteModal/DeleteModal.test.tsx | Frontend test `DeleteModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DeleteWidget/DeleteWidgetModal.test.tsx | Frontend test `DeleteWidgetModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DescriptionSection/DescriptionSection.test.tsx | Frontend test `DescriptionSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DescriptionSourceBadge/DescriptionSourceBadge.test.tsx | Frontend test `DescriptionSourceBadge.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DisplayName/DisplayName.test.tsx | Frontend test `DisplayName.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DomainDisplay/DomainDisplay.test.tsx | Frontend test `DomainDisplay.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DomainLabel/DomainLabel.test.tsx | Frontend test `DomainLabel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DomainSelectableTree/DomainSelectableTree.test.tsx | Frontend test `DomainSelectableTree.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DomainsSection/DomainsSection.test.tsx | Frontend test `DomainsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/DraggableTabs/DraggableTabs.test.tsx | Frontend test `DraggableTabs.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityDescription/EntityAttachmentProvider/EntityAttachmentProvider.test.tsx | Frontend test `EntityAttachmentProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityDetailsSection/EntityDetailsSection.test.tsx | Frontend test `EntityDetailsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityImport/EntityImport.test.tsx | Frontend test `EntityImport.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityImport/ImportStatus/ImportStatus.test.tsx | Frontend test `ImportStatus.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityPageInfos/AnnouncementCard/AnnouncementCard.test.tsx | Frontend test `AnnouncementCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityPageInfos/AnnouncementDrawer/AnnouncementDrawer.test.tsx | Frontend test `AnnouncementDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityPageInfos/ManageButton/ManageButton.test.tsx | Frontend test `ManageButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntitySummaryDetails/EntitySummaryDetails.test.tsx | Frontend test `EntitySummaryDetails.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/EntityTitleSection/EntityTitleSection.test.tsx | Frontend test `EntityTitleSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ErrorWithPlaceholder/ErrorPlaceHolder.test.tsx | Frontend test `ErrorPlaceHolder.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ErrorWithPlaceholder/ErrorPlaceHolderES.test.tsx | Frontend test `ErrorPlaceHolderES.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ErrorWithPlaceholder/ErrorPlaceHolderIngestion.test.tsx | Frontend test `ErrorPlaceHolderIngestion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ExpandableCard/ExpandableCard.test.tsx | Frontend test `ExpandableCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FeedsFilterPopover/FeedsFilterPopover.test.tsx | Frontend test `FeedsFilterPopover.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FieldCard/FieldCard.test.tsx | Frontend test `FieldCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FilterPattern/FilterPattern.test.tsx | Frontend test `FilterPattern.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FocusTrap/FocusTrapWithContainer.test.tsx | Frontend test `FocusTrapWithContainer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/FormItemLabel.test.tsx | Frontend test `FormItemLabel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JSONSchemaTemplate/FieldErrorTemplate/FieldErrorTemplate.test.tsx | Frontend test `FieldErrorTemplate.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JSONSchemaTemplate/ObjectFieldTemplate.test.tsx | Frontend test `ObjectFieldTemplate.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JSONSchemaTemplate/WorkflowArrayFieldTemplate.test.tsx | Frontend test `WorkflowArrayFieldTemplate.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/CodeWidget/CodeWidget.test.tsx | Frontend test `CodeWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/FileUploadWidget.test.tsx | Frontend test `FileUploadWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/LdapRoleMappingWidget/LdapRoleMappingWidget.test.tsx | Frontend test `LdapRoleMappingWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/ManifestJsonWidget/ManifestJsonWidget.test.tsx | Frontend test `ManifestJsonWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/PasswordWidget.test.tsx | Frontend test `PasswordWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/QueryBuilderWidget/QueryBuilderWidget.test.tsx | Frontend test `QueryBuilderWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/SelectWidget.test.tsx | Frontend test `SelectWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Form/JSONSchema/JsonSchemaWidgets/TreeSelectWidget.test.tsx | Frontend test `TreeSelectWidget.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilder/FormBuilder.test.tsx | Frontend test `FormBuilder.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/FormBuilderV1.test.tsx | Frontend test `FormBuilderV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/fields/CoreArrayField.test.tsx | Frontend test `CoreArrayField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/fields/CoreBooleanField.test.tsx | Frontend test `CoreBooleanField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/fields/CoreOneOfField.test.tsx | Frontend test `CoreOneOfField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/fields/LayoutGridField.test.tsx | Frontend test `LayoutGridField.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/templates/FormBuilderV1Templates.test.tsx | Frontend test `FormBuilderV1Templates.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/widgets/FormBuilderV1Widgets.test.tsx | Frontend test `FormBuilderV1Widgets.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormBuilderV1/widgets/coreWidgetUtils.test.ts | Frontend test `coreWidgetUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/FormCardSection/FormCardSection.test.tsx | Frontend test `FormCardSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/GlossaryTermSelectableList/GlossaryTermSelectableList.test.tsx | Frontend test `GlossaryTermSelectableList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/GlossaryTermsSection/GlossaryTermsSection.test.tsx | Frontend test `GlossaryTermsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/HeaderBreadcrumb/HeaderBreadcrumb.test.tsx | Frontend test `HeaderBreadcrumb.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/InlineAlert/InlineAlert.test.tsx | Frontend test `InlineAlert.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/InlineEdit/InlineEdit.test.tsx | Frontend test `InlineEdit.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/LineageSection/LineageSection.test.tsx | Frontend test `LineageSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ListView/ListView.test.tsx | Frontend test `ListView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Loader/Loader.test.tsx | Frontend test `Loader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ManageButtonContentItem/ManageButtonContentItem.test.tsx | Frontend test `ManageButtonContentItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/MarkdownEditor/EntityMarkdownLink/EntityMarkdownLink.test.tsx | Frontend test `EntityMarkdownLink.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/MuiDatePickerMenu/MuiDatePickerMenu.test.tsx | Frontend test `MuiDatePickerMenu.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/MuiDrawer/MuiDrawer.test.tsx | Frontend test `MuiDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/NavigationBlocker/NavigationBlocker.test.tsx | Frontend test `NavigationBlocker.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/NextPrevious/NextPrevious.test.tsx | Frontend test `NextPrevious.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/NextPreviousWithOffset/NextPreviousWithOffset.test.tsx | Frontend test `NextPreviousWithOffset.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/OverviewSection/CommonEntitySummaryInfoV1.test.tsx | Frontend test `CommonEntitySummaryInfoV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/OverviewSection/OverviewSection.test.tsx | Frontend test `OverviewSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/OwnerLabel/OwnerLabel.test.tsx | Frontend test `OwnerLabel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/OwnersSection/OwnersSection.test.tsx | Frontend test `OwnersSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/PopOverCard/EntityPopOverCard.test.tsx | Frontend test `EntityPopOverCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/PopOverCard/UserPopOverCard.test.tsx | Frontend test `UserPopOverCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ProfilePicture/ProfilePicture.test.tsx | Frontend test `ProfilePicture.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/QueryBuilderWidgetV1/QueryBuilderWidgetV1.test.tsx | Frontend test `QueryBuilderWidgetV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/QueryCount/QueryCount.test.tsx | Frontend test `QueryCount.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/QueryViewer/QueryViewer.test.tsx | Frontend test `QueryViewer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ResizablePanels/ResizableLeftPanels.test.tsx | Frontend test `ResizableLeftPanels.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ResizablePanels/ResziablePanels.test.tsx | Frontend test `ResziablePanels.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/RichTextEditor/RichTextEditor.test.tsx | Frontend test `RichTextEditor.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/RichTextEditor/RichTextEditorPreviewNew.test.tsx | Frontend test `RichTextEditorPreviewNew.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/RichTextEditor/RichTextEditorPreviewerV1.test.tsx | Frontend test `RichTextEditorPreviewerV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/RichTextEditor/TaskDescriptionPreviewer.test.tsx | Frontend test `TaskDescriptionPreviewer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SanitizedInput/SanitizedInput.test.tsx | Frontend test `SanitizedInput.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SearchBarComponent/Searchbar.test.tsx | Frontend test `Searchbar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SectionWithEdit/SectionWithEdit.test.tsx | Frontend test `SectionWithEdit.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SelectableList/SelectableList.test.tsx | Frontend test `SelectableList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ServiceDocPanel/ServiceDocPanel.test.tsx | Frontend test `ServiceDocPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Skeleton/MyData/EntityListSkeleton/EntityListSkeleton.test.tsx | Frontend test `EntityListSkeleton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SuccessScreen/SuccessScreen.test.tsx | Frontend test `SuccessScreen.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/SummaryCard/SummaryCard.test.tsx | Frontend test `SummaryCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Table/DraggableMenu/DraggableMenuItem.test.tsx | Frontend test `DraggableMenuItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/Table/Table.test.tsx | Frontend test `Table.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TableDataCardV2/TableDataCardV2.test.tsx | Frontend test `TableDataCardV2.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TabsLabel/TabsLabel.test.tsx | Frontend test `TabsLabel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TagRenderer/TagRenderer.test.tsx | Frontend test `TagRenderer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TagSelectableList/TagSelectableList.test.tsx | Frontend test `TagSelectableList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TagSuggestion/TagSuggestion.test.tsx | Frontend test `TagSuggestion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TagsSection/TagsSection.test.tsx | Frontend test `TagsSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TeamTypeSelect/TeamTypeSelect.test.tsx | Frontend test `TeamTypeSelect.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TestCaseStatusSummaryIndicator/TestCaseStatusSummaryIndicator.component.test.tsx | Frontend test `TestCaseStatusSummaryIndicator.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TestConnection/TestConnection.test.tsx | Frontend test `TestConnection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TestConnection/TestConnectionModal/TestConnectionModal.test.tsx | Frontend test `TestConnectionModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TestIndicator/TestIndicator.test.tsx | Frontend test `TestIndicator.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TierCard/TierCard.test.tsx | Frontend test `TierCard.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TierSection/TierSection.test.tsx | Frontend test `TierSection.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/TitleBreadcrumb/TitleBreadcrumb.component.test.tsx | Frontend test `TitleBreadcrumb.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/ToggleExpandButton/ToggleExpandButton.test.tsx | Frontend test `ToggleExpandButton.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/UserSelectableList/UserSelectableList.test.tsx | Frontend test `UserSelectableList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/UserTag/UserTag.test.tsx | Frontend test `UserTag.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/UserTeamSelectableList/UserTeamSelectableList.test.tsx | Frontend test `UserTeamSelectableList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/atoms/compositions/useListingData.test.tsx | Frontend test `useListingData.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/atoms/data/useUrlState.test.tsx | Frontend test `useUrlState.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/common/atoms/filters/useQuickFiltersWithComponent.test.tsx | Frontend test `useQuickFiltersWithComponent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/components/form/MUIAutocomplete/MUIAutocomplete.test.tsx | Frontend test `MUIAutocomplete.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/constants/regex.constants.test.ts | Frontend test `regex.constants.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/context/AirflowStatusProvider/AirflowStatusProvider.test.tsx | Frontend test `AirflowStatusProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/context/AsyncDeleteProvider/AsyncDeleteProvider.test.tsx | Frontend test `AsyncDeleteProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/context/LineageProvider/LineageProvider.test.tsx | Frontend test `LineageProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/context/PermissionProvider/PermissionProvider.test.tsx | Frontend test `PermissionProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/context/RuleEnforcementProvider/RuleEnforcementProvider.test.tsx | Frontend test `RuleEnforcementProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/hoc/withDomainFilter.test.tsx | Frontend test `withDomainFilter.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/AbortController/useAbortController.test.tsx | Frontend test `useAbortController.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/LocationSearch/useLocationSearch.test.ts | Frontend test `useLocationSearch.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/currentUserStore/useCurrentUserStore.test.ts | Frontend test `useCurrentUserStore.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useAlertStore.test.ts | Frontend test `useAlertStore.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useAppMode.test.ts | Frontend test `useAppMode.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useAppRoutesRegistry.test.ts | Frontend test `useAppRoutesRegistry.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useApplicationStore.test.ts | Frontend test `useApplicationStore.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useCanvasEdgeRenderer.test.ts | Frontend test `useCanvasEdgeRenderer.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useCanvasMouseEvents.test.ts | Frontend test `useCanvasMouseEvents.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useClipBoard.test.ts | Frontend test `useClipBoard.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useCustomLocation/useCustomLocation.test.ts | Frontend test `useCustomLocation.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useCustomPages.test.ts | Frontend test `useCustomPages.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useDeferredTabData.test.ts | Frontend test `useDeferredTabData.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useEditableSection.test.ts | Frontend test `useEditableSection.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useElementInView.test.ts | Frontend test `useElementInView.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useEntityRules.test.ts | Frontend test `useEntityRules.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useFqn.test.ts | Frontend test `useFqn.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useFqnDeepLink.test.ts | Frontend test `useFqnDeepLink.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useGridLayoutDirection.test.ts | Frontend test `useGridLayoutDirection.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useLineageStore.test.ts | Frontend test `useLineageStore.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useMapBasedNodesEdges.test.ts | Frontend test `useMapBasedNodesEdges.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useRovingFocus.test.tsx | Frontend test `useRovingFocus.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useSidebarItems.test.ts | Frontend test `useSidebarItems.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useWorkflowMode.capabilities.test.ts | Frontend test `useWorkflowMode.capabilities.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useWorkflowMode.test.ts | Frontend test `useWorkflowMode.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/useWorkflowNavigationBlock.test.ts | Frontend test `useWorkflowNavigationBlock.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/hooks/user-profile/useUserProfile.test.ts | Frontend test `useUserProfile.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/APICollectionPage/APICollectionPage.test.tsx | Frontend test `APICollectionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddCustomMetricPage/AddCustomMetricPage.test.tsx | Frontend test `AddCustomMetricPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddGlossary/AddGlossaryPage.test.tsx | Frontend test `AddGlossaryPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddIngestionPage/AddIngestionPage.test.tsx | Frontend test `AddIngestionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddNotificationPage/AddNotificationPage.test.tsx | Frontend test `AddNotificationPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddObservabilityPage/AddObservabilityPage.test.tsx | Frontend test `AddObservabilityPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddQueryPage/AddQueryPage.test.tsx | Frontend test `AddQueryPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AddServicePage/AddServicePage.test.tsx | Frontend test `AddServicePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AlertDetailsPage/AlertDetailsPage.test.tsx | Frontend test `AlertDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AppInstall/AppInstall.test.tsx | Frontend test `AppInstall.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AppearanceConfigSettingsPage/AppearanceConfigSettingsPage.test.tsx | Frontend test `AppearanceConfigSettingsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/Application/ApplicationPage.test.tsx | Frontend test `ApplicationPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/AuditLogsPage/AuditLogsPage.test.tsx | Frontend test `AuditLogsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/BotDetailsPage/BotDetailsPage.test.tsx | Frontend test `BotDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/BotsPageV1/BotsPageV1.test.tsx | Frontend test `BotsPageV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ClassificationVersionPage/ClassificationVersionPage.test.tsx | Frontend test `ClassificationVersionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ContainerPage/ContainerPage.test.tsx | Frontend test `ContainerPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/CreateUserPage/CreateUserPage.test.tsx | Frontend test `CreateUserPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/CustomPropertiesPageV1/CustomPropertiesPageV1.test.tsx | Frontend test `CustomPropertiesPageV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/CustomizablePage/CustomizablePage.test.tsx | Frontend test `CustomizablePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DashboardDetailsPage/DashboardDetailsPage.test.tsx | Frontend test `DashboardDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataInsightPage/DataInsightHeader/DataInsightHeader.test.tsx | Frontend test `DataInsightHeader.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataInsightPage/DataInsightLeftPanel/DataInsightLeftPanel.test.tsx | Frontend test `DataInsightLeftPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataInsightPage/DataInsightPage.test.tsx | Frontend test `DataInsightPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataInsightPage/KPIList.test.tsx | Frontend test `KPIList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataModelPage/DataModelPage.test.tsx | Frontend test `DataModelPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataQuality/DataQualityPage.test.tsx | Frontend test `DataQualityPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DataQuality/DataQualityProvider.test.tsx | Frontend test `DataQualityProvider.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DatabaseDetailsPage/DatabaseDetailsPage.test.tsx | Frontend test `DatabaseDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DatabaseSchemaPage/DatabaseSchemaPage.test.tsx | Frontend test `DatabaseSchemaPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DatabaseSchemaVersionPage/DatabaseSchemaVersionPage.test.tsx | Frontend test `DatabaseSchemaVersionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DatabaseVersionPage/DatabaseVersionPage.test.tsx | Frontend test `DatabaseVersionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/DirectoryDetailsPage/DirectoryDetailsPage.test.tsx | Frontend test `DirectoryDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/EditConnectionFormPage/EditConnectionFormPage.test.tsx | Frontend test `EditConnectionFormPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/EditEmailConfigPage/EditEmailConfigPage.test.tsx | Frontend test `EditEmailConfigPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/EmbeddedAddServicePage/EmbeddedAddServicePage.test.tsx | Frontend test `EmbeddedAddServicePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/EntityImport/BulkEntityImportPage/BulkEntityImportPage.test.tsx | Frontend test `BulkEntityImportPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/EntityVersionPage/EntityVersionPage.test.tsx | Frontend test `EntityVersionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ExplorePage/ExplorePageV1.test.tsx | Frontend test `ExplorePageV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/FileDetailsPage/FileDetailsPage.test.tsx | Frontend test `FileDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ForgotPassword/ForgotPassword.test.tsx | Frontend test `ForgotPassword.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/Glossary/GlossaryLeftPanel/GlossaryLeftPanel.test.tsx | Frontend test `GlossaryLeftPanel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/Glossary/GlossaryPage/GlossaryPage.test.tsx | Frontend test `GlossaryPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/IncidentManager/IncidentManagerDetailPage/IncidentManagerDetailPage.test.tsx | Frontend test `IncidentManagerDetailPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/IncidentManager/IncidentManagerDetailPage/TestCaseClassBase.test.ts | Frontend test `TestCaseClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/IncidentManager/IncidentManagerPage.test.tsx | Frontend test `IncidentManagerPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/KPIPage/AddKPIPage.test.tsx | Frontend test `AddKPIPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/KPIPage/EditKPIPage.test.tsx | Frontend test `EditKPIPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/LearningResourcesPage/LearningResourceForm.test.tsx | Frontend test `LearningResourceForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/LoginPage/LoginCarousel.test.tsx | Frontend test `LoginCarousel.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/LoginPage/SignInPage.test.tsx | Frontend test `SignInPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/LogoutPage/LogoutPage.test.tsx | Frontend test `LogoutPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/LogsViewerPage/LogsViewerPage.component.test.tsx | Frontend test `LogsViewerPage.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/ChatInput.test.tsx | Frontend test `ChatInput.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/ChatList.test.tsx | Frontend test `ChatList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/ConversationSidebar.test.tsx | Frontend test `ConversationSidebar.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/McpChatPage.test.tsx | Frontend test `McpChatPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/MessageBubble.test.tsx | Frontend test `MessageBubble.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/MessageList.test.tsx | Frontend test `MessageList.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/McpChatPage/NavItem.test.tsx | Frontend test `NavItem.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/MetricsPage/MetricListPage/MetricListPage.test.tsx | Frontend test `MetricListPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/MlModelPage/MlModelPage.component.test.tsx | Frontend test `MlModelPage.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/MyDataPage/MyDataPage.test.tsx | Frontend test `MyDataPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/NotificationListPage/NotificationListPage.test.tsx | Frontend test `NotificationListPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ObservabilityAlertsPage/ObservabilityAlertsPage.test.tsx | Frontend test `ObservabilityAlertsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/OmHealth/OmHealthPage.test.tsx | Frontend test `OmHealthPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/OntologyExplorerPage/OntologyExplorerPage.test.tsx | Frontend test `OntologyExplorerPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PageNotFound/PageNotFound.test.tsx | Frontend test `PageNotFound.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/Persona/PersonaDetailsPage/PersonaDetailsPage.test.tsx | Frontend test `PersonaDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/Persona/PersonaListPage/PersonaPage.test.tsx | Frontend test `PersonaPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PipelineDetails/PipelineDetailsPage.test.tsx | Frontend test `PipelineDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PlatformLineage/PlatformLineage.test.tsx | Frontend test `PlatformLineage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/AddPolicyPage/AddPolicyPage.test.tsx | Frontend test `AddPolicyPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/PoliciesDetailPage/AddRulePage.test.tsx | Frontend test `AddRulePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/PoliciesDetailPage/EditRulePage.test.tsx | Frontend test `EditRulePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/PoliciesDetailPage/PoliciesDetailPage.test.tsx | Frontend test `PoliciesDetailPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/PoliciesListPage/PoliciesListPage.test.tsx | Frontend test `PoliciesListPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/PoliciesPage/RuleForm/RuleForm.test.tsx | Frontend test `RuleForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ProfilerConfigurationPage/ProfilerConfigurationPage.test.tsx | Frontend test `ProfilerConfigurationPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/QueryPage/QueryPage.test.tsx | Frontend test `QueryPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ResetPassword/ResetPassword.test.tsx | Frontend test `ResetPassword.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/RolesPage/AddAttributeModal/AddAttributeModal.test.tsx | Frontend test `AddAttributeModal.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/RolesPage/AddRolePage/AddRolePage.test.tsx | Frontend test `AddRolePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/RolesPage/RolesDetailPage/RolesDetailPage.test.tsx | Frontend test `RolesDetailPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/RolesPage/RolesListPage/RolesListPage.test.tsx | Frontend test `RolesListPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SamlCallback/index.test.tsx | Frontend test `index.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SearchIndexDetailsPage/SearchIndexDetailsPage.test.tsx | Frontend test `SearchIndexDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SearchIndexDetailsPage/SearchIndexFieldsTab/SearchIndexFieldsTab.test.tsx | Frontend test `SearchIndexFieldsTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SearchIndexDetailsPage/SearchIndexFieldsTable/SearchIndexFieldsTable.test.tsx | Frontend test `SearchIndexFieldsTable.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SearchSettingsPage/SearchSettingsPage.test.tsx | Frontend test `SearchSettingsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ServiceDetailsPage/ServiceDetailsPage.test.tsx | Frontend test `ServiceDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ServiceVersionPage/ServiceVersionMainTabContent.test.tsx | Frontend test `ServiceVersionMainTabContent.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ServiceVersionPage/ServiceVersionPage.test.tsx | Frontend test `ServiceVersionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/ServicesPage/ServicesPage.test.tsx | Frontend test `ServicesPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SettingsNavigationPage/SettingsNavigationPage.test.tsx | Frontend test `SettingsNavigationPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SignUp/SignUpPage.test.tsx | Frontend test `SignUpPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SpreadsheetDetailsPage/SpreadsheetDetailsPage.test.tsx | Frontend test `SpreadsheetDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/StoredProcedure/StoredProcedurePage.test.tsx | Frontend test `StoredProcedurePage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/StoredProcedure/StoredProcedureTab.test.tsx | Frontend test `StoredProcedureTab.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/SwaggerPage/index.test.tsx | Frontend test `index.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TableDetailsPageV1/FrequentlyJoinedTables/FrequentlyJoinedTables.test.tsx | Frontend test `FrequentlyJoinedTables.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TableDetailsPageV1/TableConstraints/TableConstraints.test.tsx | Frontend test `TableConstraints.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TableDetailsPageV1/TableConstraints/TableConstraintsModal/TableConstraintsModal.component.test.tsx | Frontend test `TableConstraintsModal.component.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TableDetailsPageV1/TableDetailsPageV1.test.tsx | Frontend test `TableDetailsPageV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagPage/TagPage.test.tsx | Frontend test `TagPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagsPage/ClassificationFormDrawer.test.tsx | Frontend test `ClassificationFormDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagsPage/TagFormDrawer.test.tsx | Frontend test `TagFormDrawer.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagsPage/TagsForm.test.tsx | Frontend test `TagsForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagsPage/TagsPage.test.tsx | Frontend test `TagsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TagsPage/tagFormFields.test.tsx | Frontend test `tagFormFields.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TaskFormSettingsPage/TaskFormSettingsPage.test.tsx | Frontend test `TaskFormSettingsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/EntityTasks/EntityTasks.test.tsx | Frontend test `EntityTasks.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/RequestDescriptionPage/RequestDescriptionPage.test.tsx | Frontend test `RequestDescriptionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/RequestTagPage/RequestTagPage.test.tsx | Frontend test `RequestTagPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/UpdateDescriptionPage/UpdateDescriptionPage.test.tsx | Frontend test `UpdateDescriptionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/UpdateTagPage/UpdateTagPage.test.tsx | Frontend test `UpdateTagPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/Assignees.test.tsx | Frontend test `Assignees.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/DescriptionTabs.test.tsx | Frontend test `DescriptionTabs.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/DescriptionTask.test.tsx | Frontend test `DescriptionTask.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/DiffView/DiffView.test.tsx | Frontend test `DiffView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/FeedbackApprovalTask.test.tsx | Frontend test `FeedbackApprovalTask.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/TagSuggestion.test.tsx | Frontend test `TagSuggestion.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/TagsDiffView.test.tsx | Frontend test `TagsDiffView.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/TagsTabs.test.tsx | Frontend test `TagsTabs.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/TagsTask.test.tsx | Frontend test `TagsTask.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TasksPage/shared/TaskPayloadSchemaFields.test.tsx | Frontend test `TaskPayloadSchemaFields.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TeamsPage/AddTeamForm.test.tsx | Frontend test `AddTeamForm.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TeamsPage/ImportTeamsPage/ImportTeamsPage.test.tsx | Frontend test `ImportTeamsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TeamsPage/TeamsPage.test.tsx | Frontend test `TeamsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TestLibrary/TestLibraryPage.test.tsx | Frontend test `TestLibraryPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TestSuiteDetailsPage/TestSuiteDetailsPage.test.tsx | Frontend test `TestSuiteDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TestSuiteIngestionPage/TestSuiteIngestionPage.test.tsx | Frontend test `TestSuiteIngestionPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TopicDetails/TopicDetailsPage.test.tsx | Frontend test `TopicDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/TourPage/TourPage.test.tsx | Frontend test `TourPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/UserListPage/UserListPageV1.test.tsx | Frontend test `UserListPageV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/UserPage/UserPage.test.tsx | Frontend test `UserPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/pages/WorksheetDetailsPage/WorksheetDetailsPage.test.tsx | Frontend test `WorksheetDetailsPage.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/changeSummaryAPI.test.ts | Frontend test `changeSummaryAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/dataQualityDashboardAPI.test.ts | Frontend test `dataQualityDashboardAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/importExportAPI.test.ts | Frontend test `importExportAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/limitsAPI.test.ts | Frontend test `limitsAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/lineageAPI.test.ts | Frontend test `lineageAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/notificationtemplateAPI.test.ts | Frontend test `notificationtemplateAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/searchAPI.test.ts | Frontend test `searchAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/tagAPI.test.ts | Frontend test `tagAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/rest/testAPI.test.ts | Frontend test `testAPI.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/APIEndpoints/APIEndpointUtils.test.tsx | Frontend test `APIEndpointUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AdvancedSearchClassBase.test.ts | Frontend test `AdvancedSearchClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AdvancedSearchUtils.test.tsx | Frontend test `AdvancedSearchUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Alerts/AlertsUtil.test.tsx | Frontend test `AlertsUtil.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AlertsClassBase.test.ts | Frontend test `AlertsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AnnouncementsUtils.test.tsx | Frontend test `AnnouncementsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ApplicationRoutesClassBase.test.ts | Frontend test `ApplicationRoutesClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ApplicationUtils.test.ts | Frontend test `ApplicationUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AuditLogUtils.test.ts | Frontend test `AuditLogUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Auth/TokenService/TokenServiceUtil.test.ts | Frontend test `TokenServiceUtil.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/AuthProviderUtil.test.ts | Frontend test `AuthProviderUtil.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/BlockEditorExtensionsClassBase.test.ts | Frontend test `BlockEditorExtensionsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/BlockEditorUtils.test.ts | Frontend test `BlockEditorUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/BotsUtils.test.tsx | Frontend test `BotsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/BrowserNotificationUtils.test.ts | Frontend test `BrowserNotificationUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CSV/CSV.utils.test.tsx | Frontend test `CSV.utils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CSV/CSVUtilsClassBase.test.tsx | Frontend test `CSVUtilsClassBase.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CanvasUtils.test.ts | Frontend test `CanvasUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ChartUtils.test.tsx | Frontend test `ChartUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ClassificationUtils.test.tsx | Frontend test `ClassificationUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ColorUtils.test.ts | Frontend test `ColorUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ColumnUpdateUtils.test.tsx | Frontend test `ColumnUpdateUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ConnectionsRouterClassBase.test.ts | Frontend test `ConnectionsRouterClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ContainerDetailUtils.test.ts | Frontend test `ContainerDetailUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ContainerDetailUtils.test.tsx | Frontend test `ContainerDetailUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ContextCenterUtils.test.ts | Frontend test `ContextCenterUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CuratedAssetsUtils.test.tsx | Frontend test `CuratedAssetsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CustomProperty.utils.test.ts | Frontend test `CustomProperty.utils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CustomizaNavigation/CustomizeNavigation.test.ts | Frontend test `CustomizeNavigation.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CustomizableLandingPageUtils.test.tsx | Frontend test `CustomizableLandingPageUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CustomizeColumnUtils.test.tsx | Frontend test `CustomizeColumnUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/CustomizePage/CustomizePageUtils.test.ts | Frontend test `CustomizePageUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DashboardDataModelUtils.test.tsx | Frontend test `DashboardDataModelUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataAssetSummaryPanelUtils.test.tsx | Frontend test `DataAssetSummaryPanelUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataAssetsHeader.utils.test.tsx | Frontend test `DataAssetsHeader.utils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataContract/DataContractUtils.test.ts | Frontend test `DataContractUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataInsightUtils.test.tsx | Frontend test `DataInsightUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataQuality/DataQualityUtils.test.ts | Frontend test `DataQualityUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DataQuality/TestSummaryGraphUtils.test.ts | Frontend test `TestSummaryGraphUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Database/Database.util.test.tsx | Frontend test `Database.util.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DatabaseSchemaDetailsUtils.test.tsx | Frontend test `DatabaseSchemaDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DatabaseServiceUtils.test.tsx | Frontend test `DatabaseServiceUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DirectoryClassBase.test.ts | Frontend test `DirectoryClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DirectoryDetailsUtils.test.tsx | Frontend test `DirectoryDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DocumentationLinksClassBase.test.ts | Frontend test `DocumentationLinksClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Domain/DomainClassBase.test.ts | Frontend test `DomainClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Domain/LazyDomainComponents.test.tsx | Frontend test `LazyDomainComponents.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DomainUtils.test.tsx | Frontend test `DomainUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/DriveServiceUtils.test.ts | Frontend test `DriveServiceUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EdgeStyleUtils.test.ts | Frontend test `EdgeStyleUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityBreadcrumbPureUtils.test.tsx | Frontend test `EntityBreadcrumbPureUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityBulkEdit/EntityBulkEditUtils.test.tsx | Frontend test `EntityBulkEditUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityByFqnUtils.test.ts | Frontend test `EntityByFqnUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityGovernanceBreadcrumbUtils.test.tsx | Frontend test `EntityGovernanceBreadcrumbUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityImport/EntityImportUtils.test.ts | Frontend test `EntityImportUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityLineageUtils.test.tsx | Frontend test `EntityLineageUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityLink.test.ts | Frontend test `EntityLink.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityLinkUtils.test.tsx | Frontend test `EntityLinkUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityNameUtils.test.tsx | Frontend test `EntityNameUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityPatchUtils.test.ts | Frontend test `EntityPatchUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityPermissionUtils.test.tsx | Frontend test `EntityPermissionUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityRightPanelClassBase.test.tsx | Frontend test `EntityRightPanelClassBase.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntitySearchUtils.test.tsx | Frontend test `EntitySearchUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntitySortUtils.test.tsx | Frontend test `EntitySortUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntitySummaryPanelUtils.test.tsx | Frontend test `EntitySummaryPanelUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntitySummaryPanelUtilsV1.test.tsx | Frontend test `EntitySummaryPanelUtilsV1.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityUpdateUtils.test.ts | Frontend test `EntityUpdateUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityUtilClassBase.test.ts | Frontend test `EntityUtilClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityValidationUtils.test.ts | Frontend test `EntityValidationUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/EntityVersionUtils.test.tsx | Frontend test `EntityVersionUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ExplorePage/ExplorePageUtils.test.ts | Frontend test `ExplorePageUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ExploreUtils.test.ts | Frontend test `ExploreUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Export/ExportUtils.test.tsx | Frontend test `ExportUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/FeedUtils.test.tsx | Frontend test `FeedUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/FileClassBase.test.ts | Frontend test `FileClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/FileDetailsUtils.test.tsx | Frontend test `FileDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/FilterQueryUtils.test.ts | Frontend test `FilterQueryUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/FqnUtils.test.ts | Frontend test `FqnUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/GlobalSettingsClassBase.test.ts | Frontend test `GlobalSettingsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/GlobalSettingsUtils.test.tsx | Frontend test `GlobalSettingsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Glossary/GlossaryTermClassBase.test.ts | Frontend test `GlossaryTermClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Glossary/GlossaryTermUtils.test.tsx | Frontend test `GlossaryTermUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/GlossaryUtils.test.ts | Frontend test `GlossaryUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/IconUtils.test.tsx | Frontend test `IconUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/IngestionListTableUtils.test.tsx | Frontend test `IngestionListTableUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/IngestionLogs/LogsUtils.test.ts | Frontend test `LogsUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/IngestionUtils.test.tsx | Frontend test `IngestionUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/IngestionWorkflowUtils.test.ts | Frontend test `IngestionWorkflowUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/JSONLogicSearchClassBase.test.ts | Frontend test `JSONLogicSearchClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/KnowledgeGraph.utils.test.ts | Frontend test `KnowledgeGraph.utils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/KnowledgePageUtils.test.tsx | Frontend test `KnowledgePageUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/LandingPageWidget/WidgetsUtils.test.tsx | Frontend test `WidgetsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/LeftSidebarClassBase.test.ts | Frontend test `LeftSidebarClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Lineage/Layout/ELKUtil/ELKUtil.test.ts | Frontend test `ELKUtil.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Lineage/LineageUtils.test.tsx | Frontend test `LineageUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/LogsClassBase.test.ts | Frontend test `LogsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/MessagingServiceUtils.test.ts | Frontend test `MessagingServiceUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/MetricEntityUtils/MetricUtils.test.ts | Frontend test `MetricUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/MlModelDetailsUtils.test.tsx | Frontend test `MlModelDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/NavbarUtilClassBase.test.tsx | Frontend test `NavbarUtilClassBase.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/NavbarUtils.test.tsx | Frontend test `NavbarUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ObjectUtils.test.ts | Frontend test `ObjectUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ObservabilityRouterClassBase.test.ts | Frontend test `ObservabilityRouterClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ObservabilityUtils.test.ts | Frontend test `ObservabilityUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/OidcTokenStorage.test.ts | Frontend test `OidcTokenStorage.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/OktaCustomStorage.test.ts | Frontend test `OktaCustomStorage.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/PaginationUtils.test.ts | Frontend test `PaginationUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ParameterForm/ParameterFormUtils.test.ts | Frontend test `ParameterFormUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Persona/PersonaUtils.test.ts | Frontend test `PersonaUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/PipelineDetailsUtils.test.tsx | Frontend test `PipelineDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/PipelineStatusUtils.test.ts | Frontend test `PipelineStatusUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ProfileSampleConfigUtils.test.ts | Frontend test `ProfileSampleConfigUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ProfilerUtils.test.ts | Frontend test `ProfilerUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/QueryBuilderUtils.test.ts | Frontend test `QueryBuilderUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/QueryClassBase.test.ts | Frontend test `QueryClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/QuillLink/QuillLink.test.ts | Frontend test `QuillLink.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/RouterUtils.test.ts | Frontend test `RouterUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/RuleEnforcementUtils.test.ts | Frontend test `RuleEnforcementUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SSOURLUtils.test.ts | Frontend test `SSOURLUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SSOUtils.test.ts | Frontend test `SSOUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SchedularUtils.test.tsx | Frontend test `SchedularUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SchemaVersionUtils.test.ts | Frontend test `SchemaVersionUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SearchClassBase.test.ts | Frontend test `SearchClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SearchIndexUtils.test.tsx | Frontend test `SearchIndexUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SearchServiceUtils.test.ts | Frontend test `SearchServiceUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SearchUtils.test.ts | Frontend test `SearchUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ServiceConnectionUtils.test.ts | Frontend test `ServiceConnectionUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ServiceInsightsTabUtils.test.tsx | Frontend test `ServiceInsightsTabUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ServiceUtils.test.ts | Frontend test `ServiceUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ServiceUtilsClassBase.test.ts | Frontend test `ServiceUtilsClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SettingsNavigationPageUtils.test.tsx | Frontend test `SettingsNavigationPageUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SpreadsheetClassBase.test.ts | Frontend test `SpreadsheetClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SpreadsheetDetailsUtils.test.tsx | Frontend test `SpreadsheetDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/StringUtils.test.ts | Frontend test `StringUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Suggestion/SuggestionUtils.test.tsx | Frontend test `SuggestionUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SwMessenger.test.ts | Frontend test `SwMessenger.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SwTokenStorage.test.ts | Frontend test `SwTokenStorage.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/SwTokenStorageUtils.test.ts | Frontend test `SwTokenStorageUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TableColumn.util.test.tsx | Frontend test `TableColumn.util.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TableProfilerUtils.test.ts | Frontend test `TableProfilerUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TableUtils.test.tsx | Frontend test `TableUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TagClassBase.test.ts | Frontend test `TagClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TagsUtils.test.tsx | Frontend test `TagsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TaskFormDesignerUtils.test.ts | Frontend test `TaskFormDesignerUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TaskFormSchemaUtils.test.ts | Frontend test `TaskFormSchemaUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TasksUtils.test.tsx | Frontend test `TasksUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TeamUtils.test.ts | Frontend test `TeamUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TestCaseUtils.test.tsx | Frontend test `TestCaseUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TestDefinitionUtils.test.ts | Frontend test `TestDefinitionUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TestSuiteUtils.test.ts | Frontend test `TestSuiteUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ThemeUtils.test.ts | Frontend test `ThemeUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TopicDetailsUtils.test.tsx | Frontend test `TopicDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/TypeImportsIntegration.test.ts | Frontend test `TypeImportsIntegration.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/UserClassBase.test.ts | Frontend test `UserClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/UserDataUtils.test.ts | Frontend test `UserDataUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/Users.util.test.tsx | Frontend test `Users.util.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/ViewportUtils.test.ts | Frontend test `ViewportUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/WebAnalyticsUtils.test.ts | Frontend test `WebAnalyticsUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/WorksheetClassBase.test.ts | Frontend test `WorksheetClassBase.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/WorksheetDetailsUtils.test.tsx | Frontend test `WorksheetDetailsUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/date-time/DateTimeUtils.test.ts | Frontend test `DateTimeUtils.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/elasticsearchQueryBuilder.test.ts | Frontend test `elasticsearchQueryBuilder.test.ts` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/formUtils.test.tsx | Frontend test `formUtils.test.tsx` | High |
| openmetadata-ui/src/main/resources/ui/src/utils/sanitize.utils.test.ts | Frontend test `sanitize.utils.test.ts` | High |

## Main Entry Points
| File Path | Evidence | Confidence |
|---|---|---|
| openmetadata-service/src/main/java/org/openmetadata/service/OpenMetadataApplication.java | `public class OpenMetadataApplication extends Application<OpenMetadataApplicationConfig>`; `main` bootstraps Dropwizard/Jersey server | High |
| openmetadata-k8s-operator/src/main/java/org/openmetadata/operator/OMJobOperatorApplication.java | `public class OMJobOperatorApplication`; `main` starts Java Operator SDK for OMJob CRDs | High |
| openmetadata-ui/src/main/resources/ui/src/index.tsx | `createRoot(container).render(<AppRoot />)`; React SPA bootstrap | High |
| ingestion/operators/docker/main.py | Python ingestion operator Docker entry script | High |
| openmetadata-airflow-apis/openmetadata_managed_apis/api/app.py | `get_blueprint()` registers Flask REST routes under `REST_API_ENDPOINT` | High |
| ingestion/pyproject.toml | Console script `metadata = metadata.cmd:metadata` | High |
