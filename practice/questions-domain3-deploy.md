# Domain 3: Deploy AI-Powered Business Solutions - Practice Questions

## Multiple Choice Questions

### Topic: Monitor and Tune

**Question 1:**
Your Azure AI Foundry deployment shows 95th percentile latency of 8 seconds, but your SLA requires <3 seconds. What should you investigate FIRST?

A) Switch to a larger model  
B) Analyze prompt token length and retrieval overhead  
C) Increase Azure compute resources  
D) Disable safety filters

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** High latency is often caused by:
1. Long prompts consuming many tokens
2. RAG retrieval taking too long (large knowledge base, inefficient search)
3. Multiple API calls in sequence

Before increasing resources or changing models, optimize the request itself. Use Application Insights to identify where time is spent.

**Reference:** Domain 3 - Monitor and Tune, Performance Optimization
</details>

---

**Question 2:**
Which metric in Copilot Studio Analytics indicates users are not getting useful responses?

A) High engagement rate  
B) High abandon rate  
C) Low session count  
D) High messages per session

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Abandon rate (percentage of users who leave without completing their task) indicates dissatisfaction. High abandonment suggests the bot isn't providing useful responses or resolving user issues.

**Other indicators of poor quality:**
- Low resolution rate
- High escalation rate
- Low CSAT scores
- High fallback topic triggers

**Reference:** Domain 3 - Monitor and Tune, Copilot Studio Analytics
</details>

---

**Question 3:**
You're implementing semantic caching for an AI agent. Which queries benefit MOST from caching?

A) Unique user-specific questions  
B) Time-sensitive data requests  
C) Common FAQ-style questions  
D) Complex multi-step reasoning tasks

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Semantic caching stores responses for similar queries. FAQ-style questions (product info, policies, how-to guides) are asked repeatedly by different users and don't change frequently, making them ideal for caching.

**Poor candidates for caching:**
- User-specific data (account balance, order status)
- Time-sensitive info (stock prices, inventory)
- Unique complex queries unlikely to repeat

**Reference:** Domain 3 - Monitor and Tune, Caching Strategies
</details>

---

**Question 4:**
Your AI solution costs $15,000/month in Azure OpenAI API calls. Analysis shows 60% of queries use GPT-4 but only 20% require its capabilities. What optimization strategy provides the BEST cost reduction?

A) Reduce max_tokens for all queries  
B) Implement model routing (GPT-3.5 for simple, GPT-4 for complex)  
C) Decrease temperature parameter  
D) Switch entirely to GPT-3.5

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Model routing directs simple queries to cheaper models (GPT-3.5) and complex ones to more capable models (GPT-4). If 60% of queries use GPT-4 but only 20% need it, you're overpaying for 40% of queries.

**Potential savings:**
- GPT-4: ~$0.03/1K tokens
- GPT-3.5: ~$0.002/1K tokens
- 15x cost difference
- 40% of queries × 15x savings = ~60% cost reduction

**Reference:** Domain 3 - Monitor and Tune, Cost Optimization
</details>

---

### Topic: Testing

**Question 5:**
What is the PRIMARY purpose of groundedness evaluation in AI testing?

A) Measure response speed  
B) Verify responses are based on provided context, not hallucinations  
C) Check grammar and spelling  
D) Ensure responses are concise

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Groundedness measures whether the AI's response is supported by the retrieved context (knowledge base, documents). A grounded response only includes information present in the source material, preventing hallucinations.

**Evaluation method:**
```python
groundedness_score = (
    claims_supported_by_source / total_claims_in_response
)
```

**Target:** >90% groundedness for production systems

**Reference:** Domain 3 - Testing, Validation Criteria
</details>

---

**Question 6:**
You're conducting performance testing for an AI agent expecting 1,000 concurrent users. Which tool is MOST appropriate?

A) Unit tests with pytest  
B) Azure Load Testing with JMeter scripts  
C) Manual testing with multiple browsers  
D) Copilot Studio Test bot feature

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Azure Load Testing supports large-scale performance testing with:
- Concurrent user simulation (1,000+)
- Distributed load generation
- Metrics collection (latency, throughput, errors)
- Integration with Azure Monitor

