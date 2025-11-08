# Application Lifecycle Management for AI Solutions

## Overview
This section covers Application Lifecycle Management (ALM) practices for AI-powered business solutions, including version control, deployment strategies, environment management, and change management across Copilot Studio, Azure AI Foundry, Dynamics 365, and Microsoft 365.

---

## 1. ALM for Copilot Studio Agents

### 1.1 Version Control and Source Control

#### Exporting Copilot Studio Bots

**Export Process:**
```
Copilot Studio Portal:
1. Select bot
2. Settings > Export
3. Choose export format:
   - Solution (recommended for ALM)
   - Bot file (.zip)
4. Include:
   ☑ Bot content (topics, entities, variables)
   ☑ Published flows
   ☑ Custom connectors
   ☑ Environment variables
   ☑ Connection references
5. Export

Result: .zip file containing solution components
```

**Power Platform Solution Structure:**
```
CopilotStudioSolution/
├── src/
│   ├── bot/
│   │   ├── manifest.json (bot configuration)
│   │   ├── topics/
│   │   │   ├── greeting.yaml
│   │   │   ├── bookAppointment.yaml
│   │   │   └── escalate.yaml
│   │   ├── entities/
│   │   │   ├── appointmentType.json
│   │   │   └── dateTime.json
│   │   └── variables/
│   │       ├── userName.json
│   │       └── customerID.json
│   ├── flows/
│   │   ├── sendEmailNotification.json
│   │   └── createCRMRecord.json
│   ├── connectors/
│   │   └── customAPI.json
│   └── connections/
│       └── connectionReferences.json
└── solution.xml
```

**Git Integration:**
```bash
# Initialize repository
cd CopilotStudioBot
git init
git add .
git commit -m "Initial commit: Copilot Studio bot v1.0"

# Create branches for environment promotion
git branch develop
git branch staging
git branch production

# Feature development workflow
git checkout develop
git checkout -b feature/add-refund-topic

# ... make changes to bot topics ...

git add .
git commit -m "Add refund processing topic"
git push origin feature/add-refund-topic

# Create pull request for review
# After approval, merge to develop
git checkout develop
git merge feature/add-refund-topic
```

**Power Platform CLI for Source Control:**
```bash
# Install Power Platform CLI
pac install latest

# Authenticate
pac auth create --environment https://contoso.crm.dynamics.com

# Export solution
pac solution export \
  --name "ContosoSupportBot" \
  --path ./exports \
  --managed false \
  --overwrite

# Unpack solution for source control
pac solution unpack \
  --zipfile ./exports/ContosoSupportBot_1_0_0_0.zip \
  --folder ./src \
  --packagetype Both

# Commit unpacked files
git add src/
git commit -m "Export bot version 1.0"

# To import back:
# Pack solution
pac solution pack \
  --zipfile ./build/ContosoSupportBot.zip \
  --folder ./src \
  --packagetype Both

# Import to target environment
pac solution import \
  --path ./build/ContosoSupportBot.zip \
  --environment https://contoso-uat.crm.dynamics.com \
  --activate-plugins
```

#### Configuration Management

**Environment Variables:**
```json
{
  "environmentVariables": [
    {
      "name": "CRMEndpoint",
      "displayName": "CRM API Endpoint",
      "type": "string",
      "values": {
        "dev": "https://contoso-dev.crm.dynamics.com",
        "uat": "https://contoso-uat.crm.dynamics.com",
        "prod": "https://contoso.crm.dynamics.com"
      }
    },
    {
      "name": "EmailServiceKey",
      "displayName": "Email Service API Key",
      "type": "secret",
      "values": {
        "dev": "KEY_DEV_VALUE",
        "uat": "KEY_UAT_VALUE",
        "prod": "KEY_PROD_VALUE"
      }
    },
    {
      "name": "MaxRetryAttempts",
      "displayName": "Maximum API Retry Attempts",
      "type": "number",
      "values": {
        "dev": 3,
        "uat": 3,
        "prod": 5
      }
    },
    {
      "name": "EnableAdvancedLogging",
      "displayName": "Enable Verbose Logging",
      "type": "boolean",
      "values": {
        "dev": true,
        "uat": true,
        "prod": false
      }
    }
  ],
  "connectionReferences": [
    {
      "logicalName": "shared_commondataserviceforapps",
      "displayName": "Dataverse Connection",
      "connectionId": "will-be-set-per-environment"
    },
    {
      "logicalName": "shared_office365",
      "displayName": "Office 365 Outlook Connection",
      "connectionId": "will-be-set-per-environment"
    }
  ]
}
```

