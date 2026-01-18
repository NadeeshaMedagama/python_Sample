# AI Services Documentation

This directory contains comprehensive documentation for integrating and managing AI services within the Choreo platform.

---

## Overview

AI Services in Choreo enable developers to seamlessly integrate cutting-edge generative AI capabilities into their applications. Through the Choreo AI Gateway, you can connect to multiple AI providers with enterprise-grade security, cost controls, and observability.

---

## Contents

### Documentation Files

- **[Integrate_and_Manage_Generative_AI_Services.md](./Integrate_and_Manage_Generative_AI_Services.md)**  
  Complete guide to registering, configuring, and managing GenAI services in Choreo

- **[Databases/](./Databases/)**  
  Database integration guides for AI applications

---

## Supported AI Providers

Choreo AI Gateway supports the following providers:

| Provider | Description | Use Cases |
|----------|-------------|-----------|
| **OpenAI** | GPT models for text generation and completion | Chatbots, content generation, code assistance |
| **Azure OpenAI** | Enterprise OpenAI deployment on Azure | Enterprise AI applications with Azure integration |
| **Anthropic Claude** | Advanced conversational AI | Complex reasoning, analysis, coding tasks |
| **Mistral** | Open-source LLM models | Cost-effective AI solutions |
| **AWS Bedrock** | Managed AI service on AWS | Multi-model AI applications on AWS |

---

## Quick Start Guide

### 1. Prerequisites

Before integrating an AI service, ensure you have:

- ✅ API key from your chosen AI provider
- ✅ Service endpoint URL
- ✅ Appropriate credentials (client secrets, subscription keys)
- ✅ Access to Choreo platform (organization or project level)

### 2. Register an AI Service

**Organization-Level** (shared across all projects):
```
Choreo Console → Organization Settings → AI Services → Register Service
```

**Project-Level** (restricted to specific project):
```
Choreo Console → Project → Settings → AI Services → Register Service
```

### 3. Configure Authentication

Securely store your API credentials:
- Use Choreo's secret management for API keys
- Configure authentication method (API key, OAuth, etc.)
- Set up environment-specific configurations

### 4. Connect from Your Application

Use Choreo Connections to consume the registered AI service:
```python
# Example: Python application using AI service
from choreo.connections import AIServiceConnection

ai_service = AIServiceConnection('your-service-name')
response = ai_service.generate(
    prompt="Explain quantum computing",
    max_tokens=150
)
```

---

## Key Features

### 🔒 Security
- **API Key Management**: Secure storage and rotation of credentials
- **Encrypted Transport**: TLS/SSL for all communications
- **Authentication & Authorization**: Role-based access control

### 💰 Cost Control
- **Token-Based Rate Limiting**: Prevent unexpected costs
- **Usage Quotas**: Define limits based on LLM token consumption
- **Fair Usage Policies**: Enforce controls across teams

### 📊 Observability (Coming Soon)
- **Metrics & Analytics**: Track API usage and performance
- **Logging**: Detailed request/response logging
- **Monitoring**: Real-time alerts and dashboards

---

## Integration Patterns

### Pattern 1: Direct AI Integration
```
User Request → Your Application → AI Gateway → AI Provider → Response
```

### Pattern 2: AI + Database
```
User Request → Application → AI Gateway → AI Provider
                    ↓
                Database (store context/results)
```

### Pattern 3: Multi-Model Architecture
```
User Request → Application → AI Gateway → [OpenAI, Claude, Mistral]
                    ↓
              Model Router (select best model)
```

---

## Best Practices

### 1. **Choose the Right Scope**
- Use **organization-level** registration for services shared across multiple projects
- Use **project-level** registration for project-specific AI capabilities

### 2. **Implement Rate Limiting**
```yaml
# Example rate limit configuration
rate_limits:
  - name: "token-limit"
    type: "token-based"
    limit: 100000
    window: "1h"
```

### 3. **Handle Errors Gracefully**
```python
try:
    response = ai_service.generate(prompt)
except RateLimitExceeded:
    # Handle rate limit
    return "Service temporarily unavailable"
except AuthenticationError:
    # Handle auth issues
    log_error("AI service authentication failed")
```

### 4. **Monitor Token Usage**
- Track token consumption per request
- Set up alerts for approaching quota limits
- Optimize prompts to reduce token usage

### 5. **Secure Your Credentials**
- Never hardcode API keys in source code
- Use Choreo's secret management
- Rotate credentials regularly
- Use environment-specific keys

---

## Use Cases

### Content Generation
Generate articles, documentation, marketing copy using GPT models

### Code Assistance
Provide code completion, debugging, and explanation features

### Chatbots & Virtual Assistants
Build conversational AI applications with natural language understanding

### Data Analysis
Extract insights, summarize documents, analyze sentiment

### Image Generation (with compatible providers)
Create images from text descriptions

---

## Database Integration

AI applications often need persistent storage for:
- **Conversation History**: Store chat sessions and context
- **User Preferences**: Personalization data
- **Training Data**: Collect feedback for fine-tuning
- **Vector Embeddings**: Semantic search capabilities

See [Databases/README.md](./Databases/README.md) for database integration guides.

---

## Troubleshooting

### Common Issues

**Authentication Failures**
- Verify API key is correct and not expired
- Check if credentials are properly configured in Choreo
- Ensure service endpoint URL is correct

**Rate Limit Exceeded**
- Review token usage patterns
- Adjust rate limits if needed
- Implement request queuing or caching

**Slow Response Times**
- Check network connectivity
- Monitor AI provider status
- Consider caching frequent requests
- Optimize prompt length

---

## Related Resources

- [Choreo AI Gateway Documentation](../Choreo_AI_Gateway.md)
- [Main Project README](../../README.md)
- [Deployment Guides](../../Deployment_Tracks.md)
- [Choreo Marketplace](../../Choreo-Marketplace.md)

---

## Examples

Check the project repository for example implementations:
- Simple chatbot integration
- Document summarization service
- Code generation assistant
- Multi-model comparison tool

---

**Last Updated**: January 2026  
**Maintained By**: Platform Engineering Team

