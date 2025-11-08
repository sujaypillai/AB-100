# Prebuilt Agents, Custom Models, and Prompt Engineering

## Overview
This section covers decisions around using prebuilt agents vs. building custom solutions, when to create custom AI models, prompt library management, and prompt engineering best practices.

---

## 1. Use Cases for Prebuilt Agents

### Microsoft 365 Copilot Agents

#### Built-in Microsoft 365 Copilot Capabilities
**Core Functionality:**
- Summarize emails, documents, meetings
- Draft content (emails, documents, presentations)
- Analyze Excel data and create visualizations
- Generate PowerPoint presentations from documents
- Catch up on Teams chats and meetings
- Search across M365 content

**When to Use:**
✅ Users already have M365 Copilot licenses
✅ Standard productivity tasks
✅ Working with M365 content (emails, docs, meetings)
✅ No special business logic required

#### Microsoft 365 Copilot for Sales
**Capabilities:**
- CRM data in email and Teams
- Meeting preparation with account insights
- Email summaries with action items
- Opportunity tracking and updates
- Call recordings and insights

**Integration:**
- Dynamics 365 Sales
- Salesforce
- Works in Outlook, Teams

**When to Use:**
✅ Sales team uses M365 + CRM
✅ Need CRM context in daily tools
✅ Want to reduce CRM data entry
✅ Improve sales productivity

#### Microsoft 365 Copilot for Service
**Capabilities:**
- Case summarization
- Draft responses using knowledge base
- Suggest knowledge articles
- Escalation recommendations
- Sentiment analysis

**Integration:**
- Dynamics 365 Customer Service
- Salesforce Service Cloud
- Works in Outlook, Teams

**When to Use:**
✅ Customer service team uses M365
✅ Need quick access to support knowledge
✅ Want to speed up case resolution
✅ Improve response quality

### Dynamics 365 Copilot Agents

#### Copilot in Dynamics 365 Sales
**Capabilities:**
- Lead and opportunity insights
- Email generation
- Meeting preparation
- Pipeline analysis
- Next best actions
- Forecast predictions

**When to Use:**
✅ Using Dynamics 365 Sales
✅ Want AI-powered sales insights
✅ Need help with sales processes
✅ Improve win rates and productivity

#### Copilot in Dynamics 365 Customer Service
**Capabilities:**
- Case summarization and resolution
- Knowledge article suggestions
- Draft responses
- Sentiment analysis
- Conversation intelligence

**When to Use:**
✅ Using Dynamics 365 Customer Service
✅ High volume of support cases
✅ Need to improve response times
✅ Want consistent service quality

#### Copilot in Dynamics 365 Finance & Supply Chain
**Capabilities:**
- Collections predictions
- Cash flow forecasts
- Budget planning assistance
- Intelligent order promising
- Supply chain disruption alerts

**When to Use:**
✅ Using D365 Finance or Supply Chain
✅ Need predictive analytics
✅ Want to optimize operations
✅ Improve financial planning

### Power Platform AI Builder Models

#### Prebuilt AI Models
**Form Processing:**
- Extract data from invoices, receipts, ID cards
- Pre-trained on common document types
- No training required

**Object Detection:**
- Identify products in images
- Count items
- Quality inspection

**Sentiment Analysis:**
- Analyze text sentiment (positive/negative/neutral)
- Works with multiple languages

**Key Phrase Extraction:**
- Identify important topics in text
- Summarization support

**Language Detection:**
- Identify language of text
- Support for 100+ languages

**Text Recognition (OCR):**
- Extract text from images
- Handwriting recognition

**When to Use:**
✅ Common AI tasks
✅ Need quick implementation
✅ Limited AI expertise
✅ Budget constraints
✅ Standard accuracy acceptable

### Decision Framework: Prebuilt vs. Custom