**Why other options are incorrect:**
- A: Unit tests don't simulate load
- C: Manual testing can't scale to 1,000 concurrent users
- D: Test bot is for functional testing, not load testing

**Reference:** Domain 3 - Testing, Performance Testing
</details>

---

**Question 7:**
In a CI/CD pipeline for Copilot Studio, at which stage should you run automated quality evaluations (groundedness, relevance)?

A) Before exporting the solution  
B) After deployment to production  
C) During the build stage before deployment  
D) Only in manual testing

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Quality evaluations should run during the build/test stage, BEFORE deploying to production. This "shift-left" approach catches issues early.

**Pipeline flow:**
```
Export → Build → Test (quality evals) → Deploy to UAT → Deploy to Prod
                    ↑
              Block if quality < threshold
```

**Reference:** Domain 3 - Testing, CI/CD Integration
</details>

---

### Topic: Application Lifecycle Management

**Question 8:**
What is the recommended branching strategy for AI solution development in Git?

A) Single main branch for all changes  
B) Feature branching (main, develop, feature branches)  
C) One branch per developer  
D) No version control needed for AI prompts

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Feature branching (GitFlow) provides:
- **main:** Production-ready code
- **develop:** Integration branch for features
- **feature/*:** Individual features/changes
- **hotfix/*:** Urgent production fixes

This enables parallel development, code review, and controlled releases.

**Reference:** Domain 3 - ALM, Version Control Best Practices
</details>

---

**Question 9:**
You deployed a new version of your AI model that shows degraded performance. What is the FASTEST rollback approach?

A) Restore from backup and redeploy (30 minutes)  
B) Use blue-green deployment switch (instant)  
C) Revert Git commit and rebuild pipeline (20 minutes)  
D) Manually reconfigure to previous version (15 minutes)

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Blue-green deployment maintains both versions running simultaneously. Rollback is a simple traffic switch between blue (old) and green (new) environments.

**Rollback approaches ranked by speed:**
1. **Blue-green switch:** Instant (change traffic routing)
2. **Previous version container:** <5 min (redeploy existing image)
3. **Pipeline rebuild:** 15-30 min (rebuild and deploy)
4. **Backup restore:** 30+ min (restore and configure)

**Reference:** Domain 3 - ALM, Deployment Patterns
</details>

---

**Question 10:**
When versioning AI models in Azure AI Foundry, what metadata should you track? (Select all that apply)

A) Training data date range  
B) Developer's personal email  
C) Hyperparameters and configuration  
D) Evaluation metrics (accuracy, F1)

<details>
<summary>Answer</summary>

**Correct Answer: A, C, D**

**Explanation:** Essential model metadata:
- Training data characteristics (date range, size, sources)
- Hyperparameters (learning rate, batch size, epochs)
- Configuration (architecture, frameworks, dependencies)
- Performance metrics (accuracy, precision, recall, F1)
- Deployment history and rollback info

Personal information like developer emails is not relevant to model metadata.

**Reference:** Domain 3 - ALM, Model Versioning
</details>

---

### Topic: Security and Governance

**Question 11:**
A healthcare AI application processes patient data. Which compliance framework is MANDATORY?

A) SOC 2  
B) HIPAA  
C) GDPR  
D) ISO 27001

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** HIPAA (Health Insurance Portability and Accountability Act) is mandatory for healthcare applications in the US that handle Protected Health Information (PHI). Requirements include:
- Encryption at rest and in transit
- Access controls and authentication
- Audit logging
- Business Associate Agreements (BAAs)
- Breach notification procedures

**Other frameworks:**
- GDPR: Required if serving EU patients
- SOC 2: Common but not legally required
- ISO 27001: Best practice, not mandatory

**Reference:** Domain 3 - Security and Governance, HIPAA Compliance
</details>

---

**Question 12:**
Which Microsoft Responsible AI principle is MOST directly addressed by implementing bias detection in production?

A) Reliability and Safety  
B) Fairness  
C) Privacy and Security  
D) Transparency

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** The Fairness principle requires AI systems to treat all people fairly and avoid discriminatory outcomes. Bias detection identifies when the model performs differently across demographic groups.

**Fairness techniques:**
- Pre-processing: Balance training data
- In-processing: Fairness constraints during training
- Post-processing: Adjust decision thresholds per group

**Reference:** Domain 3 - Security and Governance, Responsible AI Principles
</details>

---

**Question 13:**
Under GDPR, a user requests deletion of all their data. What must your AI system do?

A) Delete data from production database only  
B) Delete from all systems including backups, logs, and training data  
C) Anonymize data in production, retain in backups  
D) Mark as deleted but retain for audit purposes

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** GDPR's "right to erasure" (right to be forgotten) requires deletion from:
- Production databases
- Backup systems
- Application logs
- Analytics data
- Training datasets (or retrain model)

**Exception:** Audit logs required for legal compliance can be retained but must be anonymized.

**Reference:** Domain 3 - Security and Governance, GDPR Compliance
</details>

---

## Scenario-Based Questions

### Scenario 1: Performance Degradation

**Context:**
Your production Copilot Studio bot shows:
- Week 1: 92% resolution rate, avg 2.3 sec latency
- Week 4: 78% resolution rate, avg 5.1 sec latency
- No code changes deployed

**Questions:**

**1.1** What are the most likely causes?

<details>
<summary>Answer</summary>

**Likely causes:**

1. **Knowledge base drift:** Data sources became stale or unavailable
2. **Increased complexity:** Users asking harder questions
3. **External API slowness:** Dependent services degraded
4. **Data volume growth:** Knowledge base grew, search slower
5. **Model performance:** Upstream AI service issues

**Investigation steps:**
```
1. Check Application Insights:
   - API dependency latency
   - Failed requests
   - Exception patterns

2. Review Copilot Studio Analytics:
   - Topic recognition trends
   - Fallback rate increase
   - Unrecognized utterances

3. Test knowledge sources:
   - SharePoint/Dataverse availability
   - Data freshness timestamps
   - Search query performance

4. Compare user queries:
   - Week 1 vs Week 4 query complexity
   - New topics emerging
```
</details>

**1.2** Design a monitoring dashboard to detect this earlier.

<details>
<summary>Answer</summary>

**Azure Dashboard - AI Bot Health**

**Section 1: Performance Metrics**
- Average latency (line chart, 7-day trend)
  - Alert: >3 sec
- 95th percentile latency
  - Alert: >5 sec
- Request rate (requests/minute)
- Error rate (%)
  - Alert: >2%

**Section 2: Quality Metrics**
- Resolution rate (%)
  - Alert: <85%
- Abandon rate (%)
  - Alert: >25%
- CSAT score (1-5)
  - Alert: <4.0
- Escalation rate (%)
  - Alert: >20%

**Section 3: Topic Performance**
- Top 10 topics by volume
- Topics with highest fallback rate
- New/emerging topics (ML-detected clusters)

**Section 4: Dependencies**
- SharePoint API latency
- Dataverse API latency
- Azure OpenAI latency
- Custom API availability

**Section 5: Cost**
- Daily token consumption
- Cost per conversation
- Monthly burn rate vs budget

**Alerts:**
- Slack/Teams notification for threshold breaches
- Weekly performance report
- Anomaly detection (statistical deviation)
</details>

---

### Scenario 2: Deployment Pipeline Failure

**Context:**
Your Azure DevOps pipeline for deploying a Copilot Studio bot failed during the "Deploy to UAT" stage:

```
Error: Solution import failed
Details: Missing dependency - 
  Flow 'ProcessOrder' not found in target environment
```

**Questions:**

**2.1** What caused the failure and how do you fix it?

<details>
<summary>Answer</summary>

**Cause:** 
The bot depends on a Power Automate flow (`ProcessOrder`) that doesn't exist in the UAT environment. This is a missing dependency issue.

**Root cause analysis:**
1. Flow created in Dev but not exported with solution
2. Flow exists but not included in solution dependencies
3. Flow in UAT has different name or was deleted

**Fix - Immediate:**
```bash
# 1. Verify flow exists in source environment
pac flow list --environment <dev-env-id>

# 2. Add flow to solution in Dev
pac solution add-reference --path . \
  --component-type 29 \  # Flow component type
  --component-id <flow-guid>

# 3. Re-export solution
pac solution export \
  --path solution.zip \
  --name ContosoBot \
  --managed

# 4. Import to UAT
pac solution import \
  --path solution.zip \
  --environment <uat-env-id>
```

**Fix - Prevent future occurrences:**
1. Use solution checker in pipeline before export
2. Dependency analysis step in pipeline
3. Automated testing in dev that exercises all flows
4. Environment comparison tool to detect drift
</details>

**2.2** Design a robust deployment pipeline with quality gates.

<details>
<summary>Answer</summary>

**Azure DevOps Pipeline - AI Solution Deployment**

```yaml
trigger:
  branches:
    include: [main]

stages:
  - stage: Build
    jobs:
      - job: Export_and_Build
        steps:
          - task: PowerPlatformToolInstaller@2
          
          - task: PowerPlatformExportSolution@2
            inputs:
              authenticationType: 'PowerPlatformSPN'
              environment: '$(DevEnvironmentUrl)'
              solutionName: '$(SolutionName)'
              solutionOutputFile: '$(Build.ArtifactStagingDirectory)\$(SolutionName).zip'
          
          - task: PowerPlatformUnpackSolution@2
            inputs:
              solutionInputFile: '$(Build.ArtifactStagingDirectory)\$(SolutionName).zip'
              sourceFolder: '$(Build.SourcesDirectory)\solutions'
          
          - script: |
              # Solution checker - validate dependencies
              pac solution check --path $(Build.ArtifactStagingDirectory)\$(SolutionName).zip
            displayName: 'Validate Solution'
          
          - task: PublishBuildArtifacts@1

  - stage: Test
    dependsOn: Build
    jobs:
      - job: Quality_Tests
        steps:
          - task: DownloadBuildArtifacts@0
          
          - script: |
              # Import to test environment
              pac solution import --path $(SolutionName).zip --environment $(TestEnvUrl)
            displayName: 'Deploy to Test'
          
          - task: PythonScript@0
            inputs:
              scriptSource: 'filepath'
              scriptPath: 'tests/test_quality.py'
            displayName: 'Run Quality Evaluations'
            # Tests groundedness, relevance, safety
          
          - script: |
              # Load testing
              artillery run load-test.yml
            displayName: 'Performance Testing'
          
          - task: PublishTestResults@2
            condition: always()

  - stage: Deploy_UAT
    dependsOn: Test
    condition: succeeded()
    jobs:
      - deployment: UAT_Deployment
        environment: 'UAT'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: PowerPlatformImportSolution@2
                  inputs:
                    environment: '$(UATEnvironmentUrl)'
                    solutionInputFile: '$(SolutionName)_managed.zip'
                
                - script: |
                    # Smoke tests
                    python tests/smoke_tests.py --env UAT
                  displayName: 'Smoke Tests'

  - stage: Deploy_Production
    dependsOn: Deploy_UAT
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Production_Deployment
        environment: 'Production'  # Manual approval required
        strategy:
          runOnce:
            preDeploy:
              steps:
                - script: |
                    # Create backup
                    pac solution clone --environment $(ProdEnvUrl) \
                      --name $(SolutionName) \
                      --output backup-$(Build.BuildId).zip
                  displayName: 'Backup Current Version'
            
            deploy:
              steps:
                - task: PowerPlatformImportSolution@2
                  inputs:
                    environment: '$(ProdEnvironmentUrl)'
                    solutionInputFile: '$(SolutionName)_managed.zip'
            
            routeTraffic:
              steps:
                - script: |
                    # Monitor for 10 minutes
                    python monitor_deployment.py --duration 600
                  displayName: 'Monitor Deployment'
            
            postRouteTraffic:
              steps:
                - script: |
                    # Automated rollback if error rate > 5%
                    python check_health.py --threshold 0.05 || \
                      pac solution import --path backup-$(Build.BuildId).zip
                  displayName: 'Health Check & Auto-Rollback'
```

**Quality Gates:**
1. **Build:** Solution checker passes
2. **Test:** Quality evals >80%, performance <3sec
3. **UAT:** Smoke tests pass
4. **Production:** Manual approval + health check
</details>

---

### Scenario 3: Security Incident

**Context:**
Your security team alerts you that an AI agent accessed unauthorized customer records. Investigation shows:
- Agent queried 500 customer records
- User's role should only access 50 customers (their territory)
- No code changes in past 30 days

**Questions:**

**3.1** What security controls failed and how do you prevent this?

<details>
<summary>Answer</summary>

**Failed Controls:**

1. **Authorization bypass:** Agent didn't respect user's data access permissions
2. **Missing row-level security:** Database allowed queries beyond user's scope
3. **Insufficient audit logging:** Didn't alert on unusual access patterns

**Root Cause Analysis:**

**Likely Issue:** Agent uses a service account with elevated privileges instead of user context

```
Current (WRONG):
User → Agent → Service Account (full access) → Database
                     ↓
              All 500 records returned

Correct:
User → Agent (with user identity) → Database
                     ↓
         Only user's 50 records returned (RLS applied)
```

**Prevention Measures:**

**1. Implement User Context Propagation:**
```python
# Pass user identity to database queries
def get_customer_data(user_id, user_token):
    # Use user's token for database access
    conn = connect_with_user_context(user_token)
    
    # Database applies RLS based on user identity
    query = "SELECT * FROM Customers"  # RLS filters automatically
    results = conn.execute(query)
    return results
```

**2. Row-Level Security in Dataverse:**
```xml
<securityrole name="Sales Rep">
  <privilege name="prvReadAccount" level="User" />
  <!-- Can only read accounts they own -->
</securityrole>
```

**3. Azure SQL RLS:**
```sql
CREATE FUNCTION fn_territoryAccess(@UserId int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN (
  SELECT 1 AS accessResult
  FROM dbo.Customers c
  INNER JOIN dbo.UserTerritories ut ON c.Territory = ut.Territory
  WHERE ut.UserId = @UserId
)

CREATE SECURITY POLICY TerritoryAccessPolicy
ADD FILTER PREDICATE fn_territoryAccess(UserId)
ON dbo.Customers
```

**4. Enhanced Monitoring:**
```python
# Alert on anomalous access patterns
def monitor_data_access(user_id, records_accessed):
    # Get user's normal access pattern
    avg_access = get_avg_daily_access(user_id)
    
    # Alert if 3x normal
    if records_accessed > avg_access * 3:
        send_alert(
            f"User {user_id} accessed {records_accessed} records "
            f"(normal: {avg_access})"
        )
        
        # Temporary access suspension
        suspend_user_access(user_id, duration=minutes(30))
```

**5. Regular Access Reviews:**
- Quarterly review of service account permissions
- Automated detection of overprivileged accounts
- Principle of least privilege enforcement
</details>

**3.2** Design an incident response plan for AI security issues.

<details>
<summary>Answer</summary>

**AI Security Incident Response Plan**

**Phase 1: Detection (0-15 minutes)**
```
Trigger: Security alert or user report
├── Automated monitoring detects anomaly
├── SOC team notified
└── Incident ticket created (P1 severity)

Actions:
1. Verify incident is real (not false positive)
2. Document: What, when, who, how many records
3. Notify: Security lead, AI system owner, Legal
4. Preserve: Logs, audit trail, system state
```

**Phase 2: Containment (15-30 minutes)**
```
Immediate:
├── Disable affected user account
├── Rotate service account credentials
├── Enable additional logging
└── Consider: Disable AI agent temporarily (if widespread)

Investigation:
├── Review audit logs (who accessed what)
├── Check: Recent deployments, config changes
├── Identify: Scope of unauthorized access
└── Determine: Root cause (authz bug, compromised account, etc.)
```

**Phase 3: Eradication (30 minutes - 2 hours)**
```
Fix the vulnerability:
├── If code bug: Deploy hotfix with RLS enforcement
├── If config issue: Correct permissions immediately
├── If compromised credentials: Rotate all secrets
└── If design flaw: Implement proper access controls

Verification:
├── Test fix in isolated environment
├── Verify RLS working correctly
├── Confirm no backdoor access remains
└── Security team approval before restore
```

**Phase 4: Recovery (2-4 hours)**
```
Restore service:
├── Deploy fix to production
├── Re-enable agent with enhanced monitoring
├── Notify users of resolution
└── Document all changes

Enhanced monitoring:
├── Increased audit logging for 30 days
├── Daily access pattern reviews
├── Weekly security reports
```

**Phase 5: Post-Incident (1-2 weeks)**
```
Activities:
├── Root cause analysis report
├── Update security controls
├── Compliance notification (if required)
│   ├── GDPR: Notify within 72 hours if personal data breach
│   ├── HIPAA: Notify within 60 days
│   └── SOC 2: Document in next audit
├── User communication (if personal data exposed)
├── Lessons learned session
└── Update incident playbooks

Process improvements:
├── Add missing security controls
├── Update deployment pipeline with security tests
├── Enhance monitoring rules
└── Team training on secure AI development
```

**Incident Severity Classification:**

| Severity | Definition | Response Time | Notification |
|----------|------------|---------------|--------------|
| P0 | Active breach, large-scale unauthorized access | Immediate | Exec team, Legal, PR |
| P1 | Confirmed unauthorized access, limited scope | <15 min | Security lead, System owner |
| P2 | Potential vulnerability, no confirmed breach | <1 hour | Security team |
| P3 | Security concern, low risk | <4 hours | Development team |

**Communication Template:**
```
TO: Affected Users
SUBJECT: Security Notification - Data Access Incident

We are writing to inform you of a security incident involving 
our AI customer service agent.

WHAT HAPPENED:
On [date], our AI agent accessed customer records beyond the 
authorized scope due to [root cause].

RECORDS AFFECTED:
[Number] customer records, including [data types].

ACTIONS TAKEN:
- Immediate containment on [date]
- Root cause fixed on [date]
- Enhanced security controls implemented
- Ongoing monitoring for 30 days

YOUR DATA:
[Specific info about their data if applicable]

WHAT YOU SHOULD DO:
[Any required actions]

QUESTIONS:
Contact security@company.com

We apologize for this incident and have taken steps to prevent 
recurrence.
```
</details>

---

## Hands-On Lab Exercises

### Lab 1: Set Up Monitoring
**Objective:** Configure Application Insights for Azure OpenAI API

**Steps:**
1. Create Application Insights resource
2. Instrument Python app with OpenTelemetry
3. Create custom metrics for token usage
4. Build dashboard with latency, cost, error rate
5. Configure alerts

**Expected Time:** 60 minutes

---

### Lab 2: Implement Quality Testing
**Objective:** Create automated quality evaluation pipeline

**Steps:**
1. Create test dataset (20 questions with reference answers)
2. Write Python script using Azure AI evaluation SDK
3. Calculate groundedness and relevance scores
4. Integrate into CI/CD pipeline
5. Set quality gate threshold (>80%)

**Expected Time:** 90 minutes

---

### Lab 3: Blue-Green Deployment
**Objective:** Set up blue-green deployment for AI model

**Steps:**
1. Deploy model v1 to "blue" Azure Container Instance
2. Deploy model v2 to "green" Azure Container Instance
3. Configure Azure Front Door for traffic splitting
4. Implement health checks
5. Practice traffic switching and rollback

**Expected Time:** 120 minutes

---

## Progress Tracker

**Multiple Choice Questions:**
- [ ] Monitor and Tune (4 questions): ___/4
- [ ] Testing (3 questions): ___/3
- [ ] ALM (3 questions): ___/3
- [ ] Security and Governance (3 questions): ___/3
- **Total: ___/13**

**Scenarios:**
- [ ] Scenario 1: Performance Degradation
- [ ] Scenario 2: Deployment Pipeline
- [ ] Scenario 3: Security Incident

**Hands-On Labs:**
- [ ] Lab 1: Set Up Monitoring
- [ ] Lab 2: Implement Quality Testing
- [ ] Lab 3: Blue-Green Deployment

**Target:** 80% correct (11/13 minimum) + all 3 scenarios + 2/3 labs

---

**Back to:** [Practice Resources Index](../PRACTICE_README.md)
