# Multi-Agent Weather System with n8n

A sophisticated 12-node n8n workflow that demonstrates multi-agent AI architecture for weather information retrieval and general question handling.

![Weather App n8n Workflow](weather-app-n8n.jpg)

## 🌟 Features

- **Weather Alerts**: Get active weather alerts by US state
- **Weather Forecasts**: Detailed 5-day forecasts by city and state
- **General Q&A**: Intelligent handling of non-weather questions with graceful redirects
- **Multi-Agent Architecture**: Specialized AI agents for different tasks
- **Real-time Data**: Live integration with National Weather Service API
- **Clean Responses**: Formatted, user-friendly output

## 🏗️ Architecture

### 12-Node Workflow Design

**Core Workflow (4 nodes)**:
1. Chat Trigger Node - Entry point
2. Supervisor AI Agent - Intent classification & general Q&A
3. JSON Parser Code Node - Data transformation
4. Intent Switch Node - Workflow routing

**General Question Path (2 nodes)**:
5. Text Extractor Code Node - Clean text extraction
6. Respond to Webhook Node - User response delivery

**Alert Sub-Workflow (2 nodes)**:
7. NWS Alerts HTTP Request - API integration
8. Alert Formatter AI Agent - Response formatting

**Forecast Sub-Workflow (4 nodes)**:
9. Geocoding AI Agent - Location to coordinates conversion
10. NWS Points HTTP Request - Grid information retrieval
11. NWS Forecast HTTP Request - Forecast data fetching
12. Forecast Formatter AI Agent - Weather data formatting

## 🚀 Quick Start

### Prerequisites

- n8n instance (cloud or self-hosted)
- OpenRouter API account (free tier available)
- Internet connection for National Weather Service API

### Installation

1. **Clone this repository**:
   ```bash
   git clone https://github.com/MikeQin/weather-app.git
   cd weather-app
   ```

2. **Import the workflow**:
   - Open your n8n instance
   - Go to Workflows → Import
   - Upload `weather-app.json`

3. **Configure API credentials**:
   - Create OpenRouter credentials in n8n (Settings → Credentials → Add)
   - Update the credential ID in all AI Agent nodes (replace `YOUR_OPENROUTER_CREDENTIAL_ID`)
   - Test the Chat Trigger webhook

4. **Deploy and test**:
   - Activate the workflow
   - Send test messages via the chat interface

## 📋 Usage Examples

### Weather Forecast
```
User: "Tell me the weather in Dallas, Texas"
Response: Detailed 5-day forecast with temperatures, conditions, and wind information
```

### Weather Alerts
```
User: "Tell me the weather alerts in Texas"
Response: Active warnings with severity levels and safety instructions
```

### General Questions
```
User: "How do I cook pasta?"
Response: Helpful answer with gentle redirect to weather services
```

## 🛠️ Configuration

### Environment Variables

The system uses n8n's built-in execution ID for session management:
```javascript
// All AI agents use this for memory store
{{ $execution.id }}
```

### API Endpoints

- **National Weather Service**: `https://api.weather.gov`
- **User-Agent**: `weather-app/1.0`
- **Timeout**: 30 seconds
- **Rate Limiting**: Handled gracefully

## 📖 Documentation

- **[N8N-WORKFLOW.md](N8N-WORKFLOW.md)**: Complete implementation guide with all node configurations
- **[CLAUDE.md](CLAUDE.md)**: Project architecture and development notes
- **[generated-PRD.md](generated-PRD.md)**: Complete product requirements document (reverse-engineered specification)
- **[PRD.md](PRD.md)**: Original product requirements and specifications  
- **[MEDIUM.md](MEDIUM.md)**: Technical article showcasing the multi-agent approach

### 📋 Complete Documentation Suite

For comprehensive understanding and implementation:
- **Implementation**: See [N8N-WORKFLOW.md](N8N-WORKFLOW.md) for step-by-step setup
- **Architecture**: See [CLAUDE.md](CLAUDE.md) for design decisions  
- **Product Spec**: See [generated-PRD.md](generated-PRD.md) for complete requirements document
- **Technical Article**: See [MEDIUM.md](MEDIUM.md) for multi-agent system insights

## 🧪 Testing

### Test Scenarios

1. **Weather forecast by city**: "Weather in Miami, Florida"
2. **Weather alerts by state**: "Alerts in California"
3. **State-only forecast**: "Weather in Texas" (defaults to largest city)
4. **General questions**: "What time is it?"
5. **Edge cases**: Invalid locations, unusual requests (handled by AI agents)

### Expected Behavior

- Weather requests return structured, formatted responses
- General questions provide helpful answers with weather service redirects
- All error conditions are handled gracefully
- Response times under 5 seconds for most requests

## 🔧 System Resilience

### AI-Driven Edge Case Management

Unlike traditional systems with explicit error handling, this architecture delegates edge cases to AI agents through natural language understanding. **This is actually a more sophisticated approach that provides better user experience through natural language interaction.**

- **API Issues**: AI agents explain problems and suggest alternatives naturally
- **Invalid Inputs**: Agents guide users with contextual, helpful responses
- **System Problems**: Conversational feedback instead of technical error codes

### Common Issues

**"No response" error**: Check Respond to Webhook Node configuration
- Ensure Response Body is set to `{{ $json.text }}`
- Verify Content-Type header is `text/plain`

**AI Agent timeout**: Verify OpenRouter API key and model availability
- Check API quotas and rate limits
- Test with different models if needed

**Weather data errors**: National Weather Service API issues
- Check internet connectivity
- Verify API endpoints are accessible
- Review timeout settings (30 seconds recommended)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Update documentation for any architectural changes
- Test all three workflow paths before submitting
- Follow existing naming conventions for nodes
- Update the Executive Summary for node count changes

## 📊 Performance

- **Cost**: ~$0.01 per interaction (OpenRouter free tier)
- **Response Time**: < 5 seconds average
- **Uptime**: Dependent on n8n instance and external APIs
- **Scalability**: Horizontal scaling via n8n cloud or self-hosted clusters

## 🎯 Future Enhancements

- [ ] Multi-language support
- [ ] Historical weather analysis
- [ ] Severe weather push notifications
- [ ] Weather image generation
- [ ] IoT sensor integration
- [ ] Mobile app interface
- [ ] Voice interaction support

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [n8n](https://n8n.io/) for the excellent workflow automation platform
- [OpenRouter](https://openrouter.ai/) for AI model access
- [National Weather Service](https://www.weather.gov/) for free weather data API
- [Claude](https://claude.ai/) for development assistance

## 📞 Support

- Create an issue for bug reports or feature requests
- Check existing documentation before asking questions
- Provide n8n logs and workflow screenshots for debugging

---

**Built with ❤️ using n8n workflow automation and multi-agent AI architecture**