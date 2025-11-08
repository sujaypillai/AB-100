# Domain 2: Design AI-Powered Business Solutions - Practice Questions

## Multiple Choice Questions

### Topic: Agent Design

**Question 1:**
You're designing a Copilot Studio agent for a retail company. The agent needs to access customer order history from Dynamics 365 and product inventory from an external API. What is the BEST approach to handle authentication?

A) Store API keys directly in the bot configuration  
B) Use OAuth 2.0 with token storage in bot variables  
C) Implement Microsoft Entra ID authentication with secure connections  
D) Hardcode credentials in the bot's Power Fx expressions

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Microsoft Entra ID provides enterprise-grade authentication with secure token management. Copilot Studio supports connections that handle authentication securely without exposing credentials in the bot logic. For Dynamics 365, use built-in authenticated connections. For external APIs, configure OAuth connections in Copilot Studio.

**Why other options are incorrect:**
- A: Storing keys directly is insecure and violates best practices
- B: While OAuth is correct, manual token management in bot variables is unnecessary and risky
- D: Hardcoding credentials is a critical security vulnerability

**Reference:** Domain 2 - Agent Design, Authentication and Security
</details>

---

**Question 2:**
A company wants their Copilot Studio agent to handle product returns. The process requires checking order eligibility (within 30 days), validating condition (unopened), and initiating refund. Which design approach is MOST appropriate?

A) Single generative answers topic for all return scenarios  
B) Multiple topic structure with conditional branching and actions  
C) Fallback topic only with custom code for all logic  
D) Power Automate flow for entire conversation

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Complex business processes with multiple decision points, validations, and actions should use structured topics with branching logic. This provides better control, testing, and maintenance than relying solely on generative AI or external flows.

**Structure:**
- Main topic: "Process Return"
- Check order date (condition node)
- Validate product condition (question + validation)
- Call action: Initiate refund (Power Automate)
- Confirmation message

**Why other options are incorrect:**
- A: Generative answers alone can't reliably execute business logic
- C: Over-reliance on code/fallback reduces maintainability
- D: Conversation logic belongs in Copilot Studio; Power Automate for actions only

**Reference:** Domain 2 - Agent Design, Topic Design and Conversation Flow
</details>

---

**Question 3:**
Which Microsoft 365 Copilot extensibility option should you use when you need to execute custom business logic and return results directly in the Copilot experience?

A) Declarative agent with Microsoft Graph connectors  
B) Message extension  
C) API plugin  
D) Copilot Studio agent

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** API plugins allow M365 Copilot to call custom APIs, execute business logic, and incorporate results directly in responses. They're ideal for dynamic data retrieval and custom calculations that require server-side processing.

**Why other options are incorrect:**
- A: Graph connectors index static content for search but don't execute custom logic
- B: Message extensions surface UI in Teams but don't integrate directly into Copilot's natural language responses
- D: Copilot Studio creates separate agents, not extensions to M365 Copilot

**Reference:** Domain 2 - Agent Design, M365 Copilot Extensibility
</details>

---

**Question 4:**
Your organization uses Microsoft Teams Contact Center. You want the AI agent to access customer sentiment from previous interactions. What capability enables this?

A) Power Virtual Agents historical analytics  
B) Real-time sentiment analysis from Azure AI Language  
C) Conversation transcript analysis in Dataverse  
D) All of the above

<details>
<summary>Answer</summary>

**Correct Answer: D**

**Explanation:** All three approaches can provide sentiment data:
- Power Virtual Agents analytics track conversation satisfaction scores
- Azure AI Language analyzes sentiment in real-time during conversations
- Dataverse stores conversation transcripts that can be analyzed for historical sentiment

The best design uses real-time sentiment (B) during active conversations and historical analysis (C) for context from past interactions.

**Reference:** Domain 2 - Agent Design, Contact Center Integration
</details>

---

### Topic: Extensibility

**Question 5:**
You're building a declarative agent for M365 Copilot that helps with expense reporting. The agent needs access to receipt images stored in SharePoint and expense policies in a custom database. Which combination of extensibility options is required?

A) Microsoft Graph connector + API plugin  
B) Message extension + Copilot Studio agent  
C) API plugin only  
D) Microsoft Graph connector only

<details>
<summary>Answer</summary>

**Correct Answer: A**

**Explanation:** 
- **Microsoft Graph connector:** Indexes SharePoint receipt images for search and retrieval within Copilot
- **API plugin:** Accesses custom expense policy database through your API

