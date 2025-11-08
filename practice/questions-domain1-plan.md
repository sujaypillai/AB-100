# Domain 1: Plan AI-Powered Business Solutions - Practice Questions

## Multiple Choice Questions

### Topic: Analyze Requirements

**Question 1:**
Your company receives 15,000 customer support tickets per month. 70% of tickets are routine questions about order status, product specifications, and return policies. What type of agent is MOST appropriate for this scenario?

A) Data analytics agent only  
B) Task automation agent with decision-making capabilities  
C) Decision-making agent only  
D) No agent - human-only approach is better

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** This scenario requires both task automation (processing routine inquiries automatically) and decision-making capabilities (routing tickets to appropriate queues or escalating complex issues). The high volume (15,000/month) and repetitive nature (70% routine) make it an excellent automation candidate, but the agent needs decision-making logic to handle routing and escalation.

**Why other options are incorrect:**
- A: Data analytics alone won't automate ticket resolution
- C: Decision-making alone won't handle the automation of responses
- D: The volume and repetitive nature strongly justify automation

**Reference:** Domain 1 - Analyze Requirements, Section 1.1-1.3
</details>

---

**Question 2:**
You're evaluating data quality for a sales forecasting agent. Historical sales data shows product names formatted as "Laptop-Pro-15", "laptop pro 15", "LAPTOP PRO 15", and "Lap Top Pro 15" for the same product. Which data quality dimension is MOST affected?

A) Accuracy  
B) Relevance  
C) Timeliness  
D) Cleanliness

<details>
<summary>Answer</summary>

**Correct Answer: D**

**Explanation:** Cleanliness issues include inconsistencies in formatting, naming conventions, and standardization. The same product with multiple name variations is a classic cleanliness problem that will fragment knowledge and reduce search/analysis effectiveness.

**Why other options are incorrect:**
- A: Accuracy relates to correctness, not formatting consistency
- B: Relevance concerns whether data is pertinent to the use case
- C: Timeliness relates to data freshness and currency

**Reference:** Domain 1 - Analyze Requirements, Section 2.4
</details>

---

**Question 3:**
A healthcare provider wants to implement an AI agent that provides treatment recommendations to doctors. The agent will analyze patient history, symptoms, lab results, and medical research. What is the PRIMARY governance consideration?

A) Cost optimization for API calls  
B) Human-in-the-loop requirement for final decisions  
C) Multi-language support for patient communications  
D) Integration with existing scheduling systems

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** For high-stakes decisions like medical treatment recommendations, human oversight is critical. The agent should support decision-making, but doctors must make final decisions. This is especially important for regulatory compliance (HIPAA) and ethical considerations in healthcare.

**Why other options are incorrect:**
- A: While cost matters, patient safety is the primary concern
- C: Language support is important but secondary to decision governance
- D: Integration is a technical consideration, not a governance priority

**Reference:** Domain 1 - Analyze Requirements, Section 1.3 (Decision-Making Agents)
</details>

---

**Question 4:**
Your AI agent requires access to customer data stored in SharePoint, Dataverse, and an on-premises SQL database. Which availability pattern provides the BEST balance of performance and data freshness?

A) Direct access to all sources for every query  
B) Cached access with 24-hour refresh for all sources  
C) Hybrid: Cache SharePoint/Dataverse, direct access to SQL  
D) Full replication of all data to local storage

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** A hybrid approach balances performance and freshness. SharePoint and Dataverse typically contain relatively static reference data (documents, policies) that benefit from caching. The SQL database likely contains transactional data requiring real-time access. This approach optimizes performance while maintaining data currency where needed.

**Why other options are incorrect:**
- A: Direct access to all sources adds latency and depends on source availability
- B: 24-hour cache may be too stale for transactional SQL data
- D: Full replication adds complexity, storage costs, and sync lag

**Reference:** Domain 1 - Analyze Requirements, Section 2.5
</details>

---

**Question 5:**
When organizing data for multiple AI systems (M365 Copilot, Copilot Studio, Azure AI Foundry), what is the PRIMARY benefit of using Microsoft Graph API?

A) Lower cost compared to direct database connections  
B) Unified access to M365 data across multiple AI platforms  
C) Automatic data quality validation  
D) Built-in AI model training capabilities

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Microsoft Graph provides a unified API for accessing M365 data (users, files, emails, calendar, Teams, etc.) with consistent authentication and permissions across platforms. This standardization simplifies integration for multiple AI systems.

