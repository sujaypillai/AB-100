# Monitor and Tune AI-Powered Business Solutions

## Overview
This section covers monitoring strategies, performance tuning, and optimization techniques for AI-powered business solutions across Microsoft 365 Copilot, Copilot Studio, Azure AI Foundry, and Dynamics 365.

---

## 1. Monitor AI Solution Performance

### 1.1 Monitoring Tools and Platforms

#### Azure AI Foundry Monitoring

**Azure AI Foundry Portal:**
```
Key Metrics Dashboard:
├─ Model Performance
│  ├─ Latency (P50, P95, P99)
│  ├─ Throughput (requests/second)
│  ├─ Token consumption
│  └─ Error rates
├─ Quality Metrics
│  ├─ Groundedness score
│  ├─ Relevance score
│  ├─ Coherence score
│  └─ Fluency score
├─ Cost Metrics
│  ├─ Token costs by model
│  ├─ Cost per request
│  └─ Daily/monthly spend
└─ User Engagement
   ├─ Active users
   ├─ Session duration
   └─ Feature usage
```

**Configuration:**
```yaml
# Azure AI Foundry Monitoring Configuration
monitoring:
  enabled: true
  workspace: "contoso-ai-workspace"
  
  metrics:
    performance:
      - latency:
          enabled: true
          alertThreshold: 5000ms  # P95 latency
      - throughput:
          enabled: true
          alertThreshold: 100  # requests/sec minimum
      - errorRate:
          enabled: true
          alertThreshold: 5  # percentage
    
    quality:
      - groundedness:
          enabled: true
          minimumScore: 3.5  # out of 5
      - relevance:
          enabled: true
          minimumScore: 3.5
      - coherence:
          enabled: true
          minimumScore: 4.0
    
    cost:
      - dailyBudget: 500  # USD
      - monthlyBudget: 10000
      - alertThreshold: 80  # percentage of budget
  
  logging:
    level: "info"
    includePrompts: true
    includeResponses: true
    includePII: false
    retentionDays: 90
  
  applicationInsights:
    enabled: true
    instrumentationKey: "${AI_INSTRUMENTATION_KEY}"
    samplingPercentage: 100
```

**Application Insights Integration:**
```python
# Python SDK for Custom Telemetry
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace
from opentelemetry.trace import Status, StatusCode

# Configure Azure Monitor
configure_azure_monitor(
    connection_string="InstrumentationKey=xxx"
)

tracer = trace.get_tracer(__name__)

def call_ai_model(prompt, user_id, session_id):
    # Create span for tracing
    with tracer.start_as_current_span("ai_model_call") as span:
        # Add attributes
        span.set_attribute("user.id", user_id)
        span.set_attribute("session.id", session_id)
        span.set_attribute("prompt.length", len(prompt))
        
        try:
            # Call AI model
            start_time = time.time()
            response = ai_client.complete(prompt)
            latency = time.time() - start_time
            
            # Log metrics
            span.set_attribute("response.latency_ms", latency * 1000)
            span.set_attribute("response.tokens", response.usage.total_tokens)
            span.set_attribute("response.cost", calculate_cost(response.usage))
            span.set_status(Status(StatusCode.OK))
            
            # Custom event for quality
            track_event("ai_response_quality", {
                "groundedness": evaluate_groundedness(response.text),
                "relevance": evaluate_relevance(prompt, response.text),
                "user_id": user_id
            })
            
            return response
            
        except Exception as e:
            span.set_status(Status(StatusCode.ERROR, str(e)))
            span.record_exception(e)
            
            # Log error
            track_exception(e, {
                "user_id": user_id,
                "prompt_preview": prompt[:100]
            })
            raise
```

#### Copilot Studio Analytics

**Analytics Dashboard:**
```
Copilot Studio Analytics:
├─ Engagement Metrics
│  ├─ Total sessions
│  ├─ Engaged sessions (% with >1 turn)
│  ├─ Average session duration
│  ├─ Resolution rate
│  └─ Abandon rate
├─ Conversation Metrics
│  ├─ Total messages
│  ├─ Messages per session
│  ├─ Topic triggering frequency
│  └─ Escalation rate
├─ Satisfaction Metrics
│  ├─ CSAT score
│  ├─ Net Promoter Score (NPS)
│  └─ User feedback sentiment
└─ AI Quality Metrics
   ├─ Generative answers usage
   ├─ Fallback rate
   └─ Answer citation rate
```

**Power BI Integration:**
```
Purpose: Advanced analytics and custom reporting

Setup:
1. Copilot Studio > Analytics > Export data
2. Configure Dataverse integration
3. Connect Power BI to Dataverse
4. Use conversation analytics tables:
   - ConversationTranscripts
   - SessionMetrics
   - TopicAnalytics
   - UserFeedback

Key Tables:
- msfp_surveyresponse (satisfaction surveys)
- msdyn_ocliveworkitem (engagement sessions)
- bot_conversationtranscript (full transcripts)
- bot_sessionmetrics (aggregated metrics)
```

**Custom Metrics with Power Automate:**
```yaml
# Track Custom Business Metrics
name: TrackConversationOutcome
trigger:
  event: ConversationEnded
  source: CopilotStudio

steps:
  - name: AnalyzeConversation
    action: ParseTranscript
    inputs:
      transcript: "@{triggerBody()?['transcript']}"
    outputs:
      - intentsDetected: list
      - entitiesExtracted: list
      - topicsTriggered: list
      - escalationOccurred: boolean
      - businessOutcome: string  # "resolved", "escalated", "abandoned"
  
  - name: CalculateMetrics
    action: Compose
    inputs:
      sessionId: "@{triggerBody()?['sessionId']}"
      userId: "@{triggerBody()?['userId']}"
      duration: "@{triggerBody()?['durationSeconds']}"
      turnCount: "@{triggerBody()?['turnCount']}"
      outcome: "@{outputs('AnalyzeConversation')?['businessOutcome']}"
      firstResponseTime: "@{triggerBody()?['firstResponseTimeMs']}"
      
      # Custom business metrics
      leadGenerated: "@{contains(outputs('AnalyzeConversation')?['topicsTriggered'], 'LeadCapture')}"
      appointmentBooked: "@{contains(outputs('AnalyzeConversation')?['topicsTriggered'], 'ScheduleAppointment')}"
      productRecommended: "@{not(empty(outputs('AnalyzeConversation')?['entitiesExtracted/product']))}"
  
  - name: LogToDataverse
    action: Dataverse.CreateRecord
    inputs:
      entity: custom_conversationmetrics
      data: "@outputs('CalculateMetrics')"
  
  - name: LogToApplicationInsights
    action: HTTP
    inputs:
      method: POST
      uri: "https://dc.services.visualstudio.com/v2/track"
      body:
        name: "ConversationCompleted"
        properties: "@outputs('CalculateMetrics')"
```

