# Orchestrate Prebuilt Agents in Business Applications

## Overview
This section covers how to orchestrate and configure prebuilt AI agents across Microsoft's business application ecosystem, including Dynamics 365, Microsoft 365 Copilot for Sales/Service, and Power Platform AI features.

---

## 1. Orchestrate AI in Dynamics 365

### 1.1 Dynamics 365 Finance AI Capabilities

#### Financial Intelligence Features

**Cash Flow Forecasting:**
```
Purpose: Predict future cash positions based on historical data and planned transactions

Configuration:
1. Navigate to Cash and bank management > Setup > Cash flow forecasting setup
2. Configure historical data parameters:
   - Historical period: 12-24 months
   - Forecast period: 30-180 days
   - Update frequency: Daily/Weekly
3. Select prediction model:
   - Azure Machine Learning (custom)
   - Built-in statistical model
4. Define account categories for analysis
5. Enable automatic forecast updates
```

**Key Configuration Elements:**
```json
{
  "forecastSetup": {
    "historicalMonths": 18,
    "forecastDays": 90,
    "updateFrequency": "Daily",
    "confidenceInterval": 0.95,
    "dataSourcesIncluded": [
      "AccountsReceivable",
      "AccountsPayable",
      "PurchaseOrders",
      "SalesOrders",
      "BankTransactions"
    ],
    "exclusions": [
      "AnomalousTransactions",
      "OneTimeEvents"
    ]
  }
}
```

**Orchestration Workflow:**
```
Daily Schedule:
1. Extract historical transaction data (T-18 months to present)
2. Apply data quality filters (remove outliers)
3. Run ML prediction model
4. Generate forecast scenarios (best/likely/worst case)
5. Update Power BI dashboards
6. Send alerts if cash position < threshold
7. Log predictions for accuracy tracking
```

**Payment Predictions:**
```
Purpose: Predict customer payment timing and risk

Configuration Steps:
1. Finance > Credit and collections > Setup > Payment predictions
2. Enable feature in Feature management
3. Configure training data:
   - Minimum: 100 invoices with payment history
   - Recommended: 1000+ invoices
4. Select features for prediction:
   - Customer payment history
   - Invoice amount and terms
   - Industry and region
   - Economic indicators
5. Set thresholds:
   - On-time probability threshold (e.g., >80%)
   - Late probability threshold (e.g., >40%)
6. Configure actions:
   - Auto-generate collection reminders
   - Adjust credit limits
   - Flag for manual review
```

**Integration Pattern:**
```
Customer Invoice → Payment Prediction Agent → Risk Score
                                                    ↓
                            ┌───────────────────────┼────────────────────┐
                            ↓                       ↓                    ↓
                    High Risk (>60%)        Medium Risk (30-60%)   Low Risk (<30%)
                            ↓                       ↓                    ↓
                  Collection workflow       Monitor closely       Standard process
```

**Budget Proposals:**
```
AI-Assisted Budget Creation:
1. Analyzes historical spending patterns
2. Incorporates growth/reduction targets
3. Considers seasonal variations
4. Suggests allocations by category
5. Flags anomalies vs. historical trends
6. Provides justification narratives

Configuration:
- General ledger > Budgeting > Budget planning
- Enable AI suggestions
- Define budget cycle and approval workflow
- Configure allocation rules
- Set variance thresholds for alerts
```

#### Finance Orchestration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Dynamics 365 Finance                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Cash Flow    │  │  Payment     │  │   Budget     │      │
│  │  Forecast    │  │ Predictions  │  │  Proposals   │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │               │
│         └──────────────────┼──────────────────┘               │
│                            ↓                                  │
│                  ┌─────────────────────┐                     │
│                  │   AI Orchestration   │                     │
│                  │       Layer          │                     │
│                  └─────────┬───────────┘                     │
│                            ↓                                  │
│         ┌──────────────────┼──────────────────┐              │
│         ↓                  ↓                  ↓               │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐          │
│  │ Dataverse│      │Azure ML  │      │  Power    │          │
│  │          │      │ Models   │      │ Automate  │          │
│  └──────────┘      └──────────┘      └──────────┘          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Dynamics 365 Supply Chain Management AI

#### Demand Forecasting

**AI-Powered Demand Prediction:**
```
Purpose: Forecast product demand considering multiple variables

Configuration Path:
Master planning > Setup > Demand forecasting > Demand forecasting parameters

Key Settings:
1. Forecasting algorithm:
   - Azure Machine Learning (recommended)
   - Statistical baseline (fallback)
2. Historical data period: 12-36 months
3. Forecast granularity:
   - Product/Product family
   - Location/Warehouse
   - Customer/Customer group
   - Time buckets (day/week/month)
4. External factors:
   - Promotions and marketing events
   - Economic indicators
   - Seasonality patterns
   - Weather data (if applicable)
5. Accuracy tracking:
   - MAPE (Mean Absolute Percentage Error) target
   - Bias detection
   - Forecast vs. actual comparison
```

**Orchestration Workflow:**
```python
# Demand Forecasting Orchestration Example
class DemandForecastOrchestrator:
    def generate_forecast(self):
        # Step 1: Gather historical data
        historical_data = self.extract_sales_history(
            months=24,
            products=self.get_active_products()
        )
        
        # Step 2: Incorporate external factors
        external_data = {
            'promotions': self.get_planned_promotions(),
            'seasonality': self.calculate_seasonality_index(),
            'economic_indicators': self.fetch_economic_data()
        }
        
        # Step 3: Run ML model
        forecast = self.azure_ml_service.predict(
            historical=historical_data,
            external=external_data,
            horizon_days=90
        )
        
        # Step 4: Apply business rules
        adjusted_forecast = self.apply_constraints(
            forecast=forecast,
            min_order_qty=self.min_order_quantities,
            max_warehouse_capacity=self.capacity_limits
        )
        
        # Step 5: Generate planning suggestions
        recommendations = self.generate_replenishment_plan(
            forecast=adjusted_forecast,
            current_inventory=self.get_inventory_levels(),
            lead_times=self.get_supplier_lead_times()
        )
        
        # Step 6: Store and distribute
        self.save_forecast(adjusted_forecast)
        self.trigger_planning_workflows(recommendations)
        
        return adjusted_forecast
```

