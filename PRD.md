# Weather App Product Requirements Document

Today's task is to design a n8n workflow for my weather app.

## Requirements:
- The chat trigger node should takes user input for inquiring US weather. 
- There are 2 ways to inquire: 1. `Alert by U.S. state`, 2. `Forecast by City/State`.
- After submitting inquire, user expects to see either alerts by a US state, or a local weather by City, State. But not both.
- If the user asks questions that have nothing to do with weather, forecast and alerts, make the supervisor AI agent directly answers the question without going through sub AI agents.

### Scenarios
- Scenario One:
  * User asks: "Tell me the weather in Dallas, Texas"
  * AI agent should interprets: "The user wants to know the weather forecast for City: Dallas, State: Texas"
  * AI agent should invoke a HTTP request node to inquire `Forecast by City/State`
- Scenario Two:
  * User asks: "Tell me the weather alerts in Texas"
  * AI agent should interprets: "The user wants to know the weather alerts for State: Texas"
  * AI agent should invoke a HTTP request node to inquire `Alert by U.S. state`
- Scenario Three:
  * User asks: "Tell me the weather forecast in Texas"
  * AI agent should interprets: "The user wants to know the weather forecast for State: Texas, but the user doesn't specify the city. I will use the largest city in the state as the default city."
  * AI agent should invoke a HTTP request node to inquire `Forecast by City/State`
- Scenario Four:
  * User asks: "Tell me the weather alerts in Dallas, TX"
  * AI agent should interprets: "The user wants to know the weather alerts for State: Texas. I can safely ingore the city Dallas. Because alerts by State."
  * AI agent should invoke a HTTP request node to inquire `Alert by U.S. state`

## Technical Design Spec:
- The chat trigger node connects to a supervisor AI agent or intent classifier AI agent, who is responsible for routing the call to a sub AI agent.
- There are 2 sub AI agents workflows: 1. handle alert by state, 2. handle forecast by city and state.
- Each sub AI agent workflow uses a HTTP request node to call a weather REST api endpoint to get the data. API call returns JSON format data.
- The entire workflow can use Code node and Switch node to make the flow easy to work with.
- Question: I am not sure which agent should format the response for user. It can be done by either the supervisor agent (intent classifier) or the sub agents.

### IMPORTANT NOTES
- I like the multi-agent architecture approach to make things easy
- Use less `Code` node because many logic can be handled by sub AI agent workflows.

## Weather REST API info:
- NWS_API_BASE = "https://api.weather.gov"
- USER_AGENT = "weather-app/1.0"

## REST API Client Implementation Code in Python
The code below is for your reference only. I don't think we need to implement it in our workflow if we leverage HTTP request node. You can make the decision by yourself.

### HTTP Request Node
```python
from typing import Any
import httpx

# Constants
NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"

async def make_nws_request(url: str) -> dict[str, Any] | None:
    """Make a request to the NWS API with proper error handling."""
    headers = {
        "User-Agent": USER_AGENT,
        "Accept": "application/geo+json"
    }
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None

def format_alert(feature: dict) -> str:
    """Format an alert feature into a readable string."""
    props = feature["properties"]
    return f"""
Event: {props.get('event', 'Unknown')}
Area: {props.get('areaDesc', 'Unknown')}
Severity: {props.get('severity', 'Unknown')}
Description: {props.get('description', 'No description available')}
Instructions: {props.get('instruction', 'No specific instructions provided')}
"""

# HTTP Request Node for alerts by state
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "Unable to fetch alerts or no alerts found."

    if not data["features"]:
        return "No active alerts for this state."

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)

# HTTP Request Node for weather forecast by city and state
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.

    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # First get the forecast grid endpoint
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    if not points_data:
        return "Unable to fetch forecast data for this location."

    # Get the forecast URL from the points response
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if not forecast_data:
        return "Unable to fetch detailed forecast."

    # Format the periods into a readable forecast
    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # Only show next 5 periods
        forecast = f"""
{period['name']}:
Temperature: {period['temperature']}°{period['temperatureUnit']}
Wind: {period['windSpeed']} {period['windDirection']}
Forecast: {period['detailedForecast']}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)
```

## Rules
- If you don't know how to do something, please ask me. Don't hallucinate.
- Use best practices in n8n workflow.

## Documentation
- Based upon the above information, generate a `CLAUDE.md` file for the project.
- If product requirements change during development, please update `CLAUDE.md` immediately.
- For every new session, always read this `PRD.md` and `CLAUDE.md` first.
- Please generate n8n workflow diagrams in `mermaid` format and save it in `N8N-WORKFLOW.md`.
- IMPORTANT: `N8N-WORKFLOW.md` MUST contain all node implementation details. This is the ONLY document I really concern about.
- IMPORTANT: Create `Executive Summary` in `CLAUDE.md` and `N8N-WORKFLOW.md` detailing how many nodes to be implemented, and their functionality.

## Conversation State Management
- For starting a new conversation session, please always read this `PRD.md` first.
- If there is a `CLAUDE.md` in the project folder, please read it as well. If `CLAUDE.md` does not exist, please create a new one after understanding the `product requirements document` - the `PRD.md` document.
- Please update `CLAUDE.md` during our conversation session immediately.