This combination allows Copilot to search documents (Graph connector) and execute custom business logic (API plugin).

**Why other options are incorrect:**
- B: Message extensions and Copilot Studio don't integrate with declarative M365 Copilot agents
- C: API plugin alone can't index SharePoint content for semantic search
- D: Graph connector alone can't access custom databases

**Reference:** Domain 2 - Extensibility, Declarative Agents
</details>

---

**Question 6:**
When designing a custom Prompt Flow in Azure AI Foundry, you need to retrieve relevant documents from your knowledge base before generating a response. What component should you use?

A) LLM node  
B) Python node with custom search code  
C) Vector Index Lookup node  
D) Prompt node

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** The Vector Index Lookup node is specifically designed for RAG (Retrieval-Augmented Generation) patterns. It searches vector embeddings to find semantically relevant documents from your indexed knowledge base, then passes them as context to the LLM.

**Flow structure:**
1. Input (user question)
2. Vector Index Lookup (retrieve relevant docs)
3. LLM node (generate answer using retrieved context)
4. Output

**Why other options are incorrect:**
- A: LLM node generates text but doesn't search knowledge bases
- B: Custom Python code works but Vector Index Lookup is purpose-built and optimized
- D: Prompt node defines LLM instructions but doesn't retrieve data

**Reference:** Domain 2 - Extensibility, Azure AI Foundry Patterns
</details>

---

**Question 7:**
What is the PRIMARY advantage of using Copilot Studio's "Generative Answers" feature over traditional topic-based responses?

A) Lower latency for responses  
B) Ability to answer questions not explicitly programmed  
C) Better integration with Dataverse  
D) Automatic translation to all languages

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Generative Answers uses AI to generate responses from knowledge sources (websites, SharePoint, Dataverse) without requiring explicit topic authoring for every possible question. It provides flexibility for diverse queries while traditional topics require pre-defined paths.

**Best practice:** Use Generative Answers for broad informational queries and structured topics for transactional processes (orders, returns, appointments).

**Why other options are incorrect:**
- A: Generative answers may have higher latency due to AI processing
- C: Both approaches integrate with Dataverse equally
- D: Translation requires separate configuration regardless of answer type

**Reference:** Domain 2 - Extensibility, Copilot Studio Generative Answers
</details>

---

### Topic: Advanced Extensibility (MCP)

**Question 8:**
Model Context Protocol (MCP) enables AI models to access external resources. Which component acts as the intermediary between the AI model and data sources?

A) MCP Client  
B) MCP Server  
C) MCP Host  
D) MCP Router

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** In MCP architecture:
- **MCP Host:** AI application (Claude, custom AI app)
- **MCP Client:** Built into the host, initiates requests
- **MCP Server:** Provides access to specific resources (databases, APIs, tools)

The MCP Server exposes resources and tools that the AI model can use through standardized protocols.

**Why other options are incorrect:**
- A: MCP Client is built into the host and initiates requests but doesn't act as intermediary
- C: MCP Host is the AI application itself
- D: MCP Router is not a standard MCP component

**Reference:** Domain 2 - Advanced Extensibility, Model Context Protocol
</details>

---

**Question 9:**
You're implementing Computer Use capabilities for an AI agent. What is the PRIMARY security concern?

A) API rate limiting  
B) Unintended actions on user's computer  
C) High token consumption  
D) Slow response times

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Computer Use allows AI to control mouse, keyboard, and interact with applications. The primary security concern is the AI performing unintended or harmful actions (deleting files, sending emails, making purchases). Safeguards include:
- Explicit user confirmation for sensitive actions
- Restricted action allowlist
- Sandboxed environments
- Human-in-the-loop for critical operations
- Comprehensive audit logging

**Why other options are incorrect:**
- A: Rate limiting is a concern but not the primary security issue
- C: Token consumption is a cost issue, not security
- D: Performance is not a security concern

**Reference:** Domain 2 - Advanced Extensibility, Computer Use
</details>

---

**Question 10:**
When should you use GPT-4 with reasoning modes (extended thinking) versus standard GPT-4?

A) Always use reasoning mode for better quality  
B) For complex multi-step problems requiring planning and verification  
C) For simple FAQ responses  
D) When cost is the primary concern

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Reasoning modes (like o1 models) excel at complex tasks requiring:
- Multi-step logical reasoning
- Problem decomposition and planning
- Mathematical calculations
- Code generation with debugging
- Verification and self-correction

