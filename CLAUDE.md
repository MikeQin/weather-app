# Weather App N8N Workflow - Project Documentation

## Project Overview
Multi-agent n8n workflow for US weather inquiries supporting alerts by state and forecasts by city/state using National Weather Service API.

### Documentation Suite
- **[generated-PRD.md](generated-PRD.md)**: Comprehensive Product Requirements Document reverse-engineered from the working system
- **[N8N-WORKFLOW.md](N8N-WORKFLOW.md)**: Complete implementation guide
- **[README.md](README.md)**: Project setup and usage instructions  
- **[MEDIUM.md](MEDIUM.md)**: Technical showcase article

## Architecture
**Multi-Agent Approach:**
- Chat Trigger → Supervisor AI Agent (Intent Classifier) → JSON Parser → Sub AI Agents
- Two sub-workflows: Alert Handler + Forecast Handler
- Minimal Code nodes, leverage AI agent capabilities
- **JSON Parser Code Node** parses Supervisor output for downstream routing

## Executive Summary

**Total Nodes Required: 12**

### Core Workflow (4 nodes):
1. **Chat Trigger Node** - Entry point
2. **Supervisor AI Agent** - Intent classification & general Q&A
3. **JSON Parser Code Node** - Parses AI output for downstream access
4. **Intent Switch Node** - Routes to appropriate workflow

### General Question Path (2 nodes):
5. **Text Extractor Code Node** - Extracts answer text from JSON
6. **Respond to Webhook Node** - Returns general responses to user

### Alert Sub-Workflow (2 nodes):
7. **NWS Alerts HTTP Request** - API call for alerts
8. **Alert Formatter AI Agent** - Formats alert response

### Forecast Sub-Workflow (4 nodes):
9. **Geocoding AI Agent** - City/state to coordinates
10. **NWS Points HTTP Request** - Gets forecast grid info
11. **NWS Forecast HTTP Request** - Gets actual forecast
12. **Forecast Formatter AI Agent** - Formats forecast response

### Response Handling
**Weather Paths**: Formatter AI Agents output directly to user
**General Path**: Intent Switch connects to Respond to Webhook Node

**Text Extractor + Respond to Webhook Configuration:**
- **Text Extractor**: Wraps answer text in `{ text: answerText }` structure
- **Respond to Webhook Parameters**:
  - **Respond With**: Text
  - **Response Body**: `{{ $json.text }}` (Expression mode enabled)
- **Respond to Webhook Options**:
  - **Response Code**: 200
  - **Response Headers**: Content-Type: text/plain
- **Connected from**: Text Extractor Code Node (Node 5)

**Note**: All paths properly terminate with user-visible responses

**Implementation Priority:**
- **Start with 4 nodes**: Chat Trigger + Supervisor + JSON Parser + Switch
- **Add General path**: Nodes 5-6 (Text Extractor + Respond to Webhook)
- **Add Alert path**: Nodes 7-8 (simpler, no geocoding)
- **Add Forecast path**: Nodes 9-12 (more complex)
- **Complete**: All paths properly terminate with user-visible responses

## User Scenarios
1. **Weather Forecast by City**: "Tell me the weather in Dallas, Texas" → Forecast workflow
2. **Weather Alerts by State**: "Tell me the weather alerts in Texas" → Alert workflow
3. **Forecast with State Only**: "Tell me the weather forecast in Texas" → Forecast workflow (defaults to largest city)
4. **Alerts with City Mentioned**: "Tell me the weather alerts in Dallas, TX" → Alert workflow (ignores city, uses state)
5. **General Questions**: "What time is it?" or "How do I cook pasta?" → Respond to Webhook Node returns Supervisor's formatted answer with weather service redirect

## Data Flow by Scenario
### Weather Requests:
`User → Supervisor (JSON) → JSON Parser → Switch → Sub-workflow → Formatter → User`

### General Questions:
`User → Supervisor (text) → JSON Parser (creates answer field) → Switch Rule 3 → Text Extractor → Respond to Webhook Node → User`

