# CarePath – An Intelligent Patient Navigation Agent

---

## 📋 Overview

**CarePath** is a multi-agent healthcare navigation system that acts as an intelligent care companion helping patients with chronic conditions (e.g., diabetes, hypertension) manage their health by:

- 🏥 **Building structured patient profiles** with conditions, medications, and health goals
- 📊 **Monitoring symptoms** over time and detecting risky patterns  
- 💊 **Managing medication adherence** and tracking compliance
- 📚 **Simplifying medical jargon** into plain language  
- 🎓 **Providing personalized lifestyle guidance** through LLM-powered recommendations
- 🔄 **Remembering patients** across sessions and days via persistent memory

### ⚠️ Ethical Foundation

**CarePath does NOT:**
- Diagnose medical conditions
- Prescribe medications or treatments
- Replace healthcare providers
- Provide personalized medical advice

**CarePath DOES:**
- Act as an intelligent care organizer and guide
- Help patients understand their conditions
- Remind patients of medication schedules
- Detect patterns for physician discussion
- Provide evidence-based lifestyle education

All examples are **simulated** for educational purposes.

---

## 🏗️ Architecture Overview

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/48735b30-a641-462f-aa9c-2a69cf29134f" />


### Multi-Agent System (5 Specialized Agents)

```
┌─────────────────────────────────────────────────────────────┐
│                    CarePath Orchestrator                     │
└─────────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────────┐
│ Sequential Flow:                                            │
│ 1. HealthProfileAgent       → Build patient profile        │
│ 2. SymptomMonitoringAgent   → Log & analyze symptoms       │
│ 3. MedicationMgmtAgent      → Track adherence & check drug interactions
│ 4. MedicalInterpretationAgent → Simplify lab results       │
│ 5. EducationAgent           → Generate personalized guidance│
└─────────────────────────────────────────────────────────────┘
```

**Execution Patterns:**
- **Sequential:** Profile → Symptoms → Medications → Interpretation → Education
- **Parallel:** Symptom & Medication agents run conceptually in parallel on same state
- **Loop:** Multi-day monitoring cycles simulating long-term care

### Tools Ecosystem

| Tool Type | Implementation | Purpose |
|-----------|----------------|---------|
| **Custom Tool** | `MedicationKnowledgeTool` | Drug info, interactions, advisory text |
| **Code Execution** | `TrendAnalysisTool` | Analyze symptom severity trends, detect patterns |
| **OpenAPI-style** | `MockEHRApiTool` | Simulate EHR API calls for lab data |
| **MCP-style** | `MedicalContentMCPTool` | Retrieve lifestyle guidance snippets |

### Sessions & Memory

- **InMemorySessionService:** Session lifecycle, checkpoints, history
- **MemoryBank:** Persistent storage of patient profiles, symptom timelines, adherence stats
- **Context Compaction:** Summarize long histories into LLM-friendly formats
- **Pause/Resume:** Tasks stored with status (PENDING/IN_PROGRESS/COMPLETED)

### Observability

- **Logging:** All agent actions captured to session logs
- **Tracing:** Ordered execution trace with timestamps & agent IDs
- **Metrics:** Interactions, risk flags, adherence ratios, session duration
- **MetricsCollector:** Real-time aggregation and summary statistics

### A2A Communication

- **AgentMessage Protocol:** Structured dataclass for inter-agent messages
- **MessageBus:** Routes messages to agents, manages orchestration
- **Parent/Child Relationships:** Tracks message lineage and dependencies

### Deployment Ready

- **REST API Interface:** FastAPI-style endpoints (create session, log symptom, get guidance, retrieve history)
- **Microservice Architecture:** Ready for Cloud Run, Agent Engine, or FastAPI deployment
- **Simulated EHR Compatibility:** HL7 FHIR-style structured data

---

## 📊 Core Components

