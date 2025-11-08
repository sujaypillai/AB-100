# Advanced Extensibility: MCP, Computer Use, and Agent Behaviors

## Overview
This section covers advanced extensibility topics including Model Context Protocol (MCP), Computer Use automation, agent behaviors (reasoning and voice mode), and optimizing solutions with Microsoft 365 agents.

---

## 1. Design Agent Extensibility with Model Context Protocol (MCP)

### Understanding Model Context Protocol

**What is MCP?**
- Open protocol for connecting AI models to external context and tools
- Standardizes how models access data sources and execute actions
- Enables interoperability between different AI systems
- Developed by Anthropic, adopted by Microsoft

**Architecture:**
```
AI Model/Agent (Client)
        ↓
    MCP Protocol
        ↓
┌───────┴────────┐
│                │
▼                ▼
Context         Tools
Providers       Providers
│                │
└─Data Sources  └─Actions/APIs
```

**Key Concepts:**
```
1. Context Providers:
   - Serve data to models
   - Database queries
   - File access
   - API calls

2. Tool Providers:
   - Enable model actions
   - Send emails
   - Create records
   - Execute commands

3. Protocol Specification:
   - Standard message format
   - Authentication
   - Error handling
   - Streaming support
```

### MCP in Copilot Studio

**Integration Points:**
```
Copilot Studio Agent
        ↓
    MCP Connector
        ↓
┌───────┴────────┐
│                │
▼                ▼
External        External
Data Sources    Actions
```

**Use Cases:**

#### Use Case 1: Custom Database Access
```
Scenario: Agent needs customer data from proprietary database

Without MCP:
- Build custom connector
- Write API wrapper
- Handle authentication
- Map data structures

With MCP:
1. Configure MCP context provider
2. Define query interface
3. Agent automatically accesses data

MCP Configuration:
{
  "name": "customer-database",
  "type": "context_provider",
  "description": "Access customer records",
  "connection": {
    "type": "sql",
    "host": "db.company.com",
    "database": "crm",
    "auth": "azure-ad"
  },
  "schema": {
    "tables": ["customers", "orders", "interactions"],
    "operations": ["read"]
  }
}

Agent Usage:
User: "Show me customer ABC123's order history"
Agent: [Uses MCP to query database]
       "Customer ABC123 has 15 orders totaling $45,000..."
```

#### Use Case 2: Multi-System Integration
```
Scenario: Agent needs data from CRM, ERP, and support systems

MCP Configuration:
contexts:
  - name: crm
    provider: dynamics365
    
  - name: erp
    provider: sap
    
  - name: support
    provider: servicenow

Agent automatically:
- Discovers available contexts
- Queries relevant system based on question
- Combines data from multiple sources

Example:
User: "What's the complete picture for Acme Corp?"
Agent:
  1. Queries CRM via MCP → Account details
  2. Queries ERP via MCP → Order status
  3. Queries Support via MCP → Open tickets
  4. Synthesizes comprehensive response
```

#### Use Case 3: Tool Execution
```
Scenario: Agent needs to perform actions across systems

MCP Tool Provider:
{
  "name": "business-actions",
  "type": "tool_provider",
  "tools": [
    {
      "name": "create_task",
      "description": "Create task in project management system",
      "parameters": {
        "title": "string",
        "description": "string",
        "assignee": "string",
        "due_date": "date"
      }
    },
    {
      "name": "send_notification",
      "description": "Send notification via Teams/email",
      "parameters": {
        "recipient": "string",
        "message": "string",
        "channel": "teams|email"
      }
    }
  ]
}

Agent Usage:
User: "Create a follow-up task for John about the Acme proposal"
Agent:
  1. Uses MCP to call create_task tool
  2. Sets parameters (assignee=John, title=Acme proposal follow-up)
  3. Confirms: "I've created a task for John due tomorrow"
```

### Design Patterns

#### Pattern 1: Semantic Layer
```
Problem: Multiple data sources, different schemas

Solution: MCP provides semantic abstraction

Architecture:
Agent Query (natural language)
        ↓
    MCP Semantic Layer
        ↓
┌───────┴────────┬─────────┐
│                │         │
▼                ▼         ▼
SQL Database   REST API   Files

Benefits:
- Agent doesn't need to know data location
- Schema changes don't break agent
- Add new sources without agent changes
```