## Technical Specifications

### API Configuration
- **Base URL**: `https://api.weather.gov`
- **User-Agent**: `weather-app/1.0`
- **Accept**: `application/geo+json`
- **Timeout**: 30 seconds

### Workflow Components
1. **Chat Trigger Node**: Captures user input
2. **Supervisor AI Agent**: Intent classification, routing, and general Q&A
   - Extracts intent, state, and city from user messages
   - Returns clean JSON for weather requests
   - Provides well-formatted responses for general questions with weather service redirects
3. **JSON Parser Code Node**: Exposes structured data for downstream access
4. **Alert Sub-Agent**: Handles state-based alert requests
5. **Forecast Sub-Agent**: Handles city/state forecast requests
6. **HTTP Request Nodes**: API calls to NWS
7. **Switch Nodes**: Flow control
8. **Response Formatting**: Sub-agents format weather responses, supervisor handles general questions directly

### API Endpoints
- **Alerts**: `GET /alerts/active/area/{state_code}`
- **Forecast Points**: `GET /points/{latitude},{longitude}`
- **Forecast**: `GET /gridpoints/{office}/{grid_x},{grid_y}/forecast`

## Implementation Requirements

### Alert Workflow
- Input: US state (2-letter code)
- Process: Direct API call to alerts endpoint
- Output: Formatted alert information

### Forecast Workflow  
- Input: City, State
- Process: Geocoding AI Agent → Points API → Forecast API
- Output: 5-period weather forecast
- Default: Use largest city when only state provided

### Geocoding Sub-Agent
- AI agent converts city/state to latitude/longitude coordinates
- Uses built-in geographical knowledge
- Handles state-only requests by defaulting to largest city
- **CRITICAL**: Must preserve ALL input data and ADD coordinates
- Input: {"intent": "forecast", "state": "TX", "city": "Dallas"}
- Output: {"intent": "forecast", "state": "TX", "city": "Dallas", "latitude": 32.7767, "longitude": -96.7970}
- NWS Points API accesses via JSON.parse($json.output).latitude

### Data Processing
- State name → 2-letter code conversion (AI agent)
- City/State → Latitude/Longitude geocoding (AI agent) with data preservation
- JSON response formatting
- Error handling for invalid locations/API failures
- **Data Flow Enhancement**: Geocoding agent preserves all workflow context while adding coordinates

## Response Formatting Strategy
- **Weather Requests**: Sub-agents handle domain-specific formatting
  - Alert Agent: Formats alert features into structured, readable format (event, area, severity, description, instructions)
  - Forecast Agent: Formats weather periods (name, temperature, wind, detailed forecast)
  - Direct output to user with clear formatting
- **General Questions**: Supervisor handles directly with natural conversational responses

## AI-Driven Edge Case Management

**Architectural Philosophy**: Instead of implementing traditional error handling code, this system delegates edge cases to AI agents through natural language understanding.

**This is actually a more sophisticated approach that provides better user experience through natural language interaction.**

### AI Agent Resilience
- **API timeout/unavailability**: AI agents explain delays and suggest alternatives naturally
- **Invalid location inputs**: Geocoding agent uses geographical knowledge to handle ambiguous inputs
- **Missing geocoding data**: Agents provide helpful guidance for location clarification
- **Rate limiting considerations**: Natural language responses about temporary unavailability
- **Unexpected inputs**: Agents handle unusual requests through conversational responses

### Benefits
- **User-friendly responses**: Conversational explanations instead of technical error codes
- **Contextual guidance**: AI agents understand user intent and provide relevant help
- **Graceful degradation**: System remains functional and helpful even when components fail
- **Natural interaction**: Users receive human-like responses to problems

## Development Guidelines
- Prioritize AI agent logic over Code nodes
- Use Switch nodes for flow control
- **Leverage AI agent intelligence for edge cases** instead of traditional error handling
- Follow n8n best practices
- **CRITICAL**: All AI Agents must use Memory Store: `{{ $execution.id }}`
- Update this documentation with any requirement changes

