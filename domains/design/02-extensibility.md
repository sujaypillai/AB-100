# Design Extensibility of AI Solutions

## Overview
This section covers extensibility patterns for AI solutions, including custom models in Azure AI Foundry, Microsoft 365 Copilot extensibility, Model Context Protocol, Computer Use automation, and agent behaviors.

---

## 1. Design AI Solutions Using Custom Models in Azure AI Foundry

### Azure AI Foundry Overview

**What is Azure AI Foundry?**
- Unified platform for building custom AI solutions
- Access to model catalog (OpenAI, open-source models)
- Tools for fine-tuning, prompt engineering, evaluation
- Deployment and management of custom models
- Integration with Azure services

**Key Capabilities:**
```
Azure AI Foundry Components:

├── Model Catalog
│   ├── Foundation models (GPT-4, Llama, Phi, etc.)
│   ├── Specialized models (vision, speech, embedding)
│   └── Open-source models
│
├── Prompt Flow
│   ├── Visual workflow designer
│   ├── Prompt engineering tools
│   ├── Chain-of-thought orchestration
│   └── Testing and evaluation
│
├── Fine-Tuning Studio
│   ├── Upload training data
│   ├── Configure hyperparameters
│   ├── Monitor training progress
│   └── Compare model versions
│
├── Evaluation & Testing
│   ├── Built-in metrics (accuracy, F1, BLEU)
│   ├── Custom evaluators
│   ├── A/B testing capabilities
│   └── Quality assessment
│
└── Deployment & Management
    ├── Managed endpoints
    ├── Scaling configuration
    ├── Monitoring and logging
    └── Cost management
```

### When to Use Custom Models in Azure AI Foundry

**Scenario 1: Domain-Specific Fine-Tuning**
```
Use Case: Legal contract analysis for specific jurisdiction

Approach:
1. Start with base model (e.g., GPT-4)
2. Gather training data:
   - 1,000+ contract examples
   - Clause classifications
   - Risk assessments
3. Fine-tune model on legal data
4. Deploy as custom endpoint
5. Integrate with Copilot Studio agent

Benefits:
- Better accuracy on legal terminology
- Consistent clause interpretation
- Faster than prompt engineering alone
```

**Scenario 2: Multi-Modal Custom Solution**
```
Use Case: Manufacturing quality inspection

Architecture:
1. Image Model: Detect defects in product images
2. Text Model: Classify defect types and severity
3. Decision Model: Recommend actions (rework, scrap, pass)

Implementation:
├── GPT-4 Vision for image analysis
├── Custom classifier for defect types
├── Business rules engine for decisions
└── Integration with manufacturing system

Workflow:
Image → Vision Model → Defect Detection → Classification → Action
```

**Scenario 3: RAG with Custom Embeddings**
```
Use Case: Enterprise knowledge retrieval

Architecture:
1. Custom embedding model trained on company docs
2. Vector database (Azure AI Search)
3. Retrieval augmented generation
4. Custom prompt templates

Benefits:
- Better semantic understanding of domain terms
- Improved retrieval relevance
- Context-aware responses
```

### Design Patterns

#### Pattern 1: Prompt Flow Orchestration
```
Design a complex workflow in Prompt Flow:

┌─────────────┐
│ User Query  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Query       │  ← Classify intent
│ Classifier  │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌────┐   ┌────┐
│LLM1│   │LLM2│  ← Route to appropriate model
└─┬──┘   └─┬──┘
  │        │
  └────┬───┘
       │
       ▼
┌─────────────┐
│ Response    │
│ Formatter   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Final Output│
└─────────────┘

Use Cases:
- Multi-step reasoning
- Complex business logic
- Model routing
- Response post-processing
```

#### Pattern 2: Custom Evaluation Pipeline
```
Evaluation Framework:

1. Ground Truth Dataset
   - Input queries
   - Expected outputs
   - Quality criteria

2. Batch Testing
   - Run model against test set
   - Collect predictions

3. Automated Scoring
   - Accuracy metrics
   - Relevance scores
   - Toxicity checks
   - Custom business metrics

4. Analysis & Iteration
   - Identify failure patterns
   - Refine prompts/model
   - Re-test

Implementation in Azure AI Foundry:
- Upload evaluation dataset
- Configure evaluation flow
- Schedule automated runs
- Dashboard for results
```