Standard GPT-4 is better for: conversational responses, creative writing, simple Q&A.

**Trade-offs:** Reasoning modes have higher latency and cost but better accuracy for complex problems.

**Why other options are incorrect:**
- A: Unnecessary for simple tasks and increases cost/latency
- C: FAQ responses don't require extended reasoning
- D: Reasoning mode has higher cost, not lower

**Reference:** Domain 2 - Advanced Extensibility, Reasoning and Voice Modes
</details>

---

### Topic: Orchestrate Prebuilt Agents

**Question 11:**
In Dynamics 365 Finance, which AI capability provides cash flow forecasting?

A) Copilot in Excel  
B) Finance Insights  
C) Customer Insights  
D) Sales Accelerator

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Finance Insights in Dynamics 365 Finance includes:
- Cash flow forecasting
- Payment predictions
- Budget proposals
- Intelligent budget proposals

It uses historical data and machine learning to predict future financial positions.

**Why other options are incorrect:**
- A: Copilot in Excel provides data analysis but not D365-specific forecasting
- C: Customer Insights is for customer data, not financial forecasting
- D: Sales Accelerator is for sales pipeline, not finance

**Reference:** Domain 2 - Orchestrate Prebuilt Agents, Dynamics 365 Finance
</details>

---

**Question 12:**
Microsoft 365 Copilot for Sales integrates with which systems? (Select all that apply)

A) Dynamics 365 Sales  
B) Salesforce  
C) SAP CRM  
D) Microsoft Outlook and Teams

<details>
<summary>Answer</summary>

**Correct Answer: A, B, D**

**Explanation:** M365 Copilot for Sales supports:
- **Dynamics 365 Sales:** Native integration
- **Salesforce:** Through certified connector
- **Outlook and Teams:** Core M365 integration for meeting prep, email summaries, CRM updates

SAP CRM is not currently supported for M365 Copilot for Sales.

**Reference:** Domain 2 - Orchestrate Prebuilt Agents, M365 Copilot for Sales
</details>

---

**Question 13:**
In Power Platform, which AI Builder capability would you use to extract invoice data (invoice number, date, amounts) from PDF invoices?

A) Text generation  
B) Form processing (prebuilt model: Invoice)  
C) Object detection  
D) Prediction model

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** AI Builder's prebuilt Invoice Processing model is specifically trained to extract structured data from invoices including:
- Invoice number, date, due date
- Vendor information
- Line items with descriptions, quantities, prices
- Subtotal, tax, total amount

**Why other options are incorrect:**
- A: Text generation creates new text, doesn't extract data
- C: Object detection identifies objects in images, not document data extraction
- D: Prediction models forecast outcomes, not extract document data

**Reference:** Domain 2 - Orchestrate Prebuilt Agents, Power Platform AI Builder
</details>

---

## Scenario-Based Questions

### Scenario 1: Multi-Channel Customer Service

**Context:**
A telecommunications company wants to deploy an AI agent across web chat, Microsoft Teams, and phone (voice). The agent should:
- Check account balance and usage
- Troubleshoot common issues (connectivity, billing)
- Schedule technician appointments
- Escalate to human agent when needed

**Questions:**

**1.1** Which platform should you use and why?

<details>
<summary>Answer</summary>

**Platform: Copilot Studio**

**Rationale:**
- **Multi-channel:** Native support for web, Teams, phone (telephony connectors)
- **Low-code:** Rapid development for common scenarios
- **Integration:** Built-in connectors for Dynamics 365, Power Automate
- **Voice:** Supports voice channel with speech-to-text and text-to-speech
- **Scalability:** Enterprise-ready with analytics

**Architecture:**
```
Copilot Studio Bot
├── Channel: Web (website embedding)
├── Channel: Microsoft Teams (app)
└── Channel: Voice (Nuance/AudioCodes gateway)
    ↓
Topics: Account Info, Troubleshooting, Scheduling
    ↓
Actions: Query Dataverse, Call Power Automate flows
    ↓
Escalation: Microsoft Teams queue or Contact Center
```
</details>

**1.2** Design the conversation flow for "Check account balance."

<details>
<summary>Answer</summary>

**Topic: Check Account Balance**