```
Decision Tree:

Does a prebuilt agent/model exist for your use case?
├─ Yes → Does it meet 80%+ of requirements?
│         ├─ Yes → Use prebuilt agent
│         │         └─ Consider: Configure/extend if needed
│         └─ No → Can you extend it to meet requirements?
│                   ├─ Yes → Extend prebuilt agent
│                   └─ No → Build custom agent
│
└─ No → Build custom agent
          └─ Consider: Combine prebuilt components where possible
```

**Evaluation Matrix:**

| Criteria | Prebuilt | Custom |
|----------|----------|--------|
| **Time to Deploy** | Days-Weeks | Months |
| **Cost (Initial)** | Low | High |
| **Cost (Ongoing)** | Subscription | Maintenance |
| **Customization** | Limited | Full control |
| **Expertise Required** | Admin/Power User | Developer |
| **Risk** | Low | Medium-High |
| **Competitive Advantage** | No | Potentially |
| **Vendor Support** | Yes | Self-managed |

---

## 2. When to Build Custom AI Models

### Scenarios Requiring Custom Models

#### 1. Domain Specialization
**Problem:** Foundation models lack specialized knowledge

**Example Scenarios:**
- Medical diagnosis with rare conditions
- Legal contract analysis for specific jurisdiction
- Manufacturing defect detection for unique products
- Financial fraud patterns specific to your business
- Scientific research in niche field

**Indicators:**
- Industry-specific terminology not understood
- Context requires deep domain knowledge
- Existing models produce inaccurate results
- Competitive advantage from specialized capability

**Approach:**
```
Option A: Fine-tune foundation model
- Start with GPT-4, Llama, etc.
- Train on domain-specific data
- Faster than training from scratch

Option B: Build specialized model
- Use Azure AI Services (Custom Vision, Custom NER)
- Train on curated dataset
- Deploy in Azure AI Foundry
```

#### 2. Data Privacy and Compliance
**Problem:** Cannot send sensitive data to external APIs

**Example Scenarios:**
- Healthcare: HIPAA-protected patient data
- Finance: PCI-DSS payment data
- Legal: Attorney-client privileged information
- Government: Classified information
- HR: Employee personal information

**Indicators:**
- Regulatory requirements prohibit data sharing
- Data residency requirements
- Customer contracts specify data handling
- Risk appetite is very low

**Approach:**
```
Self-hosted model:
1. Deploy model in your Azure tenant
2. Use Private Endpoints (no internet exposure)
3. Keep all data within your boundary
4. Full control over data processing

Options:
- Azure OpenAI (deployed in your subscription)
- Azure AI Foundry (custom model deployment)
- On-premises deployment (edge scenarios)
```

#### 3. Performance Requirements
**Problem:** Need lower latency or higher throughput

**Example Scenarios:**
- Real-time fraud detection (milliseconds)
- High-volume transaction processing
- Edge computing (offline capability)
- Cost optimization for massive scale

**Indicators:**
- Latency requirement < 100ms
- Millions of requests per day
- Network connectivity unreliable
- API costs prohibitive at scale

**Approach:**
```
Optimization Strategies:

Small Language Models (SLMs):
- Phi-3, Llama 3.2 (small variants)
- Deploy closer to users (edge)
- Lower compute requirements
- Faster inference

Model Optimization:
- Quantization (reduce precision)
- Pruning (remove unnecessary weights)
- Distillation (teach smaller model)
- Caching (common queries)
```

#### 4. Unique Capabilities
**Problem:** Need output format or capability not available

**Example Scenarios:**
- Generate code in proprietary framework
- Output in specific format (CAD files, music)
- Multi-modal inputs unique to industry
- Proprietary algorithms embedded

**Indicators:**
- No existing model produces required output
- Specialized training data available
- Competitive differentiator
- Willingness to invest in R&D

**Approach:**
```
Custom Development:
1. Define training data requirements
2. Select base architecture or build new
3. Train and validate model
4. Deploy in Azure AI Foundry
5. Continuous improvement pipeline
```

### Custom Model Development Decision Matrix