#### Microsoft 365 Copilot Monitoring

**Microsoft 365 Admin Center:**
```
Copilot Usage Analytics:
├─ Adoption Metrics
│  ├─ Active users (daily/monthly)
│  ├─ Feature utilization by app
│  ├─ Time saved per user
│  └─ Productivity gains
├─ Usage Patterns
│  ├─ Most used features
│  ├─ Peak usage times
│  ├─ Usage by department
│  └─ Usage by geography
└─ Quality Indicators
   ├─ User feedback ratings
   ├─ Feature-specific satisfaction
   └─ Problem reports
```

**Microsoft Viva Insights:**
```
Purpose: Measure Copilot impact on productivity

Metrics:
- Meeting efficiency improvements
- Time saved on email
- Focus time preservation
- Collaboration patterns
- Work-life balance impact

Access: Viva Insights > Copilot Dashboard
```

**Microsoft Graph API for Custom Analytics:**
```python
# Retrieve M365 Copilot Usage Data
import requests
from datetime import datetime, timedelta

def get_copilot_usage_metrics(access_token, days=30):
    """
    Retrieve Copilot usage metrics via Microsoft Graph
    """
    endpoint = "https://graph.microsoft.com/v1.0/reports/getCopilotUsageMetrics"
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json"
    }
    
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days)
    
    params = {
        "period": "D30",  # Last 30 days
        "date": end_date.strftime("%Y-%m-%d")
    }
    
    response = requests.get(endpoint, headers=headers, params=params)
    
    if response.status_code == 200:
        data = response.json()
        
        # Process metrics
        metrics = {
            "total_active_users": data.get("totalActiveUsers", 0),
            "features": {}
        }
        
        for feature in data.get("featureUsage", []):
            metrics["features"][feature["featureName"]] = {
                "active_users": feature["activeUserCount"],
                "total_interactions": feature["interactionCount"],
                "avg_interactions_per_user": feature["avgInteractionsPerUser"]
            }
        
        return metrics
    else:
        raise Exception(f"Failed to retrieve metrics: {response.status_code}")

# Example usage
metrics = get_copilot_usage_metrics(access_token)
print(f"Total Active Users: {metrics['total_active_users']}")
print(f"Word Copilot Usage: {metrics['features']['WordCopilot']}")
```

#### Dynamics 365 AI Monitoring

**Dynamics 365 Customer Insights:**
```
AI Feature Monitoring:
├─ Copilot Usage in D365 Apps
│  ├─ Sales Copilot interactions
│  ├─ Service Copilot usage
│  ├─ Finance Copilot queries
│  └─ Field Service AI recommendations
├─ AI Model Performance
│  ├─ Lead scoring accuracy
│  ├─ Opportunity win prediction accuracy
│  ├─ Case routing effectiveness
│  └─ Demand forecast accuracy
└─ Business Impact
   ├─ Sales cycle reduction
   ├─ Case resolution time improvement
   ├─ Revenue impact
   └─ Cost savings
```

**Power Platform Admin Center:**
```
Monitoring AI Builder Models:

Navigation: Power Platform admin center > Analytics > AI Builder

Metrics:
1. Model Performance:
   - Prediction accuracy
   - F1 score
   - Precision/Recall
   - Confusion matrix

2. Model Usage:
   - Predictions per day
   - Active flows using model
   - Active apps using model

3. Credits Consumption:
   - AI Builder credits used
   - Credits by model type
   - Usage trends

4. Model Health:
   - Last training date
   - Training data quality
   - Drift detection alerts
```

### 1.2 Interpreting Telemetry Data

#### Performance Telemetry

**Latency Analysis:**
```
Latency Breakdown:
┌─────────────────────────────────────────────────────────┐
│  Total Response Time                                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 1. Request Processing (50-200ms)                 │  │
│  │    - Authentication                               │  │
│  │    - Input validation                             │  │
│  │    - Context retrieval                            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 2. Data Retrieval (100-1000ms)                   │  │
│  │    - Database queries                             │  │
│  │    - API calls to data sources                    │  │
│  │    - Graph API calls                              │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 3. AI Model Inference (500-5000ms)               │  │
│  │    - Prompt construction                          │  │
│  │    - Model processing                             │  │
│  │    - Response generation                          │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 4. Post-Processing (50-200ms)                    │  │
│  │    - Response formatting                          │  │
│  │    - Citation addition                            │  │
│  │    - Logging                                      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘

Target Latencies:
- Simple queries: < 2 seconds
- Complex queries: < 5 seconds
- Document generation: < 10 seconds
```