```
1. Trigger Phrases:
   - "What's my balance"
   - "Check account balance"
   - "How much do I owe"

2. Authenticate User:
   - Already authenticated? → Continue
   - Not authenticated? → Ask for phone number + PIN
   - Validate against customer database

3. Retrieve Balance:
   - Action: Call Power Automate flow
   - Input: Customer ID
   - Query: Dynamics 365 Customer Service / Billing system
   - Return: Current balance, due date, recent charges

4. Present Information:
   - Message: "Your current balance is $XX.XX, due on [date]"
   - Adaptive Card: Show balance breakdown
   - Options:
     * "View recent charges" → Navigate to transaction history topic
     * "Make a payment" → Navigate to payment topic
     * "Something else" → Return to main menu

5. Confirmation:
   - "Is there anything else I can help you with?"
   - Yes → Main menu
   - No → End conversation with satisfaction survey
```

**Voice Consideration:** Use SSML for natural speech output, include verbal confirmation of amounts
</details>

**1.3** What metrics should you track to measure success?

<details>
<summary>Answer</summary>

**Engagement Metrics:**
- Total sessions by channel
- Average session duration
- Messages per session
- Return user rate

**Performance Metrics:**
- Resolution rate (% resolved without escalation)
- Escalation rate by topic
- Average time to resolution
- Abandon rate

**Quality Metrics:**
- CSAT (Customer Satisfaction) score
- Topic recognition accuracy
- Fallback rate (% unrecognized queries)
- Error rate by topic

**Business Impact:**
- Cost per interaction (vs. human agent)
- Call volume reduction
- First-contact resolution rate
- Agent time savings

**Targets:**
- Resolution rate: >70%
- CSAT: >4.0/5.0
- Escalation rate: <30%
- Topic recognition: >85%
</details>

---

### Scenario 2: Declarative Agent for M365 Copilot

**Context:**
A consulting firm wants to create a declarative agent for M365 Copilot that helps consultants prepare for client meetings. The agent should:
- Summarize recent client communications (emails, Teams chats)
- Access project documentation from SharePoint
- Retrieve billing information from custom ERP system
- Suggest talking points based on project status

**Questions:**

**2.1** Design the declarative agent architecture. List required components.

<details>
<summary>Answer</summary>

**Agent Architecture:**

**1. Declarative Agent Manifest:**
```json
{
  "name": "Client Meeting Assistant",
  "description": "Helps prepare for client meetings",
  "instructions": "You are an AI assistant specializing in client meeting preparation. Use recent communications, project docs, and billing data to create meeting briefings.",
  "capabilities": [
    {
      "name": "microsoft_graph_connector",
      "connection_id": "sharepoint_projects"
    },
    {
      "name": "api_plugin",
      "api_definition": "https://erp.contoso.com/api/openapi.json"
    }
  ],
  "conversation_starters": [
    "Prepare me for my meeting with Acme Corp",
    "What's the status of Project Phoenix?",
    "Summarize recent communications with client XYZ"
  ]
}
```

**2. Microsoft Graph Connector:**
- Index: SharePoint project sites
- Content: Project plans, status reports, deliverables
- Metadata: Client name, project ID, date
- Refresh: Daily incremental, weekly full

**3. API Plugin:**
- Endpoint: Custom ERP system
- Functions:
  * `get_client_billing(client_id)` → Recent invoices, payment status
  * `get_project_status(project_id)` → Budget, timeline, milestones
- Authentication: OAuth 2.0 with Azure AD

**4. Instructions (System Prompt):**
- Prioritize recent communications (last 30 days)
- Structure briefings: Overview, Recent Activity, Key Topics, Action Items
- Flag billing issues or project delays
- Suggest discussion topics based on project phase
</details>

**2.2** What permissions and security considerations are required?

<details>
<summary>Answer</summary>

**Microsoft Graph Permissions:**
- `Sites.Read.All` (SharePoint project docs)
- `Mail.Read` (email communications)
- `Chat.Read` (Teams messages)
- `Files.Read.All` (document access)

**API Plugin Permissions:**
- Custom ERP: `billing.read`, `projects.read`
- Scoped to user's allowed clients only

**Security Considerations:**

1. **Data Access:**
   - Agent inherits user's permissions (security trimmed)
   - Users only see clients/projects they have access to
   - Row-level security in ERP API

2. **Sensitive Data:**
   - Billing information marked as confidential
   - Don't log financial details in telemetry
   - Redact PII in analytics