**Integration with Master Planning:**
```
Demand Forecast → Supply Plan Generation → Purchase Requisitions
                                    ↓
                          Material Requirements Planning (MRP)
                                    ↓
                ┌───────────────────┼───────────────────┐
                ↓                   ↓                   ↓
          Make orders          Buy orders          Transfer orders
```

#### Inventory Optimization

**AI-Driven Safety Stock Calculation:**
```
Configuration:
1. Inventory management > Setup > Inventory optimization
2. Enable AI-based safety stock
3. Configure service level targets (e.g., 95% in-stock)
4. Define cost parameters:
   - Holding cost per unit per day
   - Stockout cost (lost sales + expediting)
5. Set review frequency (daily/weekly)
6. Configure alerts for parameter changes

Algorithm Considerations:
- Demand variability (standard deviation)
- Lead time variability
- Target service level
- Replenishment lead time
- Order frequency
```

**Predictive Maintenance:**
```
Purpose: Predict equipment failures to schedule proactive maintenance

Data Sources:
- IoT sensor data (temperature, vibration, pressure)
- Maintenance history
- Equipment specifications
- Environmental conditions

Orchestration:
1. IoT Hub receives telemetry → Azure Stream Analytics
2. Real-time anomaly detection
3. Predictive model scores failure probability
4. If risk > threshold → Create work order in D365
5. Optimize maintenance scheduling considering:
   - Production schedule impact
   - Technician availability
   - Parts inventory
   - Criticality of equipment
```

#### Supply Chain Orchestration Pattern

```yaml
# Supply Chain AI Orchestration Configuration
name: SupplyChainAIOrchestrator
triggers:
  - schedule: "0 2 * * *"  # Daily at 2 AM
  - event: "InventoryLevelCritical"
  - event: "NewSalesOrderCreated"

workflow:
  - step: demand_forecasting
    agent: DemandForecastAgent
    inputs:
      - sales_history
      - promotions
      - seasonality
    outputs:
      - forecast_by_product
      - confidence_intervals
  
  - step: inventory_optimization
    agent: InventoryOptimizationAgent
    inputs:
      - demand_forecast
      - current_inventory
      - supplier_lead_times
    outputs:
      - recommended_orders
      - safety_stock_levels
  
  - step: supplier_selection
    agent: SupplierScoringAgent
    inputs:
      - recommended_orders
      - supplier_performance_history
      - current_pricing
    outputs:
      - optimal_supplier_allocation
  
  - step: purchase_requisition_creation
    agent: ProcurementAgent
    inputs:
      - optimal_supplier_allocation
      - approval_workflows
    outputs:
      - purchase_requisitions
      - approval_requests
  
  - step: monitoring
    agent: MonitoringAgent
    actions:
      - track_forecast_accuracy
      - alert_on_exceptions
      - update_dashboards
```

### 1.3 Dynamics 365 Customer Experience AI

#### Customer Insights and Segmentation

**Unified Customer Profile:**
```
Purpose: Create 360-degree customer view by unifying data from multiple sources

Configuration (Customer Insights):
1. Navigate to Dynamics 365 Customer Insights
2. Data sources setup:
   - Connect CRM (contacts, accounts, interactions)
   - Connect ERP (orders, invoices, returns)
   - Connect marketing platform (campaigns, responses)
   - Connect support system (cases, satisfaction scores)
   - Connect web analytics (site visits, cart abandonment)
3. Data unification process:
   - Map entities from each source
   - Define match rules (email, phone, customer ID)
   - Configure merge policies for conflicting data
   - Create unified Customer table
4. AI enrichment:
   - Sentiment analysis on interaction text
   - Propensity scores (churn, purchase, upsell)
   - CLV (Customer Lifetime Value) prediction
   - Next best action recommendations
```

**Segmentation with AI:**
```
Segment Types:
1. Demographic: Age, location, industry, company size
2. Behavioral: Purchase frequency, channel preference, engagement level
3. Predictive: Churn risk, propensity to buy, response likelihood
4. Value-based: CLV tiers, profitability, growth potential

AI-Suggested Segments:
- Automatically discover customer clusters
- Identify high-value micro-segments
- Suggest expansion opportunities
- Flag at-risk segments

Example Configuration:
{
  "segmentName": "High-Value At-Risk",
  "criteria": {
    "and": [
      { "CLV": { "greaterThan": 10000 } },
      { "ChurnRisk": { "greaterThan": 0.6 } },
      { "DaysSinceLastPurchase": { "greaterThan": 180 } }
    ]
  },
  "aiEnrichment": {
    "recommendedActions": true,
    "personalizedOffers": true,
    "outreachTiming": "optimal"
  }
}
```

#### Marketing Automation AI

**Journey Orchestration:**
```
Purpose: Optimize customer journeys with AI-driven personalization

AI Capabilities:
1. Channel optimization: Select best channel per customer (email, SMS, push)
2. Send time optimization: Predict when customer most likely to engage
3. Content selection: Choose most relevant message variant
4. Next best action: Recommend optimal next touchpoint

Journey Configuration Example:
┌─────────────────────────────────────────────────────────┐
│              Customer Journey: Onboarding               │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Trigger: New Customer Created                          │
│      ↓                                                   │
│  [AI: Determine Best Send Time]                         │
│      ↓                                                   │
│  Send Welcome Email (Variant A/B/C chosen by AI)        │
│      ↓                                                   │
│  Wait 3 days                                            │
│      ↓                                                   │
│  [AI: Check Engagement Score]                           │
│      ↓                                                   │
│  ┌──────────────┬────────────────────┐                 │
│  ↓              ↓                    ↓                  │
│ Engaged    Low Engagement    No Engagement             │
│  ↓              ↓                    ↓                  │
│ Product     [AI: Best      Alternative channel         │
│  Demo      persuasion       (SMS/Call)                 │
│ Offer      approach]                                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Lead Scoring:**
```
AI-Powered Lead Scoring:

Factors Considered:
- Demographic fit (company size, industry, role)
- Behavioral signals (website visits, content downloads, email opens)
- Firmographic data (revenue, growth, technology stack)
- Engagement recency and frequency
- Similar-customer conversion patterns

