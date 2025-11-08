# Design AI and Agents for Business Solutions

## Overview
This section covers the design of AI agents and components for business solutions, including Copilot customization in Dynamics 365, agent types, topics in Copilot Studio, and data processing for AI models.

---

## 1. Design Business Terms for Copilot in Dynamics 365

### What Are Business Terms?

**Definition:** Business terms are custom vocabulary and domain-specific language that help Copilot better understand your organization's unique terminology.

**Purpose:**
- Teach Copilot industry-specific jargon
- Recognize company acronyms and internal terms
- Understand product names and codes
- Improve accuracy of responses
- Reduce ambiguity in conversations

### Examples of Business Terms

**Manufacturing Company:**
```
Business Terms:
- "SKU" → Stock Keeping Unit (product identifier)
- "WIP" → Work In Progress inventory
- "BOM" → Bill of Materials
- "Cycle time" → Time from order to delivery
- "OEE" → Overall Equipment Effectiveness

Without business terms:
User: "What's the OEE for Line 3?"
Copilot: "I'm not sure what OEE means."

With business terms:
User: "What's the OEE for Line 3?"
Copilot: "The Overall Equipment Effectiveness for Production Line 3 is currently 87%."
```

**Healthcare Organization:**
```
Business Terms:
- "EOB" → Explanation of Benefits
- "Prior auth" → Prior authorization
- "Admit date" → Patient admission date
- "DRG" → Diagnosis Related Group
- "LOS" → Length of Stay

Context: Insurance claims processing
```

**Financial Services:**
```
Business Terms:
- "AUM" → Assets Under Management
- "NAV" → Net Asset Value
- "KYC" → Know Your Customer
- "AML" → Anti-Money Laundering
- "Basis point" → 1/100th of a percent
```

### Design Process

**Step 1: Identify Business Vocabulary**
```
Sources:
├─ Industry glossaries
├─ Company style guides
├─ Product catalogs
├─ Training materials
├─ Subject matter expert interviews
└─ Customer support tickets (common questions)

Prioritize:
- High-frequency terms
- Ambiguous terms (multiple meanings)
- Critical business processes
- Customer-facing terminology
```

**Step 2: Define Terms Clearly**
```
Term Definition Template:

Term: [Exact term as users say it]
Definition: [Clear, concise explanation]
Synonyms: [Alternative ways users might say it]
Context: [When/where this term is used]
Examples: [Sample usage in sentences]

Example:
Term: "Hot lead"
Definition: A sales lead with high purchase intent, requiring immediate follow-up
Synonyms: urgent lead, priority prospect, high-intent lead
Context: Used in sales conversations and CRM
Examples:
- "Are there any hot leads in my territory?"
- "Mark this as a hot lead"
- "How many hot leads this week?"
```

**Step 3: Implement in Dynamics 365**
```
Location: Settings → Copilot → Business Terms

Fields:
- Term name
- Definition
- Category (optional)
- Related entities (optional)
- Effective date
- Owner

Configuration:
├─ Import bulk terms via CSV
├─ Version control for changes
├─ Test terms in sandbox
└─ Deploy to production
```

**Step 4: Test and Refine**
```
Testing Checklist:
□ Copilot recognizes term in questions
□ Definitions are used in responses
□ Synonyms work correctly
□ No conflicts with standard terms
□ Performance is acceptable
□ Users find responses helpful

Refinement:
- Monitor usage analytics
- Collect user feedback
- Add missing synonyms
- Clarify ambiguous definitions
- Remove unused terms
```

### Best Practices

**1. Keep Definitions Concise**
```
❌ Too verbose:
"BOM stands for Bill of Materials, which is a comprehensive list of raw materials, components, and instructions required to construct, manufacture, or repair a product or service."

✅ Clear and concise:
"Bill of Materials: A list of parts and quantities needed to build a product."
```

**2. Include Context**
```
❌ No context:
"Net 30: Payment due in 30 days"

✅ With context:
"Net 30: Payment terms requiring full payment within 30 days of invoice date. Commonly used in B2B transactions."
```

**3. Handle Ambiguity**
```
Example: "CR" could mean:
- Credit (Finance)
- Change Request (IT)
- Conversion Rate (Marketing)
- Customer Return (Operations)

Solution: Create separate terms with context
- "CR (finance)" → Credit
- "CR (IT)" → Change Request
- "CR (marketing)" → Conversion Rate
```