#### Pattern 3: A/B Testing Models
```
Scenario: Compare GPT-4 vs. fine-tuned model

Setup:
├── Deploy both models to separate endpoints
├── Traffic splitter (50/50 or 90/10)
├── Track metrics per model
└── Analyze results

Metrics to Compare:
- Response quality (user ratings)
- Latency
- Cost per query
- Task completion rate
- User satisfaction

Decision Criteria:
If fine-tuned model:
- Quality >= 95% of GPT-4
- Cost < 50% of GPT-4
→ Migrate to fine-tuned model
```

### Implementation Steps

**Step 1: Project Setup**
```
Azure AI Foundry Project Structure:

my-ai-project/
├── data/
│   ├── training/
│   ├── validation/
│   └── test/
├── prompts/
│   ├── system-prompts/
│   └── user-prompts/
├── flows/
│   ├── main-flow.yaml
│   └── evaluation-flow.yaml
├── models/
│   ├── model-config.json
│   └── deployment-config.json
└── notebooks/
    ├── data-prep.ipynb
    └── evaluation.ipynb
```

**Step 2: Data Preparation**
```python
# Example: Prepare training data for fine-tuning

import pandas as pd
import json

# Load raw data
df = pd.read_csv('customer_support_tickets.csv')

# Format for fine-tuning
training_data = []
for _, row in df.iterrows():
    training_data.append({
        "messages": [
            {"role": "system", "content": "You are a helpful customer support agent."},
            {"role": "user", "content": row['customer_question']},
            {"role": "assistant", "content": row['agent_response']}
        ]
    })

# Save in JSONL format
with open('training_data.jsonl', 'w') as f:
    for item in training_data:
        f.write(json.dumps(item) + '\n')
```

**Step 3: Model Fine-Tuning**
```
Azure AI Foundry Fine-Tuning Configuration:

Base Model: gpt-4
Training Data: training_data.jsonl (10,000 examples)
Validation Split: 10%

Hyperparameters:
- Learning rate: 0.0001
- Batch size: 4
- Epochs: 3
- Warmup steps: 100

Training Time: ~2-4 hours
Cost: $500-1000 (estimated)

Monitoring:
- Loss curve
- Validation metrics
- Sample predictions
```

**Step 4: Prompt Flow Development**
```yaml
# main-flow.yaml

inputs:
  question:
    type: string

outputs:
  answer:
    type: string

nodes:
  - name: retrieve_context
    type: python
    source:
      type: code
      path: retrieve_context.py
    inputs:
      question: ${inputs.question}
    
  - name: generate_answer
    type: llm
    source:
      type: code
      path: llm_call.py
    inputs:
      question: ${inputs.question}
      context: ${retrieve_context.output}
      deployment_name: my-custom-model
    
  - name: format_response
    type: python
    source:
      type: code
      path: format_response.py
    inputs:
      raw_answer: ${generate_answer.output}
```

**Step 5: Evaluation**
```python
# evaluation.py

from promptflow import tool
from azure.ai.evaluation import evaluate

@tool
def evaluate_model(test_data: list) -> dict:
    """Evaluate model performance"""
    
    metrics = {
        'accuracy': 0,
        'relevance': 0,
        'coherence': 0,
        'groundedness': 0
    }
    
    for item in test_data:
        prediction = model.predict(item['input'])
        
        # Built-in evaluators
        metrics['accuracy'] += accuracy_score(item['expected'], prediction)
        metrics['relevance'] += relevance_score(item['input'], prediction)
        
    # Average metrics
    for key in metrics:
        metrics[key] /= len(test_data)
    
    return metrics
```