### Key Architectural Decisions
1. **Multi-Agent Specialization**: Each agent focused on specific domain expertise
2. **AI-Driven Resilience**: Natural language handling of edge cases and errors
3. **Execution-Based Memory**: Unified session management across all agents
4. **Minimal Code Nodes**: Leverage AI capabilities over custom logic

## Memory Management
**SOLUTION**: Unified approach using n8n built-in execution ID

**Memory Store Configuration:**
- **ALL AI Agents**: `{{ $execution.id }}` (unique workflow execution identifier)
- **Consistent across all nodes**: Always available, no data flow dependencies
- **Simple and reliable**: Each workflow execution gets distinct memory space

**Data Flow**: 
```
Chat Trigger: { chatInput: "weather" }
↓
Supervisor Output: {"intent": "forecast", "state": "TX", "city": "Dallas"}
↓
JSON Parser: Exposes intent/location data for routing
↓
ALL AI Agents: Use {{ $execution.id }} for memory store
```

**Benefits**: No complex session tracking, always available, consistent throughout workflow

## Next Steps
1. ✅ Design detailed workflow architecture 
2. ✅ Create mermaid diagram
3. Implement supervisor intent classification + JSON Parser (4 nodes)
4. **Add General Question Path**: 
   - Add Text Extractor Code Node (Node 5): Extract answer text from JSON
   - Add Respond to Webhook Node (Node 6): Parameters: Respond With: Text, Response Body: `{{ $json.text }}` (Expression mode)
   - Connect: Intent Switch Rule 3 → Text Extractor → Respond to Webhook
   - Response Code: 200
5. Build alert sub-workflow (nodes 7-8)
6. Build forecast sub-workflow with geocoding (nodes 9-12)
7. Test all scenarios (weather alerts, forecasts, AND general questions)
8. Deploy and monitor

## Critical Implementation Notes
- **JSON Parser Code Node is mandatory** for intent routing and data access
- **Unified Memory Store approach**: 
  - **ALL AI Agents**: `{{ $execution.id }}` (built-in n8n variable)
  - **Consistent and reliable**: Always available regardless of node position
  - **No session complexity**: Simplified workflow with execution-based memory
- **Switch Node rules simplified** after JSON parsing
- **HTTP Request URLs updated** to use appropriate access patterns
- **Response Presenter removed**: Formatter agents output directly to user (eliminates redundant layer)
- **Geocoding AI Agent must preserve input data**: Add coordinates to all input fields instead of returning only coordinates
- **JSON.parse access pattern**: Use `{{ JSON.parse($json.output).latitude }}` for coordinate access
- **Simplified Supervisor AI Agent**: Returns only intent, state, and city data
- **No session handling complexity**: Memory management handled by execution.id across all nodes
- **No markdown formatting**: AI Agents must return raw JSON without ``` code blocks
- **Geocoding AI Agent must preserve input data**: Add coordinates to all input fields instead of returning only coordinates
- **JSON.parse access pattern**: Use `{{ JSON.parse($json.output).latitude }}` for coordinate access
- **Direct field access for parsed data**: `{{ $json.state }}`, `{{ $json.city }}`, `{{ $json.intent }}` from JSON Parser

## Reverse Engineering Methodology

**Innovation**: This project demonstrates a unique **reverse engineering approach** to PRD generation:

1. **Build working system first** → weather-app.json with real functionality
2. **Analyze implementation artifacts** → JSON workflow + visual diagram  
3. **Extract comprehensive requirements** → generated-PRD.md with actual working specifications
4. **Document proven architecture** → Complete technical specifications

**Value**: The generated-PRD.md serves as both documentation AND template for building similar multi-agent systems, ensuring specifications reflect **actual working implementation** rather than theoretical requirements.

---
*Complete project documentation with reverse-engineered PRD methodology - Keep updated with any changes*