**4. Maintain a Glossary**
```
Central repository:
├─ All approved business terms
├─ Definitions and examples
├─ Approval workflow
├─ Version history
├─ Usage guidelines
└─ Review schedule (quarterly)
```

---

## 2. Design Customizations of Copilot in Dynamics 365

### Customization Types

#### A. Copilot in Dynamics 365 Sales

**1. Email Generation Customization**
```
Customize:
├─ Email templates by scenario
│  ├─ Follow-up after demo
│  ├─ Price quote
│  ├─ Objection handling
│  └─ Thank you / closing
│
├─ Tone and style
│  ├─ Formal vs. casual
│  ├─ Industry-appropriate
│  └─ Brand voice
│
└─ Required elements
   ├─ Legal disclaimers
   ├─ Company signature
   └─ Call-to-action

Configuration:
- Provide example emails for each type
- Define brand guidelines
- Set up approval workflows for sensitive content
```

**2. Meeting Preparation Customization**
```
Customize what Copilot surfaces:
├─ Prioritize recent interactions
├─ Highlight risk factors
├─ Show competitor mentions
├─ Include industry news
└─ Suggest talking points

Data sources:
├─ CRM data (opportunities, activities)
├─ Email conversations
├─ Meeting transcripts
├─ External data (news, social media)
└─ Internal knowledge base
```

**3. Opportunity Insights Customization**
```
Custom metrics:
├─ Company-specific health scores
├─ Custom win probability model
├─ Industry-specific risk factors
├─ Territory-specific considerations
└─ Product fit scores

Implementation:
- Define calculation logic
- Configure data inputs
- Set thresholds for alerts
- Customize recommendations
```

#### B. Copilot in Dynamics 365 Customer Service

**1. Case Summarization Customization**
```
Customize summary structure:
├─ Issue description
├─ Steps taken
├─ Current status
├─ Sentiment analysis
├─ Recommended actions
└─ Related cases

Length options:
- Executive summary (2-3 sentences)
- Standard summary (paragraph)
- Detailed summary (full context)

Focus areas:
- Technical details for tech support
- Customer sentiment for escalations
- Resolution steps for knowledge base
```

**2. Knowledge Article Suggestions**
```
Customize ranking:
├─ Relevance to issue
├─ Article freshness
├─ Success rate (how often it solved problem)
├─ Customer feedback rating
└─ Author expertise

Filters:
- Product line
- Customer segment
- Issue severity
- Language
```

**3. Response Generation Customization**
```
Customize response style:
├─ Empathy level (high for complaints)
├─ Technical depth (based on customer profile)
├─ Length (quick ack vs. detailed)
└─ Formality (B2B vs. B2C)

Templates by scenario:
- First response to new case
- Status update
- Resolution confirmation
- Escalation notification
- Follow-up after resolution

Guardrails:
- Never promise specific timelines without approval
- Always offer escalation path
- Include relevant policies
- Maintain brand voice
```

#### C. Copilot in Dynamics 365 Finance & Operations

**1. Invoice Processing Customization**
```
Customize extraction:
├─ Vendor-specific formats
├─ Multi-currency handling
├─ Tax calculation rules
├─ Approval routing logic
└─ Exception handling

Validation rules:
- PO matching requirements
- Budget availability checks
- Vendor verification
- Amount thresholds
```

**2. Collections Predictions Customization**
```
Custom risk factors:
├─ Payment history weight
├─ Industry risk profile
├─ Economic indicators
├─ Relationship strength
└─ Account age

Actions:
- Recommended collection strategies
- Optimal contact timing
- Communication channel selection
- Escalation triggers
```

### Customization Implementation Process

**Step 1: Assess Current State**
```
Evaluate:
- What works well with default Copilot?
- What are common user complaints?
- Where does Copilot provide inaccurate responses?
- What business processes need customization?

Document:
- Current workflows
- Pain points
- Desired outcomes
- Success metrics
```

**Step 2: Design Customizations**
```
For each customization:
1. Define objective
2. Specify inputs (data, context)
3. Design output format
4. Set quality criteria
5. Plan testing approach

Example: Email Follow-up Customization
Objective: Generate contextual follow-up emails after customer meetings
Inputs: Meeting transcript, CRM data, previous emails
Output: 3-paragraph email with action items
Quality: Professional tone, accurate facts, clear next steps
Testing: 20 sample scenarios, sales team review
```

**Step 3: Configure in Dynamics 365**
```
Configuration options:
├─ Settings & Administration
│  └─ Copilot settings
│
├─ Customization tools
│  ├─ Power Apps maker portal
│  ├─ Solution management
│  └─ Environment variables
│
└─ Integration points
   ├─ Dataverse customization
   ├─ Power Automate flows
   └─ Custom connectors
```