**KQL Queries for Application Insights:**
```kusto
// Average latency by operation
requests
| where timestamp > ago(24h)
| where name startswith "ai_model_call"
| summarize 
    avg_latency = avg(duration),
    p50 = percentile(duration, 50),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99),
    request_count = count()
    by operation_Name
| order by avg_latency desc

// Identify slow requests
requests
| where timestamp > ago(24h)
| where duration > 5000  // > 5 seconds
| project 
    timestamp,
    operation_Name,
    duration,
    resultCode,
    customDimensions.user_id,
    customDimensions.prompt_length
| order by duration desc
| take 100

// Error rate by hour
requests
| where timestamp > ago(7d)
| summarize 
    total = count(),
    errors = countif(success == false)
    by bin(timestamp, 1h)
| extend error_rate = errors * 100.0 / total
| render timechart

// Token consumption trends
customEvents
| where timestamp > ago(30d)
| where name == "ai_response_quality"
| extend tokens = toint(customDimensions.response_tokens)
| summarize 
    total_tokens = sum(tokens),
    avg_tokens = avg(tokens),
    max_tokens = max(tokens)
    by bin(timestamp, 1d)
| render timechart
```

#### Quality Telemetry

**AI Response Quality Metrics:**
```json
{
  "evaluationFramework": {
    "groundedness": {
      "description": "Does the response contain information supported by data sources?",
      "scale": "1-5",
      "calculation": "automated_via_judge_model",
      "target": ">= 4.0",
      "sampleQuestions": [
        "Is each claim backed by a citation?",
        "Are there any hallucinated facts?",
        "Is the information current and accurate?"
      ]
    },
    "relevance": {
      "description": "Does the response address the user's question?",
      "scale": "1-5",
      "calculation": "automated_via_judge_model",
      "target": ">= 4.0",
      "sampleQuestions": [
        "Does it answer what was asked?",
        "Is the response on-topic?",
        "Is the level of detail appropriate?"
      ]
    },
    "coherence": {
      "description": "Is the response well-structured and logical?",
      "scale": "1-5",
      "calculation": "automated_via_judge_model",
      "target": ">= 4.5",
      "sampleQuestions": [
        "Is the response easy to follow?",
        "Are sentences grammatically correct?",
        "Is the flow logical?"
      ]
    },
    "fluency": {
      "description": "Is the language natural and professional?",
      "scale": "1-5",
      "calculation": "automated_via_judge_model",
      "target": ">= 4.5",
      "sampleQuestions": [
        "Does it sound natural?",
        "Is the tone appropriate?",
        "Are there awkward phrasings?"
      ]
    },
    "safetyHarmfulness": {
      "description": "Is the content safe and non-harmful?",
      "scale": "pass/fail",
      "calculation": "azure_content_safety_api",
      "target": "100% pass",
      "categories": [
        "Hate speech",
        "Violence",
        "Sexual content",
        "Self-harm"
      ]
    }
  }
}
```

**Automated Quality Evaluation:**
```python
# Azure AI Foundry - Automated Evaluation
from azure.ai.evaluation import evaluate, GroundednessEvaluator, RelevanceEvaluator

def evaluate_ai_responses(test_dataset):
    """
    Automatically evaluate AI responses against quality metrics
    """
    # Initialize evaluators
    groundedness_eval = GroundednessEvaluator(
        model_config=judge_model_config
    )
    
    relevance_eval = RelevanceEvaluator(
        model_config=judge_model_config
    )
    
    # Run evaluation
    results = evaluate(
        data=test_dataset,  # List of {question, context, response}
        evaluators={
            "groundedness": groundedness_eval,
            "relevance": relevance_eval
        }
    )
    
    # Analyze results
    metrics = {
        "avg_groundedness": results["groundedness"].mean(),
        "avg_relevance": results["relevance"].mean(),
        "pass_rate": (results["groundedness"] >= 4.0).mean(),
        "failed_examples": results[results["groundedness"] < 3.0]
    }
    
    # Flag low-quality responses
    if metrics["avg_groundedness"] < 4.0:
        send_alert("Groundedness below threshold", metrics)
    
    return metrics

# Continuous evaluation
def continuous_quality_monitoring():
    """
    Sample and evaluate production responses daily
    """
    # Sample recent conversations
    sample = get_random_sample_from_logs(
        date=datetime.now().date(),
        sample_size=100
    )
    
    # Evaluate
    metrics = evaluate_ai_responses(sample)
    
    # Log to monitoring dashboard
    log_metrics_to_appinsights("daily_quality_check", metrics)
    
    return metrics
```

#### User Satisfaction Telemetry

**CSAT (Customer Satisfaction) Tracking:**
```yaml
# Copilot Studio - CSAT Survey Configuration
survey:
  name: CopilotSatisfactionSurvey
  trigger:
    conversationEndConditions:
      - resolved: true
      - minTurnCount: 3
    sampleRate: 30  # Survey 30% of qualifying conversations
  
  questions:
    - id: q1
      type: rating
      scale: 1-5
      text: "How satisfied are you with the assistance provided?"
      required: true
    
    - id: q2
      type: multiChoice
      text: "What did you like most?"
      options:
        - "Quick responses"
        - "Accurate information"
        - "Easy to understand"
        - "Solved my problem"
      required: false
    
    - id: q3
      type: text
      text: "How can we improve?"
      maxLength: 500
      required: false
  
  thresholds:
    positive: 4  # Rating >= 4 considered satisfied
    negative: 2  # Rating <= 2 triggers alert
  
  actions:
    onNegativeResponse:
      - sendAlert:
          to: "support-managers@contoso.com"
          subject: "Low CSAT Alert"
      - createCase:
          priority: "high"
          category: "User Dissatisfaction"
```

**Sentiment Analysis on Feedback:**
```python
# Analyze user feedback sentiment
from azure.ai.textanalytics import TextAnalyticsClient

def analyze_feedback_sentiment(feedback_texts):
    """
    Analyze sentiment of user feedback comments
    """
    client = TextAnalyticsClient(endpoint=endpoint, credential=credential)
    
    # Batch analyze
    results = client.analyze_sentiment(feedback_texts, show_opinion_mining=True)
    
    aggregated = {
        "positive": 0,
        "neutral": 0,
        "negative": 0,
        "common_positive_aspects": [],
        "common_negative_aspects": []
    }
    
    for result in results:
        # Count sentiment distribution
        aggregated[result.sentiment] += 1
        
        # Extract specific opinions
        for sentence in result.sentences:
            for opinion in sentence.mined_opinions:
                aspect = opinion.target.text
                sentiment = opinion.target.sentiment
                
                if sentiment == "positive":
                    aggregated["common_positive_aspects"].append(aspect)
                elif sentiment == "negative":
                    aggregated["common_negative_aspects"].append(aspect)
    
    # Identify top issues
    from collections import Counter
    aggregated["top_complaints"] = Counter(
        aggregated["common_negative_aspects"]
    ).most_common(5)
    
    return aggregated
```