**Deployment Configuration File:**
```yaml
# deployment-config.yaml
deployment:
  solution_name: "ContosoSupportBot"
  version: "1.2.0"
  
  environments:
    development:
      url: "https://contoso-dev.crm.dynamics.com"
      solution_type: "unmanaged"
      auto_publish: true
      
    uat:
      url: "https://contoso-uat.crm.dynamics.com"
      solution_type: "managed"
      auto_publish: false
      approval_required: true
      approvers:
        - "uat-approver@contoso.com"
      
    production:
      url: "https://contoso.crm.dynamics.com"
      solution_type: "managed"
      auto_publish: false
      approval_required: true
      approvers:
        - "prod-approver1@contoso.com"
        - "prod-approver2@contoso.com"
      rollback_plan: true
      
  feature_flags:
    dev:
      enable_experimental_features: true
      enable_debug_mode: true
    uat:
      enable_experimental_features: false
      enable_debug_mode: false
    prod:
      enable_experimental_features: false
      enable_debug_mode: false
```

### 1.2 Deployment Pipeline

**Azure DevOps Pipeline for Copilot Studio:**
```yaml
# azure-pipelines-copilot.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/bot/**
      - src/flows/**

variables:
  - group: copilot-deployment-vars
  - name: solution_name
    value: 'ContosoSupportBot'

stages:
  - stage: Build
    jobs:
      - job: PackSolution
        pool:
          vmImage: 'windows-latest'
        steps:
          - task: PowerPlatformToolInstaller@2
            displayName: 'Install Power Platform Tools'
          
          - task: PowerPlatformPackSolution@2
            displayName: 'Pack Solution'
            inputs:
              SolutionSourceFolder: '$(Build.SourcesDirectory)/src'
              SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/$(solution_name)_$(Build.BuildNumber).zip'
              SolutionType: 'Both'
          
          - task: PublishBuildArtifacts@1
            displayName: 'Publish Solution Artifact'
            inputs:
              PathtoPublish: '$(Build.ArtifactStagingDirectory)'
              ArtifactName: 'solution'
  
  - stage: DeployToDev
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
    jobs:
      - deployment: DeployDev
        pool:
          vmImage: 'windows-latest'
        environment: 'Development'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: PowerPlatformImportSolution@2
                  displayName: 'Import to Dev'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(DevEnvironmentConnection)'
                    SolutionInputFile: '$(Pipeline.Workspace)/solution/$(solution_name)_$(Build.BuildNumber).zip'
                    AsyncOperation: true
                    MaxAsyncWaitTime: '60'
                    PublishCustomizationChanges: true
  
  - stage: DeployToUAT
    dependsOn: DeployToDev
    condition: succeeded()
    jobs:
      - deployment: DeployUAT
        pool:
          vmImage: 'windows-latest'
        environment: 'UAT'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: PowerPlatformSetConnectionVariables@2
                  displayName: 'Set UAT Connection Variables'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(UATEnvironmentConnection)'
                    Environment: 'https://contoso-uat.crm.dynamics.com'
                
                - task: PowerPlatformImportSolution@2
                  displayName: 'Import to UAT (Managed)'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(UATEnvironmentConnection)'
                    SolutionInputFile: '$(Pipeline.Workspace)/solution/$(solution_name)_$(Build.BuildNumber)_managed.zip'
                    ConvertToManaged: true
                    AsyncOperation: true
                    PublishCustomizationChanges: false
                
                - task: PowerShell@2
                  displayName: 'Run Smoke Tests'
                  inputs:
                    targetType: 'filePath'
                    filePath: '$(System.DefaultWorkingDirectory)/tests/smoke-tests.ps1'
                    arguments: '-Environment UAT'
  
  - stage: DeployToProduction
    dependsOn: DeployToUAT
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProd
        pool:
          vmImage: 'windows-latest'
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: PowerPlatformBackupEnvironment@2
                  displayName: 'Backup Production Environment'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(ProdEnvironmentConnection)'
                    BackupLabel: 'Pre-deployment-$(Build.BuildNumber)'
                
                - task: PowerPlatformImportSolution@2
                  displayName: 'Import to Production (Managed)'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(ProdEnvironmentConnection)'
                    SolutionInputFile: '$(Pipeline.Workspace)/solution/$(solution_name)_$(Build.BuildNumber)_managed.zip'
                    ConvertToManaged: true
                    AsyncOperation: true
                    HoldingSolution: false
                    OverwriteUnmanagedCustomizations: false
                    SkipProductUpdateDependencies: false
                    PublishCustomizationChanges: false
                
                - task: ManualValidation@0
                  displayName: 'Manual Validation'
                  inputs:
                    notifyUsers: 'prod-validators@contoso.com'
                    instructions: 'Please validate the bot is working correctly in production before completing this step'
                
                - task: PowerShell@2
                  displayName: 'Publish Customizations'
                  condition: succeeded()
                  inputs:
                    targetType: 'inline'
                    script: |
                      pac solution publish --environment $(ProdEnvironmentURL)
```