#### Pattern 2: Capability Discovery
```
Agent asks MCP: "What can I do?"

MCP Response:
{
  "contexts": [
    {
      "id": "crm",
      "description": "Customer relationship data",
      "capabilities": ["read_customer", "search_accounts"]
    },
    {
      "id": "inventory",
      "description": "Product inventory",
      "capabilities": ["check_stock", "reserve_items"]
    }
  ],
  "tools": [
    {
      "id": "create_order",
      "description": "Create sales order",
      "parameters": [...]
    }
  ]
}

Agent dynamically adapts based on available capabilities
```

#### Pattern 3: Secure Context Sharing
```
Challenge: Different users have different data access

MCP Solution:
1. User authenticates to agent
2. Agent passes user identity to MCP
3. MCP enforces row-level security
4. User only sees data they're authorized for

Flow:
User (Sales Rep A) → Agent → MCP
  ↓
MCP checks: Sales Rep A can only see their accounts
  ↓
Returns filtered data

User (Manager) → Agent → MCP
  ↓
MCP checks: Manager can see all team accounts
  ↓
Returns broader dataset
```

### Implementation Example

```python
# MCP Context Provider Implementation

from mcp import ContextProvider, Tool
from typing import List, Dict

class CRMContextProvider(ContextProvider):
    """
    MCP context provider for CRM data
    """
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        
    def get_schema(self) -> Dict:
        """Return available data schema"""
        return {
            "name": "crm",
            "description": "Customer relationship management data",
            "entities": {
                "customer": {
                    "properties": ["id", "name", "industry", "revenue"],
                    "operations": ["read", "search"]
                },
                "opportunity": {
                    "properties": ["id", "customer_id", "value", "stage"],
                    "operations": ["read", "search", "update"]
                }
            }
        }
    
    def query(self, entity: str, filters: Dict, user_context: Dict) -> List[Dict]:
        """
        Execute query with user context for security
        """
        # Check user permissions
        if not self._has_permission(user_context, entity):
            raise PermissionError(f"User cannot access {entity}")
        
        # Build query with user's data scope
        query = self._build_query(entity, filters, user_context)
        
        # Execute and return results
        return self._execute_query(query)
    
    def _has_permission(self, user_context: Dict, entity: str) -> bool:
        """Check if user can access entity"""
        # Implementation depends on your security model
        return True
    
    def _build_query(self, entity: str, filters: Dict, user_context: Dict) -> str:
        """Build SQL query with row-level security"""
        base_query = f"SELECT * FROM {entity}"
        
        # Add user's data scope (e.g., territory, department)
        if user_context.get('territory'):
            base_query += f" WHERE territory = '{user_context['territory']}'"
        
        # Add additional filters
        for key, value in filters.items():
            base_query += f" AND {key} = '{value}'"
        
        return base_query
    
    def _execute_query(self, query: str) -> List[Dict]:
        """Execute query and return results"""
        # Implementation using your database library
        pass


class CRMToolProvider:
    """
    MCP tool provider for CRM actions
    """
    
    @Tool(
        name="update_opportunity_stage",
        description="Update the stage of a sales opportunity",
        parameters={
            "opportunity_id": {"type": "string", "required": True},
            "new_stage": {"type": "string", "enum": ["Qualify", "Propose", "Close", "Won", "Lost"]}
        }
    )
    def update_opportunity_stage(
        self,
        opportunity_id: str,
        new_stage: str,
        user_context: Dict
    ) -> Dict:
        """
        Update opportunity stage
        """
        # Validate user has permission
        if not self._can_update_opportunity(user_context, opportunity_id):
            return {
                "success": False,
                "error": "Permission denied"
            }
        
        # Update in CRM
        result = self._update_crm(opportunity_id, {"stage": new_stage})
        
        # Log action for audit
        self._log_action(user_context, "update_opportunity", opportunity_id)
        
        return {
            "success": True,
            "opportunity_id": opportunity_id,
            "new_stage": new_stage,
            "updated_at": result['timestamp']
        }
    
    def _can_update_opportunity(self, user_context: Dict, opp_id: str) -> bool:
        """Check if user can update this opportunity"""
        # Implementation
        pass
    
    def _update_crm(self, opp_id: str, updates: Dict) -> Dict:
        """Perform CRM update"""
        # Implementation
        pass
    
    def _log_action(self, user_context: Dict, action: str, resource_id: str):
        """Log action for audit trail"""
        # Implementation
        pass
```

---

## 2. Design Agents to Automate Tasks Using Computer Use

### Computer Use in Copilot Studio

**What is Computer Use?**
- AI agent can interact with applications and websites
- Simulates human computer interaction (clicking, typing, navigating)
- Automates tasks across any application
- Vision-guided: AI "sees" the screen and takes actions