---

## 2. Tune AI Solution Performance

### 2.1 Prompt Optimization

**Iterative Prompt Engineering:**
```
Optimization Cycle:
1. Baseline: Deploy initial prompt
2. Measure: Collect quality metrics (groundedness, relevance)
3. Analyze: Identify failure patterns
4. Refine: Adjust prompt based on findings
5. Test: A/B test new prompt vs baseline
6. Deploy: Roll out winner
7. Repeat

Tools:
- Azure AI Foundry Prompt Flow
- Copilot Studio topic testing
- A/B testing framework
```

**Example Optimization:**
```
Initial Prompt (v1.0):
---
Summarize this document for the user.

Document: {{document_content}}
---

Issues Detected:
- Too generic, inconsistent output length
- Sometimes misses key points
- Doesn't adapt to document type

Improved Prompt (v2.0):
---
You are a professional document summarizer. Create a concise summary of the following document.

Document Type: {{document_type}}
Document: {{document_content}}

Instructions:
1. Identify and extract the 3-5 most important points
2. Keep summary under 200 words
3. Use bullet points for clarity
4. Preserve key facts, dates, and numbers exactly
5. If document is technical, maintain technical terms

Summary:
---

Results:
- Groundedness: 3.2 → 4.1
- Relevance: 3.8 → 4.3
- User satisfaction: 72% → 85%
```

**Systematic Prompt Testing:**
```python
# Prompt Flow - A/B Testing
from promptflow import flow

@flow
def prompt_ab_test(user_query, document, variant="A"):
    """
    Test two prompt variants
    """
    prompts = {
        "A": load_prompt("summarize_v1.jinja2"),
        "B": load_prompt("summarize_v2.jinja2")
    }
    
    # Get response
    response = llm_call(
        prompt=prompts[variant],
        user_query=user_query,
        document=document
    )
    
    # Evaluate quality
    quality_scores = evaluate_response(
        query=user_query,
        response=response,
        context=document
    )
    
    # Log results
    log_experiment(
        variant=variant,
        input_hash=hash(user_query + document),
        response=response,
        scores=quality_scores
    )
    
    return response

# Run experiment
def run_ab_experiment(test_cases, traffic_split=0.5):
    """
    Split traffic between variants and measure results
    """
    results = {"A": [], "B": []}
    
    for test_case in test_cases:
        variant = "A" if random.random() < traffic_split else "B"
        
        response = prompt_ab_test(
            user_query=test_case["query"],
            document=test_case["document"],
            variant=variant
        )
        
        results[variant].append({
            "test_case": test_case,
            "response": response
        })
    
    # Statistical analysis
    from scipy import stats
    
    scores_a = [r["scores"]["overall"] for r in results["A"]]
    scores_b = [r["scores"]["overall"] for r in results["B"]]
    
    t_stat, p_value = stats.ttest_ind(scores_a, scores_b)
    
    winner = "B" if np.mean(scores_b) > np.mean(scores_a) else "A"
    confidence = 1 - p_value
    
    return {
        "winner": winner,
        "confidence": confidence,
        "variant_a_avg": np.mean(scores_a),
        "variant_b_avg": np.mean(scores_b)
    }
```

### 2.2 Model Selection and Routing Optimization

**Model Router Tuning:**
```python
# Optimize model routing rules based on performance data
class ModelRouterOptimizer:
    def __init__(self, telemetry_data):
        self.data = telemetry_data
    
    def analyze_routing_effectiveness(self):
        """
        Analyze if requests are being routed to optimal models
        """
        analysis = {}
        
        for request_type in self.data["request_types"]:
            # Get all requests of this type
            requests = self.data[self.data["type"] == request_type]
            
            # Group by model used
            by_model = requests.groupby("model_used").agg({
                "latency": ["mean", "p95"],
                "quality_score": "mean",
                "cost": "mean",
                "user_satisfaction": "mean"
            })
            
            # Identify optimal model for this request type
            # Score = (quality * 0.4) + (speed * 0.3) + (cost_efficiency * 0.3)
            by_model["composite_score"] = (
                (by_model["quality_score"]["mean"] / 5.0) * 0.4 +
                (1.0 - by_model["latency"]["mean"] / by_model["latency"]["mean"].max()) * 0.3 +
                (1.0 - by_model["cost"]["mean"] / by_model["cost"]["mean"].max()) * 0.3
            )
            
            optimal_model = by_model["composite_score"].idxmax()
            current_model = self.get_current_routing_rule(request_type)
            
            analysis[request_type] = {
                "current_model": current_model,
                "optimal_model": optimal_model,
                "should_change": optimal_model != current_model,
                "expected_improvement": by_model.loc[optimal_model, "composite_score"] - 
                                       by_model.loc[current_model, "composite_score"]
            }
        
        return analysis
    
    def generate_optimized_routing_rules(self):
        """
        Generate new routing rules based on analysis
        """
        analysis = self.analyze_routing_effectiveness()
        
        new_rules = []
        for request_type, recommendation in analysis.items():
            if recommendation["should_change"]:
                new_rules.append({
                    "condition": self.get_condition_for_type(request_type),
                    "target_model": recommendation["optimal_model"],
                    "reason": f"Expected {recommendation['expected_improvement']:.1%} improvement"
                })
        
        return new_rules

# Example usage
optimizer = ModelRouterOptimizer(load_telemetry_data(days=30))
recommendations = optimizer.generate_optimized_routing_rules()

for rule in recommendations:
    print(f"Update routing: {rule['condition']} → {rule['target_model']}")
    print(f"  Reason: {rule['reason']}")
```

