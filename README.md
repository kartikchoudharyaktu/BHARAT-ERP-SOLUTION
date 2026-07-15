# BHARAT ERP

**A comprehensive, multi-domain Enterprise Resource Planning platform** built for Indian enterprises — unifying accounting, CRM, payroll, statutory compliance, and forensic audit into a single real-time console, backed by a suite of 25+ specialized executive cockpits for finance, tax, logistics, and risk management, running on an 833-file Spring Boot backend.

> Developed by **Kartik Choudhary**
> Portal Status: Active · Secure · Real-Time Sync Enabled · Backend Build: SUCCESS (833 Java files)

---

## Table of Contents

1. [Overview](#overview)
2. [Screenshots](#screenshots)
3. [Architecture Deep-Dive](#architecture-deep-dive)
4. [Repository Structure](#repository-structure)
5. [Core Modules — Full Domain Deep-Dive](#core-modules--full-domain-deep-dive)
6. [Advanced Cockpits](#advanced-cockpits)
7. [Domain Package Reference](#domain-package-reference)
8. [Complete Backend File Index (All 833 Files)](#complete-backend-file-index-all-833-files)
9. [Frontend Structure](#frontend-structure)
10. [Tech Stack](#tech-stack)
11. [Getting Started](#getting-started)
12. [Configuration](#configuration)
13. [API Documentation](#api-documentation)
14. [Architecture Governance & ADR Process](#architecture-governance--adr-process)
15. [Contributing Guide](#contributing-guide)
16. [Changelog](#changelog)
17. [Roadmap](#roadmap)
18. [License](#license)
19. [Author](#author)

---

## Overview

BHARAT ERP is a full-stack enterprise platform that consolidates the core operations of an Indian business — ledger accounting, sales/CRM pipelines, payroll & statutory compliance, and forensic-grade audit — into a single, secure, real-time console.

The backend is a large Spring Boot application (833 Java source files, verified via full recursive scan) organized into domain packages under `com.bharaterp`, spanning finance, HR, compliance, CRM, inventory, logistics, supply chain, secretarial affairs, security, AI/agentic orchestration, and a wide range of industry-specific verticals (BFSI, healthcare, construction/RERA, agriculture, education, manufacturing, retail, and more). The frontend is a React 18 + Vite single-page application, styled with Tailwind CSS, delivering a fast, real-time operational dashboard experience across every module.

Beyond the four core modules (Ledger, CRM, Payroll, Audit), the platform ships with an **Advanced Cockpits** command palette — over 25 specialized dashboards purpose-built for functions like trade finance, tax simulation, AML forensics, IoT predictive maintenance, and RERA compliance, all accessible from a single unified navigation bar.

---

## Screenshots

### Ledger — Tally-Grade Double-Entry Accounting
Multi-currency voucher posting with forex sweep, auto-balanced statutory audit reconciliation, dual posting (DR/CR) ledger matrix, and full journal voucher workflows with legal audit compliance remarks.

![Ledger Module](./screenshots/ledger-module.jpg)

### CRM — Enterprise B2B Pipeline Management
Live pipeline sync, deal valuation tracking (₹), funnel stage control (New → Negotiation → Deal Won), and lead acquisition attribution across marketing channels (Website, Trade Show, Referral).

![CRM Module](./screenshots/crm-module.jpg)

### Payroll — Statutory Payroll & Labour Compliance Console
EPF (12%) / ESI (4.75%)-compliant disbursal engine with automated deduction computation, department-level allocation, and net payout rollout per employee.

![Payroll Module](./screenshots/payroll-module.jpg)

### Audit — Statutory & Forensic Investigation Desk
ICAI & CARO-aligned statutory financial fraud forensic investigation scanner, MCA XBRL gateway sync, DGGI tax telemetry channel, CARO invariant clause validation (21/21), and immutable integrity scoring.

![Audit Module](./screenshots/audit-module.jpg)

### Advanced Cockpits — Specialized Enterprise Add-ons
A dedicated command palette exposing 25+ specialized cockpits for finance, tax, logistics, and compliance teams.

![Advanced Cockpits Menu](./screenshots/advanced-cockpits-menu.jpg)
![Advanced Cockpits List](./screenshots/advanced-cockpits-list.jpg)

---

## Architecture Deep-Dive

### High-Level Architecture

```
┌───────────────────────────────────────────────────┐
│  Frontend — React 18 + Vite + Tailwind CSS         │
│  Real-time dashboard console, cockpit navigation   │
└────────────────────┬────────────────────────────────┘
                     │  REST API (JSON over HTTPS)
┌────────────────────▼────────────────────────────────┐
│  Security Layer — Spring Security (JWT-based)       │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Application Layer — Spring Boot (Java 21 LTS)      │
│   Controllers → Services → Repositories              │
│   45 domain packages under com.bharaterp             │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│  Data Layer — Relational DB (via Spring Data JPA)    │
│  + Redis (caching layer)                              │
└─────────────────────────────────────────────────────────┘
```

### Request Lifecycle

```
Client Request
    │
    ▼
[JwtAuthenticationFilter] ──▶ validates Bearer token via JwtTokenProvider
    │
    ▼
[SecurityConfig filter chain] ──▶ resolves authorities / role checks
    │
    ▼
[Controller] ──▶ deserializes request body, delegates to Service
    │
    ▼
[Service] ──▶ business logic, validation, orchestration across Repositories
    │
    ▼
[Repository] ──▶ Spring Data JPA → MySQL (with Redis cache-aside where configured)
    │
    ▼
Response serialized back to client as JSON
```

### Security Flow (JWT)

1. Client authenticates via `AuthController` → `AuthService` validates credentials against `UserRepository` (backed by `UserDetailsServiceImpl`).
2. On success, `JwtTokenProvider` issues a signed JWT (HS256).
3. Every subsequent request carries `Authorization: Bearer <token>`.
4. `JwtAuthenticationFilter` (a `OncePerRequestFilter`) intercepts, validates, and populates the Spring Security context per-request — no server-side session state.
5. `SecurityConfig` defines which endpoints require authentication vs. public access.
6. Additional layers: `ZeroTrustMaskingService` / `ZeroTrustEnforcer` for data masking and threat detection, `RbacService` for fine-grained role/permission checks, `ApiTokenBucketShield` for rate limiting.

### Caching Strategy

- **Cache-aside pattern** via Redis (`RedisConfig`), used for frequently-read reference data (compliance rating slabs, treasury burn snapshots, dashboard aggregates).
- `HybridCacheSyncService` reconciles cache state against the source of truth on a scheduled basis.
- `CacheSyncQueue` entity tracks pending sync operations for eventual consistency scenarios.

### Data Flow — Example: Payroll Disbursal

```
PayrollController
    │
    ▼
PayrollService ──▶ TaxCalculationService (TDS) ──▶ PfChallanService (EPF)
    │                                          └──▶ GratuityService
    ▼
BankDisbursementService ──▶ generates disbursement batch
    │
    ▼
Payslip entity persisted via PayslipRepository
    │
    ▼
ActivityLogService records an audit trail entry (security.audit package)
```

---

## Repository Structure

```
bharat_erp_solution/
├── backend/
│   ├── src/main/java/com/bharaterp/    # 45 domain packages, 833 files (see full index below)
│   ├── tools/                            # Internal PowerShell static-analysis framework
│   ├── pom.xml
│   ├── mvnw / mvnw.cmd
│   └── application.properties
├── frontend/
│   ├── src/
│   │   ├── components/                   # Ledger, CRM, Payroll, Audit, Cockpit UI
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── postcss.config.js
├── screenshots/                          # Module screenshots referenced in this README
└── README.md                             # This file
```

---

## Core Modules — Full Domain Deep-Dive

This section documents every major backend domain in detail — what it covers, and representative classes within it.

### HR & Payroll (`com.bharaterp.hr`) — 114 files

The largest domain overall — **Employee lifecycle** (onboarding/offboarding), **Attendance & Leave**, **Payroll** (PF/ESI/PT/TDS, Full & Final Settlement, Gratuity, Bank Disbursement), **Performance & Appraisal cycles**, **Recruitment**, **Shift Scheduling**, **Compensation Benchmarking**, **Workforce Capacity Planning**, **Gig Worker Contracts**, **Expatriate Shadow Payroll**, and **Succession Planning**.

**Key classes:** `ApplicantTrackingService`, `AppraisalCycle`, `AppraisalCycleRepository`, `AttendanceController`, `AttendanceRecord`, `AttendanceRepository`, `AttendanceService`, `BankDisbursementService`, `BenefitController`, `Candidate`, `CandidateRepository`, `CompanyHoliday`, `CompanyHolidayRepository`, `CompensationBenchmarkController`, `CompensationBenchmarkService`, `Employee`, `EmployeeBenefit`, `EmployeeBenefitRepository`, `EmployeeBenefitService`, `EmployeeController`, `EmployeeRepository`, `EmployeeService`, `EmployeeSkill`, `EmployeeSkillRepository`, `EngagementSurvey`, `EngagementSurveyController`, `EngagementSurveyRepository`, `EngagementSurveyService`, `ExpatriatePayrollController`, `ExpatriatePayrollNode`, `ExpatriatePayrollRepository`, `ExpatriateShadowPayrollEngine`, `ExpenseAuditVoucher`, `ExpenseAuditVoucherRepository`, `ExpenseClaim`, `ExpenseClaimRepository`, `ExpenseController`, `ExpenseOcrApiController`, `ExpenseOcrAuditController`, `ExpenseOcrAuditEngine`, `ExpenseOcrClaim`, `ExpenseOcrClaimRepository`, `ExpenseService`, `FnfService`, `GigContractorPayout`, `GigContractorPayoutRepository`, `GigWorkerContractController`, `GigWorkerContractLedger`, `GratuityService`, `HrAdminController`, `HrDocument`, `HrDocumentController`, `HrDocumentRepository`, `HrDocumentService`, `InterviewSchedulerService`, `JobRequisition`, `JobRequisitionRepository`, `LeaveBalance`, `LeaveBalanceRepository`, `LeaveBalanceService` ...and 52 more (see [full file index](#complete-backend-file-index-all-833-files))

---

### Industry Verticals (`com.bharaterp.industry`) — 102 files

Vertical-specific modules covering **BFSI** (AML, KYC, loans), **Construction/RERA** (escrow, project costing), **Education** (admissions, LMS), **Healthcare/Pharma** (drug batches, prescriptions, Schedule H compliance), **Retail** (POS, loyalty), **Manufacturing** (BOM/MRP, predictive maintenance), **Agriculture** (traceability, mandi pricing), **Logistics**, **Professional Services**, **Public Sector**, and **Aviation**.

**Key classes:** `AdmissionService`, `AdvancedWindsController`, `AgriBatch`, `AgriBatchRepository`, `AgriTraceabilityService`, `AgricultureController`, `AmlAlert`, `AmlAlertRepository`, `AmlMonitoringService`, `AmlTransaction`, `AmlTransactionRepository`, `Applicant`, `ApplicantRepository`, `AviationLogisticsService`, `BfsiController`, `BomEntry`, `BomEntryRepository`, `BudgetAllocation`, `BudgetExecutionService`, `CapitalMarketsService`, `CitizenProfile`, `CitizenProfileRepository`, `CitizenServicePortal`, `ClientPortalService`, `ConstructionMilestone`, `ConstructionMilestoneRepository`, `ConstructionProject`, `ConstructionProjectRepository`, `ConstructionProjectService`, `CoreBankingIntegration`, `CourseEnrollment`, `CourseEnrollmentRepository`, `CropBatch`, `CropBatchRepository`, `DrugBatch`, `DrugBatchRepository`, `DrugMaster`, `DrugMasterRepository`, `ECommercePlatformService`, `EducationController`, `Engagement`, `EngagementRepository`, `EnvironmentalTrackerService`, `FarmInventory`, `FeeStructure`, `FleetVehicle`, `FreightOrder`, `FreightOrderRepository`, `GrantApplication`, `GrantApplicationRepository`, `HealthcareController`, `InsuranceClaimsService`, `InventorySyncService`, `KycRecord`, `KycRecordRepository`, `KycVerificationService`, `LmsIntegrationService`, `LoanApplication`, `LoanApplicationRepository`, `LogisticsController` ...and 42 more (see [full file index](#complete-backend-file-index-all-833-files))

---

### Finance (`com.bharaterp.finance`) — 96 files

The largest financial domain — covers **Accounts Payable/Receivable**, **Fixed Assets** (incl. depreciation & revaluation), **Bank Reconciliation & Cash Pooling**, **Budgeting**, **Cash Flow Forecasting**, **Consolidation**, **Credit Risk Scoring**, **Currency Triangulation**, **Hedge Accounting**, **Revenue Recognition (ASC 606)**, and **Transfer Pricing**.

**Key classes:** `AccountsPayableController`, `AccountsPayableService`, `AccountsReceivableController`, `AccountsReceivableService`, `Asc606SubscriptionApiController`, `Asc606SubscriptionEngine`, `Asc606SubscriptionLedger`, `Asc606SubscriptionLedgerRepository`, `AssetFairValue`, `AssetFairValueController`, `AssetFairValueRepository`, `AssetFairValueService`, `AssetRevaluation`, `AssetRevaluationController`, `AssetRevaluationRepository`, `AssetRevaluationService`, `BankCashPool`, `BankCashPoolRepository`, `BankCashPoolingController`, `BankCashPoolingService`, `BankReconciliationController`, `BankReconciliationService`, `BankStatementMt940Parser`, `BankStatementParserController`, `BankStatementParserRepository`, `BankTransaction`, `BankTransactionRepository`, `BudgetController`, `BudgetLine`, `BudgetLineRepository`, `BudgetService`, `CashFlowForecastController`, `CashFlowForecastService`, `ConsolidationController`, `ConsolidationService`, `CreditRiskScoringController`, `CreditRiskScoringEngine`, `CurrencyTriangulation`, `CurrencyTriangulationController`, `CurrencyTriangulationRepository`, `CurrencyTriangulationService`, `Customer`, `CustomerController`, `CustomerCreditScore`, `CustomerCreditScoreRepository`, `CustomerInvoice`, `CustomerInvoiceRepository`, `CustomerRepository`, `DepreciationService`, `EquityDividend`, `EquityDividendController`, `EquityDividendRepository`, `EquityDividendService`, `ExportController`, `ExportService`, `FixedAsset`, `FixedAssetController`, `FixedAssetRepository`, `FixedAssetService`, `HedgeAccountingController` ...and 35 more (see [full file index](#complete-backend-file-index-all-833-files))

---

### CRM (`com.bharaterp.crm`) — 52 files

Full B2B CRM: **Accounts & Contacts**, **Leads & Opportunities**, **Activities & Tasks**, **Assignment Rules** (incl. round-robin pooling), **Custom Fields**, **Deal Products**, **Marketing Campaigns**, and **Email Sequences**.

**Key classes:** `Account`, `AccountContact`, `AccountContactRepository`, `AccountController`, `AccountRepository`, `AccountService`, `Activity`, `ActivityController`, `ActivityRepository`, `ActivityService`, `AssignmentRule`, `AssignmentRuleController`, `AssignmentRuleRepository`, `AssignmentRuleService`, `Campaign`, `CampaignController`, `CampaignRecipient`, `CampaignRecipientRepository`, `CampaignRepository`, `CampaignService`, `CustomFieldController`, `CustomFieldDefinition`, `CustomFieldDefinitionRepository`, `CustomFieldService`, `CustomFieldValue`, `CustomFieldValueRepository`, `EmailSequence`, `EmailSequenceController`, `EmailSequenceRepository`, `EmailSequenceService`, `Lead`, `LeadController`, `LeadRepository`, `LeadService`, `Opportunity`, `OpportunityController`, `OpportunityProduct`, `OpportunityProductController`, `OpportunityProductRepository`, `OpportunityProductService`, `OpportunityRepository`, `OpportunityService`, `RoundRobinPool`, `RoundRobinPoolRepository`, `SequenceEnrollment`, `SequenceEnrollmentRepository`, `SequenceStep`, `SequenceStepRepository`, `Task`, `TaskController`, `TaskRepository`, `TaskService`

---

### Compliance (`com.bharaterp.compliance`) — 46 files

Statutory and regulatory compliance: **EHS Incidents**, **E-Invoice & E-Way Bill**, **DPDPA Consent Audit**, **GST**, **Legal Matter Tracking**, **MDM Golden Records**, **Data Migration**, and the **Whistleblower Vault**.

**Key classes:** `ComplianceAgentService`, `ComplianceAsService`, `DPDPComplianceApiController`, `DPDPComplianceEngine`, `DpdpConsentAudit`, `DpdpConsentAuditRepository`, `EInvoiceComplianceController`, `EInvoiceExchangeService`, `EhsIncidentApiController`, `EhsIncidentMatrixEngine`, `EhsIncidentRecord`, `EhsIncidentRecordRepository`, `EhsIncidentRepository`, `EhsIncidentTrackingService`, `EnterpriseMigrationCockpitController`, `EnterpriseMigrationCockpitService`, `EwayBillApiController`, `EwayBillDetails`, `EwayBillRepository`, `GlobalComplianceController`, `GlobalTaxEngine`, `GstInvoiceSummary`, `GstReturnApiController`, `LegalMatter`, `LegalMatterController`, `LegalMatterRepository`, `LegalMatterTrackingService`, `MdmGoldenRecord`, `MdmGoldenRecordApiController`, `MdmGoldenRecordEngine`, `MdmGoldenRecordRepository`, `MigrationDataChunk`, `MigrationDataChunkRepository`, `MigrationJobRegistry`, `MigrationJobRegistryRepository`, `TaxAgentService`, `UniversalEInvoiceExchange`, `UniversalGlobalTaxEngine`, `WhistleblowerCase`, `WhistleblowerCaseRepository`, `WhistleblowerVaultController`, `WhistleblowerVaultService`

---

### Supply Chain (`com.bharaterp.supplychain`) — 34 files

**Container/Load Optimization**, **Dropshipping Orchestration**, **GS1 Barcode Serialization**, **IoT Plant Telemetry**, **Vendor-Managed Inventory**, **Wave-Picking Optimization**, and **Distributed Order Routing (DOM)**.

**Key classes:** `ContainerLoadJob`, `ContainerLoadJobRepository`, `ContainerLoadOptimizationController`, `ContainerLoadOptimizationService`, `DistributedOrderRoute`, `DistributedOrderRouteRepository`, `DistributedOrderRoutingController`, `DistributedOrderRoutingService`, `DropshipJob`, `DropshipJobRepository`, `DropshipOrchestrator`, `DropshipOrchestratorController`, `Gs1Barcode`, `Gs1BarcodeGeneratorController`, `Gs1BarcodeGeneratorService`, `Gs1BarcodeRepository`, `MandiTradeApiController`, `MandiTradeTicket`, `MandiTradeTicketRepository`, `PlantIotApiController`, `PlantIotTelemetry`, `PlantIotTelemetryRepository`, `SupplyChainGatewayController`, `SupplyChainShipment`, `SupplyChainShipmentRepository`, `VendorManagedInventoryController`, `VendorManagedInventoryEngine`, `VmiThresholdNode`, `VmiThresholdRepository`, `WavePickingJob`, `WavePickingJobRepository`, `WavePickingOptimizationController`, `WavePickingOptimizationService`

---

### Audit & Secretarial (`com.bharaterp.audit`) — 33 files

Statutory audit and secretarial affairs: **Audit Engagements**, **Board Meetings**, **CARO Clause Compliance**, **ROC Filings**, **Secretarial Audit Reports**, and **XBRL Filing Generation**.

**Key classes:** `AuditChecklistAutomationService`, `AuditChecklistItem`, `AuditChecklistItemRepository`, `AuditEngagement`, `AuditEngagementController`, `AuditEngagementRepository`, `AuditEngagementService`, `AuditSodEnforcer`, `BoardMeeting`, `BoardMeetingController`, `BoardMeetingRepository`, `CaroClauseResponse`, `CaroClauseResponseRepository`, `CaroComplianceGuardrail`, `CaroRuleEngine`, `ContinuousEvidenceCollector`, `McaFilingExporter`, `RocFiling`, `RocFilingController`, `RocFilingRepository`, `RocFilingService`, `SecretarialAuditReport`, `SecretarialAuditReportController`, `SecretarialAuditReportRepository`, `SecretarialComplianceController`, `SecretarialComplianceService`, `StatutoryRegisterService`, `UpgradedAuditController`, `XbrlFiling`, `XbrlFilingController`, `XbrlFilingRepository`, `XbrlGeneratorService`, `XbrlReportGenerator`

---

### Inventory (`com.bharaterp.inventory`) — 33 files

**Products**, **Purchase Orders**, **Sales Orders**, **Warehouse & Stock Management**, **Customer Credit Limits**, and **Weight-Based (Catch-Weight) Stock** tracking.

**Key classes:** `CatchWeightStock`, `CatchWeightStockRepository`, `CustomerCreditController`, `CustomerCreditLimit`, `CustomerCreditLimitRepository`, `CustomerCreditService`, `InventoryApiController`, `Product`, `ProductController`, `ProductRepository`, `ProductService`, `PurchaseOrder`, `PurchaseOrderController`, `PurchaseOrderItem`, `PurchaseOrderItemRepository`, `PurchaseOrderRepository`, `PurchaseOrderService`, `SalesOrder`, `SalesOrderController`, `SalesOrderItem`, `SalesOrderItemRepository`, `SalesOrderRepository`, `SalesOrderService`, `StockItem`, `StockItemRepository`, `VariableWeightStockController`, `VariableWeightStockEngine`, `Warehouse`, `WarehouseController`, `WarehouseRepository`, `WarehouseService`, `WarehouseStock`, `WarehouseStockRepository`

---

### Infrastructure (`com.bharaterp.infrastructure`) — 28 files

Distributed systems infrastructure: **DB Grid** (sharding, replication, failover, write-ahead logging, deadlock prevention), **IoT Edge** (device auth, MQTT/OPC-UA bridges, telemetry processing), and **AI Governance Training**.

**Key classes:** `AiImplementationTrainingService`, `ClusterConfigurationRegistry`, `ClusterHeartbeatMonitor`, `ClusterReplicationAuditor`, `DataConsistencyBroker`, `DataPurgingArchiver`, `DataSlicingReconciler`, `DeadlockPreventionEngine`, `DeviceFirmwareValidator`, `DistributedPartitionManager`, `DistributedShardingRouter`, `DistributedSnapshotEngine`, `DynamicQueryOptimizer`, `EdgeDataBufferCache`, `EdgeTelemetryProcessor`, `FailoverClusterManager`, `HardwareDeviceAuthenticator`, `HardwareEdgeReconciler`, `IndustryPlclinkingService`, `IngressDataSpikeMitigator`, `IoTHeartbeatMonitor`, `MqttBrokerConnector`, `OpcUaProtocolBridge`, `QueryRoutingLoadBalancer`, `RfidBatchIngestionEngine`, `TelemetryAnomalyFilter`, `WriteAheadLogEngine`, `WriteBufferFlushEngine`

---

### Security (`com.bharaterp.security`) — 26 files

**RBAC** (roles & permissions), **SSO**, **Activity Audit Logging**, **API Token Bucket Shielding** (rate limiting), **Zero-Trust Threat Detection**, and **ZKP (Zero-Knowledge Proof) Audit Attestation**.

**Key classes:** `AIThreatDetection`, `ActivityLog`, `ActivityLogController`, `ActivityLogRepository`, `ActivityLogService`, `ApiShieldPolicy`, `ApiShieldPolicyRepository`, `ApiTokenBucketShield`, `ApiTokenBucketShieldController`, `AuditLogAspect`, `Permission`, `PermissionRepository`, `RbacController`, `RbacService`, `RolePermission`, `RolePermissionRepository`, `SecurityIntegrityGuard`, `SharedSsoCoreController`, `SharedSsoCoreService`, `SsoSessionToken`, `SsoSessionTokenRepository`, `ZeroTrustEnforcer`, `ZkpAuditAttestation`, `ZkpAuditAttestationController`, `ZkpAuditAttestationRepository`, `ZkpAuditAttestationService`

---

### Backend Core / Config (`com.bharaterp.backend`) — 25 files

Core application bootstrap and configuration: **JWT Auth**, **Kafka**, **RabbitMQ**, **Redis**, **Swagger/OpenAPI**, plus the `AuthController`/`AuthService` and core `User` entity.

**Key classes:** `AiCopilotController`, `AiCopilotService`, `AuthController`, `AuthService`, `BackendApplication`, `CustomsClearanceService`, `DataMaskingService`, `FreightBillingService`, `GlobalizationService`, `IoTEventController`, `IoTEventService`, `JwtAuthenticationFilter`, `JwtTokenProvider`, `KafkaConfig`, `OpenApiMetadataConfig`, `RabbitMqConfig`, `RedisConfig`, `RouteOptimizationService`, `SecurityConfig`, `SwaggerConfig`, `User`, `UserDetailsServiceImpl`, `UserRepository`, `VahanIntegrationService`, `ZeroTrustMaskingService`

---

### Core Platform Utilities (`com.bharaterp.core`) — 22 files

Cross-cutting platform concerns: **Audit Trail Interceptor**, **Hybrid Cache Sync**, **Feature Flag Management**, **Org Role Hierarchy**, and **Distributed Transaction Coordination**.

**Key classes:** `AuditTrailInterceptor`, `CacheSyncQueue`, `CacheSyncQueueRepository`, `DistributedTransactionRegistry`, `DistributedTransactionRegistryRepository`, `DistributedTxCoordinatorController`, `DistributedTxCoordinatorEngine`, `FeatureFlagManagementController`, `FeatureFlagManagementService`, `FeatureFlagRegistry`, `FeatureFlagRegistryRepository`, `HybridCacheSyncController`, `HybridCacheSyncService`, `OrgRoleHierarchyController`, `OrgRoleHierarchyService`, `OrgRoleNode`, `OrgRoleNodeRepository`, `SecurityAuditBlock`, `SecurityAuditBlockRepository`, `SecurityAuditController`, `TransactionParticipantNode`, `TransactionParticipantNodeRepository`

---

### GST & Tax (`com.bharaterp.gst`) — 19 files

**GST Invoicing**, **GSTR Filing**, **HSN Mapping**, and **E-Way Bill** generation.

**Key classes:** `EWayBill`, `EWayBillController`, `EWayBillRepository`, `EWayBillService`, `GstCalculationService`, `GstInvoice`, `GstInvoiceController`, `GstInvoiceItem`, `GstInvoiceItemRepository`, `GstInvoiceRepository`, `GstInvoiceService`, `GstrFilingController`, `GstrFilingRecord`, `GstrFilingRecordRepository`, `GstrFilingService`, `HsnMappingController`, `HsnMappingService`, `HsnMaster`, `HsnMasterRepository`

---

### Sales (`com.bharaterp.sales`) — 19 files

**Sales Invoicing**, **CPQ (Configure-Price-Quote) Engine**, **Partner Rebate Ledger**, and **Tiered Volume Pricing**.

**Key classes:** `CpqEngineController`, `CpqEngineService`, `CpqQuote`, `CpqQuoteRepository`, `PartnerRebateLedger`, `PartnerRebateLedgerController`, `PartnerRebateLedgerRepository`, `PartnerRebateLedgerService`, `PricingTier`, `PricingTierRepository`, `RebateSlab`, `RebateSlabRepository`, `SalesInvoice`, `SalesInvoiceApiController`, `SalesInvoiceRepository`, `TieredVolumePricing`, `TieredVolumePricingController`, `TieredVolumePricingEngineService`, `TieredVolumePricingRepository`

---

### Autonomous Agents (`com.bharaterp.autonomous`) — 18 files

Autonomous, agent-driven automation: **Collections Agent**, **Inventory Tasking Agent**, **Supply Planning Agent**, **EPM Auto-Reforecasting**, **Strategic Risk Optimizer**, **Digital Factory Orchestrator**, **Auto Journal Entry**, **Intelligent Close Manager**, and industry-specific autonomous agents (logistics, retail, pharma, smart city).

**Key classes:** `AutoJournalEntryService`, `AutoReforecastService`, `AutonomousLogisticsAgent`, `AutonomousRetailAgent`, `BusinessResourceOptimizer`, `CollectionsAgent`, `DigitalFactoryOrchestrator`, `FactoryController`, `FactoryProcessIntelligenceService`, `IndustryAutonomyController`, `IntelligentCloseManager`, `InventoryTaskingAgent`, `ManufacturingOeeCalculator`, `PharmaSupplyChainAgent`, `SmartCityOrchestrator`, `StrategicRiskOptimizer`, `SupplyPlanningAgent`, `VarianceAnalysisAgent`

---

### AI / Agent Orchestration (`com.bharaterp.ai`) — 17 files

**AI Agent Hub**, **MCP (Model Context Protocol) Tool Registry**, **Multi-Agent Orchestration**, and a **RAG (Retrieval-Augmented Generation) Pipeline**.

**Key classes:** `AgentConsensusBalancer`, `AgentGovernanceService`, `AgentHubController`, `AgentRegistry`, `AgentRegistryRepository`, `AiHubExtensionController`, `GoogleSearchIntegration`, `KnowledgeDocument`, `KnowledgeDocumentRepository`, `McpServerService`, `McpToolRegistry`, `McpToolRegistryRepository`, `MultiAgentOrchestratorService`, `OrchestrationWorkflow`, `OrchestrationWorkflowRepository`, `RagPipelineService`, `SpringAiClaudeRouter`

---

### Platform Services (`com.bharaterp.platform`) — 17 files

**Multi-Tenant Schema Routing**, **Dynamic Schema Extension**, **Data Archiving/Purging**, and **Business Intelligence** services.

**Key classes:** `CustomFieldMeta`, `CustomFieldMetaRepository`, `DataPurgeLog`, `DataPurgeLogRepository`, `DataPurgingStorageController`, `DataPurgingStorageService`, `DynamicSchemaExtensionController`, `DynamicSchemaExtensionEngine`, `MultiTenantSchemaController`, `MultiTenantSchemaRouter`, `ProcessIntelligenceService`, `SystemArchiveRegistry`, `SystemArchiveRegistryRepository`, `SystemDataArchiverController`, `SystemDataArchiverVolumeManager`, `TenantRoutingRegistry`, `TenantRoutingRegistryRepository`

---

### Integration & Connectors (`com.bharaterp.integration`) — 14 files

**Blockchain Audit**, **Generic Connectors**, **Tally Packet Parsing**, **Universal Data Import**, and **Webhook Dispatch**.

**Key classes:** `BlockchainAuditService`, `BlockchainBlock`, `BlockchainBlockRepository`, `ConnectorConfig`, `ConnectorConfigRepository`, `ConnectorController`, `GenericConnectorService`, `TallyPacketParser`, `UniversalDataImporterService`, `UniversalImportController`, `WebhookController`, `WebhookDispatchService`, `WebhookSubscription`, `WebhookSubscriptionRepository`

---

### ESG & Sustainability (`com.bharaterp.esg`) — 13 files

**Carbon Footprint Tracking**, **Green Ledger**, **CSRD Compliance**, and **ESG Reporting** — both a core `esg.carbon`/`esg.universal` implementation and a `esg.vision.green` variant.

**Key classes:** `CSRDComplianceService`, `CarbonEsgController`, `CarbonFootprintTracker`, `CarbonGreenLedgerService`, `CarbonTaxOptimizer`, `EInvoicePeppolService`, `ESGReportingService`, `EsgComplianceController`, `EsgVisionGreenLedgerService`, `GlobalESGEngine`, `SAPGreenLedgerAdapter`, `UniversalCarbonTaxOptimizer`, `UniversalEsgController`

---

### Agentic Mesh (`com.bharaterp.agentic`) — 11 files

**Agentic Mesh Coordination** (AI Mesh Coordinator, Headless API Gateway), **Process Intelligence**, and **Knowledge Graph Services**.

**Key classes:** `AIMeshCoordinator`, `AgentOrchestrationEngine`, `AgenticController`, `AgenticOrchestrator`, `HeadlessApiGateway`, `JouleWorkWorkspace`, `KnowledgeGraphService`, `MeshSessionRegistry`, `MeshSessionRegistryRepository`, `ProcessIntelligenceService`, `RAPTService`

---

### Logistics (`com.bharaterp.logistics`) — 10 files

**Freight Billing**, **Route Optimization**, **Customs Clearance**, **VAHAN Integration**, and **Multimodal Shipment Tracking**.

**Key classes:** `ConsignmentShipment`, `CustomsShipment`, `CustomsShipmentRepository`, `FreightConsignment`, `FreightConsignmentRepository`, `GlobalCustomsClearanceService`, `LogisticsController`, `NationalLogisticsVahanService`, `Vehicle`, `VehicleRepository`

---

### Quantum (R&D / Experimental) (`com.bharaterp.quantum`) — 9 files

R&D/experimental cryptography and optimization: **Quantum-Safe Crypto Engine**, **BB84 Quantum Key Distribution**, **Post-Quantum Crypto Service**, and **Simulated Annealing Optimizer**.

**Key classes:** `BB84KeyDistributionService`, `PostQuantumCryptoService`, `QuantumComputationLog`, `QuantumComputationLogRepository`, `QuantumController`, `QuantumKeyDistribution`, `QuantumOptimizationEngine`, `QuantumSafeCryptoEngine`, `SimulatedAnnealingOptimizer`

---

### Manufacturing (`com.bharaterp.manufacturing`) — 8 files

**Shop-Floor Sequencing** and **Tooling Calibration Monitoring**.

**Key classes:** `ShopFloorJob`, `ShopFloorJobRepository`, `ShopFloorSequencingController`, `ShopFloorSequencingEngine`, `ToolingCalibrationLog`, `ToolingCalibrationLogRepository`, `ToolingCalibrationMonitorController`, `ToolingCalibrationMonitorService`

---

### Marketplace (`com.bharaterp.marketplace`) — 7 files

**Partner Agent Registry**, **Agent Metering & Billing**, and **Marketplace Listings**.

**Key classes:** `AgentBillingService`, `AgentMeteringBillingService`, `AgentSmartContract`, `AgentVendorRegistry`, `MarketplaceController`, `MarketplaceListingService`, `PartnerAgent`

---

### Blockchain (`com.bharaterp.blockchain`) — 6 files

**Supply-Chain Ledger**, **Pharma Traceability Bridge**, and **Smart Contract Engine**.

**Key classes:** `AntiCounterfeitPharmaEngine`, `BlockchainLedgerService`, `IoTBlockchainBridge`, `PharmaSupplyChainService`, `RealTimeTraceabilityBridge`, `SmartContractEngine`

---

### Accounting (Ledger Core) (`com.bharaterp.account`) — 5 files

The foundational **Ledger/Accounting core** — voucher entry, dual-posting journal lines, and the base ledger persistence layer that the Ledger UI module talks to directly.

**Key classes:** `Voucher`, `VoucherController`, `VoucherLine`, `VoucherRepository`, `VoucherService`

---

### Composable Architecture (`com.bharaterp.composable`) — 5 files

**Key classes:** `ApiGatewayService`, `CleanCoreValidator`, `DynamicPluginExtensionEngine`, `EventMeshService`, `ExtensionRegistry`

---

### Procurement (`com.bharaterp.procurement`) — 5 files

**Key classes:** `PurchaseOrder`, `PurchaseOrderApiController`, `PurchaseOrderRepository`, `VendorAssessmentService`, `VendorScorecardService`

---

### Banking (UPI) (`com.bharaterp.banking`) — 4 files

**Key classes:** `UpiPaymentController`, `UpiPaymentService`, `UpiTransaction`, `UpiTransactionRepository`

---

### Financial AI (CFO Agent) (`com.bharaterp.financial`) — 4 files

**Key classes:** `CfoAgentController`, `CfoAgentProfile`, `CfoAgentProfileRepository`, `CfoAgentService`

---

### Cognitive Digital Twin (`com.bharaterp.cognitive`) — 3 files

**Key classes:** `ContinuousPlanningService`, `EnterpriseDigitalTwin`, `ScenarioSimulationEngine`

---

### Decision Intelligence (`com.bharaterp.decision`) — 3 files

**Key classes:** `ContinuousForecastingEngine`, `InvisibleCognitivePlatform`, `WhatIfSimulationModel`

---

### Migration (Strangler Pattern) (`com.bharaterp.migration`) — 3 files

**Key classes:** `MicroservicesGateway`, `MigrationController`, `MonolithicDecoupler`

---

### Sovereign Cloud / GDPR (`com.bharaterp.sovereign`) — 3 files

**Key classes:** `DataResidencyService`, `GDPRComplianceService`, `LocalizationEngine`

---

### Universal Billing (`com.bharaterp.universal`) — 3 files

**Key classes:** `UniversalBillingApiController`, `UniversalInvoice`, `UniversalInvoiceRepository`

---

### Deployment Tools (`com.bharaterp.deployment`) — 2 files

**Key classes:** `GlobalEndpointSmokeTester`, `MicroserviceDecouplerRegistry`

---

### Global / IPO Compliance (`com.bharaterp.global`) — 2 files

**Key classes:** `InvestorReportingService`, `IpoComplianceService`

---

### Government API Integration (`com.bharaterp.govt`) — 2 files

**Key classes:** `GovtApiController`, `GovtApiService`

---

### Reporting & Dashboards (`com.bharaterp.reporting`) — 2 files

**Key classes:** `DashboardController`, `DashboardService`

---

### Spatial / AR-VR (`com.bharaterp.spatial`) — 2 files

**Key classes:** `ArVrIntegrationService`, `SpatialWorkspace`

---

### Digital Workforce (`com.bharaterp.workforce`) — 2 files

**Key classes:** `LearningExperienceManager`, `SkillsGapAnalyzer`

---

### Decision Intelligence (Scenario Modeling) (`com.bharaterp.decisionintelligence`) — 1 files

**Key classes:** `ScenarioModelingService`

---

### Cognitive Mesh (Overlord) (`com.bharaterp.overlord`) — 1 files

**Key classes:** `OmnipresentCognitiveMesh`

---

### Secretarial Compliance (`com.bharaterp.secretarial`) — 1 files

**Key classes:** `BoardMeetingService`

---

### XBRL Filing (`com.bharaterp.xbrl`) — 1 files

**Key classes:** `XbrlTaxonomyMappingService`

---

## Advanced Cockpits

BHARAT ERP includes a dedicated **Advanced Cockpits** navigation dropdown — specialized dashboards layered on top of the four core modules, giving finance, compliance, and operations teams deep, domain-specific tooling without leaving the platform.

### Finance & Tax
| Cockpit | Purpose |
|---|---|
| Executive Warroom Cockpit | Consolidated executive-level overview dashboard |
| Global Forex Consolidation | Multi-currency treasury consolidation |
| Asset Depreciation Calculator | Automated fixed-asset depreciation scheduling |
| Cost-Center Budget Interceptor Lock | Budget overrun prevention at cost-center level |
| Trade Finance Letter of Credit Gateway | LC issuance and trade finance workflow tracking |
| Strategic Advance Tax Simulator | Advance tax liability projection |
| Auto Bank Statement Reconciliation | Automated bank-to-ledger matching |
| What-If Market Variance Stress Simulator | Scenario modeling for market shocks |

### Compliance & Regulatory
| Cockpit | Purpose |
|---|---|
| MCA Regulatory XBRL Filing Exporter | XBRL-format filing generation for MCA |
| Immutable Secretarial Audit Log | Tamper-evident secretarial record keeping |
| Statutory Gratuity EPFO Vault | Gratuity and EPFO compliance tracking |
| Environment Safety Incident Analytics | EHS incident logging and analytics |
| RERA Compliance & Escrow Drawdown | Real-estate escrow account compliance |
| Expat Cross-Border Shadow Payroll | Cross-border employee shadow payroll tracking |

### Risk & Forensics
| Cockpit | Purpose |
|---|---|
| AML Suspicious Activity Forensic Radar | Anti-money-laundering pattern detection |
| Anonymous Whistleblower Crypt-Vault | Encrypted, anonymous whistleblower reporting |
| Autonomous Disbursement Recovery Agent | Automated recovery workflow for flagged disbursements |
| DevSecOps Cyber SARIF Defect Exporter | Security defect export in SARIF format |

### Operations & Supply Chain
| Cockpit | Purpose |
|---|---|
| Realtime IoT Predictive Maintenance | Sensor-driven equipment maintenance forecasting |
| 3D Warehouse Visual Bin Mapping | Visual warehouse bin-location mapping |
| Live Transit Cold Chain Tracker | Cold-chain logistics temperature/transit tracking |
| Manufacturing Bill of Materials (BOM) MRP | Materials requirement planning from BOM |
| Sourcing Partner Vendor Risk Radar | Vendor risk scoring for procurement |
| GS1 Barcode RFID Serialization | Product serialization per GS1 standards |

Each cockpit is accessible directly from the top navigation bar's **Advanced Cockpits** dropdown, alongside the core Ledger, CRM, Payroll, and Audit modules.

---

## Domain Package Reference

Verified against a full recursive source scan — **833 total files** across **45 domain packages**.

| Domain Package | Files | % of Codebase |
|---|---|---|
| `com.bharaterp.hr` (HR & Payroll) | 114 | 13.7% |
| `com.bharaterp.industry` (Industry Verticals) | 102 | 12.2% |
| `com.bharaterp.finance` (Finance) | 96 | 11.5% |
| `com.bharaterp.crm` (CRM) | 52 | 6.2% |
| `com.bharaterp.compliance` (Compliance) | 46 | 5.5% |
| `com.bharaterp.supplychain` (Supply Chain) | 34 | 4.1% |
| `com.bharaterp.audit` (Audit & Secretarial) | 33 | 4.0% |
| `com.bharaterp.inventory` (Inventory) | 33 | 4.0% |
| `com.bharaterp.infrastructure` (Infrastructure) | 28 | 3.4% |
| `com.bharaterp.security` (Security) | 26 | 3.1% |
| `com.bharaterp.backend` (Backend Core / Config) | 25 | 3.0% |
| `com.bharaterp.core` (Core Platform Utilities) | 22 | 2.6% |
| `com.bharaterp.gst` (GST & Tax) | 19 | 2.3% |
| `com.bharaterp.sales` (Sales) | 19 | 2.3% |
| `com.bharaterp.autonomous` (Autonomous Agents) | 18 | 2.2% |
| `com.bharaterp.ai` (AI / Agent Orchestration) | 17 | 2.0% |
| `com.bharaterp.platform` (Platform Services) | 17 | 2.0% |
| `com.bharaterp.integration` (Integration & Connectors) | 14 | 1.7% |
| `com.bharaterp.esg` (ESG & Sustainability) | 13 | 1.6% |
| `com.bharaterp.agentic` (Agentic Mesh) | 11 | 1.3% |
| `com.bharaterp.logistics` (Logistics) | 10 | 1.2% |
| `com.bharaterp.quantum` (Quantum (R&D / Experimental)) | 9 | 1.1% |
| `com.bharaterp.manufacturing` (Manufacturing) | 8 | 1.0% |
| `com.bharaterp.marketplace` (Marketplace) | 7 | 0.8% |
| `com.bharaterp.blockchain` (Blockchain) | 6 | 0.7% |
| `com.bharaterp.account` (Accounting (Ledger Core)) | 5 | 0.6% |
| `com.bharaterp.composable` (Composable Architecture) | 5 | 0.6% |
| `com.bharaterp.procurement` (Procurement) | 5 | 0.6% |
| `com.bharaterp.banking` (Banking (UPI)) | 4 | 0.5% |
| `com.bharaterp.financial` (Financial AI (CFO Agent)) | 4 | 0.5% |
| `com.bharaterp.cognitive` (Cognitive Digital Twin) | 3 | 0.4% |
| `com.bharaterp.decision` (Decision Intelligence) | 3 | 0.4% |
| `com.bharaterp.migration` (Migration (Strangler Pattern)) | 3 | 0.4% |
| `com.bharaterp.sovereign` (Sovereign Cloud / GDPR) | 3 | 0.4% |
| `com.bharaterp.universal` (Universal Billing) | 3 | 0.4% |
| `com.bharaterp.deployment` (Deployment Tools) | 2 | 0.2% |
| `com.bharaterp.global` (Global / IPO Compliance) | 2 | 0.2% |
| `com.bharaterp.govt` (Government API Integration) | 2 | 0.2% |
| `com.bharaterp.reporting` (Reporting & Dashboards) | 2 | 0.2% |
| `com.bharaterp.spatial` (Spatial / AR-VR) | 2 | 0.2% |
| `com.bharaterp.workforce` (Digital Workforce) | 2 | 0.2% |
| `com.bharaterp.decisionintelligence` (Decision Intelligence (Scenario Modeling)) | 1 | 0.1% |
| `com.bharaterp.overlord` (Cognitive Mesh (Overlord)) | 1 | 0.1% |
| `com.bharaterp.secretarial` (Secretarial Compliance) | 1 | 0.1% |
| `com.bharaterp.xbrl` (XBRL Filing) | 1 | 0.1% |

---

## Complete Backend File Index (All 833 Files)

Every Java source file in the backend, grouped by domain package. Auto-generated from a full recursive scan of `backend/src/main/java`.

### HR & Payroll — `com.bharaterp.hr` (114 files)

```
com/bharaterp/hr/controller/EmployeeController.java
com/bharaterp/hr/controller/attendance/AttendanceController.java
com/bharaterp/hr/controller/compensation/CompensationBenchmarkController.java
com/bharaterp/hr/controller/document/HrDocumentController.java
com/bharaterp/hr/controller/engagement/EngagementSurveyController.java
com/bharaterp/hr/controller/expense/ExpenseController.java
com/bharaterp/hr/controller/leave/LeaveController.java
com/bharaterp/hr/controller/lifecycle/BenefitController.java
com/bharaterp/hr/controller/payroll/HrAdminController.java
com/bharaterp/hr/controller/payroll/PayrollController.java
com/bharaterp/hr/controller/performance/PerformanceController.java
com/bharaterp/hr/controller/recruitment/RecruitmentController.java
com/bharaterp/hr/controller/shift/ShiftController.java
com/bharaterp/hr/controller/skills/SkillController.java
com/bharaterp/hr/entity/Employee.java
com/bharaterp/hr/entity/attendance/AttendanceRecord.java
com/bharaterp/hr/entity/attendance/CompanyHoliday.java
com/bharaterp/hr/entity/benefits/EmployeeBenefit.java
com/bharaterp/hr/entity/compensation/SalaryBand.java
com/bharaterp/hr/entity/document/HrDocument.java
com/bharaterp/hr/entity/engagement/EngagementSurvey.java
com/bharaterp/hr/entity/expense/ExpenseClaim.java
com/bharaterp/hr/entity/leave/LeaveBalance.java
com/bharaterp/hr/entity/leave/LeaveRequest.java
com/bharaterp/hr/entity/lifecycle/OffboardingChecklist.java
com/bharaterp/hr/entity/lifecycle/OffboardingTask.java
com/bharaterp/hr/entity/lifecycle/OnboardingChecklist.java
com/bharaterp/hr/entity/lifecycle/OnboardingTask.java
com/bharaterp/hr/entity/payroll/Payslip.java
com/bharaterp/hr/entity/performance/AppraisalCycle.java
com/bharaterp/hr/entity/performance/PerformanceGoal.java
com/bharaterp/hr/entity/recruitment/Candidate.java
com/bharaterp/hr/entity/recruitment/JobRequisition.java
com/bharaterp/hr/entity/shift/OvertimeRecord.java
com/bharaterp/hr/entity/shift/ShiftSchedule.java
com/bharaterp/hr/entity/skills/EmployeeSkill.java
com/bharaterp/hr/entity/skills/Skill.java
com/bharaterp/hr/expense/audit/controller/ExpenseOcrAuditController.java
com/bharaterp/hr/expense/audit/entity/ExpenseAuditVoucher.java
com/bharaterp/hr/expense/audit/repository/ExpenseAuditVoucherRepository.java
com/bharaterp/hr/expense/audit/service/ExpenseOcrAuditEngine.java
com/bharaterp/hr/expense/ocr/controller/ExpenseOcrApiController.java
com/bharaterp/hr/expense/ocr/entity/ExpenseOcrClaim.java
com/bharaterp/hr/expense/ocr/repository/ExpenseOcrClaimRepository.java
com/bharaterp/hr/expense/ocr/service/ExpenseOcrAuditEngine.java
com/bharaterp/hr/lms/compliance/LmsComplianceEngineController.java
com/bharaterp/hr/lms/compliance/LmsComplianceEngineService.java
com/bharaterp/hr/lms/compliance/entity/LmsComplianceCertificate.java
com/bharaterp/hr/lms/compliance/repository/LmsComplianceCertificateRepository.java
com/bharaterp/hr/payroll/expatriate/controller/ExpatriatePayrollController.java
com/bharaterp/hr/payroll/expatriate/entity/ExpatriatePayrollNode.java
com/bharaterp/hr/payroll/expatriate/repository/ExpatriatePayrollRepository.java
com/bharaterp/hr/payroll/expatriate/service/ExpatriateShadowPayrollEngine.java
com/bharaterp/hr/payroll/service/PayrollProcessingService.java
com/bharaterp/hr/payroll/service/TaxCalculationService.java
com/bharaterp/hr/recruitment/service/ApplicantTrackingService.java
com/bharaterp/hr/recruitment/service/InterviewSchedulerService.java
com/bharaterp/hr/repository/EmployeeRepository.java
com/bharaterp/hr/repository/attendance/AttendanceRepository.java
com/bharaterp/hr/repository/attendance/CompanyHolidayRepository.java
com/bharaterp/hr/repository/benefits/EmployeeBenefitRepository.java
com/bharaterp/hr/repository/compensation/SalaryBandRepository.java
com/bharaterp/hr/repository/document/HrDocumentRepository.java
com/bharaterp/hr/repository/engagement/EngagementSurveyRepository.java
com/bharaterp/hr/repository/expense/ExpenseClaimRepository.java
com/bharaterp/hr/repository/leave/LeaveBalanceRepository.java
com/bharaterp/hr/repository/leave/LeaveRequestRepository.java
com/bharaterp/hr/repository/lifecycle/OffboardingChecklistRepository.java
com/bharaterp/hr/repository/lifecycle/OffboardingTaskRepository.java
com/bharaterp/hr/repository/lifecycle/OnboardingChecklistRepository.java
com/bharaterp/hr/repository/lifecycle/OnboardingTaskRepository.java
com/bharaterp/hr/repository/payroll/PayslipRepository.java
com/bharaterp/hr/repository/performance/AppraisalCycleRepository.java
com/bharaterp/hr/repository/performance/PerformanceGoalRepository.java
com/bharaterp/hr/repository/recruitment/CandidateRepository.java
com/bharaterp/hr/repository/recruitment/JobRequisitionRepository.java
com/bharaterp/hr/repository/shift/OvertimeRecordRepository.java
com/bharaterp/hr/repository/shift/ShiftScheduleRepository.java
com/bharaterp/hr/repository/skills/EmployeeSkillRepository.java
com/bharaterp/hr/repository/skills/SkillRepository.java
com/bharaterp/hr/service/EmployeeService.java
com/bharaterp/hr/service/attendance/AttendanceService.java
com/bharaterp/hr/service/benefits/EmployeeBenefitService.java
com/bharaterp/hr/service/compensation/CompensationBenchmarkService.java
com/bharaterp/hr/service/document/HrDocumentService.java
com/bharaterp/hr/service/engagement/EngagementSurveyService.java
com/bharaterp/hr/service/expense/ExpenseService.java
com/bharaterp/hr/service/leave/LeaveBalanceService.java
com/bharaterp/hr/service/leave/LeaveService.java
com/bharaterp/hr/service/lifecycle/OffboardingService.java
com/bharaterp/hr/service/lifecycle/OnboardingService.java
com/bharaterp/hr/service/payroll/BankDisbursementService.java
com/bharaterp/hr/service/payroll/FnfService.java
com/bharaterp/hr/service/payroll/GratuityService.java
com/bharaterp/hr/service/payroll/PayrollService.java
com/bharaterp/hr/service/payroll/PfChallanService.java
com/bharaterp/hr/service/payroll/TdsService.java
com/bharaterp/hr/service/performance/PerformanceService.java
com/bharaterp/hr/service/recruitment/RecruitmentService.java
com/bharaterp/hr/service/shift/ShiftService.java
com/bharaterp/hr/service/skills/SkillService.java
com/bharaterp/hr/talent/succession/controller/SuccessionReadinessController.java
com/bharaterp/hr/talent/succession/entity/SuccessionPlan.java
com/bharaterp/hr/talent/succession/repository/SuccessionPlanRepository.java
com/bharaterp/hr/talent/succession/service/SuccessionReadinessEngine.java
com/bharaterp/hr/workforce/capacity/WorkforceCapacityPlanningService.java
com/bharaterp/hr/workforce/capacity/controller/WorkforceCapacityPlanningController.java
com/bharaterp/hr/workforce/capacity/entity/WorkforceCapacityPlan.java
com/bharaterp/hr/workforce/capacity/repository/WorkforceCapacityPlanRepository.java
com/bharaterp/hr/workforce/capacity/service/WorkforceCapacityPlanningService.java
com/bharaterp/hr/workforce/gig/controller/GigWorkerContractController.java
com/bharaterp/hr/workforce/gig/entity/GigContractorPayout.java
com/bharaterp/hr/workforce/gig/repository/GigContractorPayoutRepository.java
com/bharaterp/hr/workforce/gig/service/GigWorkerContractLedger.java
```

### Industry Verticals — `com.bharaterp.industry` (102 files)

```
com/bharaterp/industry/agriculture/controller/AgricultureController.java
com/bharaterp/industry/agriculture/entity/AgriBatch.java
com/bharaterp/industry/agriculture/entity/CropBatch.java
com/bharaterp/industry/agriculture/entity/FarmInventory.java
com/bharaterp/industry/agriculture/entity/MandiPriceRecord.java
com/bharaterp/industry/agriculture/entity/TraceEvent.java
com/bharaterp/industry/agriculture/repository/AgriBatchRepository.java
com/bharaterp/industry/agriculture/repository/CropBatchRepository.java
com/bharaterp/industry/agriculture/repository/MandiPriceRecordRepository.java
com/bharaterp/industry/agriculture/repository/TraceEventRepository.java
com/bharaterp/industry/agriculture/service/AgriTraceabilityService.java
com/bharaterp/industry/agriculture/service/MandiPriceService.java
com/bharaterp/industry/bfsi/controller/BfsiController.java
com/bharaterp/industry/bfsi/entity/AmlAlert.java
com/bharaterp/industry/bfsi/entity/AmlTransaction.java
com/bharaterp/industry/bfsi/entity/KycRecord.java
com/bharaterp/industry/bfsi/entity/LoanApplication.java
com/bharaterp/industry/bfsi/repository/AmlAlertRepository.java
com/bharaterp/industry/bfsi/repository/AmlTransactionRepository.java
com/bharaterp/industry/bfsi/repository/KycRecordRepository.java
com/bharaterp/industry/bfsi/repository/LoanApplicationRepository.java
com/bharaterp/industry/bfsi/service/AmlMonitoringService.java
com/bharaterp/industry/bfsi/service/CoreBankingIntegration.java
com/bharaterp/industry/bfsi/service/KycVerificationService.java
com/bharaterp/industry/construction/controller/RealEstateController.java
com/bharaterp/industry/construction/entity/ConstructionMilestone.java
com/bharaterp/industry/construction/entity/ConstructionProject.java
com/bharaterp/industry/construction/entity/ProjectCosting.java
com/bharaterp/industry/construction/entity/ReraEscrowTransaction.java
com/bharaterp/industry/construction/repository/ConstructionMilestoneRepository.java
com/bharaterp/industry/construction/repository/ConstructionProjectRepository.java
com/bharaterp/industry/construction/repository/ProjectCostingRepository.java
com/bharaterp/industry/construction/repository/ReraEscrowTransactionRepository.java
com/bharaterp/industry/construction/service/ConstructionProjectService.java
com/bharaterp/industry/construction/service/ReraComplianceService.java
com/bharaterp/industry/education/controller/EducationController.java
com/bharaterp/industry/education/entity/Applicant.java
com/bharaterp/industry/education/entity/CourseEnrollment.java
com/bharaterp/industry/education/entity/FeeStructure.java
com/bharaterp/industry/education/entity/SeatAllocation.java
com/bharaterp/industry/education/entity/Student.java
com/bharaterp/industry/education/repository/ApplicantRepository.java
com/bharaterp/industry/education/repository/CourseEnrollmentRepository.java
com/bharaterp/industry/education/repository/SeatAllocationRepository.java
com/bharaterp/industry/education/repository/StudentRepository.java
com/bharaterp/industry/education/service/AdmissionService.java
com/bharaterp/industry/education/service/LmsIntegrationService.java
com/bharaterp/industry/finance/service/CapitalMarketsService.java
com/bharaterp/industry/finance/service/InsuranceClaimsService.java
com/bharaterp/industry/globaltech/service/ECommercePlatformService.java
com/bharaterp/industry/healthcare/controller/HealthcareController.java
com/bharaterp/industry/healthcare/entity/DrugBatch.java
com/bharaterp/industry/healthcare/entity/DrugMaster.java
com/bharaterp/industry/healthcare/entity/PharmaBatch.java
com/bharaterp/industry/healthcare/entity/Prescription.java
com/bharaterp/industry/healthcare/entity/PrescriptionSaleRecord.java
com/bharaterp/industry/healthcare/repository/DrugBatchRepository.java
com/bharaterp/industry/healthcare/repository/DrugMasterRepository.java
com/bharaterp/industry/healthcare/repository/PharmaBatchRepository.java
com/bharaterp/industry/healthcare/repository/PrescriptionRepository.java
com/bharaterp/industry/healthcare/repository/PrescriptionSaleRecordRepository.java
com/bharaterp/industry/healthcare/service/PharmaInventoryService.java
com/bharaterp/industry/healthcare/service/ScheduleHComplianceService.java
com/bharaterp/industry/logistics/controller/LogisticsController.java
com/bharaterp/industry/logistics/entity/FleetVehicle.java
com/bharaterp/industry/logistics/entity/FreightOrder.java
com/bharaterp/industry/logistics/repository/FreightOrderRepository.java
com/bharaterp/industry/logistics/service/MultimodalFreightService.java
com/bharaterp/industry/manufacturing/controller/ManufacturingController.java
com/bharaterp/industry/manufacturing/controller/ProductionMrpController.java
com/bharaterp/industry/manufacturing/entity/BomEntry.java
com/bharaterp/industry/manufacturing/repository/BomEntryRepository.java
com/bharaterp/industry/manufacturing/service/EnvironmentalTrackerService.java
com/bharaterp/industry/manufacturing/service/ManufacturingEngineService.java
com/bharaterp/industry/manufacturing/service/PredictiveMaintenanceService.java
com/bharaterp/industry/manufacturing/service/ProductionMrpService.java
com/bharaterp/industry/manufacturing/service/TextileMatrixService.java
com/bharaterp/industry/manufacturing/service/VehicleLifecycleService.java
com/bharaterp/industry/ngo/entity/GrantApplication.java
com/bharaterp/industry/ngo/repository/GrantApplicationRepository.java
com/bharaterp/industry/professionalservices/controller/ProfessionalServicesController.java
com/bharaterp/industry/professionalservices/entity/Engagement.java
com/bharaterp/industry/professionalservices/entity/Timesheet.java
com/bharaterp/industry/professionalservices/repository/EngagementRepository.java
com/bharaterp/industry/professionalservices/service/ClientPortalService.java
com/bharaterp/industry/professionalservices/service/PracticeManagementService.java
com/bharaterp/industry/publicsector/controller/MunicipalController.java
com/bharaterp/industry/publicsector/entity/BudgetAllocation.java
com/bharaterp/industry/publicsector/entity/CitizenProfile.java
com/bharaterp/industry/publicsector/repository/CitizenProfileRepository.java
com/bharaterp/industry/publicsector/service/BudgetExecutionService.java
com/bharaterp/industry/publicsector/service/CitizenServicePortal.java
com/bharaterp/industry/retail/controller/RetailController.java
com/bharaterp/industry/retail/entity/PosTransaction.java
com/bharaterp/industry/retail/entity/ProductCatalog.java
com/bharaterp/industry/retail/repository/PosTransactionRepository.java
com/bharaterp/industry/retail/repository/ProductCatalogRepository.java
com/bharaterp/industry/retail/service/InventorySyncService.java
com/bharaterp/industry/retail/service/LoyaltyService.java
com/bharaterp/industry/services/controller/AdvancedWindsController.java
com/bharaterp/industry/services/service/AviationLogisticsService.java
com/bharaterp/industry/services/service/TradePromotionsService.java
```

### Finance — `com.bharaterp.finance` (96 files)

```
com/bharaterp/finance/accounting/parallel/controller/ParallelAccountingController.java
com/bharaterp/finance/accounting/parallel/entity/ParallelLedgerEntry.java
com/bharaterp/finance/accounting/parallel/repository/ParallelLedgerRepository.java
com/bharaterp/finance/accounting/parallel/service/ParallelAccountingEngine.java
com/bharaterp/finance/assets/fairvalue/AssetFairValueController.java
com/bharaterp/finance/assets/fairvalue/AssetFairValueService.java
com/bharaterp/finance/assets/fairvalue/entity/AssetFairValue.java
com/bharaterp/finance/assets/fairvalue/repository/AssetFairValueRepository.java
com/bharaterp/finance/assets/revaluation/controller/AssetRevaluationController.java
com/bharaterp/finance/assets/revaluation/entity/AssetRevaluation.java
com/bharaterp/finance/assets/revaluation/repository/AssetRevaluationRepository.java
com/bharaterp/finance/assets/revaluation/service/AssetRevaluationService.java
com/bharaterp/finance/bank/parser/controller/BankStatementParserController.java
com/bharaterp/finance/bank/parser/entity/ParsedBankStatement.java
com/bharaterp/finance/bank/parser/repository/BankStatementParserRepository.java
com/bharaterp/finance/bank/parser/service/BankStatementMt940Parser.java
com/bharaterp/finance/bank/pooling/BankCashPoolingService.java
com/bharaterp/finance/bank/pooling/controller/BankCashPoolingController.java
com/bharaterp/finance/bank/pooling/entity/BankCashPool.java
com/bharaterp/finance/bank/pooling/repository/BankCashPoolRepository.java
com/bharaterp/finance/bank/pooling/service/BankCashPoolingService.java
com/bharaterp/finance/consolidation/elimination/controller/IntercompanyEliminationController.java
com/bharaterp/finance/consolidation/elimination/entity/IntercompanyTransaction.java
com/bharaterp/finance/consolidation/elimination/repository/IntercompanyTransactionRepository.java
com/bharaterp/finance/consolidation/elimination/service/IntercompanyEliminationEngine.java
com/bharaterp/finance/controller/LedgerController.java
com/bharaterp/finance/controller/ap/AccountsPayableController.java
com/bharaterp/finance/controller/ap/VendorController.java
com/bharaterp/finance/controller/ar/AccountsReceivableController.java
com/bharaterp/finance/controller/ar/CustomerController.java
com/bharaterp/finance/controller/assets/FixedAssetController.java
com/bharaterp/finance/controller/bank/BankReconciliationController.java
com/bharaterp/finance/controller/budget/BudgetController.java
com/bharaterp/finance/controller/cashflow/CashFlowForecastController.java
com/bharaterp/finance/controller/consolidation/ConsolidationController.java
com/bharaterp/finance/controller/export/ExportController.java
com/bharaterp/finance/credit/risk/controller/CreditRiskScoringController.java
com/bharaterp/finance/credit/risk/entity/CustomerCreditScore.java
com/bharaterp/finance/credit/risk/repository/CustomerCreditScoreRepository.java
com/bharaterp/finance/credit/risk/service/CreditRiskScoringEngine.java
com/bharaterp/finance/currency/triangulation/controller/CurrencyTriangulationController.java
com/bharaterp/finance/currency/triangulation/entity/CurrencyTriangulation.java
com/bharaterp/finance/currency/triangulation/repository/CurrencyTriangulationRepository.java
com/bharaterp/finance/currency/triangulation/service/CurrencyTriangulationService.java
com/bharaterp/finance/entity/LedgerEntry.java
com/bharaterp/finance/entity/ap/Vendor.java
com/bharaterp/finance/entity/ap/VendorInvoice.java
com/bharaterp/finance/entity/ar/Customer.java
com/bharaterp/finance/entity/ar/CustomerInvoice.java
com/bharaterp/finance/entity/assets/FixedAsset.java
com/bharaterp/finance/entity/bank/BankTransaction.java
com/bharaterp/finance/entity/budget/BudgetLine.java
com/bharaterp/finance/equity/dividend/EquityDividendController.java
com/bharaterp/finance/equity/dividend/EquityDividendService.java
com/bharaterp/finance/equity/dividend/entity/EquityDividend.java
com/bharaterp/finance/equity/dividend/repository/EquityDividendRepository.java
com/bharaterp/finance/hedge/accounting/HedgeAccountingController.java
com/bharaterp/finance/hedge/accounting/HedgeAccountingService.java
com/bharaterp/finance/hedge/accounting/entity/HedgeDerivative.java
com/bharaterp/finance/hedge/accounting/repository/HedgeDerivativeRepository.java
com/bharaterp/finance/intercompany/loan/controller/IntercompanyLoanController.java
com/bharaterp/finance/intercompany/loan/entity/IntercompanyLoan.java
com/bharaterp/finance/intercompany/loan/repository/IntercompanyLoanRepository.java
com/bharaterp/finance/intercompany/loan/service/IntercompanyLoanService.java
com/bharaterp/finance/repository/LedgerEntryRepository.java
com/bharaterp/finance/repository/ap/VendorInvoiceRepository.java
com/bharaterp/finance/repository/ap/VendorRepository.java
com/bharaterp/finance/repository/ar/CustomerInvoiceRepository.java
com/bharaterp/finance/repository/ar/CustomerRepository.java
com/bharaterp/finance/repository/assets/FixedAssetRepository.java
com/bharaterp/finance/repository/bank/BankTransactionRepository.java
com/bharaterp/finance/repository/budget/BudgetLineRepository.java
com/bharaterp/finance/revenue/recognition/controller/RevenueRecognitionApiController.java
com/bharaterp/finance/revenue/recognition/entity/PerformanceObligation.java
com/bharaterp/finance/revenue/recognition/entity/RevenueContract.java
com/bharaterp/finance/revenue/recognition/repository/PerformanceObligationRepository.java
com/bharaterp/finance/revenue/recognition/repository/RevenueContractRepository.java
com/bharaterp/finance/revenue/recognition/service/RevenueRecognitionAutomationService.java
com/bharaterp/finance/service/LedgerService.java
com/bharaterp/finance/service/ap/AccountsPayableService.java
com/bharaterp/finance/service/ar/AccountsReceivableService.java
com/bharaterp/finance/service/assets/DepreciationService.java
com/bharaterp/finance/service/assets/FixedAssetService.java
com/bharaterp/finance/service/bank/BankReconciliationService.java
com/bharaterp/finance/service/budget/BudgetService.java
com/bharaterp/finance/service/cashflow/CashFlowForecastService.java
com/bharaterp/finance/service/consolidation/ConsolidationService.java
com/bharaterp/finance/service/export/ExportService.java
com/bharaterp/finance/subscription/asc606/controller/Asc606SubscriptionApiController.java
com/bharaterp/finance/subscription/asc606/entity/Asc606SubscriptionLedger.java
com/bharaterp/finance/subscription/asc606/repository/Asc606SubscriptionLedgerRepository.java
com/bharaterp/finance/subscription/asc606/service/Asc606SubscriptionEngine.java
com/bharaterp/finance/transferpricing/controller/TransferPricingApiController.java
com/bharaterp/finance/transferpricing/entity/TransferPricingLedger.java
com/bharaterp/finance/transferpricing/repository/TransferPricingLedgerRepository.java
com/bharaterp/finance/transferpricing/service/TransferPricingEngine.java
```

### CRM — `com.bharaterp.crm` (52 files)

```
com/bharaterp/crm/controller/LeadController.java
com/bharaterp/crm/controller/OpportunityController.java
com/bharaterp/crm/controller/account/AccountController.java
com/bharaterp/crm/controller/activity/ActivityController.java
com/bharaterp/crm/controller/activity/TaskController.java
com/bharaterp/crm/controller/assignment/AssignmentRuleController.java
com/bharaterp/crm/controller/customfield/CustomFieldController.java
com/bharaterp/crm/controller/dealproduct/OpportunityProductController.java
com/bharaterp/crm/controller/marketing/CampaignController.java
com/bharaterp/crm/controller/sequence/EmailSequenceController.java
com/bharaterp/crm/entity/Lead.java
com/bharaterp/crm/entity/Opportunity.java
com/bharaterp/crm/entity/account/Account.java
com/bharaterp/crm/entity/account/AccountContact.java
com/bharaterp/crm/entity/activity/Activity.java
com/bharaterp/crm/entity/activity/Task.java
com/bharaterp/crm/entity/assignment/AssignmentRule.java
com/bharaterp/crm/entity/assignment/RoundRobinPool.java
com/bharaterp/crm/entity/customfield/CustomFieldDefinition.java
com/bharaterp/crm/entity/customfield/CustomFieldValue.java
com/bharaterp/crm/entity/dealproduct/OpportunityProduct.java
com/bharaterp/crm/entity/marketing/Campaign.java
com/bharaterp/crm/entity/marketing/CampaignRecipient.java
com/bharaterp/crm/entity/sequence/EmailSequence.java
com/bharaterp/crm/entity/sequence/SequenceEnrollment.java
com/bharaterp/crm/entity/sequence/SequenceStep.java
com/bharaterp/crm/repository/LeadRepository.java
com/bharaterp/crm/repository/OpportunityRepository.java
com/bharaterp/crm/repository/account/AccountContactRepository.java
com/bharaterp/crm/repository/account/AccountRepository.java
com/bharaterp/crm/repository/activity/ActivityRepository.java
com/bharaterp/crm/repository/activity/TaskRepository.java
com/bharaterp/crm/repository/assignment/AssignmentRuleRepository.java
com/bharaterp/crm/repository/assignment/RoundRobinPoolRepository.java
com/bharaterp/crm/repository/customfield/CustomFieldDefinitionRepository.java
com/bharaterp/crm/repository/customfield/CustomFieldValueRepository.java
com/bharaterp/crm/repository/dealproduct/OpportunityProductRepository.java
com/bharaterp/crm/repository/marketing/CampaignRecipientRepository.java
com/bharaterp/crm/repository/marketing/CampaignRepository.java
com/bharaterp/crm/repository/sequence/EmailSequenceRepository.java
com/bharaterp/crm/repository/sequence/SequenceEnrollmentRepository.java
com/bharaterp/crm/repository/sequence/SequenceStepRepository.java
com/bharaterp/crm/service/LeadService.java
com/bharaterp/crm/service/OpportunityService.java
com/bharaterp/crm/service/account/AccountService.java
com/bharaterp/crm/service/activity/ActivityService.java
com/bharaterp/crm/service/activity/TaskService.java
com/bharaterp/crm/service/assignment/AssignmentRuleService.java
com/bharaterp/crm/service/customfield/CustomFieldService.java
com/bharaterp/crm/service/dealproduct/OpportunityProductService.java
com/bharaterp/crm/service/marketing/CampaignService.java
com/bharaterp/crm/service/sequence/EmailSequenceService.java
```

### Compliance — `com.bharaterp.compliance` (46 files)

```
com/bharaterp/compliance/ehs/controller/EhsIncidentApiController.java
com/bharaterp/compliance/ehs/entity/EhsIncidentRecord.java
com/bharaterp/compliance/ehs/incident/EhsIncidentApiController.java
com/bharaterp/compliance/ehs/incident/EhsIncidentMatrixEngine.java
com/bharaterp/compliance/ehs/incident/entity/EhsIncidentRecord.java
com/bharaterp/compliance/ehs/incident/repository/EhsIncidentRecordRepository.java
com/bharaterp/compliance/ehs/repository/EhsIncidentRecordRepository.java
com/bharaterp/compliance/ehs/repository/EhsIncidentRepository.java
com/bharaterp/compliance/ehs/service/EhsIncidentMatrixEngine.java
com/bharaterp/compliance/ehs/service/EhsIncidentTrackingService.java
com/bharaterp/compliance/einvoice/controller/EInvoiceComplianceController.java
com/bharaterp/compliance/einvoice/service/ComplianceAsService.java
com/bharaterp/compliance/einvoice/service/EInvoiceExchangeService.java
com/bharaterp/compliance/einvoice/service/TaxAgentService.java
com/bharaterp/compliance/eway/controller/EwayBillApiController.java
com/bharaterp/compliance/eway/entity/EwayBillDetails.java
com/bharaterp/compliance/eway/repository/EwayBillRepository.java
com/bharaterp/compliance/global/ComplianceAgentService.java
com/bharaterp/compliance/global/DPDPComplianceApiController.java
com/bharaterp/compliance/global/DPDPComplianceEngine.java
com/bharaterp/compliance/global/GlobalComplianceController.java
com/bharaterp/compliance/global/GlobalTaxEngine.java
com/bharaterp/compliance/global/UniversalEInvoiceExchange.java
com/bharaterp/compliance/global/UniversalGlobalTaxEngine.java
com/bharaterp/compliance/global/entity/DpdpConsentAudit.java
com/bharaterp/compliance/global/repository/DpdpConsentAuditRepository.java
com/bharaterp/compliance/gst/controller/GstReturnApiController.java
com/bharaterp/compliance/gst/entity/GstInvoiceSummary.java
com/bharaterp/compliance/legal/controller/LegalMatterController.java
com/bharaterp/compliance/legal/entity/LegalMatter.java
com/bharaterp/compliance/legal/repository/LegalMatterRepository.java
com/bharaterp/compliance/legal/service/LegalMatterTrackingService.java
com/bharaterp/compliance/mdm/controller/MdmGoldenRecordApiController.java
com/bharaterp/compliance/mdm/entity/MdmGoldenRecord.java
com/bharaterp/compliance/mdm/repository/MdmGoldenRecordRepository.java
com/bharaterp/compliance/mdm/service/MdmGoldenRecordEngine.java
com/bharaterp/compliance/migration/EnterpriseMigrationCockpitController.java
com/bharaterp/compliance/migration/EnterpriseMigrationCockpitService.java
com/bharaterp/compliance/migration/entity/MigrationDataChunk.java
com/bharaterp/compliance/migration/entity/MigrationJobRegistry.java
com/bharaterp/compliance/migration/repository/MigrationDataChunkRepository.java
com/bharaterp/compliance/migration/repository/MigrationJobRegistryRepository.java
com/bharaterp/compliance/whistleblower/WhistleblowerVaultController.java
com/bharaterp/compliance/whistleblower/WhistleblowerVaultService.java
com/bharaterp/compliance/whistleblower/entity/WhistleblowerCase.java
com/bharaterp/compliance/whistleblower/repository/WhistleblowerCaseRepository.java
```

### Supply Chain — `com.bharaterp.supplychain` (34 files)

```
com/bharaterp/supplychain/agri/controller/MandiTradeApiController.java
com/bharaterp/supplychain/agri/entity/MandiTradeTicket.java
com/bharaterp/supplychain/agri/repository/MandiTradeTicketRepository.java
com/bharaterp/supplychain/controller/SupplyChainGatewayController.java
com/bharaterp/supplychain/dropship/controller/DropshipOrchestratorController.java
com/bharaterp/supplychain/dropship/entity/DropshipJob.java
com/bharaterp/supplychain/dropship/repository/DropshipJobRepository.java
com/bharaterp/supplychain/dropship/service/DropshipOrchestrator.java
com/bharaterp/supplychain/entity/SupplyChainShipment.java
com/bharaterp/supplychain/gs1/barcode/Gs1BarcodeGeneratorController.java
com/bharaterp/supplychain/gs1/barcode/Gs1BarcodeGeneratorService.java
com/bharaterp/supplychain/gs1/barcode/entity/Gs1Barcode.java
com/bharaterp/supplychain/gs1/barcode/repository/Gs1BarcodeRepository.java
com/bharaterp/supplychain/iot/controller/PlantIotApiController.java
com/bharaterp/supplychain/iot/entity/PlantIotTelemetry.java
com/bharaterp/supplychain/iot/repository/PlantIotTelemetryRepository.java
com/bharaterp/supplychain/logistics/optimizer/ContainerLoadOptimizationService.java
com/bharaterp/supplychain/logistics/optimizer/controller/ContainerLoadOptimizationController.java
com/bharaterp/supplychain/logistics/optimizer/entity/ContainerLoadJob.java
com/bharaterp/supplychain/logistics/optimizer/repository/ContainerLoadJobRepository.java
com/bharaterp/supplychain/logistics/optimizer/service/ContainerLoadOptimizationService.java
com/bharaterp/supplychain/order/dom/controller/DistributedOrderRoutingController.java
com/bharaterp/supplychain/order/dom/entity/DistributedOrderRoute.java
com/bharaterp/supplychain/order/dom/repository/DistributedOrderRouteRepository.java
com/bharaterp/supplychain/order/dom/service/DistributedOrderRoutingService.java
com/bharaterp/supplychain/repository/SupplyChainShipmentRepository.java
com/bharaterp/supplychain/vmi/controller/VendorManagedInventoryController.java
com/bharaterp/supplychain/vmi/entity/VmiThresholdNode.java
com/bharaterp/supplychain/vmi/repository/VmiThresholdRepository.java
com/bharaterp/supplychain/vmi/service/VendorManagedInventoryEngine.java
com/bharaterp/supplychain/warehouse/picking/WavePickingOptimizationController.java
com/bharaterp/supplychain/warehouse/picking/WavePickingOptimizationService.java
com/bharaterp/supplychain/warehouse/picking/entity/WavePickingJob.java
com/bharaterp/supplychain/warehouse/picking/repository/WavePickingJobRepository.java
```

### Audit & Secretarial — `com.bharaterp.audit` (33 files)

```
com/bharaterp/audit/controller/AuditEngagementController.java
com/bharaterp/audit/controller/BoardMeetingController.java
com/bharaterp/audit/controller/SecretarialAuditReportController.java
com/bharaterp/audit/controller/UpgradedAuditController.java
com/bharaterp/audit/controller/roc/RocFilingController.java
com/bharaterp/audit/controller/secretarial/SecretarialComplianceController.java
com/bharaterp/audit/controller/xbrl/XbrlFilingController.java
com/bharaterp/audit/entity/AuditChecklistItem.java
com/bharaterp/audit/entity/AuditEngagement.java
com/bharaterp/audit/entity/CaroClauseResponse.java
com/bharaterp/audit/entity/roc/RocFiling.java
com/bharaterp/audit/entity/secretarial/BoardMeeting.java
com/bharaterp/audit/entity/secretarial/SecretarialAuditReport.java
com/bharaterp/audit/entity/xbrl/XbrlFiling.java
com/bharaterp/audit/repository/AuditChecklistItemRepository.java
com/bharaterp/audit/repository/AuditEngagementRepository.java
com/bharaterp/audit/repository/CaroClauseResponseRepository.java
com/bharaterp/audit/repository/roc/RocFilingRepository.java
com/bharaterp/audit/repository/secretarial/BoardMeetingRepository.java
com/bharaterp/audit/repository/secretarial/SecretarialAuditReportRepository.java
com/bharaterp/audit/repository/xbrl/XbrlFilingRepository.java
com/bharaterp/audit/service/AuditChecklistAutomationService.java
com/bharaterp/audit/service/AuditEngagementService.java
com/bharaterp/audit/service/AuditSodEnforcer.java
com/bharaterp/audit/service/CaroComplianceGuardrail.java
com/bharaterp/audit/service/CaroRuleEngine.java
com/bharaterp/audit/service/ContinuousEvidenceCollector.java
com/bharaterp/audit/service/XbrlReportGenerator.java
com/bharaterp/audit/service/roc/McaFilingExporter.java
com/bharaterp/audit/service/roc/RocFilingService.java
com/bharaterp/audit/service/secretarial/SecretarialComplianceService.java
com/bharaterp/audit/service/secretarial/StatutoryRegisterService.java
com/bharaterp/audit/service/xbrl/XbrlGeneratorService.java
```

### Inventory — `com.bharaterp.inventory` (33 files)

```
com/bharaterp/inventory/controller/InventoryApiController.java
com/bharaterp/inventory/controller/ProductController.java
com/bharaterp/inventory/controller/credit/CustomerCreditController.java
com/bharaterp/inventory/controller/po/PurchaseOrderController.java
com/bharaterp/inventory/controller/so/SalesOrderController.java
com/bharaterp/inventory/controller/warehouse/WarehouseController.java
com/bharaterp/inventory/entity/Product.java
com/bharaterp/inventory/entity/StockItem.java
com/bharaterp/inventory/entity/credit/CustomerCreditLimit.java
com/bharaterp/inventory/entity/po/PurchaseOrder.java
com/bharaterp/inventory/entity/po/PurchaseOrderItem.java
com/bharaterp/inventory/entity/so/SalesOrder.java
com/bharaterp/inventory/entity/so/SalesOrderItem.java
com/bharaterp/inventory/entity/warehouse/Warehouse.java
com/bharaterp/inventory/entity/warehouse/WarehouseStock.java
com/bharaterp/inventory/repository/ProductRepository.java
com/bharaterp/inventory/repository/StockItemRepository.java
com/bharaterp/inventory/repository/credit/CustomerCreditLimitRepository.java
com/bharaterp/inventory/repository/po/PurchaseOrderItemRepository.java
com/bharaterp/inventory/repository/po/PurchaseOrderRepository.java
com/bharaterp/inventory/repository/so/SalesOrderItemRepository.java
com/bharaterp/inventory/repository/so/SalesOrderRepository.java
com/bharaterp/inventory/repository/warehouse/WarehouseRepository.java
com/bharaterp/inventory/repository/warehouse/WarehouseStockRepository.java
com/bharaterp/inventory/service/ProductService.java
com/bharaterp/inventory/service/credit/CustomerCreditService.java
com/bharaterp/inventory/service/po/PurchaseOrderService.java
com/bharaterp/inventory/service/so/SalesOrderService.java
com/bharaterp/inventory/service/warehouse/WarehouseService.java
com/bharaterp/inventory/weight/controller/VariableWeightStockController.java
com/bharaterp/inventory/weight/entity/CatchWeightStock.java
com/bharaterp/inventory/weight/repository/CatchWeightStockRepository.java
com/bharaterp/inventory/weight/service/VariableWeightStockEngine.java
```

### Infrastructure — `com.bharaterp.infrastructure` (28 files)

```
com/bharaterp/infrastructure/dbgrid/service/ClusterConfigurationRegistry.java
com/bharaterp/infrastructure/dbgrid/service/ClusterHeartbeatMonitor.java
com/bharaterp/infrastructure/dbgrid/service/ClusterReplicationAuditor.java
com/bharaterp/infrastructure/dbgrid/service/DataConsistencyBroker.java
com/bharaterp/infrastructure/dbgrid/service/DataPurgingArchiver.java
com/bharaterp/infrastructure/dbgrid/service/DataSlicingReconciler.java
com/bharaterp/infrastructure/dbgrid/service/DeadlockPreventionEngine.java
com/bharaterp/infrastructure/dbgrid/service/DistributedPartitionManager.java
com/bharaterp/infrastructure/dbgrid/service/DistributedShardingRouter.java
com/bharaterp/infrastructure/dbgrid/service/DistributedSnapshotEngine.java
com/bharaterp/infrastructure/dbgrid/service/DynamicQueryOptimizer.java
com/bharaterp/infrastructure/dbgrid/service/FailoverClusterManager.java
com/bharaterp/infrastructure/dbgrid/service/QueryRoutingLoadBalancer.java
com/bharaterp/infrastructure/dbgrid/service/WriteAheadLogEngine.java
com/bharaterp/infrastructure/dbgrid/service/WriteBufferFlushEngine.java
com/bharaterp/infrastructure/governance/service/AiImplementationTrainingService.java
com/bharaterp/infrastructure/iotedge/service/DeviceFirmwareValidator.java
com/bharaterp/infrastructure/iotedge/service/EdgeDataBufferCache.java
com/bharaterp/infrastructure/iotedge/service/EdgeTelemetryProcessor.java
com/bharaterp/infrastructure/iotedge/service/HardwareDeviceAuthenticator.java
com/bharaterp/infrastructure/iotedge/service/HardwareEdgeReconciler.java
com/bharaterp/infrastructure/iotedge/service/IndustryPlclinkingService.java
com/bharaterp/infrastructure/iotedge/service/IngressDataSpikeMitigator.java
com/bharaterp/infrastructure/iotedge/service/IoTHeartbeatMonitor.java
com/bharaterp/infrastructure/iotedge/service/MqttBrokerConnector.java
com/bharaterp/infrastructure/iotedge/service/OpcUaProtocolBridge.java
com/bharaterp/infrastructure/iotedge/service/RfidBatchIngestionEngine.java
com/bharaterp/infrastructure/iotedge/service/TelemetryAnomalyFilter.java
```

### Security — `com.bharaterp.security` (26 files)

```
com/bharaterp/security/audit/controller/ActivityLogController.java
com/bharaterp/security/audit/entity/ActivityLog.java
com/bharaterp/security/audit/repository/ActivityLogRepository.java
com/bharaterp/security/audit/service/ActivityLogService.java
com/bharaterp/security/audit/service/AuditLogAspect.java
com/bharaterp/security/config/SecurityIntegrityGuard.java
com/bharaterp/security/mesh/AIThreatDetection.java
com/bharaterp/security/mesh/ZeroTrustEnforcer.java
com/bharaterp/security/rbac/controller/RbacController.java
com/bharaterp/security/rbac/entity/Permission.java
com/bharaterp/security/rbac/entity/RolePermission.java
com/bharaterp/security/rbac/repository/PermissionRepository.java
com/bharaterp/security/rbac/repository/RolePermissionRepository.java
com/bharaterp/security/rbac/service/RbacService.java
com/bharaterp/security/shield/controller/ApiTokenBucketShieldController.java
com/bharaterp/security/shield/entity/ApiShieldPolicy.java
com/bharaterp/security/shield/repository/ApiShieldPolicyRepository.java
com/bharaterp/security/shield/service/ApiTokenBucketShield.java
com/bharaterp/security/sso/controller/SharedSsoCoreController.java
com/bharaterp/security/sso/entity/SsoSessionToken.java
com/bharaterp/security/sso/repository/SsoSessionTokenRepository.java
com/bharaterp/security/sso/service/SharedSsoCoreService.java
com/bharaterp/security/zkp/audit/ZkpAuditAttestationController.java
com/bharaterp/security/zkp/audit/ZkpAuditAttestationService.java
com/bharaterp/security/zkp/audit/entity/ZkpAuditAttestation.java
com/bharaterp/security/zkp/audit/repository/ZkpAuditAttestationRepository.java
```

### Backend Core / Config — `com.bharaterp.backend` (25 files)

```
com/bharaterp/backend/BackendApplication.java
com/bharaterp/backend/config/GlobalizationService.java
com/bharaterp/backend/config/JwtAuthenticationFilter.java
com/bharaterp/backend/config/JwtTokenProvider.java
com/bharaterp/backend/config/KafkaConfig.java
com/bharaterp/backend/config/OpenApiMetadataConfig.java
com/bharaterp/backend/config/RabbitMqConfig.java
com/bharaterp/backend/config/RedisConfig.java
com/bharaterp/backend/config/SecurityConfig.java
com/bharaterp/backend/config/SwaggerConfig.java
com/bharaterp/backend/controller/AiCopilotController.java
com/bharaterp/backend/controller/AuthController.java
com/bharaterp/backend/controller/IoTEventController.java
com/bharaterp/backend/entity/User.java
com/bharaterp/backend/logistics/CustomsClearanceService.java
com/bharaterp/backend/logistics/FreightBillingService.java
com/bharaterp/backend/logistics/RouteOptimizationService.java
com/bharaterp/backend/logistics/VahanIntegrationService.java
com/bharaterp/backend/repository/UserRepository.java
com/bharaterp/backend/security/AiCopilotService.java
com/bharaterp/backend/security/DataMaskingService.java
com/bharaterp/backend/security/IoTEventService.java
com/bharaterp/backend/security/ZeroTrustMaskingService.java
com/bharaterp/backend/service/AuthService.java
com/bharaterp/backend/service/UserDetailsServiceImpl.java
```

### Core Platform Utilities — `com.bharaterp.core` (22 files)

```
com/bharaterp/core/audit/AuditTrailInterceptor.java
com/bharaterp/core/audit/controller/SecurityAuditController.java
com/bharaterp/core/audit/entity/SecurityAuditBlock.java
com/bharaterp/core/audit/repository/SecurityAuditBlockRepository.java
com/bharaterp/core/cache/HybridCacheSyncController.java
com/bharaterp/core/cache/HybridCacheSyncService.java
com/bharaterp/core/cache/entity/CacheSyncQueue.java
com/bharaterp/core/cache/repository/CacheSyncQueueRepository.java
com/bharaterp/core/flags/FeatureFlagManagementController.java
com/bharaterp/core/flags/FeatureFlagManagementService.java
com/bharaterp/core/flags/entity/FeatureFlagRegistry.java
com/bharaterp/core/flags/repository/FeatureFlagRegistryRepository.java
com/bharaterp/core/org/tree/OrgRoleHierarchyController.java
com/bharaterp/core/org/tree/OrgRoleHierarchyService.java
com/bharaterp/core/org/tree/entity/OrgRoleNode.java
com/bharaterp/core/org/tree/repository/OrgRoleNodeRepository.java
com/bharaterp/core/tx/coordinator/DistributedTxCoordinatorController.java
com/bharaterp/core/tx/coordinator/DistributedTxCoordinatorEngine.java
com/bharaterp/core/tx/coordinator/entity/DistributedTransactionRegistry.java
com/bharaterp/core/tx/coordinator/entity/TransactionParticipantNode.java
com/bharaterp/core/tx/coordinator/repository/DistributedTransactionRegistryRepository.java
com/bharaterp/core/tx/coordinator/repository/TransactionParticipantNodeRepository.java
```

### GST & Tax — `com.bharaterp.gst` (19 files)

```
com/bharaterp/gst/EWayBill.java
com/bharaterp/gst/EWayBillController.java
com/bharaterp/gst/EWayBillRepository.java
com/bharaterp/gst/controller/GstInvoiceController.java
com/bharaterp/gst/controller/GstrFilingController.java
com/bharaterp/gst/controller/HsnMappingController.java
com/bharaterp/gst/entity/GstInvoice.java
com/bharaterp/gst/entity/GstInvoiceItem.java
com/bharaterp/gst/entity/GstrFilingRecord.java
com/bharaterp/gst/entity/HsnMaster.java
com/bharaterp/gst/repository/GstInvoiceItemRepository.java
com/bharaterp/gst/repository/GstInvoiceRepository.java
com/bharaterp/gst/repository/GstrFilingRecordRepository.java
com/bharaterp/gst/repository/HsnMasterRepository.java
com/bharaterp/gst/service/EWayBillService.java
com/bharaterp/gst/service/GstCalculationService.java
com/bharaterp/gst/service/GstInvoiceService.java
com/bharaterp/gst/service/GstrFilingService.java
com/bharaterp/gst/service/HsnMappingService.java
```

### Sales — `com.bharaterp.sales` (19 files)

```
com/bharaterp/sales/controller/SalesInvoiceApiController.java
com/bharaterp/sales/cpq/controller/CpqEngineController.java
com/bharaterp/sales/cpq/entity/CpqQuote.java
com/bharaterp/sales/cpq/entity/PricingTier.java
com/bharaterp/sales/cpq/repository/CpqQuoteRepository.java
com/bharaterp/sales/cpq/repository/PricingTierRepository.java
com/bharaterp/sales/cpq/service/CpqEngineService.java
com/bharaterp/sales/entity/SalesInvoice.java
com/bharaterp/sales/rebate/ledger/controller/PartnerRebateLedgerController.java
com/bharaterp/sales/rebate/ledger/entity/PartnerRebateLedger.java
com/bharaterp/sales/rebate/ledger/entity/RebateSlab.java
com/bharaterp/sales/rebate/ledger/repository/PartnerRebateLedgerRepository.java
com/bharaterp/sales/rebate/ledger/repository/RebateSlabRepository.java
com/bharaterp/sales/rebate/ledger/service/PartnerRebateLedgerService.java
com/bharaterp/sales/repository/SalesInvoiceRepository.java
com/bharaterp/sales/tiered/pricing/controller/TieredVolumePricingController.java
com/bharaterp/sales/tiered/pricing/entity/TieredVolumePricing.java
com/bharaterp/sales/tiered/pricing/repository/TieredVolumePricingRepository.java
com/bharaterp/sales/tiered/pricing/service/TieredVolumePricingEngineService.java
```

### Autonomous Agents — `com.bharaterp.autonomous` (18 files)

```
com/bharaterp/autonomous/agent/service/CollectionsAgent.java
com/bharaterp/autonomous/agent/service/InventoryTaskingAgent.java
com/bharaterp/autonomous/agent/service/SupplyPlanningAgent.java
com/bharaterp/autonomous/epm/service/AutoReforecastService.java
com/bharaterp/autonomous/epm/service/StrategicRiskOptimizer.java
com/bharaterp/autonomous/factory/controller/FactoryController.java
com/bharaterp/autonomous/factory/service/BusinessResourceOptimizer.java
com/bharaterp/autonomous/factory/service/DigitalFactoryOrchestrator.java
com/bharaterp/autonomous/factory/service/FactoryProcessIntelligenceService.java
com/bharaterp/autonomous/finance/service/AutoJournalEntryService.java
com/bharaterp/autonomous/finance/service/IntelligentCloseManager.java
com/bharaterp/autonomous/finance/service/VarianceAnalysisAgent.java
com/bharaterp/autonomous/industry/AutonomousLogisticsAgent.java
com/bharaterp/autonomous/industry/AutonomousRetailAgent.java
com/bharaterp/autonomous/industry/IndustryAutonomyController.java
com/bharaterp/autonomous/industry/ManufacturingOeeCalculator.java
com/bharaterp/autonomous/industry/PharmaSupplyChainAgent.java
com/bharaterp/autonomous/industry/SmartCityOrchestrator.java
```

### AI / Agent Orchestration — `com.bharaterp.ai` (17 files)

```
com/bharaterp/ai/agent/controller/AgentHubController.java
com/bharaterp/ai/agent/controller/AiHubExtensionController.java
com/bharaterp/ai/agent/entity/AgentRegistry.java
com/bharaterp/ai/agent/repository/AgentRegistryRepository.java
com/bharaterp/ai/agent/service/AgentConsensusBalancer.java
com/bharaterp/ai/agent/service/AgentGovernanceService.java
com/bharaterp/ai/agent/service/GoogleSearchIntegration.java
com/bharaterp/ai/mcp/entity/McpToolRegistry.java
com/bharaterp/ai/mcp/repository/McpToolRegistryRepository.java
com/bharaterp/ai/mcp/service/McpServerService.java
com/bharaterp/ai/orchestrator/SpringAiClaudeRouter.java
com/bharaterp/ai/orchestrator/entity/OrchestrationWorkflow.java
com/bharaterp/ai/orchestrator/repository/OrchestrationWorkflowRepository.java
com/bharaterp/ai/orchestrator/service/MultiAgentOrchestratorService.java
com/bharaterp/ai/rag/entity/KnowledgeDocument.java
com/bharaterp/ai/rag/repository/KnowledgeDocumentRepository.java
com/bharaterp/ai/rag/service/RagPipelineService.java
```

### Platform Services — `com.bharaterp.platform` (17 files)

```
com/bharaterp/platform/archive/controller/SystemDataArchiverController.java
com/bharaterp/platform/archive/entity/SystemArchiveRegistry.java
com/bharaterp/platform/archive/repository/SystemArchiveRegistryRepository.java
com/bharaterp/platform/archive/service/SystemDataArchiverVolumeManager.java
com/bharaterp/platform/bi/service/ProcessIntelligenceService.java
com/bharaterp/platform/purging/controller/DataPurgingStorageController.java
com/bharaterp/platform/purging/entity/DataPurgeLog.java
com/bharaterp/platform/purging/repository/DataPurgeLogRepository.java
com/bharaterp/platform/purging/service/DataPurgingStorageService.java
com/bharaterp/platform/schema/controller/DynamicSchemaExtensionController.java
com/bharaterp/platform/schema/entity/CustomFieldMeta.java
com/bharaterp/platform/schema/repository/CustomFieldMetaRepository.java
com/bharaterp/platform/schema/service/DynamicSchemaExtensionEngine.java
com/bharaterp/platform/tenant/controller/MultiTenantSchemaController.java
com/bharaterp/platform/tenant/entity/TenantRoutingRegistry.java
com/bharaterp/platform/tenant/repository/TenantRoutingRegistryRepository.java
com/bharaterp/platform/tenant/service/MultiTenantSchemaRouter.java
```

### Integration & Connectors — `com.bharaterp.integration` (14 files)

```
com/bharaterp/integration/blockchain/entity/BlockchainBlock.java
com/bharaterp/integration/blockchain/repository/BlockchainBlockRepository.java
com/bharaterp/integration/blockchain/service/BlockchainAuditService.java
com/bharaterp/integration/connector/controller/ConnectorController.java
com/bharaterp/integration/connector/controller/UniversalImportController.java
com/bharaterp/integration/connector/entity/ConnectorConfig.java
com/bharaterp/integration/connector/repository/ConnectorConfigRepository.java
com/bharaterp/integration/connector/service/GenericConnectorService.java
com/bharaterp/integration/connector/service/TallyPacketParser.java
com/bharaterp/integration/connector/service/UniversalDataImporterService.java
com/bharaterp/integration/webhook/controller/WebhookController.java
com/bharaterp/integration/webhook/entity/WebhookSubscription.java
com/bharaterp/integration/webhook/repository/WebhookSubscriptionRepository.java
com/bharaterp/integration/webhook/service/WebhookDispatchService.java
```

### ESG & Sustainability — `com.bharaterp.esg` (13 files)

```
com/bharaterp/esg/carbon/controller/CarbonEsgController.java
com/bharaterp/esg/carbon/service/CarbonFootprintTracker.java
com/bharaterp/esg/carbon/service/CarbonGreenLedgerService.java
com/bharaterp/esg/carbon/service/CarbonTaxOptimizer.java
com/bharaterp/esg/carbon/service/ESGReportingService.java
com/bharaterp/esg/universal/CSRDComplianceService.java
com/bharaterp/esg/universal/GlobalESGEngine.java
com/bharaterp/esg/universal/SAPGreenLedgerAdapter.java
com/bharaterp/esg/universal/UniversalCarbonTaxOptimizer.java
com/bharaterp/esg/universal/UniversalEsgController.java
com/bharaterp/esg/vision/green/EInvoicePeppolService.java
com/bharaterp/esg/vision/green/EsgVisionGreenLedgerService.java
com/bharaterp/esg/vision/green/controller/EsgComplianceController.java
```

### Agentic Mesh — `com.bharaterp.agentic` (11 files)

```
com/bharaterp/agentic/mesh/AIMeshCoordinator.java
com/bharaterp/agentic/mesh/AgenticOrchestrator.java
com/bharaterp/agentic/mesh/HeadlessApiGateway.java
com/bharaterp/agentic/mesh/ProcessIntelligenceService.java
com/bharaterp/agentic/mesh/entity/MeshSessionRegistry.java
com/bharaterp/agentic/mesh/repository/MeshSessionRegistryRepository.java
com/bharaterp/agentic/orchestrator/controller/AgenticController.java
com/bharaterp/agentic/orchestrator/entity/JouleWorkWorkspace.java
com/bharaterp/agentic/orchestrator/service/AgentOrchestrationEngine.java
com/bharaterp/agentic/orchestrator/service/KnowledgeGraphService.java
com/bharaterp/agentic/orchestrator/service/RAPTService.java
```

### Logistics — `com.bharaterp.logistics` (10 files)

```
com/bharaterp/logistics/multimodal/controller/LogisticsController.java
com/bharaterp/logistics/multimodal/entity/ConsignmentShipment.java
com/bharaterp/logistics/multimodal/entity/CustomsShipment.java
com/bharaterp/logistics/multimodal/entity/FreightConsignment.java
com/bharaterp/logistics/multimodal/entity/Vehicle.java
com/bharaterp/logistics/multimodal/repository/CustomsShipmentRepository.java
com/bharaterp/logistics/multimodal/repository/FreightConsignmentRepository.java
com/bharaterp/logistics/multimodal/repository/VehicleRepository.java
com/bharaterp/logistics/multimodal/service/GlobalCustomsClearanceService.java
com/bharaterp/logistics/multimodal/service/NationalLogisticsVahanService.java
```

### Quantum (R&D / Experimental) — `com.bharaterp.quantum` (9 files)

```
com/bharaterp/quantum/core/controller/QuantumController.java
com/bharaterp/quantum/core/crypto/QuantumSafeCryptoEngine.java
com/bharaterp/quantum/core/entity/QuantumComputationLog.java
com/bharaterp/quantum/core/optimization/SimulatedAnnealingOptimizer.java
com/bharaterp/quantum/core/qkd/BB84KeyDistributionService.java
com/bharaterp/quantum/core/repository/QuantumComputationLogRepository.java
com/bharaterp/quantum/core/service/PostQuantumCryptoService.java
com/bharaterp/quantum/core/service/QuantumKeyDistribution.java
com/bharaterp/quantum/core/service/QuantumOptimizationEngine.java
```

### Manufacturing — `com.bharaterp.manufacturing` (8 files)

```
com/bharaterp/manufacturing/shopfloor/controller/ShopFloorSequencingController.java
com/bharaterp/manufacturing/shopfloor/entity/ShopFloorJob.java
com/bharaterp/manufacturing/shopfloor/repository/ShopFloorJobRepository.java
com/bharaterp/manufacturing/shopfloor/service/ShopFloorSequencingEngine.java
com/bharaterp/manufacturing/tooling/ToolingCalibrationMonitorController.java
com/bharaterp/manufacturing/tooling/ToolingCalibrationMonitorService.java
com/bharaterp/manufacturing/tooling/entity/ToolingCalibrationLog.java
com/bharaterp/manufacturing/tooling/repository/ToolingCalibrationLogRepository.java
```

### Marketplace — `com.bharaterp.marketplace` (7 files)

```
com/bharaterp/marketplace/agent/AgentMeteringBillingService.java
com/bharaterp/marketplace/agent/AgentSmartContract.java
com/bharaterp/marketplace/agent/AgentVendorRegistry.java
com/bharaterp/marketplace/controller/MarketplaceController.java
com/bharaterp/marketplace/entity/PartnerAgent.java
com/bharaterp/marketplace/service/AgentBillingService.java
com/bharaterp/marketplace/service/MarketplaceListingService.java
```

### Blockchain — `com.bharaterp.blockchain` (6 files)

```
com/bharaterp/blockchain/scm/service/BlockchainLedgerService.java
com/bharaterp/blockchain/scm/service/IoTBlockchainBridge.java
com/bharaterp/blockchain/scm/service/PharmaSupplyChainService.java
com/bharaterp/blockchain/scm/service/SmartContractEngine.java
com/bharaterp/blockchain/vision/scm/service/AntiCounterfeitPharmaEngine.java
com/bharaterp/blockchain/vision/scm/service/RealTimeTraceabilityBridge.java
```

### Accounting (Ledger Core) — `com.bharaterp.account` (5 files)

```
com/bharaterp/account/controller/VoucherController.java
com/bharaterp/account/entity/Voucher.java
com/bharaterp/account/entity/VoucherLine.java
com/bharaterp/account/repository/VoucherRepository.java
com/bharaterp/account/service/VoucherService.java
```

### Composable Architecture — `com.bharaterp.composable` (5 files)

```
com/bharaterp/composable/entity/ExtensionRegistry.java
com/bharaterp/composable/service/ApiGatewayService.java
com/bharaterp/composable/service/CleanCoreValidator.java
com/bharaterp/composable/service/DynamicPluginExtensionEngine.java
com/bharaterp/composable/service/EventMeshService.java
```

### Procurement — `com.bharaterp.procurement` (5 files)

```
com/bharaterp/procurement/controller/PurchaseOrderApiController.java
com/bharaterp/procurement/entity/PurchaseOrder.java
com/bharaterp/procurement/repository/PurchaseOrderRepository.java
com/bharaterp/procurement/vms/service/VendorAssessmentService.java
com/bharaterp/procurement/vms/service/VendorScorecardService.java
```

### Banking (UPI) — `com.bharaterp.banking` (4 files)

```
com/bharaterp/banking/controller/UpiPaymentController.java
com/bharaterp/banking/entity/UpiTransaction.java
com/bharaterp/banking/repository/UpiTransactionRepository.java
com/bharaterp/banking/service/UpiPaymentService.java
```

### Financial AI (CFO Agent) — `com.bharaterp.financial` (4 files)

```
com/bharaterp/financial/controller/ai/CfoAgentController.java
com/bharaterp/financial/entity/ai/CfoAgentProfile.java
com/bharaterp/financial/repository/ai/CfoAgentProfileRepository.java
com/bharaterp/financial/service/ai/CfoAgentService.java
```

### Cognitive Digital Twin — `com.bharaterp.cognitive` (3 files)

```
com/bharaterp/cognitive/twin/ContinuousPlanningService.java
com/bharaterp/cognitive/twin/EnterpriseDigitalTwin.java
com/bharaterp/cognitive/twin/ScenarioSimulationEngine.java
```

### Decision Intelligence — `com.bharaterp.decision` (3 files)

```
com/bharaterp/decision/vision/intelligence/service/ContinuousForecastingEngine.java
com/bharaterp/decision/vision/intelligence/service/InvisibleCognitivePlatform.java
com/bharaterp/decision/vision/intelligence/service/WhatIfSimulationModel.java
```

### Migration (Strangler Pattern) — `com.bharaterp.migration` (3 files)

```
com/bharaterp/migration/strangler/controller/MigrationController.java
com/bharaterp/migration/strangler/service/MicroservicesGateway.java
com/bharaterp/migration/strangler/service/MonolithicDecoupler.java
```

### Sovereign Cloud / GDPR — `com.bharaterp.sovereign` (3 files)

```
com/bharaterp/sovereign/cloud/service/DataResidencyService.java
com/bharaterp/sovereign/cloud/service/GDPRComplianceService.java
com/bharaterp/sovereign/cloud/service/LocalizationEngine.java
```

### Universal Billing — `com.bharaterp.universal` (3 files)

```
com/bharaterp/universal/controller/UniversalBillingApiController.java
com/bharaterp/universal/entity/UniversalInvoice.java
com/bharaterp/universal/repository/UniversalInvoiceRepository.java
```

### Deployment Tools — `com.bharaterp.deployment` (2 files)

```
com/bharaterp/deployment/service/GlobalEndpointSmokeTester.java
com/bharaterp/deployment/service/MicroserviceDecouplerRegistry.java
```

### Global / IPO Compliance — `com.bharaterp.global` (2 files)

```
com/bharaterp/global/service/InvestorReportingService.java
com/bharaterp/global/service/IpoComplianceService.java
```

### Government API Integration — `com.bharaterp.govt` (2 files)

```
com/bharaterp/govt/controller/GovtApiController.java
com/bharaterp/govt/service/GovtApiService.java
```

### Reporting & Dashboards — `com.bharaterp.reporting` (2 files)

```
com/bharaterp/reporting/controller/DashboardController.java
com/bharaterp/reporting/service/DashboardService.java
```

### Spatial / AR-VR — `com.bharaterp.spatial` (2 files)

```
com/bharaterp/spatial/entity/SpatialWorkspace.java
com/bharaterp/spatial/service/ArVrIntegrationService.java
```

### Digital Workforce — `com.bharaterp.workforce` (2 files)

```
com/bharaterp/workforce/digital/service/LearningExperienceManager.java
com/bharaterp/workforce/digital/service/SkillsGapAnalyzer.java
```

### Decision Intelligence (Scenario Modeling) — `com.bharaterp.decisionintelligence` (1 files)

```
com/bharaterp/decisionintelligence/service/ScenarioModelingService.java
```

### Cognitive Mesh (Overlord) — `com.bharaterp.overlord` (1 files)

```
com/bharaterp/overlord/service/OmnipresentCognitiveMesh.java
```

### Secretarial Compliance — `com.bharaterp.secretarial` (1 files)

```
com/bharaterp/secretarial/compliance/BoardMeetingService.java
```

### XBRL Filing — `com.bharaterp.xbrl` (1 files)

```
com/bharaterp/xbrl/filing/service/XbrlTaxonomyMappingService.java
```

---

## Frontend Structure

The frontend is a React 18 single-page application, built with Vite and styled using Tailwind CSS.

### Known Structure

```
frontend/
├── index.html
├── package.json
├── vite.config.js
├── tailwind.config.js
├── postcss.config.js
└── src/
    ├── main.jsx              # Application entry point
    ├── App.jsx                # Root component — navigation, routing between modules
    └── components/
        ├── ledger/            # Ledger module UI (voucher entry, journal matrix)
        ├── crm/                # CRM pipeline UI
        ├── payroll/            # Payroll disbursal console UI
        ├── audit/              # Audit & forensic scanner UI
        └── cockpits/           # Advanced Cockpits UI components
```

### Complete Frontend File Index

> 📝 **Pending:** the full recursive frontend file list has not yet been provided. Once available, it will be embedded here in the same domain-grouped format as the backend index above.
>
> To generate it, run this from the project root:
> ```powershell
> cd C:\bharat_erp_solution\frontend
> Get-ChildItem -Path .\src -Recurse -Include *.jsx,*.js,*.css |
>   ForEach-Object { $_.FullName.Substring((Resolve-Path .\src).Path.Length + 1) } |
>   Sort-Object | Out-File -Encoding utf8 all_frontend_files.txt
> Get-Content .\all_frontend_files.txt
> ```
> Paste the output back and it will be merged into this section.

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Backend Runtime | Java (LTS) | 21.0.11 |
| Backend Framework | Spring Boot (Maven) | — |
| Security | Spring Security + JWT | — |
| ORM / Persistence | Spring Data JPA | — |
| Caching | Redis | via Spring Data Redis |
| Messaging | Kafka, RabbitMQ | via `KafkaConfig`, `RabbitMqConfig` |
| Document Generation | iText | 9.6.0 (`itext-core`) |
| API Documentation | Swagger / OpenAPI | via `SwaggerConfig`, `OpenApiMetadataConfig` |
| Frontend Runtime | Node.js | v22.16.0 |
| Frontend Framework | React | 18.3.1 |
| Build Tool (frontend) | Vite | 5.4.1 |
| Styling | Tailwind CSS | 3.4.1 |
| Icons | lucide-react | 0.344.0 |
| Build Tool (backend) | Maven Wrapper (`mvnw`) | — |
| Internal Audit Tooling | Custom PowerShell static-analysis framework | v2.0 |

---

## Getting Started

### Prerequisites
- Java 21 (LTS)
- Node.js 18+ (tested on v22.16.0)
- MySQL (or compatible relational database)
- Redis

### Clone

```bash
git clone <your-repo-url>
cd bharat_erp_solution
```

### Backend Setup

```bash
cd backend
./mvnw clean compile
./mvnw spring-boot:run
```

Last verified build: **BUILD SUCCESS**, compiling 833 source files in ~22 seconds (Java 21, `javac` with debug parameters, release 21).

### Frontend Setup

```bash
cd frontend
npm install
npm run dev       # local development server
npm run build     # production build (outputs to /dist)
npm run preview   # preview the production build locally (localhost:4173)
```

### Full Local Stack

1. Start MySQL and Redis locally (or via Docker).
2. Start the backend (`./mvnw spring-boot:run`).
3. Start the frontend (`npm run dev`).
4. Open the app in your browser and log in via the portal.

---

## Configuration

Backend configuration lives in `application.properties` / environment variables:

```properties
# Database
spring.datasource.url=jdbc:mysql://localhost:3306/bharat_erp_core_ledger
spring.datasource.username=${DB_USERNAME:root}
spring.datasource.password=${DB_PASSWORD}

# JWT
app.jwt.secret=${JWT_SECRET}
app.jwt.expiration-ms=86400000

# Redis
spring.redis.host=${REDIS_HOST:localhost}
spring.redis.port=${REDIS_PORT:6379}

# Messaging (optional, if Kafka/RabbitMQ features are enabled)
spring.kafka.bootstrap-servers=${KAFKA_BROKERS:localhost:9092}
spring.rabbitmq.host=${RABBITMQ_HOST:localhost}
```

Always provide `DB_PASSWORD` and `JWT_SECRET` via environment variables in any shared or production environment — never hardcode credentials in source control.

---

## API Documentation

### Conventions
- Base path: `/api/v1/{domain}` (e.g. `/api/v1/crm/leads`, `/api/v1/autonomous-treasury`)
- Authentication: `Authorization: Bearer <JWT>` header on all protected endpoints
- Response format: JSON, with domain-specific payload DTOs
- Error handling: centralized via `@ControllerAdvice` global exception interceptors, returning structured error responses with appropriate HTTP status codes
- Interactive API docs: available via Swagger/OpenAPI (see `SwaggerConfig`, `OpenApiMetadataConfig`) — typically at `/swagger-ui.html` once the backend is running

### Representative Endpoints by Domain

| Domain | Example Endpoint | Controller |
|---|---|---|
| Ledger | `POST /api/v1/ledger/vouchers` | `VoucherController` |
| CRM — Leads | `GET/POST /api/v1/crm/leads` | `LeadController` |
| CRM — Opportunities | `GET/POST /api/v1/crm/opportunities` | `OpportunityController` |
| Payroll | `POST /api/v1/hr/payroll/run` | `PayrollController` |
| HR — Employees | `GET/POST /api/v1/hr/employees` | `EmployeeController` |
| Audit — Engagements | `GET/POST /api/v1/audit/engagements` | `AuditEngagementController` |
| Audit — ROC Filings | `POST /api/v1/audit/roc/filings` | `RocFilingController` |
| GST — Invoices | `POST /api/v1/gst/invoices` | `GstInvoiceController` |
| Inventory — Products | `GET/POST /api/v1/inventory/products` | `ProductController` |
| Inventory — Warehouse | `GET /api/v1/inventory/warehouse` | `WarehouseController` |
| Finance — Accounts Payable | `GET/POST /api/v1/finance/ap` | `AccountsPayableController` |
| Finance — Budget | `GET/POST /api/v1/finance/budget` | `BudgetController` |
| Security — RBAC | `GET/POST /api/v1/security/rbac` | `RbacController` |
| Auth | `POST /api/v1/auth/login` | `AuthController` |

---

## Architecture Governance & ADR Process

The core architecture is version-locked following the current blueprint revision. Any structural change — new domain package, new cockpit, cross-cutting infrastructure change — must go through a formal **Architecture Decision Record (ADR)** before merging.

### What Requires an ADR
- Adding a new top-level domain package under `com.bharaterp`
- Changing the authentication/authorization model
- Introducing a new persistence technology or messaging system
- Any change affecting more than one domain package's public service contracts

### What Doesn't Require an ADR
- Bug fixes within an existing service/controller
- Adding a new field to an existing entity (with a corresponding migration)
- New endpoints within an already-established domain

### ADR Format

```markdown
# ADR-XXXX: <Title>

## Status
Proposed | Accepted | Superseded

## Context
What problem are we solving? What constraints apply?

## Decision
What are we doing about it?

## Consequences
What becomes easier or harder as a result?
```

### Development Workflow
- Backend changes follow a step-by-step, file-level workflow with a clean compile verification (`./mvnw clean compile`) after each change — never batch multiple unverified changes together.
- Always read the current file content before editing; don't rely on assumptions about what a file contains.
- Frontend changes are validated against the existing cockpit navigation and dashboard state patterns before merging.
- Naming conventions follow domain-first packaging (`com.bharaterp.<domain>.<subdomain>.<layer>`) to keep controllers, services, entities, and repositories consistently discoverable.

---

## Contributing Guide

### Branching Strategy
- `main` — production-ready code only
- `feature/<domain>-<short-description>` — new feature work (e.g. `feature/hr-gratuity-calc`)
- `fix/<short-description>` — bug fixes
- `audit/<short-description>` — audit/refactor work arising from code quality reviews

### Commit Message Convention

```
<type>(<domain>): <short summary>

<optional longer description>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `audit`

Example:
```
feat(hr): add gratuity slab calculation for FY2026-27

Implements updated gratuity ceiling per latest Payment of Gratuity
(Amendment) rules. Adds unit tests for edge cases at 5-year boundary.
```

### Pull Request Checklist
- [ ] `./mvnw clean compile` passes with no errors (backend changes)
- [ ] `npm run build` passes with no errors (frontend changes)
- [ ] New services are wired to a real repository — no placeholder/stub logic
- [ ] No hardcoded credentials or secrets introduced
- [ ] Relevant section of this README / file index updated if the change adds/removes/moves files
- [ ] For structural changes, an ADR has been added under `docs/adr/`

### Code Style
- Java: standard Spring Boot conventions — `Controller` → `Service` → `Repository` layering, DTOs for request/response payloads, `@ControllerAdvice` for exception handling
- React: functional components with hooks, Tailwind utility classes for styling, `lucide-react` for icons

---

## Changelog

### Backend Build Milestones

| Date | Java Files | Notes |
|---|---|---|
| Earlier baseline | 525 | First full scan by the internal Enterprise Validator tool (v2.0) |
| Jul 9, 2026 | 806 | 99% health score on keyword-based scan; 1 stub file, 1 empty-method file identified |
| Jul 10, 2026 | 811 | Incremental growth |
| Jul 15, 2026 | 833 | Latest clean Maven build — **BUILD SUCCESS**, verified via full recursive file scan |

### Feature Milestones (from project blueprint history)

- **Phase 2 (June 2026):** HR & Payroll module built out — Employee master, Attendance/Leave, Payroll (PF/ESI/PT/TDS), Full & Final Settlement, Gratuity, TDS New Tax Regime FY2025-26, Bank Disbursement CSV, Shift scheduling, Expense claims, Performance Goals, Appraisal cycles.
- **Phase 5 (June 2026):** CA/CS Audit Suite added.
- **Blueprint v2.0 → v3.0:** Feature count grew from 248 to 310+ across 20 modules, adding BFSI & Treasury, Sustainability & ESG, and Platform & Developer Ecosystem modules.
- **Blueprint v8.0:** Governance-locked feature freeze — all further structural changes now require a formal ADR.
- **July 2026:** Multi-pass PowerShell audit campaign (Steps 1–6b + Master Audit) across the full codebase — duplicate class-name resolution, hollow-implementation fixes, Spring bean-name collision fixes, and restoration of real AES-256-GCM encryption + genuine persistence in `WhistleblowerVaultService`.

---

## Roadmap

- Expand the Advanced Cockpits suite with additional industry-specific dashboards
- Deepen automated test coverage across Ledger, Payroll, and Audit modules
- CI/CD pipeline integration for automated build & deployment
- Complete the frontend file index (pending full file list)
- Public API documentation portal for third-party integrations
- Multi-tenant support for enterprise group companies (building on the existing `platform.tenant` package)
- Formal ADR archive under `docs/adr/` for all structural decisions going forward

---

## License

Proprietary — All rights reserved unless otherwise licensed by the project owner.

---

## Author

**Kartik Choudhary**
B.Tech CSE, Raj Kumar Goel Institute of Technology and Management, Ghaziabad (AKTU)
📧 [kartikmzn7@gmail.com](mailto:kartikmzn7@gmail.com)
🔗 [GitHub](https://github.com/kartikchoudharyaktu) · [LinkedIn](https://linkedin.com/in/kartik-choudharyaktu)