| Factor | Use Foundation Model | Fine-tune | Build Custom |
|--------|---------------------|-----------|--------------|
| **Budget** | $ | $$ | $$$$ |
| **Timeline** | Weeks | Months | 6+ months |
| **Expertise** | Prompt engineering | ML engineering | AI research |
| **Data Required** | Examples only | Thousands | Millions |
| **Accuracy Need** | Good enough | High | Specialized |
| **Maintenance** | Minimal | Moderate | Significant |

---

## 3. Prompt Library Guidelines

### Why a Prompt Library?

**Benefits:**
- **Consistency:** Same quality across users/agents
- **Reusability:** Don't recreate prompts
- **Version Control:** Track what works
- **Knowledge Sharing:** Learn from best prompts
- **Governance:** Review and approve prompts
- **Optimization:** Continuously improve

### Prompt Library Structure

```
/PromptLibrary/
├── README.md (usage guidelines)
├── /Sales/
│   ├── lead-qualification.md
│   ├── email-response-positive.md
│   ├── email-response-objection.md
│   ├── proposal-generation.md
│   └── meeting-summary.md
│
├── /CustomerService/
│   ├── case-summarization.md
│   ├── empathy-response.md
│   ├── troubleshooting-guide.md
│   └── escalation-notice.md
│
├── /Marketing/
│   ├── content-ideas.md
│   ├── social-media-post.md
│   ├── email-campaign.md
│   └── audience-analysis.md
│
├── /Operations/
│   ├── data-analysis.md
│   ├── report-generation.md
│   └── process-documentation.md
│
└── /Templates/
    ├── generic-analysis.md
    ├── content-generation.md
    ├── data-extraction.md
    └── summarization.md
```

### Prompt Template Format

```markdown
# Prompt Name

## Purpose
Brief description of what this prompt accomplishes

## Use Cases
- Scenario 1
- Scenario 2

## Prerequisites
- Data/context required
- Permissions needed
- Systems accessed

## Parameters
| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| {{customer_name}} | String | Yes | Customer company name | "Acme Corp" |
| {{product}} | String | Yes | Product being discussed | "Enterprise Plan" |
| {{context}} | Text | No | Additional background | "Renewal opportunity" |

## Prompt Template

```
You are a {{role}} helping with {{task}}.

Context:
{{context}}

Task:
{{specific_instructions}}

Output Format:
{{format_requirements}}

Constraints:
{{limitations}}
```

## Example Input
```json
{
  "role": "sales assistant",
  "task": "drafting follow-up email",
  "customer_name": "Acme Corp",
  "product": "Enterprise Plan",
  "context": "Customer requested pricing after demo"
}
```

## Example Output
```
Subject: Enterprise Plan Pricing for Acme Corp

Hi [Customer Name],

Thank you for your interest in our Enterprise Plan...
[rest of email]
```

## Performance Metrics
- Average response quality: 4.2/5
- Token usage: ~500 tokens
- Estimated cost: $0.01 per use
- Success rate: 85%

## Version History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-01-15 | J. Smith | Initial version |
| 1.1 | 2025-02-20 | A. Jones | Added output format constraints |

## Approval
- Reviewed by: [Name]
- Approved by: [Name]
- Date: [Date]
- Next Review: [Date]

## Notes
Any special considerations, limitations, or tips
```

### Prompt Library Management

**Version Control:**
```bash
# Store in Git repository
git add prompts/sales/lead-qualification.md
git commit -m "Improved lead qualification prompt clarity"
git push origin main

# Tag releases
git tag -a v1.0 -m "Initial prompt library release"
```

**Governance Process:**
```
1. Creation: Author drafts prompt
2. Testing: Validate with sample data
3. Review: Peer review for quality
4. Approval: Manager/AI CoE approves
5. Publishing: Add to library
6. Monitoring: Track performance
7. Iteration: Update based on feedback
```

**Access Control:**
```
├── Public Prompts (all users)
├── Department Prompts (team-specific)
├── Restricted Prompts (approved users only)
└── Experimental Prompts (testing only)
```