**Dynamic Routing Configuration:**
```yaml
# Adaptive Model Router
modelRouter:
  name: AdaptiveRouter
  evaluationPeriod: 7days  # Re-evaluate routing every week
  
  models:
    - id: gpt-4-turbo
      cost: 0.03  # per 1K tokens
      avgLatency: 2000ms
      capabilities: [complex_reasoning, code_generation, long_context]
    
    - id: gpt-35-turbo
      cost: 0.002
      avgLatency: 800ms
      capabilities: [general_chat, simple_qa, summarization]
    
    - id: gpt-4o-mini
      cost: 0.0004
      avgLatency: 500ms
      capabilities: [simple_tasks, high_volume]
  
  routingRules:
    - name: ComplexAnalysis
      condition: 
        or:
          - prompt_complexity_score > 0.7
          - user_query contains ["analyze", "compare", "evaluate"]
          - context_size > 5000
      targetModel: gpt-4-turbo
      fallback: gpt-35-turbo
    
    - name: CodeGeneration
      condition:
        and:
          - intent == "code_generation"
          - language in ["python", "javascript", "csharp"]
      targetModel: gpt-4-turbo
      fallback: gpt-35-turbo
    
    - name: SimpleQA
      condition:
        and:
          - prompt_complexity_score < 0.3
          - context_size < 1000
          - intent in ["factual_qa", "greeting", "navigation"]
      targetModel: gpt-4o-mini
      fallback: gpt-35-turbo
    
    - name: Default
      condition: true  # Catch-all
      targetModel: gpt-35-turbo
      fallback: gpt-4o-mini
  
  optimization:
    enabled: true
    autoAdjust: true  # Automatically adjust rules based on performance
    costOptimizationWeight: 0.3
    qualityWeight: 0.4
    latencyWeight: 0.3
```

### 2.3 Retrieval-Augmented Generation (RAG) Optimization

**Indexing Optimization:**
```python
# Azure AI Search - Optimize Index Configuration
index_definition = {
    "name": "knowledge-base-optimized",
    "fields": [
        {
            "name": "id",
            "type": "Edm.String",
            "key": True
        },
        {
            "name": "content",
            "type": "Edm.String",
            "searchable": True,
            "analyzer": "en.microsoft",  # Use language-specific analyzer
            "searchAnalyzer": "en.microsoft"
        },
        {
            "name": "title",
            "type": "Edm.String",
            "searchable": True,
            "analyzer": "en.microsoft",
            "searchAnalyzer": "en.microsoft",
            # Boost title matches
            "weight": 2.0
        },
        {
            "name": "category",
            "type": "Edm.String",
            "filterable": True,
            "facetable": True
        },
        {
            "name": "lastModified",
            "type": "Edm.DateTimeOffset",
            "filterable": True,
            "sortable": True
        },
        {
            "name": "contentVector",
            "type": "Collection(Edm.Single)",
            "searchable": True,
            "vectorSearchDimensions": 1536,
            "vectorSearchProfileName": "hnsw-profile"
        }
    ],
    "vectorSearch": {
        "algorithms": [
            {
                "name": "hnsw-algorithm",
                "kind": "hnsw",  # Hierarchical Navigable Small World
                "hnswParameters": {
                    "metric": "cosine",
                    "m": 16,  # Number of connections per layer
                    "efConstruction": 400,  # Build-time accuracy
                    "efSearch": 500  # Query-time accuracy
                }
            }
        ],
        "profiles": [
            {
                "name": "hnsw-profile",
                "algorithm": "hnsw-algorithm"
            }
        ]
    },
    "semantic": {
        "configurations": [
            {
                "name": "semantic-config",
                "prioritizedFields": {
                    "titleField": {"fieldName": "title"},
                    "contentFields": [{"fieldName": "content"}],
                    "keywordsFields": [{"fieldName": "category"}]
                }
            }
        ]
    }
}
```

**Retrieval Strategy Tuning:**
```python
# Optimize retrieval parameters based on quality metrics
class RAGOptimizer:
    def __init__(self):
        self.search_client = get_search_client()
    
    def optimize_retrieval(self, query, ground_truth_answer):
        """
        Test different retrieval parameters to find optimal configuration
        """
        # Parameters to test
        top_k_values = [3, 5, 10, 15]
        reranking_options = [True, False]
        search_modes = ["hybrid", "vector_only", "text_only"]
        
        best_config = None
        best_score = 0
        
        for top_k in top_k_values:
            for rerank in reranking_options:
                for search_mode in search_modes:
                    # Retrieve documents
                    docs = self.retrieve_documents(
                        query=query,
                        top_k=top_k,
                        rerank=rerank,
                        search_mode=search_mode
                    )
                    
                    # Generate answer with retrieved docs
                    answer = self.generate_answer(query, docs)
                    
                    # Evaluate quality
                    score = self.evaluate_answer_quality(
                        generated=answer,
                        ground_truth=ground_truth_answer
                    )
                    
                    if score > best_score:
                        best_score = score
                        best_config = {
                            "top_k": top_k,
                            "rerank": rerank,
                            "search_mode": search_mode,
                            "score": score
                        }
        
        return best_config
    
    def retrieve_documents(self, query, top_k, rerank, search_mode):
        """
        Retrieve documents with specified configuration
        """
        if search_mode == "hybrid":
            # Combine vector and keyword search
            results = self.search_client.search(
                search_text=query,
                vector_queries=[{
                    "vector": self.embed_query(query),
                    "k_nearest_neighbors": top_k,
                    "fields": "contentVector"
                }],
                top=top_k,
                query_type="semantic" if rerank else "simple"
            )
        elif search_mode == "vector_only":
            results = self.search_client.search(
                vector_queries=[{
                    "vector": self.embed_query(query),
                    "k_nearest_neighbors": top_k,
                    "fields": "contentVector"
                }],
                top=top_k
            )
        else:  # text_only
            results = self.search_client.search(
                search_text=query,
                top=top_k,
                query_type="semantic" if rerank else "simple"
            )
        
        return list(results)

# Run optimization on test set
optimizer = RAGOptimizer()
test_queries = load_test_dataset()

optimal_configs = []
for test_case in test_queries:
    config = optimizer.optimize_retrieval(
        query=test_case["query"],
        ground_truth_answer=test_case["answer"]
    )
    optimal_configs.append(config)

# Analyze results
avg_optimal_top_k = np.mean([c["top_k"] for c in optimal_configs])
rerank_preference = np.mean([c["rerank"] for c in optimal_configs])
print(f"Recommended top_k: {int(avg_optimal_top_k)}")
print(f"Enable reranking: {rerank_preference > 0.5}")
```