### 1.3 Change Management

**Change Tracking:**
```json
{
  "changeLog": [
    {
      "version": "1.2.0",
      "date": "2025-11-08",
      "type": "feature",
      "changes": [
        {
          "component": "topic",
          "name": "RefundProcessing",
          "action": "added",
          "description": "New topic for handling refund requests",
          "author": "developer@contoso.com",
          "reviewedBy": "reviewer@contoso.com",
          "jiraTicket": "PROJ-1234"
        },
        {
          "component": "flow",
          "name": "SendRefundEmail",
          "action": "added",
          "description": "Automated email notification for refund status",
          "author": "developer@contoso.com"
        }
      ],
      "testingNotes": "Tested with 50 sample refund scenarios",
      "deployedTo": {
        "dev": "2025-11-01",
        "uat": "2025-11-05",
        "prod": "2025-11-08"
      }
    },
    {
      "version": "1.1.5",
      "date": "2025-10-20",
      "type": "bugfix",
      "changes": [
        {
          "component": "topic",
          "name": "AppointmentBooking",
          "action": "modified",
          "description": "Fixed date parsing for 'next week' queries",
          "bugId": "BUG-456",
          "author": "developer2@contoso.com"
        }
      ]
    }
  ]
}
```

---

## 2. ALM for Custom AI Models (Azure AI Foundry)

### 2.1 Model Versioning

**Model Registry:**
```python
# Register model version in Azure AI Foundry
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Model

ml_client = MLClient(
    credential=credential,
    subscription_id=subscription_id,
    resource_group_name=resource_group,
    workspace_name=workspace_name
)

# Register new model version
model = Model(
    name="customer-intent-classifier",
    version="2.1.0",
    path="./models/intent-classifier-v2.1",
    type="custom_model",
    description="Intent classification model with improved accuracy for refund requests",
    tags={
        "framework": "transformers",
        "base_model": "bert-base-uncased",
        "training_date": "2025-11-01",
        "accuracy": "0.94",
        "f1_score": "0.92",
        "trained_by": "data-scientist@contoso.com"
    },
    properties={
        "training_data_version": "3.2",
        "hyperparameters": {
            "learning_rate": 2e-5,
            "batch_size": 16,
            "epochs": 3
        },
        "evaluation_metrics": {
            "test_accuracy": 0.94,
            "test_f1": 0.92,
            "test_precision": 0.93,
            "test_recall": 0.91
        }
    }
)

registered_model = ml_client.models.create_or_update(model)
print(f"Model registered: {registered_model.name} version {registered_model.version}")
```