Configuration:
1. Marketing > Lead scoring models
2. Select features for model training
3. Define conversion event (e.g., Opportunity Created)
4. Train model on historical data (min 500 converted leads)
5. Set score thresholds:
   - Hot Lead: >80
   - Warm Lead: 50-80
   - Cold Lead: <50
6. Configure auto-actions:
   - Hot → Auto-assign to sales rep + immediate notification
   - Warm → Add to nurture campaign
   - Cold → Long-term drip campaign

Real-time Scoring:
- Scores update as new data arrives
- Trigger workflows on score changes
- Integrate with sales queue prioritization
```

#### Service AI Orchestration

**Case Routing with AI:**
```
Purpose: Automatically route support cases to optimal agent/queue

Configuration:
1. Customer Service > Settings > AI-based case routing
2. Define routing criteria:
   - Case topic/category
   - Customer priority/tier
   - Agent skills and availability
   - Historical resolution patterns
   - SLA urgency
3. Train routing model:
   - Historical case data (min 1000 cases)
   - Case-to-agent assignments
   - Resolution outcomes (time, quality, satisfaction)
4. Configure business rules:
   - VIP customer override
   - Language matching
   - Workload balancing
5. Enable automatic assignment

Orchestration Logic:
```

```python
class CaseRoutingOrchestrator:
    def route_case(self, case):
        # Step 1: AI predicts case attributes
        predictions = self.ai_model.predict({
            'case_description': case.description,
            'customer_history': case.customer.past_cases,
            'product': case.product
        })
        
        predicted_category = predictions['category']
        predicted_complexity = predictions['complexity']
        predicted_resolution_time = predictions['estimated_hours']
        
        # Step 2: Find qualified agents
        qualified_agents = self.get_agents_with_skill(predicted_category)
        
        # Step 3: Score agents
        agent_scores = []
        for agent in qualified_agents:
            score = self.calculate_agent_score(
                agent=agent,
                case_complexity=predicted_complexity,
                current_workload=agent.current_cases,
                past_performance=agent.resolution_metrics,
                availability=agent.schedule
            )
            agent_scores.append((agent, score))
        
        # Step 4: Select best agent
        best_agent = max(agent_scores, key=lambda x: x[1])[0]
        
        # Step 5: Assign and notify
        self.assign_case(case, best_agent)
        self.send_notification(best_agent, case, predicted_resolution_time)
        
        return best_agent
```

**Knowledge Article Suggestions:**
```
AI-Powered Knowledge Recommendations:

Context: Agent working on case or customer using self-service portal

Process:
1. Analyze case description/customer question (NLP)
2. Extract key concepts and entities
3. Search knowledge base with semantic similarity
4. Rank articles by relevance
5. Consider:
   - Product/version match
   - Historical usefulness (thumbs up/down)
   - Recency of article
   - User role/permissions
6. Present top 3-5 articles in sidebar

Configuration:
- Customer Service > Knowledge management > AI suggestions
- Enable knowledge article suggestions
- Configure indexing frequency
- Set relevance threshold (min 0.7 similarity)
- Enable feedback collection for continuous improvement
```

---

## 2. Configure Microsoft 365 Copilot for Sales and Service

### 2.1 Microsoft 365 Copilot for Sales

#### Setup and Configuration

**Prerequisites:**
```
Licenses Required:
- Microsoft 365 Copilot license
- Dynamics 365 Sales (or Salesforce)
- Teams
- Outlook

Admin Setup:
1. Microsoft 365 admin center
2. Setup > Copilot for Sales
3. Connect CRM:
   - Select Dynamics 365 or Salesforce
   - Authenticate with admin credentials
   - Map standard objects (Account, Contact, Opportunity, Lead)
4. Configure Teams integration
5. Enable Outlook add-in
6. Set up permissions and security groups
```

**CRM Connection Configuration:**
```json
{
  "crmConnection": {
    "platform": "Dynamics365",
    "environment": "https://contoso.crm.dynamics.com",
    "authentication": "OAuth2",
    "syncFrequency": "realtime",
    "objectsEnabled": [
      {
        "object": "Account",
        "fields": ["name", "industry", "revenue", "owner"],
        "bidirectional": true
      },
      {
        "object": "Contact",
        "fields": ["firstname", "lastname", "email", "jobtitle"],
        "bidirectional": true
      },
      {
        "object": "Opportunity",
        "fields": ["name", "estimatedvalue", "closeprobability", "estimatedclosedate"],
        "bidirectional": true
      },
      {
        "object": "Lead",
        "fields": ["subject", "leadsource", "rating"],
        "bidirectional": false
      }
    ],
    "activityTracking": {
      "emails": true,
      "meetings": true,
      "calls": true,
      "automaticSync": true
    }
  }
}
```

#### Sales Orchestration Capabilities

**Email Intelligence:**
```
Copilot in Outlook for Sales:

Features:
1. CRM record identification:
   - Detects customer contacts in email
   - Shows relevant Account/Opportunity context
   - Displays recent interactions timeline

2. AI-generated email summaries:
   - Key discussion points
   - Action items
   - Commitments made
   - Sentiment analysis

3. Suggested responses:
   - Context-aware reply suggestions
   - Tone matching (formal/casual)
   - Product information insertion
   - Pricing and availability lookup

4. Follow-up reminders:
   - AI detects promised follow-ups
   - Creates tasks automatically
   - Suggests optimal follow-up timing

Configuration:
- Settings > Copilot for Sales > Email intelligence
- Enable automatic record matching
- Configure summary generation (on-demand vs automatic)
- Set up suggested content approval workflow
```

**Meeting Intelligence:**
```
Copilot in Teams for Sales:

Before Meeting:
- Pulls up relevant CRM records
- Summarizes recent interactions
- Highlights key talking points
- Shows open opportunities/deals
- Suggests questions to ask

During Meeting:
- Real-time transcription
- Live sentiment analysis
- Action item capture
- Key topic detection
- Competitor mention alerts

After Meeting:
- Auto-generated summary
- Identified action items → Tasks
- Updates CRM opportunity stage if discussed
- Suggests follow-up actions
- Analyzes meeting effectiveness