**Chunking Strategy Optimization:**
```python
# Test different document chunking strategies
def evaluate_chunking_strategies(documents):
    """
    Test various chunking approaches to maximize retrieval quality
    """
    strategies = {
        "fixed_500": {"size": 500, "overlap": 50, "method": "tokens"},
        "fixed_1000": {"size": 1000, "overlap": 100, "method": "tokens"},
        "semantic": {"method": "semantic_sentences", "max_size": 800},
        "paragraph": {"method": "paragraphs", "max_size": 1200}
    }
    
    results = {}
    
    for strategy_name, params in strategies.items():
        # Chunk documents
        chunks = chunk_documents(documents, params)
        
        # Index chunks
        index_name = f"test-index-{strategy_name}"
        create_and_populate_index(index_name, chunks)
        
        # Test retrieval quality on test set
        test_queries = load_test_queries()
        metrics = evaluate_retrieval_quality(index_name, test_queries)
        
        results[strategy_name] = {
            "avg_chunk_count": len(chunks) / len(documents),
            "retrieval_precision": metrics["precision"],
            "retrieval_recall": metrics["recall"],
            "answer_quality": metrics["answer_quality"],
            "avg_latency": metrics["avg_latency"]
        }
        
        # Cleanup
        delete_index(index_name)
    
    # Identify best strategy
    best_strategy = max(results.items(), 
                       key=lambda x: x[1]["answer_quality"])
    
    return results, best_strategy

# Semantic chunking implementation
def semantic_chunk(text, max_size=800):
    """
    Chunk text at natural semantic boundaries
    """
    import nltk
    from sentence_transformers import SentenceTransformer
    
    # Split into sentences
    sentences = nltk.sent_tokenize(text)
    
    # Embed sentences
    model = SentenceTransformer('all-MiniLM-L6-v2')
    embeddings = model.encode(sentences)
    
    # Calculate similarity between adjacent sentences
    from sklearn.metrics.pairwise import cosine_similarity
    similarities = []
    for i in range(len(embeddings) - 1):
        sim = cosine_similarity(
            [embeddings[i]], 
            [embeddings[i+1]]
        )[0][0]
        similarities.append(sim)
    
    # Find chunk boundaries (low similarity = semantic shift)
    threshold = np.percentile(similarities, 25)  # Bottom quartile
    boundaries = [0]
    
    current_size = 0
    for i, sim in enumerate(similarities):
        current_size += len(sentences[i].split())
        
        # Start new chunk if semantic shift OR size limit reached
        if sim < threshold or current_size > max_size:
            boundaries.append(i + 1)
            current_size = 0
    
    boundaries.append(len(sentences))
    
    # Create chunks
    chunks = []
    for i in range(len(boundaries) - 1):
        chunk = " ".join(sentences[boundaries[i]:boundaries[i+1]])
        chunks.append(chunk)
    
    return chunks
```

### 2.4 Caching Optimization

**Response Caching Strategy:**
```python
# Implement semantic caching for AI responses
import hashlib
from redis import Redis
from sentence_transformers import SentenceTransformer

class SemanticCache:
    def __init__(self, redis_client, similarity_threshold=0.95):
        self.redis = redis_client
        self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
        self.similarity_threshold = similarity_threshold
    
    def get_cache_key(self, query, context_hash):
        """Generate cache key from query embedding"""
        embedding = self.encoder.encode(query)
        # Store embedding in Redis with context
        key = f"query:{context_hash}:{hashlib.md5(query.encode()).hexdigest()}"
        return key, embedding
    
    def get(self, query, context_hash):
        """
        Retrieve cached response if semantically similar query exists
        """
        query_key, query_embedding = self.get_cache_key(query, context_hash)
        
        # Check exact match first
        cached = self.redis.get(query_key)
        if cached:
            return json.loads(cached)
        
        # Search for semantically similar queries
        # In production, use vector database like Azure AI Search
        pattern = f"query:{context_hash}:*"
        for key in self.redis.scan_iter(match=pattern, count=100):
            cached_data = json.loads(self.redis.get(key))
            cached_embedding = cached_data["embedding"]
            
            # Calculate similarity
            similarity = cosine_similarity(
                [query_embedding],
                [cached_embedding]
            )[0][0]
            
            if similarity >= self.similarity_threshold:
                # Cache hit!
                cached_data["cache_hit"] = True
                cached_data["similarity_score"] = float(similarity)
                return cached_data
        
        return None
    
    def set(self, query, context_hash, response, ttl=3600):
        """Cache query and response"""
        query_key, query_embedding = self.get_cache_key(query, context_hash)
        
        cache_data = {
            "query": query,
            "response": response,
            "embedding": query_embedding.tolist(),
            "timestamp": datetime.now().isoformat()
        }
        
        self.redis.setex(
            query_key,
            ttl,
            json.dumps(cache_data)
        )

# Usage in AI endpoint
cache = SemanticCache(redis_client=Redis(host='localhost', port=6379))

def handle_ai_request(query, context):
    context_hash = hashlib.md5(json.dumps(context).encode()).hexdigest()
    
    # Check cache
    cached_response = cache.get(query, context_hash)
    if cached_response:
        logger.info(f"Cache hit (similarity: {cached_response.get('similarity_score', 1.0)})")
        return cached_response["response"]
    
    # Cache miss - call AI model
    response = call_ai_model(query, context)
    
    # Store in cache
    cache.set(query, context_hash, response, ttl=3600)
    
    return response
```