**Step 6: Deployment**
```
Deployment Options:

1. Managed Endpoint (Recommended):
   - Automatic scaling
   - Load balancing
   - Monitoring built-in
   - Cost: Pay per usage

2. Container Deployment:
   - Full control
   - Can deploy anywhere
   - Manual scaling
   - Cost: Infrastructure cost

Configuration:
- Instance type: Standard_DS3_v2
- Min instances: 1
- Max instances: 10
- Auto-scale: CPU > 70%
- Authentication: Key-based or Azure AD
```

---

## 2. Design Agents in Microsoft 365 Copilot

### Microsoft 365 Copilot Extensibility

**Extension Types:**

#### A. Declarative Agents
```
What: Configure agents with JSON manifest
No Code Required

Manifest Structure:
{
  "name": "Sales Assistant",
  "description": "Helps sales team with customer insights",
  "instructions": "You are a sales assistant...",
  "capabilities": {
    "graph_connectors": ["crm-data"],
    "plugins": ["send-email", "create-task"],
    "files": ["sales-playbook.pdf"]
  },
  "conversation_starters": [
    "Show me my top opportunities",
    "Summarize this week's pipeline",
    "Draft follow-up email"
  ]
}

Use Cases:
- Department-specific assistants
- Role-based helpers
- Process-specific agents
```

#### B. Plugins (API-Based Extensions)
```
What: Connect Copilot to external systems via APIs

Architecture:
Microsoft 365 Copilot
         ↓
    Your Plugin (API)
         ↓
  External System

Plugin Manifest:
{
  "schema_version": "v2.1",
  "name_for_human": "CRM Integration",
  "name_for_model": "crm_plugin",
  "description_for_human": "Access customer data",
  "description_for_model": "Retrieve customer information...",
  "api": {
    "type": "openapi",
    "url": "https://api.company.com/crm/openapi.yaml"
  },
  "auth": {
    "type": "oauth2"
  }
}

Common Use Cases:
- CRM data access
- Project management tools
- Custom LOB apps
- Third-party services
```

#### C. Microsoft Graph Connectors
```
What: Index external data for Copilot search

Purpose:
- Make external content searchable in M365
- Copilot can retrieve and cite this data
- Respects permissions

Data Sources:
├── Custom databases
├── File shares
├── Third-party SaaS
├── Legacy systems
└── APIs

Implementation:
1. Create Graph connector
2. Define schema mapping
3. Ingest content (full or incremental)
4. Set up access controls
5. Enable in Copilot
```

### Design Patterns for M365 Copilot Agents

#### Pattern 1: Department-Specific Agent
```
Use Case: Finance Department Assistant

Configuration:
Name: "Finance Copilot"
Instructions: "You are a finance assistant helping with 
               budgeting, reporting, and compliance questions."

Knowledge Sources:
├── SharePoint: /sites/finance/policies
├── Graph Connector: ERP system
├── Files: Budget templates, compliance guides
└── Plugin: Financial reporting API

Capabilities:
- Answer policy questions
- Generate budget reports
- Check compliance requirements
- Create expense reports

Conversation Starters:
- "What's our Q4 budget status?"
- "Show me travel expense policy"
- "Generate monthly financial report"
- "Check approval workflow for this expense"

Access Control:
- Only Finance department members
- Role-based data access
```

#### Pattern 2: Process Automation Agent
```
Use Case: Employee Onboarding Assistant

Configuration:
Name: "Onboarding Helper"
Instructions: "Guide new employees through onboarding process."

Workflow:
1. New employee asks question
2. Agent retrieves relevant info from:
   - HR knowledge base
   - IT setup guides
   - Benefits information
   - Company policies
3. Provides step-by-step guidance
4. Creates tasks in Planner
5. Schedules meetings if needed

Plugins:
- Create Planner task
- Schedule Teams meeting
- Assign training courses
- Submit IT requests

Example Interaction:
Employee: "How do I set up my email?"
Agent: "I'll help you set up your email. 
        [Retrieves IT guide]
        Here are the steps...
        [Creates task in Planner]
        I've also created a task reminder.
        Would you like to schedule a call with IT if you need help?"
```