### Best Practices

**1. Parameterization:**
```
❌ Bad: Hardcoded values
"Create a sales email to Acme Corp about Product X"

✅ Good: Parameterized
"Create a sales email to {{customer_name}} about {{product_name}}"
```

**2. Clear Instructions:**
```
❌ Bad: Vague
"Write something about the product"

✅ Good: Specific
"Write a 3-paragraph product description that:
1. Highlights the main benefit
2. Lists 3 key features
3. Includes a call-to-action"
```

**3. Output Format:**
```
❌ Bad: No format specified
"Analyze this data"

✅ Good: Format defined
"Analyze this data and output as JSON:
{
  'summary': '...',
  'top_insights': [...],
  'recommendations': [...]
}"
```

**4. Examples (Few-Shot):**
```
Include 2-3 examples in prompt:

"Example 1:
Input: [sample]
Output: [expected result]

Example 2:
Input: [sample]
Output: [expected result]

Now process: [actual input]"
```

---

## 4. Customized Small Language Models (SLMs)

### What Are SLMs?

**Comparison:**
| Aspect | Small Language Model | Large Language Model |
|--------|---------------------|---------------------|
| **Parameters** | 1-7 billion | 70-175+ billion |
| **Training Cost** | Thousands | Millions |
| **Inference Cost** | Low | High |
| **Latency** | Fast (<100ms) | Slower (500ms+) |
| **Deployment** | CPU, small GPU, edge | Large GPU clusters |
| **Use Cases** | Focused tasks | General purpose |
| **Examples** | Phi-3, Llama 3.2-3B | GPT-4, Claude, Llama 3.1-70B |

### When to Use SLMs

**Ideal Scenarios:**

**1. Classification Tasks:**
```
Examples:
- Sentiment analysis
- Intent detection
- Category assignment
- Priority scoring
- Spam detection

Why SLM works:
- Limited output space (finite classes)
- Fast inference needed
- High volume of requests
- Cost-sensitive application
```

**2. Edge Deployment:**
```
Examples:
- Mobile apps (on-device AI)
- IoT devices
- Offline scenarios
- Retail kiosks
- Manufacturing floor

Why SLM works:
- Limited compute resources
- No network connectivity
- Privacy (data stays on device)
- Low latency requirements
```

**3. High-Volume/Low-Cost:**
```
Examples:
- Processing millions of transactions
- Real-time filtering
- Automated tagging
- Simple data extraction

Why SLM works:
- Cost per request is critical
- Task is well-defined
- Accuracy requirements moderate
```

**4. Domain-Specific Fine-Tuned:**
```
Examples:
- Medical coding from clinical notes
- Legal document classification
- Financial report analysis
- Manufacturing quality assessment

Why SLM works:
- Fine-tuned on domain data
- Narrow task scope
- Competitive performance vs LLM
- Lower operational cost
```

### SLM Development Process

**Step 1: Define Scope**
```
Questions to Answer:
- What is the specific task?
- What are the inputs and outputs?
- What accuracy is required?
- What is the expected volume?
- What are latency requirements?

Example:
Task: Classify customer support emails
Input: Email text
Output: Category (billing, technical, sales, other)
Accuracy: >90%
Volume: 10,000 emails/day
Latency: <100ms
```

**Step 2: Data Collection**
```
Requirements:
- High-quality labeled data
- Representative of real-world inputs
- Balanced across classes
- Sufficient volume (thousands-hundreds of thousands)

Example:
- Collect 50,000 historical support emails
- Label with correct category
- Balance classes (25% each category)
- Split: 70% train, 15% validation, 15% test
```

**Step 3: Model Selection**
```
Popular SLM Options:
- Phi-3 (Microsoft): 3.8B parameters, strong reasoning
- Llama 3.2 (Meta): 1B, 3B variants
- Gemma (Google): 2B, 7B variants
- Mistral 7B: Open-source, commercial use

Selection Criteria:
- Size fits deployment target
- License allows use case
- Pre-training aligns with domain
- Community/vendor support
```