**Model Comparison:**
```python
# Compare model versions before promotion
def compare_model_versions(model_name, version1, version2):
    """Compare two model versions"""
    model_v1 = ml_client.models.get(name=model_name, version=version1)
    model_v2 = ml_client.models.get(name=model_name, version=version2)
    
    comparison = {
        "model": model_name,
        "versions": {
            "current": version1,
            "candidate": version2
        },
        "metrics_comparison": {
            "accuracy": {
                "v1": model_v1.properties["evaluation_metrics"]["test_accuracy"],
                "v2": model_v2.properties["evaluation_metrics"]["test_accuracy"],
                "improvement": (
                    model_v2.properties["evaluation_metrics"]["test_accuracy"] -
                    model_v1.properties["evaluation_metrics"]["test_accuracy"]
                )
            },
            "f1_score": {
                "v1": model_v1.properties["evaluation_metrics"]["test_f1"],
                "v2": model_v2.properties["evaluation_metrics"]["test_f1"],
                "improvement": (
                    model_v2.properties["evaluation_metrics"]["test_f1"] -
                    model_v1.properties["evaluation_metrics"]["test_f1"]
                )
            }
        },
        "training_data": {
            "v1": model_v1.properties["training_data_version"],
            "v2": model_v2.properties["training_data_version"]
        },
        "recommendation": None
    }
    
    # Determine recommendation
    if comparison["metrics_comparison"]["accuracy"]["improvement"] > 0.02:
        comparison["recommendation"] = "PROMOTE"
    elif comparison["metrics_comparison"]["accuracy"]["improvement"] > 0:
        comparison["recommendation"] = "CONSIDER"
    else:
        comparison["recommendation"] = "DO_NOT_PROMOTE"
    
    return comparison

# Example usage
comparison = compare_model_versions("customer-intent-classifier", "2.0.0", "2.1.0")
print(json.dumps(comparison, indent=2))
```

### 2.2 Prompt Flow ALM

**Prompt Flow Source Control:**
```yaml
# flow.dag.yaml
$schema: https://azuremlschemas.azureedge.net/promptflow/latest/Flow.schema.json
name: customer_support_flow
display_name: Customer Support AI Flow
type: standard
version: 2.1.0

inputs:
  user_query:
    type: string
    description: Customer's question or request
  
  customer_id:
    type: string
    description: Customer identifier

outputs:
  response:
    type: string
    description: AI-generated response
  
  confidence:
    type: number
    description: Confidence score (0-1)
  
  requires_escalation:
    type: boolean
    description: Whether to escalate to human agent

nodes:
  - name: retrieve_customer_context
    type: python
    source:
      type: code
      path: retrieve_customer.py
    inputs:
      customer_id: ${inputs.customer_id}
    outputs:
      customer_data: object
  
  - name: retrieve_knowledge
    type: retrieval
    source:
      type: azure_ai_search
      connection: azure_search_connection
      index_name: knowledge_base
    inputs:
      query: ${inputs.user_query}
      top_k: 5
    outputs:
      documents: array
  
  - name: classify_intent
    type: llm
    source:
      type: azure_openai
      connection: azure_openai_connection
      deployment_name: gpt-4-turbo
    inputs:
      prompt: |
        Classify the user's intent.
        
        Query: ${inputs.user_query}
        
        Choose from: question, complaint, request, feedback
    outputs:
      intent: string
  
  - name: generate_response
    type: llm
    source:
      type: azure_openai
      connection: azure_openai_connection
      deployment_name: gpt-4-turbo
    inputs:
      prompt: |
        You are a customer support agent.
        
        Customer: ${nodes.retrieve_customer_context.outputs.customer_data}
        Question: ${inputs.user_query}
        Intent: ${nodes.classify_intent.outputs.intent}
        Knowledge: ${nodes.retrieve_knowledge.outputs.documents}
        
        Provide a helpful, accurate response.
    outputs:
      response: string
  
  - name: evaluate_confidence
    type: python
    source:
      type: code
      path: evaluate_confidence.py
    inputs:
      response: ${nodes.generate_response.outputs.response}
      documents: ${nodes.retrieve_knowledge.outputs.documents}
    outputs:
      confidence: number
      requires_escalation: boolean
```

