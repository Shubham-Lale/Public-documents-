<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Logic App Consumption Deployment Technical Playbook and Signoff Document

## Project Information

**Project Name:** Invoice Processing Automation System
**Logic App Name:** LA-Invoice-Processing-Consumption
**Version/Release:** v1.2.3
**Deployment Environment:** Production
**Deployment Date:** July 20, 2025
**Project Manager:** Anita Sharma
**Technical Lead:** Rohit Gupta
**Prepared By:** Priya Verma
**Date Prepared:** July 18, 2025

## Executive Summary

This comprehensive document combines technical playbook documentation with deployment signoff requirements for the Logic App Consumption deployment. It serves as both a technical reference guide and formal approval mechanism for production deployment. The document encompasses architecture specifications, deployment procedures, testing validation, monitoring guidelines, and operational management protocols to ensure successful Logic App Consumption deployment and ongoing maintenance. The Consumption model provides a cost-effective, pay-per-execution solution ideal for variable workloads with automatic scaling capabilities[^1][^2].

## System Architecture and Design

### Logic App Details

- **Logic App Type:** Consumption (Multitenant)
- **Resource Group:** RG-Invoice-Processing-Prod
- **Azure Region:** East US
- **Hosting Environment:** Multitenant Azure Logic Apps
- **Integration Account:** IA-Invoice-Processing-Basic
- **Workflow Count:** 1 (single workflow per Logic App)
- **Runtime Configuration:** Shared multitenant infrastructure with automatic scaling[^1][^2]


### Key Components and Architecture

The Logic App Consumption architecture follows a serverless, event-driven design pattern optimized for cost efficiency and automatic scaling. The solution implements enterprise integration patterns using Azure's multitenant Logic Apps service[^1][^3]:

- **Primary Workflow:**
    - Process-Invoice-Documents: Handles incoming invoice requests from multiple channels, validates data, processes approvals, and integrates with accounting systems
- **Integration Connections:**
    - Office 365 Outlook for email processing
    - SharePoint Online for document storage
    - Azure SQL Database for invoice tracking
    - Azure Blob Storage for document archival
    - External ERP system via HTTP connector
    - Power Automate connector for approval workflows
- **Supporting Infrastructure:**
    - Azure Key Vault for secrets management
    - Azure Monitor for logging and monitoring
    - Integration Account for B2B scenarios
    - Azure Storage Account for Logic Apps runtime data


### Consumption Model Characteristics

The Consumption model provides specific benefits and limitations that shape the architecture[^1][^2][^4]:

**Benefits:**

- **Pay-per-execution:** Cost-effective for variable workloads
- **Automatic scaling:** No infrastructure management required
- **Fully managed:** Microsoft handles all infrastructure concerns
- **Easy to get started:** Minimal setup and configuration
- **Built-in high availability:** Geo-redundant storage and paired region replication[^1]

**Limitations:**

- **Single workflow per Logic App:** Each Logic App resource can contain only one workflow[^4][^5]
- **Cold start latency:** Potential delays after periods of inactivity[^6]
- **Shared resources:** Performance can be affected by "noisy neighbors"[^3]
- **Limited customization:** Less control over runtime settings compared to Standard[^2]


### Security Architecture

The security model implements multitenant security principles with shared responsibility[^7][^8]:

- **Identity and Access Management:**
    - Azure Active Directory integration for authentication
    - Role-Based Access Control (RBAC) for resource permissions
    - Managed Identity for service-to-service authentication where supported
- **Data Protection:**
    - Encryption in transit using TLS 1.2
    - Encryption at rest using Azure Storage Service Encryption
    - Azure Key Vault integration for sensitive data management
- **Network Security:**
    - HTTPS endpoints for all external communications
    - Access restrictions through Azure portal settings
    - IP allowlisting for trigger endpoints


## Technical Documentation

### Workflow Design Patterns

The Logic App Consumption implementation follows established integration patterns optimized for the multitenant environment[^9]:

#### 1. Event-Driven Processing Pattern

The workflow uses webhook and polling triggers to process invoice documents as they arrive, optimizing for the pay-per-execution model.

#### 2. Error Handling Strategy

Comprehensive error handling using built-in retry policies and error scoping:

```json
{
  "retryPolicy": {
    "type": "exponential",
    "count": 4,
    "interval": "PT7S",
    "maximumInterval": "PT1H"
  },
  "runAfter": {
    "Previous_Action": ["Failed", "Skipped", "TimedOut"]
  }
}
```


#### 3. Conditional Logic Pattern

Business rules implemented through condition actions and switch statements for routing invoices based on amount, vendor, and approval requirements.

#### 4. Batch Processing Considerations

Given Consumption model limitations, batch processing is designed to work within the 500 action limit per workflow[^5].

### Performance Optimization

#### Action Limitations and Considerations

- **Maximum actions per workflow:** 500 actions[^5]
- **Maximum workflow name length:** 80 characters[^5]
- **Maximum parameters per workflow:** 50 parameters[^5]
- **Run duration:** 90 days maximum[^5]
- **Single trigger or action input/output size:** 104 MB[^5]


#### Retry Policy Configuration

Optimized retry policies for different operation types:

- **HTTP actions:** Exponential backoff with 4 retries, maximum 1-hour interval
- **SQL operations:** Fixed interval with 3 retries, 10-second intervals
- **File operations:** Exponential backoff with 5 retries, starting at 7 seconds


#### Trigger Optimization

- **Polling frequency:** Balanced between responsiveness and cost
- **Event-driven triggers:** Preferred over polling to minimize costs[^9]
- **Batch triggers:** SplitOn property configured for high-volume scenarios


### Data Flow Architecture

The data flow follows a simple linear pattern suitable for Consumption model:

1. **Trigger Layer:** Email or HTTP trigger receives invoice documents
2. **Validation Layer:** Schema validation and business rule enforcement
3. **Processing Layer:** Data extraction and transformation
4. **Integration Layer:** External system interactions via connectors
5. **Notification Layer:** Status updates and approval notifications

## Deployment Procedures

### Pre-Deployment Checklist

#### Infrastructure Requirements

- [x] Azure subscription with adequate quotas
- [x] Resource group created with standard naming convention
- [x] Azure Key Vault provisioned for secrets management
- [x] Integration Account created (Basic tier)
- [x] Storage account configured for Logic Apps metadata
- [x] Azure Monitor Log Analytics workspace configured
- [x] Required service connections validated


#### Security and Configuration

- [x] Azure RBAC roles assigned appropriately
- [x] Key Vault access policies configured
- [x] Managed Identity enabled where applicable
- [x] Connection authentication credentials stored securely
- [x] Network access restrictions configured
- [x] Trigger endpoint security validated


#### Development Environment Setup

- [x] Visual Studio 2019 with Azure Logic Apps tools installed[^10]
- [x] Azure Resource Manager templates prepared
- [x] Parameter files configured for each environment
- [x] Source control repository structured properly


### Deployment Process

#### CI/CD Pipeline Configuration

The deployment follows automated CI/CD practices using Azure DevOps[^10]:

1. **Source Control Management**
    - Git repository with feature branch workflow
    - Azure Resource Manager templates versioned
    - Parameter files for each environment
2. **Build Process**
    - ARM template validation and testing
    - Parameter file validation
    - Security scanning of templates and configurations
3. **Release Pipeline Stages**
    - Development environment deployment
    - User acceptance testing validation
    - Production deployment with approval gates

#### ARM Template Structure

Key components of the Consumption Logic App ARM template[^11][^12][^13]:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "logicAppName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Logic App"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Logic/workflows",
      "apiVersion": "2019-05-01",
      "name": "[parameters('logicAppName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "definition": {
          "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "triggers": {},
          "actions": {}
        },
        "parameters": {}
      }
    }
  ]
}
```


#### Deployment Validation Script

```powershell
# Logic App Consumption Deployment Validation
param(
    [Parameter(Mandatory=$true)]
    [string]$ResourceGroupName,
    
    [Parameter(Mandatory=$true)]
    [string]$LogicAppName
)