**Why other options are incorrect:**
- A: Cost reduction is not the primary benefit
- C: Microsoft Graph doesn't validate data quality automatically
- D: Microsoft Graph is for data access, not model training

**Reference:** Domain 1 - Analyze Requirements, Section 3.4
</details>

---

### Topic: AI Strategy Design

**Question 6:**
Your organization is adopting the Cloud Adoption Framework for AI. In which phase should you define responsible AI principles and governance structure?

A) Strategy  
B) Plan  
C) Ready  
D) Adopt

<details>
<summary>Answer</summary>

**Correct Answer: A**

**Explanation:** The Strategy phase establishes foundational principles, including responsible AI guidelines, governance structures, and ethical frameworks. This must happen before planning specific implementations.

**Why other options are incorrect:**
- B: Plan phase focuses on specific projects and roadmaps
- C: Ready phase prepares the technical environment
- D: Adopt phase implements solutions

**Reference:** Domain 1 - AI Strategy Design, Cloud Adoption Framework
</details>

---

**Question 7:**
Which architectural pattern is MOST appropriate for a scenario where a customer service agent needs to consult multiple specialized agents (billing, technical support, account management) to resolve complex inquiries?

A) Hierarchical orchestration  
B) Peer-to-peer collaboration  
C) Sequential pipeline  
D) Independent parallel processing

<details>
<summary>Answer</summary>

**Correct Answer: A**

**Explanation:** Hierarchical orchestration uses a coordinator agent that routes requests to specialized agents based on the inquiry type. This is ideal when one agent needs to decide which specialized agents to consult and synthesize their responses.

**Why other options are incorrect:**
- B: Peer-to-peer works when agents have equal roles, not when coordination is needed
- C: Sequential pipeline implies fixed order, but routing should be dynamic
- D: Independent parallel doesn't allow synthesis of results

**Reference:** Domain 1 - AI Strategy Design, Multi-Agent Architecture
</details>

---

**Question 8:**
Your company has a policy that all AI agents must be explainable and auditable. Which characteristic should be prioritized in agent design?

A) Maximum token efficiency  
B) Transparency with reasoning traces  
C) Lowest possible latency  
D) Support for the largest language model

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** Explainability and auditability require transparency in how the agent makes decisions. Reasoning traces show the agent's thought process, data sources used, and decision logic, enabling both explanation to users and audit trails.

**Why other options are incorrect:**
- A: Token efficiency is important for cost but doesn't ensure explainability
- C: Latency is a performance metric, not related to explainability
- D: Larger models don't guarantee explainability; may reduce it

**Reference:** Domain 1 - AI Strategy Design, Agentic-First Principles
</details>

---

### Topic: Agents and Models

**Question 9:**
A retail company wants to add AI capabilities to their existing customer service operations. They need to respond to product inquiries, check order status, and process returns. Which solution offers the FASTEST time to value?

A) Build custom models in Azure AI Foundry with Prompt Flow  
B) Deploy Microsoft 365 Copilot for Sales  
C) Create a Copilot Studio bot using prebuilt connectors  
D) Develop a custom application using Azure OpenAI Service

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Copilot Studio with prebuilt connectors provides low-code development with ready-made integrations to common business systems. For standard customer service scenarios with existing data sources, this offers the fastest deployment.

**Why other options are incorrect:**
- A: Custom model development requires significant time and expertise
- B: M365 Copilot for Sales is optimized for sales scenarios, not general customer service
- D: Custom development has longest time to value

**Reference:** Domain 1 - Agents and Models, Prebuilt vs. Custom Solutions
</details>

---

**Question 10:**
When should you consider using a Small Language Model (SLM) instead of a Large Language Model (LLM)?

A) When you need maximum accuracy regardless of cost  
B) For domain-specific tasks with limited scope and latency requirements  
C) When processing complex multi-step reasoning tasks  
D) For general-purpose conversational AI with broad knowledge

<details>
<summary>Answer</summary>

**Correct Answer: B**

**Explanation:** SLMs excel at focused, domain-specific tasks where lower latency and cost are priorities. They can be fine-tuned for specific domains and deployed on smaller infrastructure. They're ideal when the task scope is well-defined.