**Capabilities:**
```
Actions the Agent Can Perform:
├── Click buttons and links
├── Fill in forms
├── Navigate menus
├── Read screen content
├── Upload/download files
├── Switch between applications
├── Copy/paste data
└── Take screenshots
```

**Architecture:**
```
User Request
    ↓
Copilot Studio Agent (decides what to do)
    ↓
Computer Use Module
    ↓
┌───────────┬────────────┬──────────────┐
│           │            │              │
▼           ▼            ▼              ▼
Web       Desktop    Legacy App    Multiple Apps
Browser   App       (No API)      (Workflow)
```

### Use Cases

#### Use Case 1: Legacy System Automation
```
Scenario: Agent needs to retrieve data from old system without API

Problem:
- Legacy application has no API
- Data entry is manual
- Time-consuming and error-prone

Solution with Computer Use:
1. Agent opens legacy application
2. Navigates to search screen
3. Enters search criteria
4. Reads results from screen
5. Extracts and formats data
6. Returns to user

Example:
User: "Get inventory levels for product SKU-12345 from the old system"

Agent Computer Use Steps:
1. Launch legacy inventory app
2. Click "Search" menu
3. Type "SKU-12345" in search box
4. Click "Search" button
5. Wait for results (vision-based detection)
6. Read quantity from screen: "450 units"
7. Return to user: "SKU-12345 has 450 units in stock"
```

#### Use Case 2: Multi-System Workflow
```
Scenario: Process requires data entry in multiple systems

Workflow: New Employee Onboarding
1. Create user account in HR system
2. Add to Active Directory
3. Assign email license in M365 admin
4. Enroll in benefits portal
5. Create ticket for laptop provisioning

Computer Use Automation:
Agent executes each step:
- Logs into each system
- Fills required forms
- Submits information
- Verifies completion
- Moves to next step

Saves: ~2 hours of manual work
```

#### Use Case 3: Web Data Extraction
```
Scenario: Gather competitive intelligence from websites

Task: Monitor competitor pricing

Agent Actions:
1. Navigate to competitor website
2. Search for product category
3. Extract pricing information
4. Take screenshot for reference
5. Store in database
6. Alert if prices changed significantly

Schedule: Daily at 6 AM
Result: Automated competitive pricing dashboard
```

### Design Considerations

#### 1. Reliability
```
Challenges:
- UI changes break automation
- Network delays cause timeouts
- Unexpected popups or alerts
- Application crashes

Solutions:
├── Retry logic with exponential backoff
├── Alternative paths if primary fails
├── Visual verification at each step
├── Error screenshots for debugging
└── Human escalation for failures

Example Retry Logic:
def click_button(button_text, max_retries=3):
    for attempt in range(max_retries):
        try:
            # Use vision to find button
            button_location = find_on_screen(button_text)
            if button_location:
                click(button_location)
                return True
            else:
                wait(2)  # Button might still be loading
        except Exception as e:
            if attempt == max_retries - 1:
                # Final attempt failed
                escalate_to_human(f"Could not click {button_text}")
                return False
            else:
                wait(2 ** attempt)  # Exponential backoff
    return False
```

#### 2. Security
```
Concerns:
- Credentials management
- Screen content exposure
- Audit trail
- Compliance

Best Practices:
├── Use secure credential store (Key Vault)
├── Never log passwords
├── Mask sensitive data in screenshots
├── Full audit log of actions
├── User consent for each action type
└── Role-based access control

Credential Handling:
# Never hardcode credentials
username = get_credential_from_vault("app_username")
password = get_credential_from_vault("app_password")

# Use on behalf of user when possible
execute_as_user(user_context)
```

#### 3. Performance
```
Optimization Strategies:
├── Cache screen regions that don't change
├── Parallel execution where possible
├── Minimize unnecessary steps
├── Use keyboard shortcuts instead of mouse
└── Pre-load applications in background

Example:
Slow: Navigate menu → submenu → form (5 seconds)
Fast: Keyboard shortcut directly to form (1 second)
```

#### 4. User Experience
```
Design Principles:
├── Show progress to user
├── Provide estimated time
├── Allow cancellation
├── Explain what agent is doing
└── Handle interruptions gracefully

Progress Indicator:
"I'm working on your request...
✓ Logged into system
⟳ Searching for records (30% complete)
  Extracting data
  Formatting results"
```

### Implementation Pattern