### 1. Data Structures (`@dataclass`)
- `Patient`: Name, age, conditions, medications, allergies, goals
- `Symptom`: Description, severity, timestamp, related conditions
- `SymptomTrend`: Analyzed trends with risk flags
- `Medication`: Drug info, indications, side effects, interactions
- `MedicationAdherence`: Tracking compliance metrics
- `Task`: Long-running task state (status, payload, result)
- `AgentMessage`: A2A protocol message format

### 2. Agents

**HealthProfileAgent**
- Builds/maintains structured patient profile
- Validates conditions, medications, goals
- Stores in MemoryBank for cross-session access

**SymptomMonitoringAgent**
- Logs patient symptoms with severity
- Analyzes trends using TrendAnalysisTool
- Detects risky patterns (increasing severity, high thresholds)
- Flags for physician escalation

**MedicationManagementAgent**
- Tracks medication adherence (scheduled vs. taken doses)
- Checks for drug interactions using MedicationKnowledgeTool
- Simulates medication refill reminders
- Stores adherence stats in MemoryBank

**MedicalInterpretationAgent**
- Fetches lab results from MockEHRApiTool
- Simplifies medical jargon into plain language
- Explains normal vs. abnormal values
- Contextualizes to patient's conditions

**EducationAgent**
- LLM-powered personalized recommendations (via Gemini)
- Retrieves topic-specific guidance (nutrition, exercise, stress, adherence)
- Adapts to patient's risk flags and conditions
- Always includes disclaimer to consult doctor

### 3. Tools

**MedicationKnowledgeTool**
```python
med_info = MEDICATION_TOOL.get_medication_info("Metformin")
has_interaction, msg = MEDICATION_TOOL.check_interaction("Metformin", "Lisinopril")
```

**TrendAnalysisTool**
```python
trend = TREND_TOOL.analyze_trend(symptoms)
# Returns: SymptomTrend with trend_direction, risk_flag, severity stats
```

**MockEHRApiTool**
```python
labs = EHR_TOOL.fetch_patient_labs(patient_id)
refills = EHR_TOOL.fetch_medication_refill_status(patient_id)
```

**MedicalContentMCPTool**
```python
guidance = CONTENT_TOOL.get_lifestyle_guidance("diabetes_nutrition")
```

### 4. Orchestration

**CarePathOrchestrator**
- `run_full_patient_workflow()`: Execute complete agent pipeline for a patient
- `run_multi_day_monitoring()`: Simulate multi-day monitoring loop
- Manages session state, tracks metrics, stores checkpoints

**MessageBus**
- Registers agents
- Routes AgentMessages to appropriate agents
- Handles error cases gracefully

---

## 🎯 10 Comprehensive Demos

The notebook includes **10 demonstration scenarios** showcasing all key features:

### Demo 1: Single Patient Full Workflow
- Complete end-to-end execution of all 5 agents
- Shows profile building, symptom logging, adherence tracking, lab interpretation, education

### Demo 2: Multi-Day Monitoring (Loop Pattern)
- Simulates 3 days of daily patient check-ins
- Tracks symptom severity trends over time
- Demonstrates consistency of medication adherence

### Demo 3: Parallel Agent Execution
- Shows SymptomMonitoringAgent & MedicationManagementAgent running conceptually in parallel
- Validates execution trace for proper ordering

### Demo 4: Session Persistence & Pause/Resume
- Creates a session, runs workflow, saves checkpoint
- Resumes session with new data, demonstrating long-running task support
- Shows full session history across multiple interactions

### Demo 5: Tools & External API Integration
- Demonstrates each tool type in action
- Custom tool: medication knowledge lookup & interaction checking
- Code execution: symptom trend analysis
- API tool: simulated EHR lab data fetching
- MCP tool: lifestyle guidance retrieval

### Demo 6: A2A Communication & MessageBus
- Shows direct agent-to-agent messaging via MessageBus
- Validates message routing and response handling

### Demo 7: Observability & Metrics
- Displays collected metrics (interactions, risk flags, adherence, duration)
- Shows memory bank contents (patient profiles, symptom timelines)
- Demonstrates logging & tracing infrastructure