**Deployment Script:**
```python
# deploy_prompt_flow.py
from azure.ai.ml import MLClient
from azure.ai.ml.entities import OnlineDeployment, ManagedOnlineEndpoint

def deploy_prompt_flow(environment, version):
    """Deploy Prompt Flow to specified environment"""
    
    ml_client = get_ml_client(environment)
    
    # Create or get endpoint
    endpoint_name = f"customer-support-{environment}"
    try:
        endpoint = ml_client.online_endpoints.get(endpoint_name)
    except:
        endpoint = ManagedOnlineEndpoint(
            name=endpoint_name,
            description=f"Customer Support Flow - {environment}",
            auth_mode="key",
            tags={"environment": environment}
        )
        ml_client.online_endpoints.begin_create_or_update(endpoint).result()
    
    # Create deployment
    deployment = OnlineDeployment(
        name=f"v{version.replace('.', '-')}",
        endpoint_name=endpoint_name,
        model=f"customer_support_flow:{version}",
        instance_type="Standard_DS3_v2",
        instance_count=2 if environment == "prod" else 1,
        environment_variables={
            "ENVIRONMENT": environment,
            "LOG_LEVEL": "INFO" if environment == "prod" else "DEBUG"
        },
        request_settings={
            "request_timeout_ms": 30000,
            "max_concurrent_requests_per_instance": 10
        },
        liveness_probe={
            "initial_delay": 10,
            "period": 30,
            "timeout": 5
        }
    )
    
    # Deploy
    ml_client.online_deployments.begin_create_or_update(deployment).result()
    
    # Route traffic (blue-green deployment)
    if environment == "prod":
        # Route 10% to new version initially
        endpoint.traffic = {
            f"v{version.replace('.', '-')}": 10,
            # Remaining traffic to previous version
        }
    else:
        # Route 100% in non-prod
        endpoint.traffic = {f"v{version.replace('.', '-')}": 100}
    
    ml_client.online_endpoints.begin_create_or_update(endpoint).result()
    
    print(f"Deployed version {version} to {environment}")
    print(f"Endpoint: {endpoint.scoring_uri}")

# Usage
if __name__ == "__main__":
    import sys
    environment = sys.argv[1]  # dev, uat, prod
    version = sys.argv[2]       # e.g., "2.1.0"
    
    deploy_prompt_flow(environment, version)
```

### 2.3 Model Monitoring and Rollback

**Deployment Monitoring:**
```python
# monitor_deployment.py
from azure.ai.ml import MLClient
from azure.monitor.query import LogsQueryClient
from datetime import datetime, timedelta

class DeploymentMonitor:
    def __init__(self, environment):
        self.ml_client = get_ml_client(environment)
        self.logs_client = LogsQueryClient(credential)
        self.environment = environment
    
    def monitor_deployment_health(self, deployment_name, duration_minutes=30):
        """Monitor deployment after release"""
        end_time = datetime.utcnow()
        start_time = end_time - timedelta(minutes=duration_minutes)
        
        # Query metrics
        query = f"""
        requests
        | where timestamp between (datetime({start_time.isoformat()}) .. datetime({end_time.isoformat()}))
        | where cloud_RoleName == "{deployment_name}"
        | summarize 
            request_count = count(),
            avg_duration = avg(duration),
            p95_duration = percentile(duration, 95),
            error_count = countif(success == false),
            error_rate = (countif(success == false) * 100.0) / count()
        """
        
        results = self.logs_client.query_workspace(
            workspace_id=workspace_id,
            query=query,
            timespan=timedelta(minutes=duration_minutes)
        )
        
        metrics = results.tables[0].rows[0] if results.tables[0].rows else None
        
        if not metrics:
            return {"status": "NO_DATA", "action": "WAIT"}
        
        # Evaluate health
        health_checks = {
            "error_rate": metrics["error_rate"] < 5.0,  # < 5%
            "avg_duration": metrics["avg_duration"] < 5000,  # < 5s
            "p95_duration": metrics["p95_duration"] < 10000,  # < 10s
            "request_volume": metrics["request_count"] > 10  # Sufficient data
        }
        
        all_healthy = all(health_checks.values())
        
        return {
            "status": "HEALTHY" if all_healthy else "UNHEALTHY",
            "metrics": metrics,
            "health_checks": health_checks,
            "action": "PROMOTE" if all_healthy else "ROLLBACK"
        }
    
    def rollback_deployment(self, endpoint_name, previous_version):
        """Rollback to previous version"""
        endpoint = self.ml_client.online_endpoints.get(endpoint_name)
        
        # Route 100% traffic to previous version
        endpoint.traffic = {previous_version: 100}
        
        self.ml_client.online_endpoints.begin_create_or_update(endpoint).result()
        
        print(f"Rolled back {endpoint_name} to {previous_version}")
        
        # Send alert
        send_alert(
            subject=f"Deployment Rollback: {endpoint_name}",
            message=f"Deployment rolled back to {previous_version} due to health check failure",
            severity="high"
        )

# Automated monitoring
monitor = DeploymentMonitor("production")
health = monitor.monitor_deployment_health("customer-support-v2-1-0", duration_minutes=30)

if health["action"] == "ROLLBACK":
    monitor.rollback_deployment("customer-support-prod", "v2-0-0")
elif health["action"] == "PROMOTE":
    # Gradually increase traffic to new version
    promote_deployment("customer-support-prod", "v2-1-0", traffic_percentage=100)
```