**Why other options are incorrect:**
- A: Maximum accuracy typically requires LLMs with broader training
- C: Complex reasoning benefits from LLMs' extensive knowledge
- D: General-purpose conversational AI requires LLMs' broad capabilities

**Reference:** Domain 1 - Agents and Models, Small Language Models
</details>

---

### Topic: ROI and Cost Evaluation

**Question 11:**
A company is deciding between building a custom AI solution (12 months, $500K) versus buying a prebuilt solution ($100K annually). The custom solution offers 20% better performance. What is the break-even point?

A) 2 years  
B) 3 years  
C) 5 years  
D) Never breaks even

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** 
- Build: $500K upfront
- Buy: $100K/year
- Break-even: $500K ÷ $100K = 5 years

After 5 years, cumulative buy cost equals build cost. Beyond 5 years, build becomes more cost-effective (assuming minimal maintenance costs). The 20% performance benefit should be weighed against the 5-year payback period.

**Reference:** Domain 1 - ROI and Cost Evaluation, Build vs. Buy Analysis
</details>

---

**Question 12:**
Which cost optimization strategy is MOST effective for an agent that handles both simple FAQs (80% of queries) and complex analysis (20% of queries)?

A) Use the most powerful model for all queries  
B) Use the least expensive model for all queries  
C) Implement model routing based on query complexity  
D) Process all queries in batches to reduce costs

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Model routing directs simple queries to smaller, cheaper models (like GPT-3.5) and complex queries to more powerful models (like GPT-4). This optimizes both cost and performance by matching model capabilities to task requirements.

**Why other options are incorrect:**
- A: Wastes resources on simple queries
- B: May provide poor results for complex queries
- D: Batching doesn't address the cost/performance balance

**Reference:** Domain 1 - ROI and Cost Evaluation, Model Routing Strategy
</details>

---

**Question 13:**
When calculating TCO (Total Cost of Ownership) for an AI solution, which cost component is often UNDERESTIMATED?

A) Azure OpenAI API consumption  
B) Initial development and training  
C) Ongoing maintenance, monitoring, and retraining  
D) Licensing for development tools

<details>
<summary>Answer</summary>

**Correct Answer: C**

**Explanation:** Organizations often focus on upfront costs (development, APIs, licenses) but underestimate ongoing operational costs: monitoring, incident response, model retraining, data quality maintenance, security updates, and user support. These can equal or exceed initial costs over the solution's lifetime.

**Why other options are incorrect:**
- A: API costs are usually well-documented and predictable
- B: Development costs are typically estimated carefully
- D: Tool licensing is generally known upfront

**Reference:** Domain 1 - ROI and Cost Evaluation, TCO Calculation
</details>

---

## Scenario-Based Questions

### Scenario 1: Manufacturing Quality Control

**Context:**
GlobalTech Manufacturing produces 50,000 units daily across 3 production lines. Quality inspectors manually review 5% of units (2,500/day), taking 2 minutes per inspection. Defect rate is 2%. The company wants to implement AI-powered visual inspection.

**Questions:**

**1.1** What type of agent is most appropriate?
<details>
<summary>Answer</summary>
Data analytics agent with decision-making capabilities. The agent needs to analyze visual data (images of products), detect defects using computer vision, and make classification decisions (pass/fail).
</details>

**1.2** Calculate the potential time savings if AI inspects 100% of units in 5 seconds per unit.

<details>
<summary>Answer</summary>

**Current state:**
- 2,500 units inspected manually
- 2 minutes per unit = 5,000 minutes = 83.3 hours/day

**With AI (100% inspection):**
- 50,000 units inspected by AI
- 5 seconds per unit = 250,000 seconds = 69.4 hours/day
- But this replaces only 83.3 human hours, so net increase in inspection time is offset by automation

**Actual savings:** Eliminates 83.3 hours of manual labor daily while increasing inspection coverage from 5% to 100%, improving defect detection from 2% sample to full coverage.

**Annual savings:** ~30 FTE at $50K/year = $1.5M + improved quality/reduced waste
</details>

**1.3** What data quality dimensions are critical for this use case?

<details>
<summary>Answer</summary>

