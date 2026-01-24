# AI Gateway Quick Start Guide

Get up and running with Choreo AI Gateway in minutes. This guide walks you through your first AI service integration.

---

## 🚀 Quick Start in 5 Minutes

### Step 1: Choose Your AI Provider

Select from supported providers:
- **OpenAI** - Best for: General purpose, GPT-4, content generation
- **Azure OpenAI** - Best for: Enterprise compliance, Microsoft ecosystem
- **Anthropic Claude** - Best for: Long context, complex reasoning
- **Mistral** - Best for: Cost-effective, open-source models
- **AWS Bedrock** - Best for: Multi-model access, AWS integration

### Step 2: Get Your API Credentials

#### OpenAI
```bash
# Sign up at https://platform.openai.com
# Navigate to: API Keys → Create new secret key
# Copy your key: sk-proj-...
```

#### Azure OpenAI
```bash
# Required credentials:
# - Endpoint URL: https://your-resource.openai.azure.com/
# - API Key: Your subscription key
# - Deployment Name: Your model deployment name
```

#### Anthropic Claude
```bash
# Sign up at https://console.anthropic.com
# Navigate to: Settings → API Keys
# Copy your key: sk-ant-...
```

### Step 3: Register Service in Choreo

**Via Choreo Console:**

1. Navigate to **Settings** → **AI Services**
2. Click **Register Service**
3. Fill in the form:
   ```
   Service Name: my-openai-service
   Provider: OpenAI
   API Key: [Your API Key]
   Model: gpt-4
   ```
4. Click **Save**

**Via Choreo CLI:**

```bash
# Login to Choreo
choreo login

# Register AI service
choreo ai-service register \
  --name my-openai-service \
  --provider openai \
  --api-key $OPENAI_API_KEY \
  --model gpt-4
```

### Step 4: Create a Connection

```yaml
# choreo-connections.yaml
connections:
  - name: ai-assistant
    type: ai-service
    service: my-openai-service
    config:
      max_tokens: 150
      temperature: 0.7
```

### Step 5: Use in Your Application

**Python Example:**

```python
from choreo.connections import get_connection
import json

# Get AI service connection
ai_service = get_connection('ai-assistant')

# Simple text generation
response = ai_service.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing in simple terms"}
    ],
    max_tokens=150
)

print(response.choices[0].message.content)
```

**Node.js Example:**

```javascript
const { getConnection } = require('@choreo/connections');

async function generateText() {
    const aiService = await getConnection('ai-assistant');
    
    const response = await aiService.chat.completions.create({
        model: 'gpt-4',
        messages: [
            { role: 'system', content: 'You are a helpful assistant.' },
            { role: 'user', content: 'Explain quantum computing in simple terms' }
        ],
        max_tokens: 150
    });
    
    console.log(response.choices[0].message.content);
}

generateText();
```

**Java Example:**

```java
import com.choreo.connections.Connection;
import com.choreo.connections.AIServiceConnection;

public class AIExample {
    public static void main(String[] args) {
        AIServiceConnection aiService = 
            Connection.getAIService("ai-assistant");
        
        String response = aiService.chat()
            .model("gpt-4")
            .addMessage("system", "You are a helpful assistant.")
            .addMessage("user", "Explain quantum computing in simple terms")
            .maxTokens(150)
            .execute();
        
        System.out.println(response);
    }
}
```

---

## 🎯 Common Use Cases

### 1. Chatbot Integration

```python
def chatbot_handler(user_message, conversation_history):
    messages = conversation_history + [
        {"role": "user", "content": user_message}
    ]
    
    response = ai_service.chat.completions.create(
        model="gpt-4",
        messages=messages,
        max_tokens=200,
        temperature=0.8
    )
    
    return response.choices[0].message.content
```

### 2. Content Summarization

```python
def summarize_document(document_text):
    response = ai_service.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "Summarize the following text concisely."},
            {"role": "user", "content": document_text}
        ],
        max_tokens=300
    )
    
    return response.choices[0].message.content
```