---

## 3. ALM for Dynamics 365 AI Features

### 3.1 Configuration as Code

**Dynamics 365 Configuration Export:**
```powershell
# Export Dynamics 365 AI configurations
Connect-CrmOnline -ServerUrl "https://contoso.crm.dynamics.com"

# Export AI Builder models
$models = Get-CrmRecords -EntityLogicalName "msdyn_aimodel"

foreach ($model in $models.CrmRecords) {
    $modelExport = @{
        name = $model.msdyn_name
        type = $model.msdyn_modeltype
        version = $model.msdyn_version
        status = $model.statuscode
        configuration = $model.msdyn_modelconfiguration
    }
    
    $modelExport | ConvertTo-Json -Depth 10 | Out-File "./config/aimodels/$($model.msdyn_name).json"
}

# Export Copilot configurations
$copilotSettings = Get-CrmRecord -EntityLogicalName "organization" -Id $orgId -Fields "msdyn_copilotconfig"

$copilotSettings | ConvertTo-Json -Depth 10 | Out-File "./config/copilot-settings.json"
```

**Import Script:**
```powershell
# Import configurations to target environment
param(
    [Parameter(Mandatory=$true)]
    [string]$TargetEnvironment,
    
    [Parameter(Mandatory=$true)]
    [string]$ConfigPath
)

Connect-CrmOnline -ServerUrl $TargetEnvironment

# Import AI Builder models
$modelFiles = Get-ChildItem "$ConfigPath/aimodels/*.json"

foreach ($file in $modelFiles) {
    $config = Get-Content $file.FullName | ConvertFrom-Json
    
    # Check if model exists
    $existing = Get-CrmRecords -EntityLogicalName "msdyn_aimodel" `
        -FilterAttribute "msdyn_name" -FilterOperator "eq" -FilterValue $config.name
    
    if ($existing.CrmRecords.Count -eq 0) {
        # Create new
        $newModel = @{
            "msdyn_name" = $config.name
            "msdyn_modeltype" = $config.type
            "msdyn_modelconfiguration" = $config.configuration
        }
        
        New-CrmRecord -EntityLogicalName "msdyn_aimodel" -Fields $newModel
        Write-Host "Created AI model: $($config.name)"
    } else {
        # Update existing
        $modelId = $existing.CrmRecords[0].msdyn_aimodelid
        
        $updates = @{
            "msdyn_modelconfiguration" = $config.configuration
            "msdyn_version" = $config.version
        }
        
        Set-CrmRecord -EntityLogicalName "msdyn_aimodel" -Id $modelId -Fields $updates
        Write-Host "Updated AI model: $($config.name)"
    }
}
```

### 3.2 Environment Strategy

**Environment Promotion Flow:**
```
┌──────────────────────────────────────────────────────────┐
│              Dynamics 365 ALM Pipeline                    │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  Development                                              │
│  ├─ Unmanaged solution                                   │
│  ├─ Frequent deployments                                 │
│  ├─ Developer testing                                     │
│  └─ Continuous integration                                │
│                    ↓                                       │
│             [Export & Build]                              │
│                    ↓                                       │
│  User Acceptance Testing (UAT)                           │
│  ├─ Managed solution                                      │
│  ├─ QA testing                                            │
│  ├─ User validation                                       │
│  └─ Performance testing                                   │
│                    ↓                                       │
│          [Approval Required]                              │
│                    ↓                                       │
│  Production                                               │
│  ├─ Managed solution                                      │
│  ├─ Change advisory board approval                       │
│  ├─ Scheduled maintenance window                         │
│  ├─ Backup before deployment                             │
│  └─ Rollback plan ready                                   │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