**Step 4: Fine-Tuning**
```
Process in Azure AI Foundry:

1. Upload training data
2. Select base model
3. Configure training:
   - Learning rate
   - Batch size
   - Number of epochs
   - Evaluation metrics
4. Monitor training
5. Evaluate on validation set
6. Iterate if needed
```

**Step 5: Evaluation**
```
Compare against:
- Baseline (rule-based system)
- LLM performance (GPT-4)
- Human accuracy

Metrics:
- Accuracy
- Precision/Recall/F1
- Latency
- Cost per inference
- Resource utilization

Decision:
If SLM performance is within 5% of LLM but 10x cheaper → Use SLM
```

**Step 6: Optimization**
```
Techniques:
- Quantization: Reduce precision (FP32 → INT8)
- Pruning: Remove unnecessary weights
- Distillation: Teach smaller model from larger one
- ONNX Runtime: Optimize inference

Result: Faster, smaller model with minimal accuracy loss
```

**Step 7: Deployment**
```
Deployment Options:

Cloud:
- Azure AI Foundry managed endpoint
- Azure Container Instances
- Azure Kubernetes Service

Edge:
- Azure IoT Edge
- ONNX Runtime on device
- Windows ML / CoreML

Monitoring:
- Latency, throughput
- Accuracy over time (drift)
- Resource utilization
- Cost tracking
```

---

## 5. Prompt Engineering Guidelines and Techniques

### Core Principles

**1. Clarity**
```
❌ Vague:
"Tell me about the customer"

✅ Clear:
"Provide a summary of customer 'Acme Corp' including:
- Industry and size
- Products purchased
- Last interaction date
- Current account status
- Open opportunities"
```

**2. Context**
```
❌ No context:
"Draft an email"

✅ With context:
"You are a customer success manager. Draft a follow-up email to a customer who just completed onboarding. The customer is in the healthcare industry and purchased our Enterprise plan. Tone should be professional and supportive."
```

**3. Constraints**
```
❌ No constraints:
"Summarize this document"

✅ With constraints:
"Summarize this 10-page document in exactly 3 bullet points. Each bullet point should be one sentence. Focus only on action items."
```

**4. Format Specification**
```
❌ Unspecified:
"Analyze sales data"

✅ Specified:
"Analyze sales data and provide output as a JSON object with this structure:
{
  'total_revenue': number,
  'top_products': [array of strings],
  'growth_rate': percentage,
  'recommendations': [array of strings]
}"
```

### Prompt Engineering Techniques

#### Technique 1: Zero-Shot Prompting
**Definition:** Task instruction without examples

**When to use:**
- Task is straightforward
- Model has general knowledge of task
- Need quick results

**Example:**
```
Prompt: "Classify the sentiment of this customer review as positive, negative, or neutral:

Review: 'The product arrived on time and works as expected. Good value for money.'

Sentiment:"

Response: "Positive"
```

**Pros:**
- Fast to create
- No example data needed
- Works for common tasks

**Cons:**
- May not understand specific requirements
- Inconsistent output format
- Lower accuracy on complex tasks

#### Technique 2: Few-Shot Prompting
**Definition:** Provide examples before the actual task

**When to use:**
- Need specific output format
- Task has nuances
- Want more consistent results

**Example:**
```
Prompt:
"Classify customer support tickets by urgency.

Example 1:
Ticket: 'Cannot log in to my account'
Urgency: High

Example 2:
Ticket: 'How do I change my password?'
Urgency: Low

Example 3:
Ticket: 'Payment processing error - order not completed'
Urgency: Critical

Now classify:
Ticket: 'Would like to upgrade my plan'
Urgency:"

Response: "Low"
```

**Pros:**
- More consistent format
- Better accuracy
- Can teach specific patterns

**Cons:**
- Longer prompts (more tokens/cost)
- Need good examples
- May overfit to examples