Configuration Example:
{
  "meetingIntelligence": {
    "preCallBrief": {
      "enabled": true,
      "includeOpenOpportunities": true,
      "includeRecentEmails": true,
      "includeSocialInsights": true
    },
    "realTimeAssist": {
      "transcription": true,
      "sentimentTracking": true,
      "competitorAlerts": ["Competitor A", "Competitor B"],
      "battleCards": true
    },
    "postMeetingActions": {
      "autoSaveToCRM": true,
      "suggestStageChange": true,
      "createFollowUpTasks": true,
      "updateOpportunityNotes": true
    }
  }
}
```

**Sales Data Grounding:**
```
Configuring Data Sources for Copilot:

1. CRM Data:
   - Accounts, Contacts, Leads, Opportunities (automatic)
   - Custom entities (configure mapping)

2. Product Catalog:
   - Product names, descriptions, pricing
   - Inventory levels (if integrated)
   - Sales collateral and datasheets

3. Competitive Intelligence:
   - Competitor profiles
   - Battle cards
   - Win/loss analysis

4. Sales Playbooks:
   - Stage-specific guidance
   - Objection handling scripts
   - Discovery question templates

Configuration Path:
- Admin center > Copilot for Sales > Data sources
- Enable each source
- Configure refresh frequency
- Set permission boundaries (all users vs specific roles)
- Test with sample queries
```

**Opportunity Summaries:**
```
AI-Generated Opportunity Insights:

Copilot generates:
1. Deal summary:
   - Current stage and health score
   - Key stakeholders and their roles
   - Timeline of interactions
   - Products/solutions in scope

2. Risk assessment:
   - Competitors involved
   - Long sales cycle alert
   - Low engagement warning
   - Budget/authority concerns

3. Recommended actions:
   - "Schedule executive alignment meeting"
   - "Share case study from similar customer"
   - "Address pricing concerns raised in last email"
   - "Engage solution architect for technical validation"

4. Competitive positioning:
   - Where you're winning vs competitors
   - Differentiators to emphasize
   - Competitor weaknesses to exploit

Access:
- Dynamics 365 Sales: Copilot pane in Opportunity form
- Outlook: Copilot panel when viewing email from stakeholder
- Teams: "/copilot summarize opportunity [name]"
```

### 2.2 Microsoft 365 Copilot for Service

#### Setup and Configuration

**Prerequisites:**
```
Licenses:
- Microsoft 365 Copilot
- Dynamics 365 Customer Service
- Teams (for collaboration scenarios)

Configuration Steps:
1. Admin center > Copilot for Service
2. Connect Customer Service environment
3. Configure knowledge sources:
   - Internal knowledge base
   - SharePoint sites
   - External websites (approved domains)
4. Set up case summarization
5. Enable email assistance features
6. Configure generative answers settings
```

**Knowledge Source Configuration:**
```yaml
knowledgeSources:
  - name: InternalKnowledgeBase
    type: Dynamics365Knowledge
    enabled: true
    indexing:
      frequency: hourly
      fullCrawl: weekly
    filters:
      status: Published
      expirationDate: future
    
  - name: ProductDocumentation
    type: SharePointSite
    url: https://contoso.sharepoint.com/sites/ProductDocs
    enabled: true
    security:
      respectSharePointPermissions: true
    indexing:
      includeFileTypes: [".docx", ".pdf", ".html"]
      excludeFolders: ["/Archive", "/Drafts"]
  
  - name: SupportFAQ
    type: Website
    url: https://support.contoso.com/faq
    enabled: true
    crawlDepth: 3
    updateFrequency: daily
    
  - name: CommunityForum
    type: ExternalWebsite
    url: https://community.contoso.com
    enabled: false  # Can be enabled after moderation setup
```

#### Service Orchestration Capabilities

**Case Summarization:**
```
Purpose: Quickly understand case context and history

Copilot generates:
1. Issue summary:
   - Customer's primary problem (in plain language)
   - Product/service affected
   - Impact and urgency

2. Interaction timeline:
   - Chronological summary of conversations
   - Channels used (email, chat, phone)
   - Agents involved

3. Resolution attempts:
   - Troubleshooting steps already taken
   - Knowledge articles shared
   - Workarounds provided

4. Current status:
   - Escalation state
   - Open tasks/pending actions
   - Time to SLA breach

5. Suggested next steps:
   - "Check if recent product update resolves issue"
   - "Escalate to engineering for bug investigation"
   - "Offer temporary workaround while awaiting patch"

Configuration:
{
  "caseSummarization": {
    "autoGenerateOnOpen": true,
    "includeCustomerSentiment": true,
    "highlightSLARisks": true,
    "suggestNextSteps": true,
    "maxSummaryLength": 500,
    "includeRelatedCases": true
  }
}
```

**Draft Responses:**
```
AI-Assisted Response Generation:

Context: Agent is responding to customer inquiry

Process:
1. Agent clicks "Draft with Copilot" in email reply
2. Copilot analyzes:
   - Customer's question/issue
   - Case history and context
   - Knowledge articles related to topic
   - Company tone and style guidelines
3. Generates draft response:
   - Acknowledges customer's concern
   - Provides relevant information/solution
   - Offers additional resources
   - Appropriate tone (empathetic, professional)
4. Agent reviews, edits, and sends

Prompt Examples:
- "Draft a response explaining the refund process"
- "Write an apology for the service outage and explain resolution"
- "Explain the difference between Product A and Product B"
- "Provide troubleshooting steps for error code 500"

Configuration:
- Service > Copilot settings > Response generation
- Upload style guide documents
- Configure approved tone (formal, friendly, technical)
- Set up approval workflow for sensitive topics
- Enable/disable AI content watermarking
```

**Knowledge Article Suggestions:**
```
Real-Time Knowledge Recommendations:

As agent types case notes or reads customer message:
1. Copilot performs semantic search on knowledge base
2. Surfaces relevant articles in sidebar
3. Shows confidence score for each suggestion
4. Allows quick insert into response

Configuration:
{
  "knowledgeSuggestions": {
    "triggerMode": "automatic",  // or "onDemand"
    "minimumConfidence": 0.75,
    "maxSuggestions": 5,
    "rankingFactors": [
      "semanticSimilarity",
      "historicalUsefulness",
      "articleFreshness",
      "customerProductVersion"
    ],
    "displayFormat": "sidePanel",
    "allowDirectInsert": true
  }
}
```

**Conversation Summaries:**
```
Summarizing Multi-Turn Conversations:

For live chat or phone transcripts:
1. Copilot summarizes each conversation segment
2. Identifies key information:
   - Customer problem statement
   - Error messages or symptoms
   - Product details (version, configuration)
   - Troubleshooting performed
   - Resolution or next steps
3. Extracts structured data:
   - Product name → Case product field
   - Error code → Case subject/description
   - Callback number → Contact phone

Benefits:
- Reduces post-conversation documentation time
- Ensures consistent case notes
- Improves handoff to next agent
- Enhances knowledge creation from conversations

Configuration Example:
```

```json
{
  "conversationSummary": {
    "channels": ["chat", "phoneTranscript", "email"],
    "autoGenerateSummary": {
      "afterConversationEnd": true,
      "duringLongConversations": {
        "enabled": true,
        "messageThreshold": 20
      }
    },
    "structuredDataExtraction": {
      "enabled": true,
      "fieldsToExtract": [
        {"field": "productName", "crmField": "product"},
        {"field": "errorCode", "crmField": "subject"},
        {"field": "symptomDescription", "crmField": "description"},
        {"field": "customerEnvironment", "crmField": "customerenvironment"}
      ]
    },
    "languageSupport": ["en", "es", "fr", "de", "ja"],
    "customInstructions": "Focus on technical details and customer sentiment"
  }
}
```

---

## 3. Design AI Features in Power Platform

### 3.1 AI Builder Overview

**AI Builder Capabilities:**

```
┌───────────────────────────────────────────────────────────┐
│                      AI Builder Models                     │
├───────────────────────────────────────────────────────────┤
│                                                             │
│  Prebuilt Models (no training required):                   │
│  ├─ Text Recognition (OCR)                                 │
│  ├─ Receipt Processing                                     │
│  ├─ Business Card Reader                                   │
│  ├─ Invoice Processing                                     │
│  ├─ ID Document Reader                                     │
│  ├─ Language Detection                                     │
│  ├─ Key Phrase Extraction                                  │
│  ├─ Sentiment Analysis                                     │
│  ├─ Entity Extraction                                      │
│  └─ Category Classification                                │
│                                                             │
│  Custom Models (require training):                         │
│  ├─ Object Detection                                       │
│  ├─ Document Processing (custom forms)                     │
│  ├─ Text Classification                                    │
│  └─ Prediction Models                                      │
│                                                             │
│  Generative AI (prompt-based):                             │
│  ├─ Custom Prompts (GPT)                                   │
│  └─ Generative Answers in Copilot Studio                   │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

### 3.2 Prebuilt AI Models in Power Platform

#### Document Processing Use Case

**Invoice Processing Scenario:**
```
Business Need: Extract data from supplier invoices automatically

Power Automate Flow:
1. Trigger: When email arrives with attachment (Office 365 Outlook)
2. Condition: If subject contains "Invoice" OR attachment is PDF
3. AI Builder: Extract information from invoice
   - Model: Invoice Processing (prebuilt)
   - Input: Email attachment
4. Parse JSON: Structure extracted data
5. Create record in Dataverse:
   - Entity: Invoices
   - Fields mapped from AI output:
     * Vendor Name → vendorname
     * Invoice Number → invoicenumber
     * Invoice Date → invoicedate
     * Total Amount → totalamount
     * Line Items → related entity
6. Condition: If total > $10,000
   - Yes: Create approval request
   - No: Auto-approve
7. Send confirmation email

AI Builder Output Structure:
{
  "vendorName": "Acme Supplies Inc.",
  "vendorAddress": "123 Main St, City, State 12345",
  "invoiceNumber": "INV-2025-001234",
  "invoiceDate": "2025-11-01",
  "dueDate": "2025-11-30",
  "totalAmount": 15750.50,
  "taxAmount": 1312.54,
  "lineItems": [
    {
      "description": "Widget Type A",
      "quantity": 100,
      "unitPrice": 125.00,
      "amount": 12500.00
    },
    {
      "description": "Widget Type B",
      "quantity": 50,
      "unitPrice": 65.01,
      "amount": 3250.50
    }
  ]
}
```

**Receipt Processing in Power Apps:**
```
Use Case: Expense reporting mobile app

Configuration:
1. Create Canvas App in Power Apps
2. Add Camera control
3. Add Button "Scan Receipt"
4. On button press:
   - Capture image from camera
   - Send to AI Builder Receipt Processing model
   - Display extracted data in form:
     * Merchant name
     * Transaction date
     * Total amount
     * Tax amount
     * Receipt items
5. Allow user to review and edit
6. Submit to Dataverse (Expense Reports entity)

Power Fx Formula:
ClearCollect(
    colReceiptData,
    ReceiptProcessor.Predict(
        Camera1.Photo
    )
);

Set(varMerchantName, First(colReceiptData).MerchantName);
Set(varTransactionDate, First(colReceiptData).TransactionDate);
Set(varTotalAmount, First(colReceiptData).TotalAmount);
```

#### Sentiment Analysis Use Case

**Customer Feedback Analysis:**
```
Scenario: Analyze customer feedback forms for sentiment and escalate negative feedback

Power Automate Flow:
1. Trigger: When item is created (SharePoint list: Customer Feedback)
2. AI Builder: Analyze sentiment
   - Model: Sentiment Analysis (prebuilt)
   - Text: Feedback comments field
   - Language: English
3. Parse sentiment result:
   - Overall sentiment: Positive/Neutral/Negative
   - Confidence score: 0-1
4. Update SharePoint item:
   - Set Sentiment column
   - Set Confidence column
5. Condition: If sentiment is Negative AND confidence > 0.8
   - Create high-priority case in Dynamics 365
   - Send alert to customer service manager
   - Trigger customer retention workflow
6. Condition: If sentiment is Positive AND confidence > 0.9
   - Add customer to "Brand Champions" list
   - Trigger request for review/testimonial

Configuration in Power Automate:
{
  "sentimentAnalysis": {
    "model": "AIBuilder.SentimentAnalysis",
    "inputText": "@{triggerBody()?['Comments']}",
    "language": "en",
    "thresholds": {
      "negative": {
        "confidence": 0.8,
        "action": "escalate"
      },
      "positive": {
        "confidence": 0.9,
        "action": "nurture"
      }
    }
  }
}
```