# Validate Logic App deployment
$logicApp = Get-AzLogicApp -ResourceGroupName $ResourceGroupName -Name $LogicAppName

if ($logicApp.State -ne "Enabled") {
    Write-Error "Logic App is not enabled. Current state: $($logicApp.State)"
    exit 1
}

# Test trigger endpoint
$triggerUrl = Get-AzLogicAppTriggerCallbackUrl -ResourceGroupName $ResourceGroupName -Name $LogicAppName -TriggerName "manual"

try {
    $response = Invoke-RestMethod -Uri $triggerUrl.Value -Method POST -ContentType "application/json" -Body '{"test": "deployment validation"}'
    Write-Host "Trigger endpoint test successful"
} catch {
    Write-Error "Trigger endpoint test failed: $($_.Exception.Message)"
    exit 1
}

Write-Host "Deployment validation completed successfully"
```


## Testing and Validation

### Comprehensive Testing Strategy

#### Unit Testing

Individual workflow components tested using Azure Logic Apps testing capabilities:

- Action-level validation with static results
- Expression function testing with sample data
- Connector configuration validation
- Trigger endpoint testing


#### Integration Testing

End-to-end workflow testing across connected systems:

- Email processing integration validation
- SharePoint document operations testing
- Database connectivity and operations verification
- External API integration testing


#### Load Testing Considerations

Performance validation considering Consumption model characteristics[^14]:

- **Normal load testing:** 50 executions per hour
- **Peak load testing:** 200 executions per hour
- **Performance monitoring:** Response time tracking and billing analysis
- **Throttling validation:** Testing action and trigger throttling limits[^15]


#### User Acceptance Testing

Business process validation with stakeholder involvement:

- Invoice processing workflow end-to-end testing
- Approval process validation
- Error handling scenario testing
- Business rule verification


### Test Results Documentation

| Test Category | Status | Test Count | Pass Rate | Coverage | Evidence Location |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Unit Tests | Pass | 234 | 100% | 89% | Azure DevOps Test Results \#1234 |
| Integration Tests | Pass | 67 | 98.5% | 85% | Test Report \#1235 |
| Load Tests | Pass | 8 scenarios | 100% | N/A | Load Test Report \#1236 |
| User Acceptance Tests | Pass | 28 | 100% | N/A | UAT Signoff Document |

### Known Issues and Limitations

#### Current Known Issues

1. **Issue ID INV-2025-001:** Occasional timeout during high-volume email processing
    - **Severity:** Low
    - **Mitigation:** Retry policy configured with exponential backoff
    - **Resolution Timeline:** Monitoring and optimization in Q4 2025

#### Consumption Model Limitations

- **Single workflow constraint:** Each Logic App can contain only one workflow[^4][^5]
- **Cold start delays:** 2-5 second delays after periods of inactivity[^6]
- **Action limit:** Maximum 500 actions per workflow execution[^5]
- **Trigger limitations:** Only 1 trigger in designer, up to 10 in JSON[^5]


## Monitoring and Observability

### Monitoring Strategy

#### Azure Monitor Integration

Comprehensive monitoring using Azure Monitor and Log Analytics[^16]:

- **Diagnostic Settings Configuration:**
    - WorkflowRuntime logs enabled with 90-day retention
    - Metrics collection for all workflow executions
    - Failed runs and trigger events logging
- **Logic Apps Management Solution:**
    - Installed in Log Analytics workspace for enhanced monitoring[^16]
    - Custom dashboards for business KPIs
    - Automated alert rules for critical failures


#### Key Performance Indicators

| Metric Category | KPI | Target | Alert Threshold |
| :-- | :-- | :-- | :-- |
| Availability | Workflow success rate | >99% | <95% |
| Performance | Average execution time | <30 seconds | >60 seconds |
| Cost | Executions per day | <500 | >750 |
| Error Rate | Failed runs | <1% | >5% |
| Business | Invoice processing success | >98% | <95% |

#### Consumption-Specific Monitoring

Monitoring tailored for Consumption model characteristics[^15]:

- **Billing Metrics:**
    - Total billable executions tracking
    - Action execution cost analysis
    - Trigger execution monitoring
    - Standard connector usage analysis
- **Performance Metrics:**
    - Run latency monitoring
    - Trigger fire latency tracking
    - Action completion times
    - Throttling event detection


#### Alert Configuration

**Critical Alerts:** Immediate notification to operations team

- Workflow failure rate exceeding 5% in 15-minute window
- Service unavailability lasting more than 5 minutes
- Critical business process failures

**Warning Alerts:** Email notifications to support team

- Performance degradation exceeding baseline by 200%
- Billing threshold approaching monthly limits
- Error rate trends indicating potential issues


### Troubleshooting Procedures

#### Common Error Scenarios for Consumption Model

| Error Type | Symptoms | Investigation Steps | Resolution |
| :-- | :-- | :-- | :-- |
| Cold Start Delays | Extended trigger response times | Check Activity Log for inactivity periods | Accept as normal behavior, optimize trigger frequency |
| Action Limit Exceeded | Workflow failures at 500 actions | Review workflow design for optimization | Refactor workflow, use nested Logic Apps |
| Connector Throttling | HTTP 429 errors from connectors | Monitor connector usage patterns | Implement exponential backoff, optimize polling |
| Timeout Issues | Workflow timeouts after 90 days | Review long-running processes | Break into smaller workflows, use checkpointing |

#### Diagnostic Queries for Consumption Model

```kusto
// Failed workflow runs analysis
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.LOGIC"
| where Category == "WorkflowRuntime"
| where OperationName == "Microsoft.Logic/workflows/workflowRunCompleted"
| where status_s == "Failed"
| summarize count() by Resource, error_code_s
| order by count_ desc