### Demo 8: Agent Evaluation & Quality Metrics
- Computes profile completeness (0-1 score)
- Counts risk flags detected
- Measures medication adherence ratio
- Verifies agent coverage (all executed successfully)
- Calculates overall quality score

### Demo 9: REST API Interface (Deployment)
- POST /api/v1/sessions/patient → Create session & run workflow
- GET /api/v1/sessions/{session_id}/history → Retrieve session logs
- POST /api/v1/sessions/{session_id}/symptoms → Log new symptom
- GET /api/v1/sessions/{session_id}/guidance → Get personalized guidance

### Demo 10: Full System Summary & Key Takeaways
- Comprehensive recap of all features
- Ethical guardrails emphasized
- Production deployment guidance
- Use cases illustrated

---

## 🔑 Concepts Demonstrated

✅ **Multi-Agent Systems**
- Sequential, parallel, and loop execution patterns
- Agent orchestration with state management
- Proper error handling and logging

✅ **Tool Integration**
- Custom tool implementation
- Code execution tool (TrendAnalysisTool)
- OpenAPI-style tool (MockEHRApiTool)
- MCP-style tool (MedicalContentMCPTool)

✅ **Sessions & Memory**
- In-memory session service with history
- MemoryBank for persistent patient data
- Checkpoint system for pause/resume

✅ **Context Engineering**
- Context compaction (long histories → LLM-friendly summaries)
- Token optimization for LLM calls
- Incremental context updates

✅ **Observability**
- Structured logging with timestamps
- Execution tracing with agent IDs
- Metrics collection & aggregation
- Real-time monitoring

✅ **Agent-to-Agent (A2A) Communication**
- AgentMessage protocol with structured data
- MessageBus for message routing
- Parent/child task relationships

✅ **Long-Running Operations**
- Task objects with status tracking
- Checkpoint persistence
- Resume capability across sessions

✅ **Agent Evaluation**
- Synthetic patient generation
- Quality metrics (completeness, coverage, adherence)
- Performance benchmarking

✅ **Deployment**
- REST API interface (FastAPI-style)
- Microservice-ready architecture
- Session management for scalability

✅ **Ethical AI**
- Clear disclaimers and limitations
- Transparency about AI capabilities
- Guardrails preventing diagnosis/prescription
- User-centric design

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- Google Gemini API key (set as environment variable `GOOGLE_API_KEY` or via Kaggle secrets)
- Kaggle Notebook environment (or local Jupyter)

### Running the Notebook

1. **Open in Kaggle:**
   - Upload to Kaggle Notebooks
   - Add `GOOGLE_API_KEY` to Kaggle Secrets
   - Run all cells

2. **Run Locally:**
   ```bash
   # Install dependencies
   pip install google-generativeai pandas numpy python-dateutil
   
   # Set API key
   export GOOGLE_API_KEY="your-key-here"
   
   # Run in Jupyter
   jupyter notebook carepath-agent.ipynb
   ```

### Key Cells to Run
1. **Setup & Imports** (Cell 1-3): Initialize environment
2. **Observability & Data Structures** (Cell 4-5): Setup logging and types
3. **Session & Memory** (Cell 6-7): Initialize state management
4. **Tools** (Cell 8): Load all tool implementations
5. **Agents** (Cell 9): Define all 5 agents
6. **Orchestrator** (Cell 10-11): Setup orchestration
7. **Demos** (Cell 12+): Run all 10 demonstration scenarios

---

## 📈 Sample Output