3. **Audit:**
   - Log all agent interactions with client data
   - Track which documents were accessed
   - Compliance with data retention policies

4. **Consent:**
   - Users must consent to agent accessing their data
   - Clear disclosure in agent description
   - Ability to revoke consent

5. **Testing:**
   - Use test clients/projects for development
   - Don't expose production data in dev environments
</details>

**2.3** Write sample instructions for the declarative agent.

<details>
<summary>Answer</summary>

**Agent Instructions (System Prompt):**

```
You are the Client Meeting Assistant, an AI specialized in helping consultants prepare for client meetings.

## Core Responsibilities
- Summarize recent client communications from emails and Teams chats
- Retrieve relevant project documentation from SharePoint
- Access billing and budget information from the ERP system
- Generate meeting briefings with key talking points

## Response Structure
When preparing meeting briefings, use this format:

### Meeting Brief: [Client Name]
**Date:** [Meeting Date]
**Attendees:** [List if available]

**1. Quick Overview**
- Client relationship status
- Active projects summary
- Recent interactions (last 30 days)

**2. Key Updates**
- Project milestones achieved/upcoming
- Budget status vs. planned
- Outstanding deliverables
- Recent communications highlights

**3. Topics to Discuss**
- Items requiring decisions
- Risks or blockers
- Opportunities for expansion
- Follow-ups from previous meeting

**4. Action Items**
- Outstanding tasks
- Deadlines approaching
- Documents to bring

## Guidelines
- Prioritize information from the last 30 days
- Flag billing issues or overdue payments
- Highlight project delays or risks
- Suggest proactive topics based on project phase
- If billing info is sensitive, ask before displaying
- Cite sources (link to emails, documents, Teams conversations)
- If information is missing, state what's not available

## When to Search
- Use SharePoint connector for project docs, status reports
- Use Graph for recent emails and Teams conversations
- Use ERP API for billing, budget, project financials

## Tone
- Professional and concise
- Focus on actionable insights
- Assume user is time-constrained
```
</details>

---

### Scenario 3: Dynamics 365 + Power Platform Integration

**Context:**
A retail company uses:
- Dynamics 365 Customer Insights for customer segmentation
- Dynamics 365 Marketing for campaigns
- Power Apps for store operations
- Power Automate for order processing

They want to create an AI-powered assistant for store managers that provides:
- Real-time inventory alerts
- Customer behavior insights for their store
- Campaign performance for their region
- Automated reordering suggestions

**Questions:**

**3.1** Which AI components would you use?

<details>
<summary>Answer</summary>

**AI Component Architecture:**

**1. Copilot Studio Bot (Store Manager Assistant)**
- Platform for natural language interface
- Integrates with all required systems
- Mobile-friendly (Power Apps portal or Teams)

**2. AI Builder Models:**
- **Prediction Model:** Reorder point prediction based on sales velocity
- **Text Analysis:** Analyze customer feedback sentiment

**3. Dynamics 365 Customer Insights AI:**
- Customer segment insights
- RFM (Recency, Frequency, Monetary) analysis
- Purchase propensity scores

**4. Power Automate Flows (AI-Triggered):**
- Inventory alert flow (threshold-based)
- Automated reorder flow (calls supplier API)
- Campaign performance digest (scheduled)

**5. Azure AI Language (via API Plugin):**
- Advanced text analytics for customer reviews
- Key phrase extraction from feedback

**Integration Flow:**
```
Store Manager → Copilot Studio Bot
                     ↓
    ┌───────────────┼───────────────┐
    ↓               ↓               ↓
Customer Insights  AI Builder    Power Automate
(Segments)         (Predictions)  (Alerts/Actions)
    ↓               ↓               ↓
Dataverse ← Unified Data Platform →
```
</details>

**3.2** Design the conversation for "Inventory Alert."

<details>
<summary>Answer</summary>

**Topic: Inventory Alerts**

**Trigger:** Proactive message from Power Automate flow when inventory falls below threshold

**Flow Design:**