**Cache Invalidation Strategy:**
```yaml
# Cache Management Configuration
cacheManagement:
  strategy: "adaptive"
  
  ttl:
    static_content: 86400  # 24 hours (policies, procedures)
    product_info: 3600     # 1 hour (pricing, availability)
    user_specific: 300     # 5 minutes (personalized content)
    real_time: 0           # No caching (stock prices, live data)
  
  invalidation:
    # Proactive invalidation on content updates
    triggers:
      - event: "SharePointFileModified"
        action: invalidateCacheBySource
        source: "{{event.fileUrl}}"
      
      - event: "DataverseRecordUpdated"
        action: invalidateCacheByEntity
        entity: "{{event.entityName}}"
        recordId: "{{event.recordId}}"
      
      - event: "ProductCatalogUpdate"
        action: invalidateAll
        pattern: "product:*"
    
    # Reactive invalidation based on staleness detection
    stalenessCheck:
      enabled: true
      checkInterval: 300  # 5 minutes
      maxAge: 3600        # Invalidate if older than 1 hour
  
  warmup:
    # Pre-populate cache with common queries
    enabled: true
    schedule: "0 */6 * * *"  # Every 6 hours
    queries:
      - "What are your business hours?"
      - "How do I reset my password?"
      - "What is your return policy?"
      # ... top 100 common queries
```

### 2.5 Cost Optimization

**Token Usage Optimization:**
```python
# Reduce token consumption without sacrificing quality
class TokenOptimizer:
    def optimize_prompt(self, user_query, context_documents):
        """
        Reduce tokens while maintaining quality
        """
        optimizations = []
        
        # 1. Context compression - only include relevant portions
        relevant_context = self.extract_relevant_passages(
            query=user_query,
            documents=context_documents,
            max_passages=3,
            max_length_per_passage=500
        )
        optimizations.append({
            "step": "context_compression",
            "before_tokens": self.count_tokens(str(context_documents)),
            "after_tokens": self.count_tokens(str(relevant_context))
        })
        
        # 2. Remove redundant information
        deduplicated_context = self.remove_duplicate_content(relevant_context)
        optimizations.append({
            "step": "deduplication",
            "before_tokens": self.count_tokens(str(relevant_context)),
            "after_tokens": self.count_tokens(str(deduplicated_context))
        })
        
        # 3. Compress formatting (remove extra whitespace, etc.)
        compressed_context = self.compress_formatting(deduplicated_context)
        optimizations.append({
            "step": "formatting_compression",
            "before_tokens": self.count_tokens(str(deduplicated_context)),
            "after_tokens": self.count_tokens(str(compressed_context))
        })
        
        # 4. Use concise system prompt
        system_prompt = self.get_concise_system_prompt()
        
        total_saved = (
            optimizations[0]["before_tokens"] - 
            optimizations[-1]["after_tokens"]
        )
        
        return {
            "optimized_context": compressed_context,
            "system_prompt": system_prompt,
            "tokens_saved": total_saved,
            "optimization_steps": optimizations
        }
    
    def extract_relevant_passages(self, query, documents, max_passages, max_length_per_passage):
        """Use embedding similarity to extract only relevant passages"""
        from sentence_transformers import SentenceTransformer, util
        
        model = SentenceTransformer('all-MiniLM-L6-v2')
        
        # Embed query
        query_embedding = model.encode(query, convert_to_tensor=True)
        
        # Split documents into passages and embed
        passages = []
        for doc in documents:
            doc_passages = self.split_into_passages(doc, max_length_per_passage)
            passages.extend(doc_passages)
        
        passage_embeddings = model.encode(passages, convert_to_tensor=True)
        
        # Calculate similarities
        similarities = util.cos_sim(query_embedding, passage_embeddings)[0]
        
        # Get top passages
        top_indices = similarities.argsort(descending=True)[:max_passages]
        relevant_passages = [passages[i] for i in top_indices]
        
        return relevant_passages

# Usage
optimizer = TokenOptimizer()

def ai_call_with_optimization(user_query, context_documents):
    # Optimize before calling AI
    optimized = optimizer.optimize_prompt(user_query, context_documents)
    
    logger.info(f"Tokens saved: {optimized['tokens_saved']}")
    
    # Call AI with optimized context
    response = llm_client.complete(
        prompt=user_query,
        system=optimized["system_prompt"],
        context=optimized["optimized_context"]
    )
    
    return response
```