```
DEMO 1: Single Patient Full Workflow
======================================================================

👤 Patient: Alice
   Conditions: Type 2 Diabetes, Hypertension
   Medications: Metformin, Lisinopril

✅ Workflow completed in 2.34 seconds
   Session ID: SES-a1b2c3d4
   Total interactions: 5

📋 Patient Profile:
   ID: PAT-xyz789
   Name: Alice
   Goals: ['Maintain HbA1c < 7%', 'Lose 10 lbs']

⚠️  Symptom Monitoring:
   • Morning glucose spike: increasing trend (severity avg: 6.5)

💊 Medication Adherence:
   Metformin: ██████████ 100%
   Lisinopril: ████████░░ 80%

🧪 Lab Interpretation:
   • HbA1c is 6.8 (normal). Keep up current habits!
   • Fasting Glucose is 115 mg/dL (a bit high). Increase exercise...
```

---

## 🔧 Customization

### Add New Patient Template
```python
PATIENT_GENERATOR.patient_templates.append({
    "name": "David",
    "age": 55,
    "conditions": ["Type 1 Diabetes"],
    "medications": [{"name": "Insulin", "dose": "20 units"}],
    # ...
})
```

### Add Custom Medication Knowledge
```python
MEDICATION_TOOL.knowledge_base["NewDrug"] = {
    "indication": "...",
    "typical_dose": "...",
    "side_effects": [...],
    "interactions": [...],
    "advisory": "..."
}
```

### Add New Agent
```python
class MyCustomAgent(BaseAgent):
    def handle(self, msg: AgentMessage) -> Dict[str, Any]:
        # Your implementation
        return {...}

MESSAGE_BUS.register_agent(MyCustomAgent())
```

---

## 📦 Production Deployment Checklist

- [ ] Replace `InMemorySessionService` with persistent DB (PostgreSQL, Firestore)
- [ ] Replace `MockEHRApiTool` with real HL7 FHIR API integration
- [ ] Add HIPAA compliance & end-to-end encryption
- [ ] Implement OAuth2 authentication & RBAC authorization
- [ ] Deploy with message queue (Celery, RabbitMQ) for async task processing
- [ ] Add vector DB (Pinecone, Weaviate) for semantic search of medical content
- [ ] Integrate with real calendar/reminder systems (Google Calendar API, Twilio)
- [ ] Establish clinical validation & feedback loop
- [ ] Set up monitoring & alerting (Datadog, New Relic)
- [ ] Create audit logs for compliance
- [ ] Add rate limiting & cost controls
- [ ] Scale to Kubernetes / Cloud Run

---

## 🎓 Learning Outcomes

By studying this capstone, you'll understand:

1. **How to build complex multi-agent systems** with proper orchestration
2. **Tool integration patterns** (custom, API, MCP, code execution)
3. **Session management** for long-lived, multi-turn interactions
4. **Context engineering** for efficient LLM usage
5. **Observability best practices** (logging, tracing, metrics)
6. **A2A communication protocols** for scalable agent networks
7. **Deployment architecture** for production microservices
8. **Ethical AI principles** in healthcare applications
9. **Evaluation frameworks** for multi-agent systems
10. **Real-world constraints** (HIPAA, patient privacy, liability)

---

## 📚 References

- [Google Agentic AI Course](https://kaggle.com/learn/agents-for-good)
- [HL7 FHIR Standard](https://www.hl7.org/fhir/)
- [HIPAA Compliance Guide](https://www.hhs.gov/hipaa/)
- [Gemini API Documentation](https://ai.google.dev/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

## ⚖️ Legal & Ethical Disclaimer

**CarePath is an educational demonstration system, not a medical device.**

- All patient data is synthetic and simulated
- No real medical advice is provided
- System does not diagnose or prescribe
- Users must consult healthcare providers for medical decisions
- Intended solely for learning purposes
- Not for deployment in actual patient care without regulatory approval

---

**Built with ❤️ for better patient care.**  
**Jahnavi Umarji | Google Agentic AI Course | Capstone Submission**

---

## 📄 Files Included

- `carepath-agent.ipynb` — Main Jupyter notebook (all code, demos, documentation)
- `CAREPATH_README.md` — This file (comprehensive reference guide)

---

**Status: ✅ Complete & Ready for Evaluation**

Last Updated: November 27, 2025