### 3. Code Generation

```python
def generate_code(description):
    response = ai_service.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are a code generation assistant."},
            {"role": "user", "content": f"Generate Python code for: {description}"}
        ],
        max_tokens=500,
        temperature=0.3  # Lower temperature for more deterministic code
    )
    
    return response.choices[0].message.content
```

### 4. Sentiment Analysis

```python
def analyze_sentiment(text):
    response = ai_service.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "Analyze sentiment: positive, negative, or neutral."},
            {"role": "user", "content": text}
        ],
        max_tokens=50
    )
    
    return response.choices[0].message.content
```

---

## 🔧 Configuration Options

### Rate Limiting

```yaml
connections:
  - name: ai-assistant
    type: ai-service
    service: my-openai-service
    rate_limits:
      - type: token-based
        limit: 100000
        window: 1h
      - type: request-based
        limit: 1000
        window: 1h
```

### Error Handling

```python
from choreo.exceptions import (
    RateLimitExceeded,
    AuthenticationError,
    ServiceUnavailable
)

try:
    response = ai_service.chat.completions.create(...)
except RateLimitExceeded as e:
    print(f"Rate limit exceeded. Retry after: {e.retry_after}")
except AuthenticationError:
    print("Authentication failed. Check your API key.")
except ServiceUnavailable:
    print("AI service temporarily unavailable.")
```

### Streaming Responses

```python
# For long responses, use streaming
def stream_response(prompt):
    stream = ai_service.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    for chunk in stream:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end='')
```

---

## 📊 Monitoring Your Usage

### View Usage Statistics

```bash
# Via Choreo CLI
choreo ai-service stats --service my-openai-service

# Output:
# Service: my-openai-service
# Total Requests: 1,234
# Total Tokens: 456,789
# Average Response Time: 2.3s
# Success Rate: 99.2%
```

### Set Up Alerts

```yaml
# choreo-alerts.yaml
alerts:
  - name: high-token-usage
    service: my-openai-service
    condition: token_usage > 80000
    window: 1h
    action: notify
    channels:
      - email: team@example.com
      - slack: #ai-alerts
```

---

## 🐛 Troubleshooting

### Issue: "Authentication Failed"

**Solution:**
```bash
# Verify your API key is correct
choreo ai-service test --service my-openai-service

# Update API key if needed
choreo ai-service update my-openai-service --api-key $NEW_API_KEY
```

### Issue: "Rate Limit Exceeded"

**Solution:**
```python
# Implement exponential backoff
import time

def call_with_retry(func, max_retries=3):
    for i in range(max_retries):
        try:
            return func()
        except RateLimitExceeded as e:
            if i == max_retries - 1:
                raise
            wait_time = 2 ** i
            time.sleep(wait_time)
```

### Issue: "Slow Response Times"

**Solutions:**
- Reduce `max_tokens` to get faster responses
- Use a smaller model (e.g., gpt-3.5-turbo instead of gpt-4)
- Implement caching for common queries
- Use streaming for long responses

---

## 🎓 Next Steps

1. **Explore Advanced Features**: [Provider Configuration Guide](./Provider_Configuration_Guide.md)
2. **Learn Best Practices**: [Best Practices and Examples](./Best_Practices_and_Examples.md)
3. **Database Integration**: [Databases/README.md](./Databases/README.md)
4. **Full Documentation**: [Integrate and Manage GenAI Services](./Integrate_and_Manage_Generative_AI_Services.md)

---

## 📚 Additional Resources

- [Choreo AI Gateway Documentation](../Choreo_AI_Gateway.md)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Anthropic Claude Documentation](https://docs.anthropic.com)
- [Azure OpenAI Service](https://learn.microsoft.com/en-us/azure/ai-services/openai/)

---

**Need Help?**
- Join our [Community Slack](https://choreo.dev/slack)
- Check [Stack Overflow](https://stackoverflow.com/questions/tagged/choreo)
- Contact support: support@choreo.dev

---

**Last Updated**: January 2026