1. **Accuracy:** Training images must correctly label defects vs. good products
2. **Relevance:** Images should represent actual production conditions (lighting, angles, backgrounds)
3. **Cleanliness:** Remove corrupted images, duplicates, mislabeled data
4. **Availability:** Real-time image capture and processing required
5. **Timeliness:** Less critical unless product specifications change frequently
</details>

---

### Scenario 2: Financial Services Chatbot

**Context:**
A bank wants to deploy an AI chatbot for customer inquiries. The chatbot must handle:
- Account balance inquiries (40% of queries)
- Transaction history (30%)
- General banking questions (20%)
- Complex issues requiring human agent (10%)

Security requirements: PCI-DSS compliance, data encryption, audit logging.

**Questions:**

**2.1** Design the agent architecture. Include security considerations.

<details>
<summary>Answer</summary>

**Architecture:**
```
Customer → Authentication (MFA) → Chatbot Agent
                                       ↓
                    ┌──────────────────┼──────────────────┐
                    ↓                  ↓                  ↓
            Simple Queries      Account Services    Human Escalation
            (GPT-3.5 Turbo)     (Secure API)        (Queue + Context)
                                       ↓
                              Core Banking System
                              (Read-only for bot)
```

**Security:**
- Microsoft Entra ID authentication
- Encrypt data in transit (TLS 1.2+) and at rest (AES-256)
- PII detection/redaction in logs
- Rate limiting to prevent abuse
- Audit logging of all interactions
- Session timeouts (10 minutes)
- Read-only access to banking systems
</details>

**2.2** What governance policies should be established?

<details>
<summary>Answer</summary>

1. **Approval Process:** High-risk classification requires board approval
2. **Human Oversight:** Support team monitors random sample of conversations
3. **Escalation Rules:** Clear triggers for human takeover
4. **Data Retention:** 90 days for transcripts, 7 years for audit logs
5. **Regular Reviews:** Quarterly bias assessment, monthly performance review
6. **Incident Response:** Defined process for security breaches or AI errors
7. **User Consent:** Clear disclosure of AI usage and data handling
</details>

---

### Scenario 3: Healthcare Appointment Scheduling

**Context:**
A medical clinic receives 500 appointment requests daily via phone, web, and email. Current manual scheduling takes 5 minutes per appointment. The clinic wants AI automation but must comply with HIPAA.

**Questions:**

**3.1** Assess agent suitability using the framework.

<details>
<summary>Answer</summary>

**Agent Fit Assessment (1-5 scale):**

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Process repeatability | 5 | Appointment scheduling follows standard rules |
| Data availability | 4 | Calendar, patient records, provider schedules available |
| Volume justifies automation | 5 | 500/day × 5 min = 2,500 min/day = significant savings |
| Clear success metrics | 5 | Booking accuracy, time saved, patient satisfaction |
| Limited contextual judgment | 4 | Most scheduling is rule-based, some requires judgment |
| Stakeholder willingness | 3 | Healthcare staff may be cautious about automation |

**Total: 26/30 = Excellent candidate**

**Considerations:**
- HIPAA compliance mandatory
- Need human override for complex cases
- Integration with EHR system required
</details>

**3.2** What HIPAA compliance measures must be implemented?

<details>
<summary>Answer</summary>

1. **Authentication:** Verify patient identity before accessing PHI
2. **Authorization:** Role-based access to patient data
3. **Encryption:** At-rest and in-transit for all PHI
4. **Audit Logging:** Track all PHI access with immutable logs
5. **Business Associate Agreement:** Required with Microsoft (Azure, Office 365)
6. **Minimum Necessary:** Agent accesses only required data fields
7. **Breach Notification:** Process for detection and reporting within 60 days
8. **De-identification:** Remove/mask PHI in analytics and training data
9. **Secure Disposal:** Follow retention policies and secure deletion
10. **Training:** Staff educated on HIPAA requirements and agent usage
</details>

---

## Performance Metrics

Track your progress:

- [ ] Domain 1 - Questions 1-13 completed
- [ ] Scenario 1 completed
- [ ] Scenario 2 completed
- [ ] Scenario 3 completed
- [ ] Score: ____/13 multiple choice
- [ ] Scenarios: ____/3 completed

**Target:** 80% correct (11/13 questions minimum)

---

**Next:** [Domain 2 Practice Questions](questions-domain2-design.md)