### 3.3 Custom AI Models

#### Prediction Model Example

**Customer Churn Prediction:**
```
Training Data Requirements:
- Historical customer records (min 1000 records)
- Outcome field: Churned (Yes/No)
- Features:
  * Account age (days)
  * Number of support cases (last 90 days)
  * Number of logins (last 30 days)
  * Total purchase amount (last 12 months)
  * Days since last purchase
  * Product usage metrics
  * Subscription tier

AI Builder Configuration:
1. Navigate to Power Apps > AI Builder > Build
2. Select "Prediction" model
3. Choose Dataverse table: Customers
4. Select historical outcome field: Churned
5. Choose features (columns to consider)
6. Train model (takes 2-4 hours)
7. Review model performance:
   - Accuracy score
   - Precision/Recall
   - Feature importance
8. Publish model

Using the Prediction Model:
- Power Automate: Scheduled flow runs prediction on all active customers daily
- Canvas App: Displays churn risk score on customer detail screen
- Model-driven App: Inline churn risk indicator in customer grid
```

**Power Automate Integration:**
```yaml
name: DailyChurnPrediction
trigger:
  schedule: "0 6 * * *"  # Daily at 6 AM

steps:
  - name: GetActiveCustomers
    action: Dataverse.ListRecords
    inputs:
      entity: accounts
      filter: "statuscode eq 1"  # Active accounts
      select: ["accountid", "accountage", "supportcases", "logincount", "totalspend"]
  
  - name: PredictChurn
    action: AIBuilder.Predict
    inputs:
      model: "ChurnPredictionModel"
      records: "@outputs('GetActiveCustomers')?['body/value']"
    
  - name: ProcessPredictions
    forEach: "@outputs('PredictChurn')?['predictions']"
    actions:
      - name: UpdateCustomerRisk
        action: Dataverse.UpdateRecord
        inputs:
          entity: accounts
          recordId: "@item()['accountid']"
          fields:
            churnrisk: "@item()['prediction']"
            churnprobability: "@item()['probability']"
            lastupdated: "@utcNow()"
      
      - name: CheckHighRisk
        condition: "@greater(item()['probability'], 0.75)"
        ifTrue:
          - action: Office365.SendEmail
            inputs:
              to: "accountmanager@contoso.com"
              subject: "High Churn Risk Alert: @{item()['accountname']}"
              body: "Customer @{item()['accountname']} has @{int(mul(item()['probability'], 100))}% churn probability."
          
          - action: Dataverse.CreateRecord
            entity: tasks
            inputs:
              subject: "Retention outreach for @{item()['accountname']}"
              priority: 1
              ownerid: "@item()['accountownerid']"
```

#### Custom Document Processing

**Training a Custom Form Processor:**
```
Use Case: Extract data from company-specific purchase requisition forms

Steps:
1. Prepare training documents:
   - Collect 15-50 example forms (filled out)
   - Ensure variety in handwriting, formats
   - Save as PDF or images

2. Create model in AI Builder:
   - Select "Form Processing"
   - Upload training documents
   - Define fields to extract:
     * Request ID (text)
     * Requester Name (text)
     * Department (text)
     * Request Date (date)
     * Vendor (text)
     * Line Items (table)
       - Description (text)
       - Quantity (number)
       - Unit Price (number)
       - Total (number)
     * Approval Status (text)

3. Tag documents:
   - For each training document, draw boxes around fields
   - Label each box with field name
   - For tables, mark column headers and rows

4. Train model (1-2 hours)

5. Test with sample documents:
   - Upload test document
   - Review extracted data accuracy
   - Retrain if accuracy < 90%

6. Publish and use in flows

Example Integration:
Power Automate:
- Trigger: New file in SharePoint (Purchase Requisitions folder)
- Action: AI Builder - Extract information from form
- Action: Create record in Dataverse (Purchase Requisitions table)
- Action: Start approval workflow
```

### 3.4 Generative AI in Copilot Studio

#### Custom Prompts for Agents

**Prompt Engineering in Copilot Studio:**
```
Use Case: Product recommendation agent

Topic: Recommend Product
Description: Suggests products based on customer needs

Prompt Configuration:
┌──────────────────────────────────────────────────────────┐
│ System Prompt (sets behavior):                           │
│                                                           │
│ You are a helpful product advisor for Contoso Corp.      │
│ Your goal is to recommend products that best fit the     │
│ customer's needs based on their requirements and budget. │
│ Always ask clarifying questions before making            │
│ recommendations. Be concise and friendly.                │
│                                                           │
│ Available products (loaded from Dataverse):              │
│ {{Topic.Products}}                                       │
│                                                           │
│ Customer context:                                         │
│ - Industry: {{User.Industry}}                            │
│ - Company Size: {{User.CompanySize}}                     │
│ - Past purchases: {{User.PurchaseHistory}}              │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ User Prompt (dynamic):                                    │
│                                                           │
│ Customer message: {{Activity.Text}}                      │
│                                                           │
│ Please recommend up to 3 products with:                  │
│ 1. Product name and description                          │
│ 2. Why it's a good fit                                   │
│ 3. Approximate price                                     │
│ 4. Next steps to purchase                                │
└──────────────────────────────────────────────────────────┘
```

**Prompt with Grounding Data:**
```yaml
promptConfiguration:
  name: GenerateProjectProposal
  description: Creates a project proposal based on customer requirements
  
  dataSources:
    - type: Dataverse
      table: Projects
      filter: "statuscode eq 'Completed' and industrysector eq '{User.Industry}'"
      columns: ["projectname", "description", "duration", "cost", "outcomes"]
      maxRecords: 5
    
    - type: SharePoint
      site: "https://contoso.sharepoint.com/sites/Templates"
      library: "ProposalTemplates"
      filter: "Industry eq '{User.Industry}'"
      
  systemPrompt: |
    You are a proposal writer for Contoso Consulting.
    Create a professional project proposal based on:
    - Customer requirements provided in user message
    - Similar past projects from our database
    - Proposal templates for structure and tone
    
    Include:
    1. Executive Summary
    2. Proposed Approach
    3. Timeline (estimate)
    4. Budget Range
    5. Expected Outcomes
    
    Use past project data as examples but customize for this customer.
    
  userPrompt: |
    Customer Requirements: {{Activity.Text}}
    
    Similar Projects We've Done:
    {{DataSource.Projects}}
    
    Template Structure:
    {{DataSource.ProposalTemplates}}
    
    Generate a proposal document addressing their requirements.
    
  outputFormat: markdown
  
  validationRules:
    - maxLength: 5000
    - mustInclude: ["Executive Summary", "Timeline", "Budget"]
    - tone: "professional"
```