#### Pattern 3: Multi-Agent Collaboration
```
Scenario: Customer 360 View

Architecture:
Main Agent: Customer Insights Copilot
    ↓
    Delegates to:
    ├── Sales Agent (Dynamics 365 Sales data)
    ├── Service Agent (Support tickets)
    ├── Marketing Agent (Campaign interactions)
    └── Finance Agent (Payment history)

User Query: "Give me a complete view of Acme Corp"

Main Agent:
1. Queries Sales Agent → Revenue, opportunities
2. Queries Service Agent → Support history, issues
3. Queries Marketing Agent → Campaign engagement
4. Queries Finance Agent → Payment status
5. Synthesizes comprehensive response

Response:
"Acme Corp Overview:
- Annual Revenue: $2M (↑15% YoY)
- Active Opportunities: 3 totaling $500K
- Support: 2 open tickets (low priority)
- Marketing: High engagement on recent campaign
- Finance: Payment terms Net 30, always on time
- Recommendation: Great customer for upsell opportunity"
```

### Implementation Steps

**Step 1: Create Declarative Agent**
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "1.0",
  "name": "Sales Assistant",
  "description": "Helps sales team with customer insights and tasks",
  "instructions": "You are an AI assistant for the sales team. Help them with customer information, sales processes, and administrative tasks. Always be professional and data-driven in your responses.",
  
  "capabilities": [
    {
      "name": "GraphConnectors",
      "items": [
        {
          "connection_id": "crm-connector",
          "description": "CRM customer data"
        }
      ]
    },
    {
      "name": "OneDriveAndSharePoint",
      "items": [
        {
          "site_url": "https://contoso.sharepoint.com/sites/sales",
          "description": "Sales team documentation"
        }
      ]
    },
    {
      "name": "Plugins",
      "items": [
        {
          "plugin_id": "crm-plugin",
          "description": "CRM operations"
        }
      ]
    }
  ],
  
  "conversation_starters": [
    {
      "title": "My top deals",
      "text": "Show me my top 5 opportunities this month"
    },
    {
      "title": "Customer summary",
      "text": "Summarize {{customer_name}}'s account"
    },
    {
      "title": "Send follow-up",
      "text": "Draft a follow-up email for {{opportunity_name}}"
    }
  ],
  
  "actions": [
    {
      "id": "create-follow-up-task",
      "file": "./actions/create-task.json"
    }
  ]
}
```

**Step 2: Deploy Agent**
```
Deployment Process:

1. Package agent files:
   ├── manifest.json
   ├── icon.png
   ├── actions/
   └── README.md

2. Upload to Teams App Studio

3. Test in development environment

4. Submit for admin approval

5. Publish to organization

6. Monitor usage and feedback
```

**Step 3: Monitor and Iterate**
```
Monitoring:
├── Usage analytics (queries per day)
├── Success rate (completed tasks)
├── User satisfaction (ratings)
├── Error rates
└── Performance (response time)

Iteration:
- Add frequently requested capabilities
- Refine instructions based on misunderstandings
- Add conversation starters for common tasks
- Update knowledge sources
- Improve error handling
```

---

## 3. Design Agent Extensibility in Copilot Studio

### Extensibility Options

#### A. Custom Connectors
```
Purpose: Integrate with any REST API

Use Cases:
- Proprietary systems
- Third-party services
- Custom applications
- Legacy systems

Creation Process:
1. Define API operations (OpenAPI/Swagger)
2. Configure authentication
3. Test API calls
4. Add to Copilot Studio agent
5. Use in topics via actions
```

**Example: Custom CRM Connector**
```yaml
# OpenAPI specification excerpt

openapi: 3.0.0
info:
  title: Custom CRM API
  version: 1.0.0

paths:
  /customers/{id}:
    get:
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
        revenue:
          type: number

  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.crm.com/oauth/authorize
          tokenUrl: https://auth.crm.com/oauth/token
          scopes:
            read: Read customer data
```

#### B. Power Automate Flows
```
Purpose: Execute complex business logic

Capabilities:
- Multi-step workflows
- Conditional logic
- Error handling
- Integration with 100s of services
- Data transformations