#### Technique 3: Chain-of-Thought (CoT)
**Definition:** Ask model to show reasoning steps

**When to use:**
- Complex reasoning required
- Multi-step problems
- Want to verify logic
- Improve accuracy on hard problems

**Example:**
```
Prompt:
"Analyze this sales opportunity and recommend whether to prioritize it. Show your reasoning step-by-step.

Opportunity Details:
- Company: TechStart Inc
- Industry: Software
- Deal Size: $50,000
- Stage: Proposal sent
- Last contact: 2 weeks ago
- Competitor: Yes
- Budget approved: No

Think through:
1. Deal value assessment
2. Stage and momentum
3. Risk factors
4. Likelihood to close
5. Recommendation

Analysis:"

Response:
"1. Deal value assessment: $50K is moderate size...
2. Stage and momentum: Proposal stage is mid-funnel...
[reasoning for each step]
5. Recommendation: Medium priority..."
```

**Pros:**
- Better accuracy on complex tasks
- Explainable decisions
- Can catch errors in reasoning

**Cons:**
- More tokens (higher cost)
- Slower response
- May be verbose

#### Technique 4: Retrieval-Augmented Generation (RAG)
**Definition:** Retrieve relevant documents, inject into prompt

**When to use:**
- Need factual accuracy
- Large knowledge base
- Answers not in model's training
- Updatable knowledge

**Process:**
```
1. User asks question
2. System searches knowledge base (semantic search)
3. Retrieve top N relevant documents
4. Inject documents into prompt
5. Model generates answer grounded in documents
6. Cite sources in response
```

**Example:**
```
User Question: "What is our return policy?"

Retrieved Documents:
[Doc 1: Return Policy - 30 days, Doc 2: Warranty Info]

Prompt to Model:
"Answer the user's question based ONLY on the following documents. Cite the source.

Documents:
[Document 1]: Customers may return items within 30 days...
[Document 2]: Product warranty covers defects...

User Question: What is our return policy?

Answer:"

Response: "According to Document 1, customers may return items within 30 days..."
```

**Pros:**
- Factually accurate
- Can update knowledge without retraining
- Provides citations
- Reduces hallucinations

**Cons:**
- Requires search infrastructure
- Longer prompts
- Quality depends on retrieval
- More complex to implement

#### Technique 5: Iterative Refinement
**Definition:** Use model output as input for next iteration

**When to use:**
- Complex generation tasks
- Quality improvement needed
- Multi-stage processes

**Example:**
```
Iteration 1 (Draft):
Prompt: "Write a product description for enterprise software"
Output: [initial draft]

Iteration 2 (Improve):
Prompt: "Improve this description to be more compelling: [initial draft]"
Output: [better version]

Iteration 3 (Optimize):
Prompt: "Make this more concise: [better version]"
Output: [final version]
```

#### Technique 6: Role-Based Prompting
**Definition:** Assign a role/persona to the model

**When to use:**
- Want specific expertise
- Need appropriate tone
- Industry-specific responses

**Example:**
```
Prompt:
"You are an experienced financial advisor. A client is concerned about market volatility. Draft a reassuring email that:
- Acknowledges their concern
- Provides perspective on short-term volatility
- Recommends staying the course
- Offers to discuss their portfolio

Keep tone professional, empathetic, and confident."
```

**Pros:**
- Appropriate expertise level
- Consistent tone
- Better quality for specialized tasks

**Cons:**
- May be overly formal/technical
- Requires clear role definition

---

## Study Resources

- [Azure AI Foundry Model Catalog](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/model-catalog)
- [Prompt Engineering Guide](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)
- [AI Builder Prebuilt Models](https://learn.microsoft.com/en-us/ai-builder/prebuilt-overview)
- [Small Language Models Overview](https://azure.microsoft.com/en-us/blog/introducing-phi-3/)

**Next Topic:** [ROI and Cost Evaluation](04-roi-cost-evaluation.md)