**Step 4: Test and Validate**
```
Testing phases:
1. Unit testing (individual features)
2. Integration testing (with other systems)
3. User acceptance testing (UAT)
4. Performance testing
5. Security testing

Validation criteria:
- Accuracy of generated content
- Response time acceptable
- User satisfaction >80%
- Error rate <2%
- Adoption rate >60%
```

**Step 5: Deploy and Monitor**
```
Deployment:
├─ Pilot group (10-20% of users)
├─ Gather feedback (2-4 weeks)
├─ Refine based on feedback
├─ Gradual rollout (waves)
└─ Full deployment

Monitoring:
- Usage analytics
- Error rates
- User satisfaction scores
- Business impact metrics
- Continuous improvement backlog
```

---

## 3. Design Connectors for Copilot in Dynamics 365 Sales

### Purpose of Connectors

**What Are Connectors?**
- Integrations that allow Copilot to access external data and systems
- Enable Copilot to take actions in other applications
- Extend Copilot's capabilities beyond Dynamics 365

**Common Use Cases:**
```
Sales Intelligence:
├─ LinkedIn Sales Navigator → Prospect insights
├─ ZoomInfo → Company and contact data
├─ Clearbit → Firmographic enrichment
└─ DiscoverOrg → Decision-maker information

Communication:
├─ Teams → Meeting insights
├─ Outlook → Email analysis
├─ Slack → Collaboration context
└─ Zoom → Call recordings and transcripts

Documentation:
├─ SharePoint → Sales collateral
├─ OneDrive → Proposal templates
├─ DocuSign → Contract status
└─ PandaDoc → Document analytics

Competitive Intelligence:
├─ Klue → Competitor tracking
├─ Crayon → Win/loss analysis
├─ G2 → Product reviews
└─ News APIs → Industry updates
```

### Connector Design Process

**Step 1: Identify Integration Needs**
```
Questions to answer:
- What external data would help sales teams?
- Which systems do sales reps use daily?
- Where is data manually copied between systems?
- What actions could be automated?

Example needs:
- Pull LinkedIn profile data into Copilot responses
- Check contract status in DocuSign
- Get latest pricing from external system
- Create tasks in project management tool
```

**Step 2: Evaluate Connector Options**
```
Option A: Prebuilt Connectors
- Check Power Platform connector gallery
- 1000+ connectors available
- No development needed
- Limited customization

Option B: Custom Connectors
- Build connector for proprietary systems
- Full control over functionality
- Requires development effort
- Maintained by your team

Option C: API Integration
- Direct API calls in Power Automate
- Flexible but manual
- Good for simple integrations
```

**Step 3: Design Connector Architecture**
```
Architecture Pattern:

Dynamics 365 Sales Copilot
         ↓
   Connector Layer
         ↓
   ┌──────┴──────┐
   ↓             ↓
External API   Authentication
   ↓             (OAuth, API Key)
External System

Components:
├─ Authentication mechanism
├─ API endpoint definitions
├─ Request/response mapping
├─ Error handling
├─ Rate limiting
└─ Caching strategy
```

**Step 4: Define Data Mapping**
```
Map external data to Dynamics 365 entities:

Example: LinkedIn Sales Navigator Connector

LinkedIn Field → Dynamics 365 Field
────────────────────────────────────
Company Name → Account.Name
Industry → Account.Industry
Employee Count → Account.NumberOfEmployees
Recent News → Account.Description (append)
Decision Makers → Contacts (create/update)
Engagement Signals → Activities (create)

Transformation rules:
- Standardize industry names
- Format phone numbers
- De-duplicate contacts
- Enrich with additional data
```

**Step 5: Configure Copilot Integration**
```
Integration points:

1. Automatic enrichment:
   - Trigger: New account created
   - Action: Fetch LinkedIn data
   - Update: Dynamics 365 fields

2. On-demand queries:
   - User asks: "Show me recent news about this account"
   - Copilot: Calls connector
   - Returns: Formatted news summaries

3. Suggested actions:
   - Copilot detects: Key decision maker changed
   - Suggests: "Update contact information?"
   - Executes: When user confirms
```

### Connector Security Design