**Deployment Checklist:**
```markdown
## Pre-Deployment Checklist

### Planning
- [ ] Change request approved by CAB
- [ ] Deployment scheduled in maintenance window
- [ ] Stakeholders notified
- [ ] Rollback plan documented

### Environment Preparation
- [ ] Target environment backed up
- [ ] Required licenses available
- [ ] Connection references configured
- [ ] Environment variables set
- [ ] Users notified of downtime (if applicable)

### Solution Readiness
- [ ] Solution tested in UAT
- [ ] All test cases passed
- [ ] Performance benchmarks met
- [ ] Security review completed
- [ ] Documentation updated

### Deployment
- [ ] Import solution (managed)
- [ ] Activate flows/plugins
- [ ] Publish customizations
- [ ] Verify connections
- [ ] Run smoke tests
- [ ] Monitor for errors (30 min)

### Post-Deployment
- [ ] Smoke tests passed
- [ ] Key scenarios validated
- [ ] Users can access system
- [ ] No critical errors in logs
- [ ] Performance within acceptable range
- [ ] Stakeholders notified of completion

### Rollback (if needed)
- [ ] Restore from backup
- [ ] OR uninstall solution
- [ ] Verify system restored
- [ ] Incident investigation initiated
```

---

## 4. ALM for Microsoft 365 Copilot

### 4.1 Declarative Agent Management

**Agent Manifest Versioning:**
```json
{
  "manifestVersion": "1.0",
  "id": "com.contoso.sales-assistant",
  "version": "2.1.0",
  "name": {
    "short": "Sales Assistant",
    "full": "Contoso Sales Assistant Copilot"
  },
  "description": {
    "short": "AI assistant for sales professionals",
    "full": "Intelligent assistant that helps sales teams with customer insights, opportunity management, and deal progression"
  },
  "developer": {
    "name": "Contoso IT",
    "websiteUrl": "https://contoso.com",
    "privacyUrl": "https://contoso.com/privacy",
    "termsOfUseUrl": "https://contoso.com/terms"
  },
  "capabilities": {
    "declarativeCopilot": {
      "actions": [
        {
          "id": "getCustomerInsights",
          "file": "actions/customer-insights.json"
        },
        {
          "id": "updateOpportunity",
          "file": "actions/update-opportunity.json"
        }
      ],
      "instructions": {
        "file": "instructions.txt"
      },
      "knowledgeSources": [
        {
          "type": "sharepoint",
          "siteUrl": "https://contoso.sharepoint.com/sites/Sales",
          "documentLibraries": ["Product Information", "Sales Playbooks"]
        },
        {
          "type": "graphConnector",
          "connectorId": "salesforceConnector"
        }
      ]
    }
  },
  "permissions": [
    {
      "name": "files.read",
      "type": "delegated"
    },
    {
      "name": "user.read",
      "type": "delegated"
    }
  ],
  "validDomains": [
    "contoso.sharepoint.com",
    "api.contoso.com"
  ]
}
```

**Deployment via Teams Toolkit:**
```bash
# Install Teams Toolkit CLI
npm install -g @microsoft/teamsfx-cli

# Login
teamsfx account login azure
teamsfx account login m365

# Deploy to dev environment
teamsfx provision --env dev
teamsfx deploy --env dev
teamsfx publish --env dev

# Promote to production
teamsfx provision --env prod
teamsfx deploy --env prod
teamsfx publish --env prod

# Update app package
teamsfx update package --env prod --manifest-path ./appPackage/manifest.json
```

### 4.2 Plugin Lifecycle Management

**Plugin Versioning:**
```yaml
# Plugin manifest with versioning
openapi: 3.0.0
info:
  title: Contoso CRM Plugin
  description: Plugin for accessing Contoso CRM data
  version: 2.1.0
  x-microsoft-metadata:
    changeLog:
      - version: 2.1.0
        date: 2025-11-08
        changes:
          - Added opportunity forecast endpoint
          - Improved error handling
      - version: 2.0.0
        date: 2025-10-01
        changes:
          - Breaking: Changed authentication method
          - Added batch operations

servers:
  - url: https://api.contoso.com/v2
    description: Production API

paths:
  /customers/{id}:
    get:
      operationId: getCustomer
      summary: Get customer details
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Customer details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Customer'

components:
  schemas:
    Customer:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        industry:
          type: string
```