```yaml
# Copilot Studio Computer Use Action

name: "Extract Legacy Inventory"
trigger: User asks for inventory from old system
steps:
  - step: 1
    action: launch_application
    application: "InventorySystem.exe"
    wait_for: "window_title='Inventory Management'"
    
  - step: 2
    action: click
    target: "visual_match='Search' + button"
    timeout: 5000ms
    
  - step: 3
    action: type_text
    target: "input_field_near='SKU:'"
    text: "${product_sku}"
    
  - step: 4
    action: click
    target: "button_text='Search'"
    
  - step: 5
    action: wait
    condition: "screen_contains='Results:'"
    max_wait: 10000ms
    
  - step: 6
    action: extract_text
    region: "below='Results:'"
    pattern: "Quantity: (\\d+)"
    save_as: "quantity"
    
  - step: 7
    action: take_screenshot
    save_as: "evidence"
    
  - step: 8
    action: close_application
    
  - step: 9
    action: return_result
    message: "Found ${quantity} units"
    attachments: ["evidence"]
```

---

## 3. Design Agent Behaviors: Reasoning and Voice Mode

### Reasoning Mode

**What is Reasoning Mode?**
- Agent shows its thought process
- Step-by-step problem solving
- Explicit decision-making
- Better for complex tasks

**When to Use:**
```
Use Reasoning Mode:
✓ Complex problem-solving
✓ Multi-step analysis
✓ User wants to understand logic
✓ Debugging or validation needed
✓ High-stakes decisions

Don't Use Reasoning Mode:
✗ Simple Q&A
✗ Fast responses critical
✗ User just wants answer
✗ Overhead not justified
```

**Example:**

```
Without Reasoning:
User: "Should we approve this expense?"
Agent: "Yes, approve it."

With Reasoning:
User: "Should we approve this expense?"
Agent: "Let me analyze this expense request step by step:

1. Expense amount: $2,500
   → Under department limit ($5,000) ✓

2. Budget availability:
   → Department has $15,000 remaining ✓

3. Policy compliance:
   → Business travel category ✓
   → Receipt attached ✓
   → Approved by manager ✓

4. Unusual factors:
   → Amount is typical for this type of travel ✓
   → Vendor is on approved list ✓

Conclusion: All criteria met. Recommend approval.

Would you like me to process the approval?"
```

**Configuration in Copilot Studio:**
```yaml
agent_config:
  reasoning_mode: true
  reasoning_settings:
    show_steps: true
    verbosity: medium  # low, medium, high
    format: structured # structured, narrative
    include_confidence: true
    
  reasoning_template: |
    Let me think through this step by step:
    
    1. [Analysis]
    ${step_1_reasoning}
    
    2. [Evaluation]
    ${step_2_reasoning}
    
    3. [Conclusion]
    ${conclusion}
    
    Confidence: ${confidence_score}%
```

### Voice Mode

**What is Voice Mode?**
- Natural voice conversation
- Speech-to-text and text-to-speech
- Conversational flow optimized for speaking
- Hands-free interaction

**Design Considerations:**

#### 1. Conversation Style
```
Text Mode:
- Can be verbose
- Lists and tables work
- User reads at own pace

Voice Mode:
- Must be concise
- Spoken format (no visual formatting)
- Optimized for listening

Example:
Text: "Here are your top 5 opportunities:
       1. Acme Corp - $500K - Closing 12/15
       2. TechStart - $250K - Closing 12/20
       3. ..."

Voice: "You have 5 high-value opportunities. 
        The largest is Acme Corp at $500,000, 
        expected to close December 15th. 
        Would you like details on any of these?"
```

#### 2. Confirmation Pattern
```
Voice requires explicit confirmation:

Text: User clicks "Yes" button
Voice: User must say "yes" or similar

Agent: "I found 3 matching records. 
        Would you like me to read them?"
User: "Yes"
Agent: "Okay, here's the first one..."

OR

User: "Just the summaries"
Agent: "Got it, I'll summarize all three..."
```

#### 3. Error Handling
```
Text: Show error message, user reads and retries
Voice: Must handle gracefully with voice

Example:
Agent: "I didn't catch that. Could you repeat?"
Agent: "I'm not sure I understood. Did you say [x]?"
Agent: "Let me try rephrasing: [clarification]"
```

#### 4. Multi-Turn Conversation
```
Voice mode favors natural back-and-forth:

User: "Schedule a meeting"
Agent: "Sure, with whom?"
User: "John Smith"
Agent: "And when would you like to meet?"
User: "Tomorrow at 2"
Agent: "Tomorrow at 2 PM. How long?"
User: "One hour"
Agent: "Perfect. I'll send the invite for tomorrow 
        at 2 PM for one hour with John Smith. 
        Should I include anyone else?"
User: "No, that's it"
Agent: "Done! I've sent the meeting invite."
```