**Authentication Methods:**
```
OAuth 2.0 (Recommended):
├─ User consent flow
├─ Token-based access
├─ Refresh tokens
├─ Scope-limited permissions
└─ Revocable access

API Key:
├─ Simple implementation
├─ Less secure (static key)
├─ Good for internal systems
└─ Requires key management

Service Principal:
├─ App-level authentication
├─ No user interaction
├─ Good for background processes
└─ Azure AD integration
```

**Data Security:**
```
Considerations:
├─ Data in transit: HTTPS/TLS encryption
├─ Data at rest: Encrypted storage if cached
├─ Access control: Role-based permissions
├─ Audit logging: Track all connector usage
├─ Data residency: Comply with regulations
└─ PII handling: Anonymize when possible
```

**Error Handling:**
```
Scenarios to handle:
├─ External API is down
│  └─ Fallback: Use cached data or graceful degradation
│
├─ Rate limit exceeded
│  └─ Retry with exponential backoff
│
├─ Authentication failure
│  └─ Prompt user to reauthorize
│
├─ Invalid data format
│  └─ Log error, notify admin, skip field
│
└─ Timeout
   └─ Cancel request, inform user
```

### Example: LinkedIn Sales Navigator Connector

**Functional Requirements:**
```
Capabilities:
1. Fetch account insights
   - Company overview
   - Employee count and growth
   - Recent news and updates
   - Similar companies

2. Get contact information
   - Decision makers
   - Job titles and tenure
   - Contact details
   - Engagement signals

3. Track engagement
   - Profile views
   - InMail responses
   - Connection requests
   - Content interactions

4. Suggested actions
   - When to reach out
   - Talking points
   - Mutual connections
   - Recent job changes
```

**Technical Design:**
```
Connector Configuration:

Base URL: https://api.linkedin.com/v2/
Authentication: OAuth 2.0

Endpoints:
├─ /organizations/{id} → Company data
├─ /people/{id} → Contact data
├─ /sales-navigator/insights → Intelligence
└─ /ugcPosts → Recent activity

Triggers:
├─ Account view in Dynamics 365
├─ Opportunity stage change
├─ Manual user request
└─ Scheduled refresh (daily)

Actions:
├─ Enrich account data
├─ Find decision makers
├─ Get recent news
└─ Track engagement

Data flow:
1. User opens account in Dynamics 365
2. Copilot detects LinkedIn URL in account
3. Calls connector to fetch insights
4. Displays formatted insights in Copilot panel
5. Offers to update Dynamics 365 fields
```

**Implementation:**
```
Power Platform custom connector:

1. Create connector definition
   - Import OpenAPI spec
   - Configure authentication
   - Test connection

2. Add to solution
   - Package connector
   - Add dependencies
   - Deploy to environments

3. Configure in Copilot Studio
   - Add connector action
   - Map inputs/outputs
   - Design conversation flow

4. Test end-to-end
   - Unit tests for each endpoint
   - Integration test with Dynamics 365
   - User acceptance testing
```

---

## 4. Design Agents for Integration with Dynamics 365 Contact Center

### Contact Center Integration Patterns

**Omnichannel Architecture:**
```
Customer Channels:
├─ Phone (Voice)
├─ Chat (Website)
├─ SMS/Text
├─ Email
├─ Social Media (Facebook, Twitter)
├─ WhatsApp
└─ Microsoft Teams

          ↓
    
Dynamics 365 Contact Center
(Omnichannel routing and queuing)

          ↓

┌────────┴────────┐
↓                 ↓
AI Agents      Human Agents
(Automation)    (Complex cases)
```

### Agent Design for Contact Center

#### 1. Virtual Agent (First Line)

**Purpose:** Handle common inquiries without human intervention

**Capabilities:**
```
Self-Service Functions:
├─ Account balance inquiry
├─ Order status check
├─ Password reset
├─ FAQ responses
├─ Appointment scheduling
├─ Product information
└─ Return/exchange requests

Design considerations:
- Must authenticate customer
- Access to customer data (Dataverse)
- Ability to create cases
- Escalation to human when needed
- Multi-language support
```

**Conversation Flow Example:**
```
Bot: "Hi! I'm here to help. What can I do for you today?"

Customer: "I need to check my order status"

Bot: [Authenticates customer via email/phone]
     "I found your recent order #12345 for $299.99. 
      It shipped yesterday and will arrive on Nov 10. 
      Would you like tracking details?"

Customer: "Yes please"

Bot: [Provides tracking link and details]
     "Anything else I can help with today?"

Customer: "No, thanks"

Bot: "Great! Please rate this interaction."
     [Thumbs up/down feedback]
```

#### 2. Agent Assist (Human Support)