#### Generative Answers Configuration

**Knowledge-Grounded Responses:**
```
Copilot Studio Configuration:
1. Create new Generative Answers topic
2. Configure data sources:
   
   a) Public websites:
      - URL: https://support.contoso.com
      - Crawl depth: 2 levels
      - Update frequency: Weekly
      - Authentication: None
   
   b) SharePoint:
      - Site: Product Documentation
      - Include: All document libraries
      - Exclude: Archive folders
      - Authentication: Inherit from user
   
   c) Dataverse:
      - Table: Knowledge Articles
      - Filter: Published articles only
      - Search fields: Title, Content, Keywords
   
   d) Files:
      - Upload: Product manuals (PDF)
      - FAQ documents
      - Policy documents

3. Configure moderation:
   - Enable content moderation
   - Block: Offensive language, PII exposure
   - Require: Citations for factual claims
   
4. Set guardrails:
   - Stay on topic (product support only)
   - Don't make promises (refunds, pricing changes)
   - Escalate: Legal questions, account issues
   
5. Response settings:
   - Tone: Friendly and professional
   - Length: Concise (200-300 words)
   - Citations: Always show sources
   - Fallback: "Let me connect you to a specialist"
```

**Example Generative Answers Topic:**
```json
{
  "topicName": "GenerativeAnswers_ProductSupport",
  "trigger": {
    "type": "anyMessage",
    "excludeTopics": ["Greeting", "EndConversation", "Escalate"]
  },
  "generativeAnswersConfig": {
    "dataSources": [
      {
        "id": "sharepoint_productdocs",
        "weight": 10,
        "enabled": true
      },
      {
        "id": "dataverse_knowledgearticles",
        "weight": 8,
        "enabled": true
      },
      {
        "id": "website_supportsite",
        "weight": 6,
        "enabled": true
      }
    ],
    "promptConfiguration": {
      "systemMessage": "You provide technical support for Contoso products. Use the knowledge sources to answer questions accurately. If you're unsure, say so and offer to escalate.",
      "userMessage": "{{Activity.Text}}",
      "temperature": 0.3,
      "maxTokens": 800,
      "topP": 0.9
    },
    "citationSettings": {
      "includeCitations": true,
      "citationStyle": "inline",
      "minCitations": 1
    },
    "contentModeration": {
      "enabled": true,
      "blockOffensiveContent": true,
      "blockPII": true,
      "customFilters": ["competitor_names"]
    },
    "fallbackBehavior": {
      "noAnswerMessage": "I don't have enough information to answer that confidently. Would you like me to connect you with a specialist?",
      "lowConfidenceThreshold": 0.6,
      "escalateTopic": "EscalateToAgent"
    }
  }
}
```

---

## 4. Orchestration Best Practices

### 4.1 Multi-Agent Coordination

**Agent Handoff Pattern:**
```
Scenario: Customer inquiry requires both sales and technical expertise

Flow:
1. Customer contacts Sales Copilot (via Teams)
2. Sales Copilot provides product information
3. Customer asks technical question beyond sales scope
4. Sales Copilot detects technical context
5. Handoff initiated:
   - Sales Copilot: "Let me bring in our technical specialist"
   - Context passed: Customer info, conversation history, question
6. Technical Copilot (Copilot Studio) joins conversation
7. Technical Copilot answers technical question
8. If solution requires demo:
   - Create opportunity in CRM
   - Schedule demo meeting
   - Assign to solution engineer
9. Return control to Sales Copilot for next steps
```

**Implementation:**
```yaml
# Sales Copilot Topic: Detect Technical Question
name: DetectAndEscalateTechnical
trigger:
  conditions:
    - userMessage contains ["API", "integration", "technical specs", "architecture"]
    - OR confidence(TechnicalIntent) > 0.7

actions:
  - action: CaptureContext
    variables:
      customerName: "{{User.Name}}"
      company: "{{CRM.Account.Name}}"
      conversationHistory: "{{Conversation.Transcript}}"
      technicalQuestion: "{{Activity.Text}}"
  
  - action: SendMessage
    message: "Great question! Let me bring in our technical expert who can provide detailed information."
  
  - action: InvokeAgent
    agent: TechnicalSupportCopilot
    inputs:
      context: "{{var_CaptureContext}}"
      mode: "collaborative"  # Both agents visible
    
  - action: WaitForAgentComplete
    agent: TechnicalSupportCopilot
  
  - action: ResumeControl
    message: "Thank you to our technical team. How else can I help you today?"
```

### 4.2 Data Consistency Across Agents

**Shared Context Management:**
```
Challenge: Multiple agents need access to same customer context

Solution: Centralized context service

Architecture:
┌──────────────────────────────────────────────────────┐
│           Centralized Context Service                 │
│                  (Dataverse)                          │
├──────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────────────────────────────────┐       │
│  │     Shared Context Entity                 │       │
│  ├──────────────────────────────────────────┤       │
│  │  - SessionID (unique)                     │       │
│  │  - CustomerID                             │       │
│  │  - ConversationHistory (JSON)            │       │
│  │  - CurrentIntent                          │       │
│  │  - AgentStack (which agents involved)    │       │
│  │  - Variables (key-value pairs)           │       │
│  │  - LastUpdated                            │       │
│  └──────────────────────────────────────────┘       │
│                                                        │
└────────┬─────────────┬────────────────┬──────────────┘
         ↓             ↓                ↓
    Sales Agent   Service Agent   Marketing Agent
    (reads/writes) (reads/writes)  (reads/writes)
```