**Voice Mode Configuration:**
```yaml
voice_config:
  enabled: true
  
  speech_settings:
    language: "en-US"
    voice: "neural-female-1"  # Natural sounding voice
    speed: 1.0  # Normal pace
    tone: "professional"  # professional, friendly, casual
    
  conversation_settings:
    max_silence: 3000ms  # Wait 3 seconds for user speech
    barge_in: true  # Allow user to interrupt
    confirmation_required: true  # Confirm important actions
    
  response_format:
    max_length: 100_words  # Keep responses concise
    use_ssml: true  # Speech Synthesis Markup Language
    
  prompts:
    greeting: "Hi, I'm your AI assistant. How can I help?"
    listening: "I'm listening..."
    thinking: "Let me check on that..."
    goodbye: "Goodbye! Let me know if you need anything else."
```

**SSML for Natural Speech:**
```xml
<speak>
  <voice name="en-US-Neural2-F">
    <prosody rate="medium" pitch="medium">
      I found <emphasis level="moderate">three</emphasis> urgent items.
    </prosody>
    
    <break time="500ms"/>
    
    <prosody rate="slow">
      The first is the Acme contract, 
      <emphasis>due tomorrow</emphasis>.
    </prosody>
  </voice>
</speak>
```

---

## 4. Optimize Solution Design Using Agents in Microsoft 365

### Integration Patterns

#### Pattern 1: Embedded Agents in Workflows
```
Optimize: Reduce context switching

Traditional:
User in Outlook → Switch to CRM → Get customer info → Switch back → Write email

Optimized:
User in Outlook → Ask Copilot for customer info → Drafts email with context
(Never leaves Outlook)

Implementation:
- Microsoft 365 Copilot with CRM plugin
- Customer data retrieved inline
- Email drafted with relevant context
```

#### Pattern 2: Proactive Agents
```
Optimize: Anticipate needs

Traditional: User initiates every interaction

Optimized: Agent proactively surfaces information

Examples:
- Meeting starting in 15 min → Copilot surfaces prep materials
- Deadline approaching → Copilot reminds and offers to draft update
- Customer email received → Copilot suggests response template

Configuration:
triggers:
  - event: "calendar_event_soon"
    condition: "within_15_minutes"
    action: "prepare_meeting_brief"
    
  - event: "email_received"
    condition: "from_customer AND high_priority"
    action: "suggest_response"
```

#### Pattern 3: Agent Collaboration
```
Optimize: Seamless hand-offs between agents

Scenario: Complex customer inquiry

Flow:
User asks in Teams → M365 Copilot
  ↓ (Recognizes needs CRM data)
Delegates to → Dynamics 365 Sales Copilot
  ↓ (Gets customer info)
Returns to → M365 Copilot
  ↓ (Needs support history)
Delegates to → Customer Service Copilot
  ↓ (Gets support cases)
Returns to → M365 Copilot
  ↓
Synthesizes → Comprehensive response

From user's perspective: One conversation, seamless
```

### Performance Optimization

**Caching Strategy:**
```
Cache frequently accessed data:
- User profile
- Recent documents
- Common queries
- Static reference data

Cache TTL:
- User data: 15 minutes
- Documents: 5 minutes
- Reference data: 1 hour

Example:
First query: "What's our refund policy?" (500ms - fetch from knowledge base)
Same query 5 min later: "What's our refund policy?" (50ms - from cache)
```

**Parallel Processing:**
```
Sequential (Slow):
Query CRM → Wait → Query Support → Wait → Query Finance → Wait → Respond
Total: 3 seconds

Parallel (Fast):
Query CRM
Query Support    } Execute simultaneously
Query Finance
Wait for all → Respond
Total: 1 second
```

**Progressive Response:**
```
Traditional:
User waits → Full response arrives

Optimized:
User sees → "I'm gathering information..."
         → "Found customer record ✓"
         → "Checking support history ✓"
         → "Analyzing...✓"
         → Complete response

User Experience: Feels faster, less anxiety
```

---

## Study Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Copilot Studio Computer Use](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-computer-use)
- [Agent Reasoning Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/reasoning)
- [Microsoft 365 Copilot Optimization](https://learn.microsoft.com/en-us/microsoft-365-copilot/best-practices)

**Next Topic:** [Orchestrate Prebuilt Agents](03-prebuilt-orchestration.md)