**Purpose:** Provide real-time assistance to human agents

**Capabilities:**
```
Real-Time Assistance:
├─ Suggest knowledge articles
├─ Draft responses
├─ Summarize case history
├─ Recommend next actions
├─ Sentiment analysis
├─ Compliance checks
└─ Auto-fill forms

UI Integration:
- Side panel in agent workspace
- Contextual suggestions
- Quick actions
- Case insights
```

**Example Scenario:**
```
Human agent receives call from upset customer

Agent Assist automatically:
1. Shows customer history (recent cases, purchases)
2. Displays sentiment: "High frustration detected"
3. Suggests empathy response template
4. Recommends similar resolved cases
5. Highlights relevant policy (return window)
6. Prepares draft resolution

Agent uses suggestions to resolve quickly and effectively
```

#### 3. Routing Agent

**Purpose:** Intelligently route inquiries to best resource

**Routing Logic:**
```
Factors to consider:
├─ Issue type (technical, billing, general)
├─ Customer tier (VIP, standard)
├─ Language preference
├─ Agent skills and availability
├─ Current queue lengths
├─ Case complexity
└─ Customer sentiment

Decision tree:
1. Can virtual agent handle it?
   ├─ Yes → Route to virtual agent
   └─ No → Continue

2. What skill required?
   ├─ Technical → Technical support queue
   ├─ Billing → Billing specialist queue
   └─ General → General support queue

3. Customer tier?
   ├─ VIP → Priority queue
   └─ Standard → Normal queue

4. Agent availability?
   ├─ Available → Assign immediately
   └─ Busy → Queue with estimated wait time
```

### Integration Design

#### Data Integration
```
Dynamics 365 Contact Center ← → Data Sources

Data flows:
├─ Customer profile (Dataverse)
├─ Order history (ERP)
├─ Support history (Case management)
├─ Knowledge base (SharePoint)
├─ Product catalog (Dynamics 365 Sales)
└─ Custom systems (APIs)

Real-time access:
- Agent needs data immediately
- Latency < 2 seconds
- Fallback if source unavailable
```

#### Channel Integration
```
Each channel requires specific handling:

Voice (Phone):
├─ Speech-to-text for transcription
├─ Real-time sentiment analysis
├─ Automatic call summarization
└─ Post-call wrap-up automation

Chat (Website, Teams):
├─ Typing indicators
├─ Rich media support (images, files)
├─ Persistent conversation history
└─ Chatbot-to-human handoff

SMS/WhatsApp:
├─ Character limits
├─ Async nature (delayed responses OK)
├─ Media sharing
└─ Template messages for notifications

Email:
├─ Email parsing and classification
├─ Automatic case creation
├─ Attachment handling
└─ Threaded conversations
```

#### Escalation Design
```
Escalation triggers:
├─ Customer explicitly requests human
├─ Agent confidence score < 60%
├─ Multiple failed attempts
├─ High sentiment negativity
├─ Complex issue detected
└─ Timeout (too long in bot)

Escalation process:
1. Inform customer of transfer
2. Summarize conversation for human agent
3. Transfer context (history, customer info)
4. Find best available agent
5. Introduce human agent
6. Virtual agent available for assist
```

### Performance Metrics

**Agent Metrics:**
```
Containment Rate:
- % of inquiries resolved by virtual agent
- Target: 60-70% for common issues

Average Handle Time:
- Time to resolve inquiry
- Target: < 3 minutes for self-service

First Contact Resolution:
- % resolved in single interaction
- Target: > 80%

Customer Satisfaction:
- Post-interaction rating
- Target: > 4.0/5.0

Escalation Rate:
- % transferred to human
- Target: < 30%
```

**Channel-Specific Metrics:**
```
Voice:
- Call abandonment rate
- Average speed to answer
- Call duration

Chat:
- Response time
- Concurrent conversations per agent
- Chat duration

Email:
- Response time (first and subsequent)
- Resolution time
- Email volume

SMS:
- Delivery rate
- Response rate
- Opt-out rate
```

---

## Study Resources

- [Dynamics 365 Copilot Customization](https://learn.microsoft.com/en-us/dynamics365/sales/copilot-overview)
- [Power Platform Connectors](https://learn.microsoft.com/en-us/connectors/)
- [Dynamics 365 Contact Center](https://learn.microsoft.com/en-us/dynamics365/contact-center/)
- [Copilot Studio Integration](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)

**Next Topic:** [Design Extensibility of AI Solutions](02-extensibility.md)