```
Power Automate Flow: Inventory Monitor
├── Trigger: Scheduled (runs hourly)
├── Get store inventory from Dataverse
├── For each product:
│   ├── Check if quantity < reorder point
│   └── If yes:
│       ├── Get AI Builder prediction (days until stockout)
│       └── Send proactive message to Copilot Studio bot
└── Log alert in Dataverse

Copilot Studio: Receive Proactive Message
├── Display alert with Adaptive Card:
│   ├── Product name & SKU
│   ├── Current quantity
│   ├── Reorder point
│   ├── Predicted stockout date
│   └── Action buttons:
│       ├── "View details" → Show sales velocity, supplier info
│       ├── "Reorder now" → Trigger reorder flow
│       └── "Adjust threshold" → Update reorder point
├── User selects "Reorder now":
│   ├── Call Power Automate flow: Create Purchase Order
│   ├── Input: Product ID, suggested quantity (from AI)
│   ├── Flow creates PO in D365 Supply Chain
│   └── Confirmation: "PO #12345 created for 500 units"
└── Log action in Dataverse
```

**Adaptive Card Example:**
```json
{
  "type": "AdaptiveCard",
  "body": [
    {
      "type": "TextBlock",
      "text": "⚠️ Low Inventory Alert",
      "weight": "Bolder",
      "size": "Large"
    },
    {
      "type": "FactSet",
      "facts": [
        {"title": "Product", "value": "Blue Widget XL"},
        {"title": "Current Stock", "value": "45 units"},
        {"title": "Reorder Point", "value": "50 units"},
        {"title": "Predicted Stockout", "value": "November 12 (4 days)"}
      ]
    }
  ],
  "actions": [
    {"type": "Action.Submit", "title": "View Details"},
    {"type": "Action.Submit", "title": "Reorder Now"},
    {"type": "Action.Submit", "title": "Snooze Alert"}
  ]
}
```
</details>

**3.3** How would you use Customer Insights AI to personalize recommendations?

<details>
<summary>Answer</summary>

**Customer Insights AI Integration:**

**1. Segment-Based Recommendations:**
```
Topic: "What products should I promote?"

Flow:
├── Get store location from user profile
├── Query Customer Insights:
│   ├── Get dominant customer segments for this store
│   │   Example: "Budget-Conscious Families" (35%), "Tech Enthusiasts" (25%)
│   └── Get segment preferences and behaviors
├── Call AI Builder Prediction:
│   ├── Input: Segment + current inventory + season
│   └── Output: Recommended products to feature
├── Display recommendations with rationale:
│   "Based on your customer base (35% Budget-Conscious Families):
│    1. Feature value packs and multi-buy offers
│    2. Promote household essentials
│    3. Highlight clearance items"
└── Provide campaign templates from D365 Marketing
```

**2. Personalized Customer Service:**
```
Scenario: Store manager asks about high-value customer visiting today

Flow:
├── Customer Insights lookup by name/phone
├── Retrieve customer profile:
│   ├── Segment: "VIP - High Lifetime Value"
│   ├── Purchase history: Average order $500
│   ├── Preferences: Premium products, sustainable brands
│   ├── RFM Score: R:5, F:5, M:5
│   └── Propensity scores: 
│       • Next purchase probability: 85% within 7 days
│       • Churn risk: Low (15%)
│       • Category affinity: Electronics (0.9), Home Goods (0.7)
├── Generate service recommendations:
│   "VIP Customer: Jane Smith
│    • Lifetime value: $12,500 (top 2% of customers)
│    • Last visit: 12 days ago (due for visit)
│    • Preferred categories: Electronics, Home Goods
│    • Recommendation: 
│      - Show new electronics arrivals
│      - Offer preview of upcoming premium line
│      - Provide complimentary gift wrapping"
└── Log interaction for future insights
```

**3. Campaign Performance Insights:**
```
Topic: "How is the fall campaign performing?"

Flow:
├── Query D365 Marketing campaign results for store's region
├── Join with Customer Insights:
│   ├── Campaign engagement by segment
│   └── Conversion rates by customer profile
├── AI analysis:
│   ├── Identify overperforming/underperforming segments
│   ├── Recommend adjustments
│   Example: "Tech Enthusiasts segment showing 2x higher engagement
│            → Recommend increasing budget allocation to this segment"
└── Provide actionable insights with confidence scores
```
</details>

---

## Performance Metrics

Track your progress:

- [ ] Domain 2 - Questions 1-13 completed
- [ ] Scenario 1 completed
- [ ] Scenario 2 completed
- [ ] Scenario 3 completed
- [ ] Score: ____/13 multiple choice
- [ ] Scenarios: ____/3 completed

**Target:** 80% correct (11/13 questions minimum)

---

**Next:** [Domain 3 Practice Questions](questions-domain3-deploy.md)