**Implementation Pattern:**
```python
# Context Service API
class ConversationContextService:
    def __init__(self, dataverse_client):
        self.client = dataverse_client
    
    def get_context(self, session_id):
        """Retrieve current context for session"""
        record = self.client.retrieve(
            entity='conversationcontexts',
            id=session_id
        )
        return {
            'customer_id': record['customerid'],
            'history': json.loads(record['conversationhistory']),
            'intent': record['currentintent'],
            'variables': json.loads(record['variables'])
        }
    
    def update_context(self, session_id, updates):
        """Update context with new information"""
        current = self.get_context(session_id)
        
        # Merge updates
        current['variables'].update(updates.get('variables', {}))
        current['history'].append(updates.get('latest_exchange', {}))
        
        # Write back
        self.client.update(
            entity='conversationcontexts',
            id=session_id,
            data={
                'conversationhistory': json.dumps(current['history']),
                'variables': json.dumps(current['variables']),
                'lastupdated': datetime.utcnow().isoformat()
            }
        )
    
    def add_agent_to_stack(self, session_id, agent_name):
        """Track which agents have been involved"""
        record = self.client.retrieve('conversationcontexts', session_id)
        agent_stack = json.loads(record['agentstack'])
        agent_stack.append({
            'agent': agent_name,
            'timestamp': datetime.utcnow().isoformat()
        })
        self.client.update(
            'conversationcontexts',
            session_id,
            {'agentstack': json.dumps(agent_stack)}
        )
```

### 4.3 Performance Optimization

**Caching Strategy:**
```
Product catalog frequently accessed by sales agents

Solution: Multi-tier caching

Tier 1: In-Memory Cache (Agent runtime)
- TTL: 5 minutes
- Scope: Current conversation session
- Invalidation: On session end or explicit refresh

Tier 2: Distributed Cache (Redis/Azure Cache)
- TTL: 1 hour
- Scope: All agent instances
- Invalidation: On product catalog update event

Tier 3: Database (Dataverse)
- Source of truth
- Updated by ETL processes

Flow:
Agent needs product info
  → Check Tier 1 (in-memory)
    → If miss, check Tier 2 (Redis)
      → If miss, query Tier 3 (Dataverse)
        → Populate Tier 2
        → Populate Tier 1
  → Return to agent
```

**Parallel Data Retrieval:**
```yaml
# Optimize opportunity summary by fetching data in parallel
optimizedOpportunitySummary:
  - step: ParallelDataFetch
    parallel:
      - name: FetchOpportunityDetails
        source: Dataverse
        entity: opportunities
        filter: "opportunityid eq '{OpportunityID}'"
      
      - name: FetchRelatedContacts
        source: Dataverse
        entity: contacts
        filter: "parentcustomerid eq '{AccountID}'"
      
      - name: FetchRecentEmails
        source: MicrosoftGraph
        endpoint: "/users/{OwnerEmail}/messages"
        filter: "recipients/any(r:r/emailAddress/address eq '{ContactEmail}')"
        top: 10
      
      - name: FetchCompetitorInfo
        source: SharePoint
        site: "SalesIntelligence"
        query: "ContentType:BattleCard AND Account:{AccountName}"
      
      - name: FetchSimilarDeals
        source: CustomAPI
        endpoint: "/api/similardeals"
        params:
          accountIndustry: "{Industry}"
          dealSize: "{EstimatedValue}"
          limit: 5
  
  - step: AggregateResults
    action: GenerateSummary
    inputs:
      opportunity: "{{FetchOpportunityDetails.result}}"
      contacts: "{{FetchRelatedContacts.result}}"
      emails: "{{FetchRecentEmails.result}}"
      competitors: "{{FetchCompetitorInfo.result}}"
      similarDeals: "{{FetchSimilarDeals.result}}"
```

---

## Practice Scenarios

### Scenario 1: Multi-System Orchestration
**Context:** Customer case requires data from Dynamics 365, SharePoint, and external ERP

**Task:** Design orchestration pattern for case resolution agent

**Solution:**
1. Unified context in Dataverse
2. Parallel API calls to all systems
3. Data aggregation layer
4. AI summarization of combined data
5. Present unified view to agent

### Scenario 2: Sales and Service Coordination
**Context:** Customer calls service about product issue, but also interested in upgrade

**Task:** Design handoff between Service Copilot and Sales Copilot

**Solution:**
1. Service Copilot detects sales intent
2. Resolve service issue first
3. Capture context (product usage, pain points)
4. Warm handoff to Sales Copilot
5. Sales Copilot uses service history to personalize pitch

---

## Key Takeaways

✅ **Dynamics 365 AI Orchestration:**
- Each D365 app has specific AI capabilities (Finance: forecasting, SCM: demand planning, CE: customer insights)
- Configure data sources, training periods, and thresholds carefully
- Monitor accuracy and retrain models regularly

✅ **M365 Copilot for Sales/Service:**
- Connect to CRM (D365 or Salesforce) for data grounding
- Configure which data sources to include
- Set up automatic activity tracking
- Enable appropriate features per user role

✅ **Power Platform AI:**
- AI Builder provides prebuilt and custom models
- Use prebuilt for common scenarios (invoices, receipts, sentiment)
- Train custom models when you have specific needs and data
- Integrate via Power Automate flows and Power Apps

✅ **Generative AI in Copilot Studio:**
- Prompt engineering is critical for quality responses
- Ground prompts with relevant data sources
- Configure content moderation and guardrails
- Always include citations for trustworthiness

✅ **Orchestration Best Practices:**
- Design for multi-agent scenarios with clear handoff protocols
- Centralize shared context to avoid data inconsistency
- Implement caching and parallel retrieval for performance
- Monitor agent interactions and outcomes for continuous improvement

---

## Study Resources

- [Dynamics 365 AI Capabilities](https://learn.microsoft.com/en-us/dynamics365/ai/)
- [Microsoft 365 Copilot for Sales](https://learn.microsoft.com/en-us/microsoft-sales-copilot/)
- [Microsoft 365 Copilot for Service](https://learn.microsoft.com/en-us/microsoft-service-copilot/)
- [AI Builder Documentation](https://learn.microsoft.com/en-us/ai-builder/)
- [Copilot Studio - Generative Answers](https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers)

**Previous Topic:** [Advanced Extensibility](02-extensibility-advanced.md)  
**Next Domain:** [Deploy AI-Powered Business Solutions](../3-DEPLOY-AI-SOLUTIONS.md)