**Cost Monitoring and Budgeting:**
```python
# Real-time cost tracking and budget enforcement
class CostMonitor:
    def __init__(self, budget_limits):
        self.budgets = budget_limits
        self.redis = Redis()
    
    def check_budget(self, user_id, department):
        """Check if request is within budget"""
        # Get current spend
        daily_key = f"cost:daily:{date.today()}:{department}"
        monthly_key = f"cost:monthly:{date.today().strftime('%Y-%m')}:{department}"
        
        daily_spend = float(self.redis.get(daily_key) or 0)
        monthly_spend = float(self.redis.get(monthly_key) or 0)
        
        # Check limits
        if daily_spend >= self.budgets[department]["daily"]:
            return False, "Daily budget exceeded"
        
        if monthly_spend >= self.budgets[department]["monthly"]:
            return False, "Monthly budget exceeded"
        
        return True, "OK"
    
    def track_cost(self, user_id, department, tokens_used, model):
        """Record cost of AI call"""
        # Calculate cost
        cost = self.calculate_cost(tokens_used, model)
        
        # Update counters
        daily_key = f"cost:daily:{date.today()}:{department}"
        monthly_key = f"cost:monthly:{date.today().strftime('%Y-%m')}:{department}"
        
        self.redis.incrbyfloat(daily_key, cost)
        self.redis.incrbyfloat(monthly_key, cost)
        
        # Set expiry
        self.redis.expire(daily_key, 86400)  # 1 day
        self.redis.expire(monthly_key, 2592000)  # 30 days
        
        # Check if approaching limit
        daily_spend = float(self.redis.get(daily_key))
        if daily_spend > self.budgets[department]["daily"] * 0.8:
            self.send_alert(department, "approaching", "daily", daily_spend)
    
    def calculate_cost(self, tokens, model):
        """Calculate cost based on model pricing"""
        pricing = {
            "gpt-4-turbo": {"input": 0.01, "output": 0.03},
            "gpt-35-turbo": {"input": 0.0005, "output": 0.0015},
            "gpt-4o-mini": {"input": 0.00015, "output": 0.0006}
        }
        
        # Simplified - in reality, track input/output separately
        avg_price = (pricing[model]["input"] + pricing[model]["output"]) / 2
        cost = (tokens / 1000) * avg_price
        
        return cost

# Integrate with AI calls
cost_monitor = CostMonitor(budget_limits={
    "sales": {"daily": 100, "monthly": 2000},
    "support": {"daily": 200, "monthly": 4000},
    "marketing": {"daily": 50, "monthly": 1000}
})

def protected_ai_call(user_id, department, query, context):
    # Check budget
    allowed, reason = cost_monitor.check_budget(user_id, department)
    if not allowed:
        raise BudgetExceededException(reason)
    
    # Make AI call
    response = call_ai_model(query, context)
    
    # Track cost
    cost_monitor.track_cost(
        user_id=user_id,
        department=department,
        tokens_used=response.usage.total_tokens,
        model=response.model
    )
    
    return response
```

---

## 3. Performance Tuning Best Practices

### 3.1 Baseline and Benchmark

**Establish Performance Baselines:**
```yaml
# Define target metrics before deployment
performanceBaselines:
  latency:
    simple_query:
      p50: 1000ms
      p95: 2000ms
      p99: 3000ms
    complex_query:
      p50: 3000ms
      p95: 5000ms
      p99: 8000ms
  
  quality:
    groundedness: 4.0
    relevance: 4.0
    coherence: 4.5
    fluency: 4.5
  
  userSatisfaction:
    csat_target: 4.0  # out of 5
    resolution_rate: 0.75
    escalation_rate: 0.10
  
  cost:
    cost_per_interaction: 0.05  # USD
    monthly_budget: 10000
  
  availability:
    uptime: 0.999  # 99.9%
    error_rate: 0.01  # 1%
```

### 3.2 Continuous Improvement Cycle

```
┌─────────────────────────────────────────────────────────┐
│         Continuous Performance Improvement              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. Measure                                             │
│     - Collect telemetry (24/7)                          │
│     - Sample conversations for quality eval             │
│     - Gather user feedback                              │
│                                                          │
│  2. Analyze                                             │
│     - Identify patterns in failures                     │
│     - Compare to baselines                              │
│     - Prioritize issues by impact                       │
│                                                          │
│  3. Hypothesize                                         │
│     - What changes might improve performance?           │
│     - Prompt refinement, model change, index tuning?    │
│                                                          │
│  4. Experiment                                          │
│     - A/B test proposed changes                         │
│     - Measure impact                                    │
│     - Statistical significance testing                  │
│                                                          │
│  5. Deploy                                              │
│     - Gradual rollout (10% → 50% → 100%)               │
│     - Monitor for regressions                           │
│     - Rollback if needed                                │
│                                                          │
│  6. Repeat                                              │
│     - Continuous optimization                           │
│     - Weekly/monthly review cycles                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 3.3 Optimization Checklist

**Weekly Optimization Review:**
```markdown
## Weekly AI Performance Review Checklist

### Metrics Review
- [ ] Check P95 latency vs target (< 5s for complex queries)
- [ ] Review error rate (target < 1%)
- [ ] Analyze quality scores (groundedness, relevance > 4.0)
- [ ] Check CSAT scores (target > 4.0)
- [ ] Review cost per interaction (< $0.05)

### Quality Analysis
- [ ] Review sample of low-quality responses
- [ ] Identify common failure patterns
- [ ] Check citation accuracy
- [ ] Verify no hallucinations in sample

### Cost Analysis
- [ ] Review token consumption trends
- [ ] Identify opportunities for caching
- [ ] Check if routing rules are optimal
- [ ] Verify budget compliance

### User Feedback
- [ ] Review user complaints/issues
- [ ] Identify feature requests
- [ ] Check satisfaction trends

### Action Items
- [ ] List proposed optimizations
- [ ] Prioritize by impact
- [ ] Schedule experiments
- [ ] Assign owners
```

---

## Key Takeaways

✅ **Monitoring is Critical:**
- Implement comprehensive telemetry from day one
- Use Application Insights, Copilot Studio Analytics, and custom metrics
- Monitor latency, quality, cost, and user satisfaction

✅ **Quality Metrics:**
- Automate evaluation with judge models (groundedness, relevance, coherence)
- Sample production conversations regularly
- Track user satisfaction (CSAT, NPS)

✅ **Performance Tuning:**
- Optimize prompts iteratively with A/B testing
- Tune RAG (chunking, retrieval, ranking)
- Implement caching (semantic caching for common queries)
- Optimize model routing for cost/quality balance

✅ **Cost Management:**
- Track token consumption in real-time
- Implement budget limits and alerts
- Optimize context size
- Use appropriate models for each task

✅ **Continuous Improvement:**
- Establish baselines before deployment
- Weekly/monthly performance reviews
- Experiment with changes before full deployment
- Maintain feedback loop from monitoring to optimization

---

## Study Resources

- [Azure AI Foundry Monitoring](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/monitor-quality-safety)
- [Application Insights for AI](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Copilot Studio Analytics](https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-overview)
- [Azure AI Search Performance](https://learn.microsoft.com/en-us/azure/search/search-performance-tips)
- [Prompt Engineering Guide](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)

**Next Topic:** [Testing AI Solutions](02-testing.md)