Example Flow: Create Customer Case
┌─────────────────┐
│ Trigger: From   │
│ Copilot Studio  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Validate Input  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Create Case in  │
│ Dynamics 365    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Send Email      │
│ Notification    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Return Case ID  │
│ to Agent        │
└─────────────────┘
```

#### C. Generative Answers
```
Purpose: Use AI to answer from knowledge sources

Configuration:
- Add knowledge sources (URLs, files, SharePoint)
- Configure generative answers settings
- Set content moderation level
- Enable citations

Behavior:
User: "What's our return policy?"
Agent:
1. Searches knowledge sources
2. Finds relevant policy documents
3. Generates natural language answer
4. Includes citations

Response: "Our return policy allows returns within 30 days 
          of purchase with original receipt. Items must be 
          unopened and in original condition. [1]
          
          [1] Return Policy v2.0, updated Nov 2025"
```

#### D. Plugin Actions (Preview)
```
Purpose: Extend agent with code capabilities

Languages Supported:
- Python
- TypeScript/JavaScript
- C#

Use Cases:
- Complex calculations
- Data transformations
- Integration with services without REST API
- Custom algorithms

Example: Lead Scoring Plugin
```python
from copilot_studio import plugin, tool

@plugin(name="lead_scorer")
class LeadScorer:
    
    @tool(description="Calculate lead score based on profile and behavior")
    def score_lead(
        self,
        company_size: int,
        industry: str,
        engagement_score: float,
        budget: float
    ) -> dict:
        """
        Calculate comprehensive lead score
        """
        score = 0
        
        # Company size scoring
        if company_size > 1000:
            score += 30
        elif company_size > 100:
            score += 20
        else:
            score += 10
        
        # Industry scoring
        high_value_industries = ['technology', 'finance', 'healthcare']
        if industry.lower() in high_value_industries:
            score += 25
        
        # Engagement scoring
        score += engagement_score * 0.3
        
        # Budget qualification
        if budget > 100000:
            score += 25
        elif budget > 50000:
            score += 15
        
        # Classification
        if score >= 75:
            priority = "Hot"
        elif score >= 50:
            priority = "Warm"
        else:
            priority = "Cold"
        
        return {
            "score": score,
            "priority": priority,
            "recommendation": self._get_recommendation(priority)
        }
    
    def _get_recommendation(self, priority: str) -> str:
        recommendations = {
            "Hot": "Immediate follow-up recommended. Assign to senior sales rep.",
            "Warm": "Follow up within 48 hours. Send targeted content.",
            "Cold": "Add to nurture campaign. Monthly check-ins."
        }
        return recommendations[priority]
```

### Design Patterns

#### Pattern 1: Hybrid Intelligence
```
Combine generative AI with structured logic:

User Query → Agent
    ↓
    Decision Point: Can generative AI handle this?
    ├─ Yes → Use generative answers
    │         └─ Return AI response
    │
    └─ No → Requires specific action
              └─ Call custom connector/flow
                  └─ Return structured result

Example:
Query: "What's our refund policy?" → Generative AI
Query: "Process a refund for order #123" → Structured flow
```

#### Pattern 2: Progressive Enhancement
```
Start simple, add complexity:

Phase 1: FAQ Bot
- Topics for common questions
- Static responses
- Simple branching

Phase 2: Add Knowledge Base
- Enable generative answers
- Upload documentation
- Dynamic responses

Phase 3: Add Integrations
- Connect to business systems
- Fetch real-time data
- Execute transactions

Phase 4: Advanced AI
- Custom models
- Complex workflows
- Multi-agent orchestration
```

#### Pattern 3: Graceful Degradation
```
Design for failure scenarios:

Primary Path:
User Query → API Call → Success → Response

Fallback 1:
API Call Fails → Try cached data → Response

Fallback 2:
No cached data → Generative AI answer → Response

Fallback 3:
AI uncertain → Escalate to human → Transfer

Always have a fallback to avoid dead ends
```

---

## Study Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/)
- [Microsoft 365 Copilot Extensibility](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/)
- [Copilot Studio Connectors](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-custom-connector)
- [Prompt Flow Documentation](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/prompt-flow)

**Next Topic:** [Model Context Protocol and Computer Use](02-extensibility-advanced.md)