**Plugin Deployment Pipeline:**
```yaml
# GitHub Actions workflow for plugin deployment
name: Deploy Copilot Plugin

on:
  push:
    branches: [main, develop]
    paths:
      - 'plugins/**'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Validate OpenAPI spec
        run: |
          npm install -g @apidevtools/swagger-cli
          swagger-cli validate plugins/crm-plugin/openapi.yaml
      
      - name: Build plugin package
        run: |
          cd plugins/crm-plugin
          npm run build
      
      - name: Deploy to Azure API Management
        env:
          AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
        run: |
          az login --service-principal -u ${{ secrets.AZURE_CLIENT_ID }} -p ${{ secrets.AZURE_CLIENT_SECRET }} --tenant ${{ secrets.AZURE_TENANT_ID }}
          az apim api import \
            --resource-group contoso-rg \
            --service-name contoso-apim \
            --path /crm \
            --specification-format OpenApi \
            --specification-path plugins/crm-plugin/openapi.yaml \
            --api-id crm-plugin
      
      - name: Update M365 App Catalog
        run: |
          teamsfx deploy --env production --manifest-path ./plugins/crm-plugin/manifest.json
```

---

## 5. Best Practices

### 5.1 Version Control Best Practices

**Branching Strategy:**
```
main (production)
├── release/v2.1 (release candidate)
├── develop (integration branch)
    ├── feature/new-refund-topic
    ├── feature/improve-intent-recognition
    └── bugfix/date-parsing-issue
```

**Commit Message Convention:**
```
<type>(<scope>): <subject>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- refactor: Code refactoring
- test: Test additions/changes
- chore: Build/config changes

Examples:
feat(copilot): add refund processing topic
fix(intent): correct date parsing for relative dates
docs(readme): update deployment instructions
test(integration): add tests for CRM integration
```

### 5.2 Deployment Best Practices

**Blue-Green Deployment:**
```python
# Implement blue-green deployment strategy
def blue_green_deploy(endpoint_name, new_version):
    """
    Deploy new version alongside current, then switch traffic
    """
    # Deploy new version (green)
    deploy_version(endpoint_name, new_version, traffic=0)
    
    # Monitor health
    if monitor_health(endpoint_name, new_version, duration=15):
        # Gradually shift traffic
        shift_traffic(endpoint_name, new_version, percentage=10)
        time.sleep(300)  # Wait 5 minutes
        
        if monitor_health(endpoint_name, new_version, duration=10):
            shift_traffic(endpoint_name, new_version, percentage=50)
            time.sleep(300)
            
            if monitor_health(endpoint_name, new_version, duration=10):
                shift_traffic(endpoint_name, new_version, percentage=100)
                print("Deployment successful")
            else:
                rollback(endpoint_name, previous_version)
        else:
            rollback(endpoint_name, previous_version)
    else:
        rollback(endpoint_name, previous_version)
```

**Canary Deployment:**
```yaml
# Canary deployment configuration
canaryDeployment:
  stages:
    - name: initial
      trafficPercentage: 5
      duration: 30m
      successCriteria:
        errorRate: "< 2%"
        latency_p95: "< 5000ms"
    
    - name: expand
      trafficPercentage: 25
      duration: 1h
      successCriteria:
        errorRate: "< 1%"
        latency_p95: "< 5000ms"
        userSatisfaction: "> 4.0"
    
    - name: full
      trafficPercentage: 100
      successCriteria:
        errorRate: "< 1%"
        latency_p95: "< 5000ms"
  
  rollback:
    automatic: true
    conditions:
      - errorRate: "> 5%"
      - latency_p95: "> 10000ms"
      - crashRate: "> 0.1%"
```

---

## Key Takeaways

✅ **Source Control Everything:**
- Version control all AI assets (prompts, models, configurations)
- Use Git for code and Power Platform solutions
- Maintain environment-specific configurations

✅ **Automated Deployment:**
- CI/CD pipelines for all environments
- Automated testing before promotion
- Environment-specific validation

✅ **Environment Strategy:**
- Dev → UAT → Production promotion flow
- Managed solutions for UAT/Prod
- Environment variables for configuration

✅ **Change Management:**
- Document all changes in changelog
- Require approvals for production
- Maintain rollback procedures

✅ **Monitoring and Rollback:**
- Monitor deployments actively
- Automated health checks
- Quick rollback capabilities

---

## Study Resources

- [Power Platform ALM](https://learn.microsoft.com/en-us/power-platform/alm/)
- [Azure AI Foundry MLOps](https://learn.microsoft.com/en-us/azure/machine-learning/concept-model-management-and-deployment)
- [Teams App Lifecycle](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/apps-publish-overview)
- [Copilot Studio ALM](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-export-import-bots)

**Previous Topic:** [Testing AI Solutions](02-testing.md)  
**Next Topic:** [Security and Governance](04-security-governance.md)