// Billing analysis query
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.LOGIC"
| where Category == "WorkflowRuntime"
| where OperationName contains "Action"
| summarize TotalActions = count() by bin(TimeGenerated, 1d), Resource
| order by TimeGenerated desc
```


## Operational Management

### Business Continuity and Disaster Recovery

#### High Availability in Multitenant Environment

- **Built-in redundancy:** Automatic replication to paired regions[^1]
- **Geo-redundant storage:** Enabled by default for high availability[^1]
- **Automatic failover:** Managed by Azure platform
- **Data replication:** Cross-region replication for disaster recovery


#### Backup and Recovery Procedures

- **Workflow Definitions:** ARM templates in source control
- **Configuration Data:** Parameter files and connection configurations
- **Run History:** 90-day retention with Azure Monitor integration[^5]
- **Business Data:** Dependent on connected systems' backup policies


#### Recovery Objectives

- **RTO (Recovery Time Objective):** 15 minutes (platform managed)
- **RPO (Recovery Point Objective):** 5 minutes maximum
- **Business Impact:** Minimal disruption due to automatic failover


### Support and Maintenance

#### Support Team Structure

- **Level 1 Support:** Operations team monitoring dashboards
    - Contact: ops-support@company.com
    - Coverage: 24/7 monitoring
- **Level 2 Support:** Technical specialists
    - Contact: tech-team@company.com
    - Coverage: Business hours with on-call rotation
- **Level 3 Support:** Development team
    - Contact: dev-team@company.com
    - Coverage: On-call for critical issues


#### Maintenance Procedures

- **Routine Maintenance:** Minimal due to fully managed service
- **Connection Updates:** Quarterly review of connector configurations
- **Security Updates:** Monthly security policy reviews
- **Performance Optimization:** Quarterly workflow performance analysis


## Risk Assessment and Mitigation

### Identified Risks and Mitigation Strategies

| Risk Category | Risk Description | Impact | Probability | Mitigation Strategy | Owner |
| :-- | :-- | :-- | :-- | :-- | :-- |
| Cost Management | Unexpected billing spikes due to high execution volume | Medium | Medium | Implement billing alerts and action limits | Technical Lead |
| Performance | Cold start delays affecting user experience | Low | High | Optimize trigger frequency, accept as normal behavior | Operations Team |
| Scalability | Single workflow limitation constraining growth | Medium | Low | Plan for Standard model migration if needed | Technical Lead |
| Dependencies | Connector service outages affecting workflow | High | Low | Implement retry policies and alternative paths | Technical Lead |

### Rollback Plan

#### Rollback Scenarios

- Critical workflow failures affecting business operations
- Unexpected cost escalation due to runaway executions
- Security incidents requiring immediate service isolation
- Data corruption in connected systems


#### Rollback Procedures

1. **Immediate Response (0-5 minutes)**
    - Disable Logic App workflow through Azure portal
    - Activate incident response team
    - Document rollback decision
2. **Recovery Actions (5-30 minutes)**
    - Deploy previous ARM template version
    - Restore previous connection configurations
    - Validate basic functionality
3. **Validation (30-60 minutes)**
    - End-to-end testing of rolled-back version
    - Stakeholder communication
    - Post-incident analysis planning

## Compliance and Governance

### Regulatory Compliance Framework

#### Data Protection Standards

- **GDPR Compliance:** Personal data handling in email processing
- **SOX Controls:** Financial data integrity for invoice processing
- **Industry Standards:** Following NIST cybersecurity framework


#### Audit Trail Implementation

- **Workflow Execution Logs:** Complete execution history maintained
- **Access Logging:** All administrative actions logged
- **Change Management:** Configuration changes tracked in source control
- **Data Processing:** All invoice data processing logged


### Change Management Process

#### Change Categories

- **Minor Changes:** Configuration updates following standard procedures
- **Major Changes:** Workflow logic modifications requiring full testing
- **Emergency Changes:** Critical security or business continuity fixes


#### Approval Workflow

1. Change request with business impact assessment
2. Technical review by development team
3. Security assessment by compliance team
4. Business approval by stakeholders
5. Deployment scheduling and execution

## Approval and Signoff

### Technical Validation

#### Technical Architecture Review

I certify that the Logic App Consumption architecture follows enterprise integration best practices, implements appropriate security controls within the multitenant environment constraints, and meets all technical requirements for production deployment. The solution has been designed to optimize for the pay-per-execution model while maintaining reliability and performance.

**Technical Lead:** Rohit Gupta
**Signature:** ____________________
**Date:** July 18, 2025

#### Infrastructure Validation

I certify that all Azure infrastructure components have been configured according to Consumption model specifications, monitoring systems are operational, and the multitenant environment provides adequate isolation and security for production workloads.

**Cloud Infrastructure Manager:** Sunil Patel
**Signature:** ____________________
**Date:** July 18, 2025

#### Security Review

I certify that comprehensive security assessment has been completed within the constraints of the multitenant Consumption model, all available security controls are properly implemented, and the solution meets organizational security policies and compliance requirements.

**Information Security Manager:** Kavya Nair
**Signature:** ____________________
**Date:** July 19, 2025

### Business Approval

#### Business Requirements Validation

I certify that the Logic App Consumption solution meets all business requirements for invoice processing, provides cost-effective automation within budget constraints, and delivers the expected return on investment through process automation.

**Product Owner:** Ajay Kumar
**Signature:** ____________________
**Date:** July 19, 2025

#### Financial Approval

I certify that the Consumption model pricing has been analyzed and approved, billing alerts and cost controls are in place, and the solution operates within approved budget parameters for production deployment.

**Finance Manager:** Rekha Singh
**Signature:** ____________________
**Date:** July 19, 2025

### Operations Approval

#### Operational Readiness Certification

I certify that operational procedures have been adapted for the Consumption model characteristics, monitoring systems account for cold start behaviors and cost tracking, and the support team is prepared to maintain this solution with appropriate SLAs.

**Operations Manager:** Manoj Krishna
**Signature:** ____________________
**Date:** July 19, 2025

#### Support Team Readiness

I certify that support procedures have been updated for Consumption model troubleshooting, team training includes cost management and billing analysis, and escalation procedures account for the limitations of the multitenant environment.

**Support Manager:** Deepika Gupta
**Signature:** ____________________
**Date:** July 19, 2025

### Final Authorization

#### Project Management Approval

Based on comprehensive validations above and successful completion of all deployment readiness criteria specific to the Logic App Consumption model, I authorize the deployment to production environment with the understanding of the cost implications and architectural constraints.

**Deployment Decision:** [x] Approved [ ] Conditionally Approved [ ] Denied

**Project Manager:** Anita Sharma
**Signature:** ____________________
**Date:** July 20, 2025

#### Executive Sponsorship

I provide executive approval for this Logic App Consumption deployment, acknowledging the cost model implications and architectural constraints, while approving the solution for its cost-effectiveness and business value in automating invoice processing workflows.

**Executive Sponsor:** Dr. Rajesh Menon, CTO
**Signature:** ____________________
**Date:** July 20, 2025

## Post-Deployment Activities

### Immediate Post-Deployment Validation (0-24 hours)

#### Health Check Procedures

- [x] Logic App Consumption workflow status verification
- [x] Trigger endpoint accessibility and response testing
- [x] Integration connection validation with all external systems
- [x] Azure Monitor diagnostic settings confirmation
- [x] Billing meter activation verification
- [x] Error handling mechanism testing
- [x] Performance baseline establishment


#### Business Process Validation

- [x] End-to-end invoice processing workflow testing
- [x] Email trigger functionality verification
- [x] Document processing and approval flow validation
- [x] External system integration confirmation
- [x] Notification system functionality testing


#### Cost Monitoring Setup

- [x] Billing alerts configured for daily/monthly thresholds
- [x] Action execution tracking activated
- [x] Cost analysis dashboard configured
- [x] Budget monitoring enabled


### Short-term Monitoring (1-30 days)

#### Performance and Cost Analysis

- **Execution Patterns:** Analysis of trigger frequency and action usage
- **Cost Optimization:** Review of actual vs. projected costs
- **Performance Trends:** Cold start frequency and impact assessment
- **Error Pattern Analysis:** Identification of recurring issues


#### Operational Adjustments

- **Trigger Optimization:** Fine-tuning polling intervals based on usage patterns
- **Cost Controls:** Adjusting billing alerts based on actual consumption
- **Support Procedures:** Refining troubleshooting guides based on experience
- **Documentation Updates:** Updating operational procedures with lessons learned


### Long-term Optimization (30+ days)

#### Strategic Assessment

- **Cost Efficiency Review:** Comparing actual costs with Standard model projections
- **Performance Analysis:** Evaluating if Consumption model meets performance requirements
- **Scalability Planning:** Assessing growth trajectory and model suitability
- **Technology Roadmap:** Planning for potential migration to Standard model if needed


#### Success Metrics Review

- **Business KPIs:** Invoice processing efficiency and accuracy improvements
- **Technical KPIs:** Workflow reliability, performance, and cost per execution
- **Operational KPIs:** Support ticket volume, resolution time, and team productivity


## Document Control and Governance

**Document Version:** 1.2.3
**Document Type:** Consumption Model Technical Playbook and Signoff Document
**Classification:** Internal Use Only
**Last Updated:** July 20, 2025
**Next Review Date:** October 20, 2025
**Document Owner:** Priya Verma, Senior Integration Specialist
**Approval Required For Changes:** Yes - Technical Lead and Project Manager
**Distribution List:** Project Team, Operations Team, Finance Team, Management

### Version History

| Version | Date | Author | Changes |
| :-- | :-- | :-- | :-- |
| 1.0 | June 20, 2025 | Priya Verma | Initial document creation for Consumption model |
| 1.1 | July 5, 2025 | Priya Verma | Added cost monitoring and billing considerations |
| 1.2 | July 15, 2025 | Rohit Gupta | Technical architecture review and updates |
| 1.2.3 | July 20, 2025 | Priya Verma | Final signoff approvals and deployment validation |

This comprehensive document serves as both the technical playbook for managing the Logic App Consumption deployment and the formal signoff document ensuring all stakeholders understand the specific characteristics, benefits, and limitations of the Consumption model. The document addresses the unique aspects of the multitenant, pay-per-execution model while providing complete reference for deployment, operations, and ongoing maintenance of the Logic App solution.

<div style="text-align: center">⁂</div>

[^1]: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-pricing

[^2]: https://learn.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare

[^3]: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview

[^4]: https://www.linkedin.com/pulse/azure-logic-apps-choosing-between-consumption-orestis-meikopoulos

[^5]: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config

[^6]: https://www.avatech.nz/post/what-is-azure-logic-apps-consumption

[^7]: https://www.linkedin.com/pulse/best-practices-securing-your-azure-logic-apps-divyesh-gohil-i5zlf

[^8]: https://www.linkedin.com/pulse/securing-azure-logic-apps-best-practices-sardar-mudassar-ali-khan-

[^9]: https://dev.to/hitesh_dhamecha_9c59b7c12/best-practices-and-use-cases-for-azure-logic-apps-1pkm

[^10]: https://blog.sandro-pereira.com/2022/04/22/logic-app-ci-cd-from-zero-to-hero-whitepaper/

[^11]: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-create-azure-resource-manager-templates

[^12]: https://learn.microsoft.com/bs-latn-ba/Azure/logic-apps/quickstart-create-deploy-azure-resource-manager-template?tabs=azure-portal

[^13]: https://learn.microsoft.com/en-us/azure/logic-apps/quickstart-create-deploy-azure-resource-manager-template

[^14]: https://stackoverflow.com/questions/79599997/performance-difference-between-azure-logic-apps-consumption-and-standard-workflo

[^15]: https://www.manageengine.com/products/applications_manager/help/azure-logic-apps-consumption-monitoring-tools.html

[^16]: https://learn.microsoft.com/en-us/azure/logic-apps/monitor-workflows-collect-diagnostic-data

[^17]: https://github.com/marnixcox/logicapp-consumption

[^18]: https://learn.microsoft.com/en-us/azure/logic-apps/quickstart-create-example-consumption-workflow

[^19]: https://www.tech-findings.com/2022/07/logic-App-consumption-logic-app-standard.html

[^20]: https://www.manageengine.com/products/applications_manager/help/azure-logic-apps-standard-monitoring-tools.html

[^21]: https://stackoverflow.com/questions/74799025/how-to-deploy-workflow-in-standard-logic-app-with-the-help-of-arm-template/74810534

[^22]: https://blog.sandro-pereira.com/2022/11/10/logic-app-standard-ci-cd-from-zero-to-hero-whitepaper/

[^23]: https://thomasthornton.cloud/2024/10/17/automating-logic-app-deployments-from-designer-to-terraform/

[^24]: https://andrewilson.co.uk/post/2023/02/automate-deployment-of-azure-consumption-logic-apps/

[^25]: https://techcommunity.microsoft.com/blog/appsonazureblog/azure-logic-apps-plans-consumption-based-vs-standard/3793997

[^26]: https://learn.microsoft.com/en-us/azure/logic-apps/devops-deployment-single-tenant-azure-logic-apps

[^27]: https://dev.to/sardarmudassaralikhan/securing-azure-logic-apps-best-practices-and-considerations-3h2o

[^28]: https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-azure-resource-manager-templates-overview

