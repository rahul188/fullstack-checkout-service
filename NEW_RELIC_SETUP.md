# 🚀 New Relic Monitoring Setup

This document provides instructions for setting up and configuring New Relic APM monitoring for the Fullstack Checkout Service.

## 📋 Overview

New Relic has been integrated into this FastAPI application to provide:
- **Application Performance Monitoring (APM)**: Track response times, throughput, and error rates
- **Distributed Tracing**: Follow requests across service boundaries
- **Error Tracking**: Automatic exception capture and reporting
- **Custom Attributes**: Business context for transactions
- **Logs in Context**: Correlate logs with transactions

## 🔧 Setup Instructions

### 1. Prerequisites

- Python 3.8 or higher (as required by this application)
- New Relic account (sign up at [newrelic.com](https://newrelic.com))
- New Relic license key

### 2. Installation

The New Relic Python agent is already included in `requirements.txt`. When you install dependencies, it will be installed automatically:

```bash
pip install -r requirements.txt
```

### 3. Configuration

#### Get Your New Relic License Key

1. Log in to your New Relic account
2. Navigate to: **Account settings** > **API keys**
3. Copy your **License key** (or create a new one)
4. Or visit: https://one.newrelic.com/launcher/api-keys-ui.api-keys-launcher

#### Set Environment Variables

Create a `.env` file in the project root (copy from `.env.example`):

```bash
cp .env.example .env
```

Edit the `.env` file and add your credentials:

```bash
NEW_RELIC_LICENSE_KEY=your_actual_license_key_here
NEW_RELIC_APP_NAME=Fullstack Checkout Service
```

**For different environments**, you can customize the app name:

```bash
# Development
NEW_RELIC_APP_NAME=Fullstack Checkout Service (Dev)

# Staging
NEW_RELIC_APP_NAME=Fullstack Checkout Service (Staging)

# Production
NEW_RELIC_APP_NAME=Fullstack Checkout Service (Production)
```

#### Export Environment Variables (Alternative Method)

If you don't want to use a `.env` file, export the variables directly:

```bash
export NEW_RELIC_LICENSE_KEY="your_license_key_here"
export NEW_RELIC_APP_NAME="Fullstack Checkout Service"
export NEW_RELIC_ENVIRONMENT="production"  # optional
```

### 4. Running the Application

Start the application as usual:

```bash
python main.py
```

Or with uvicorn:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The New Relic agent will automatically initialize and start reporting data.

### 5. Verify the Integration

#### Check Application Logs

Look for New Relic initialization messages in your logs:

```
INFO: New Relic agent initialized successfully
```

#### Check New Relic Dashboard

1. Log in to [New Relic One](https://one.newrelic.com)
2. Navigate to **APM & Services**
3. Look for your application name (e.g., "Fullstack Checkout Service")
4. You should start seeing data within 1-2 minutes

#### Test with API Requests

Generate some traffic to verify monitoring:

```bash
# Test the products endpoint
curl http://localhost:8000/api/products

# Test the checkout endpoint
curl -X POST http://localhost:8000/api/checkout \
  -H "Content-Type: application/json" \
  -d '{
    "items": [{"product_id": 1, "quantity": 2}],
    "customer_name": "Test User",
    "customer_email": "test@example.com",
    "shipping_address": "123 Test St",
    "payment_method": "credit_card"
  }'
```

## 📊 What's Being Monitored

### Automatic Instrumentation

The New Relic Python agent automatically instruments:

- **FastAPI Routes**: All API endpoints (`/api/products`, `/api/checkout`, etc.)
- **HTTP Requests**: Request/response times, status codes, throughput
- **Database Queries**: (When a database is added)
- **External Services**: HTTP calls to external APIs
- **Background Tasks**: Async operations
- **Exceptions**: Unhandled errors and exceptions

### Key Metrics

Monitor these important metrics in New Relic:

1. **Response Time**: Average, median, 95th percentile
2. **Throughput**: Requests per minute
3. **Error Rate**: Percentage of failed requests
4. **Apdex Score**: Application performance index
5. **Transaction Traces**: Detailed breakdown of slow requests

## 🎯 Configuration Options

### newrelic.ini Configuration

The `newrelic.ini` file contains all configuration options. Key settings:

```ini
# Application name (can be overridden by environment variable)
app_name = ${NEW_RELIC_APP_NAME:Fullstack Checkout Service}

# License key (should be set via environment variable)
license_key = ${NEW_RELIC_LICENSE_KEY}

# Enable distributed tracing
distributed_tracing.enabled = true

# Log level (debug, info, warning, error, critical)
log_level = info

# Transaction tracer settings
transaction_tracer.enabled = true
transaction_tracer.transaction_threshold = apdex_f
```

### Environment-Specific Settings

The configuration file includes sections for different environments:

- `[newrelic:development]` - Development settings with debug logging
- `[newrelic:staging]` - Staging environment configuration
- `[newrelic:production]` - Production settings

Set the environment using:

```bash
export NEW_RELIC_ENVIRONMENT=production
```

## 🚨 Alerts and Notifications

### Recommended Alerts

Set up alerts for:

1. **High Error Rate**: Alert when error rate > 5%
2. **Slow Response Time**: Alert when average response time > 1s
3. **Low Throughput**: Alert when requests/min drops significantly
4. **Application Down**: Alert when app stops reporting data

### Creating Alerts in New Relic

1. Navigate to **Alerts & AI** > **Alert Policies**
2. Click **Create a policy**
3. Add conditions for the metrics above
4. Configure notification channels (email, Slack, PagerDuty, etc.)

## 🔍 Custom Instrumentation

### Adding Custom Attributes

You can add custom business context to transactions:

```python
import newrelic.agent

@app.post("/api/checkout")
async def checkout(checkout_request: CheckoutRequest):
    # Add custom attributes
    newrelic.agent.add_custom_attribute("customer_email", checkout_request.customer_email)
    newrelic.agent.add_custom_attribute("order_total", total_amount)
    newrelic.agent.add_custom_attribute("item_count", len(checkout_request.items))
    
    # Your existing code...
```

### Custom Transactions

For background tasks or custom operations:

```python
@newrelic.agent.background_task()
def process_order_async(order_id):
    # Process order
    pass
```

### Recording Custom Events

```python
newrelic.agent.record_custom_event("OrderPlaced", {
    "order_id": order_id,
    "total_amount": total_amount,
    "payment_method": payment_method
})
```

## 📈 Dashboard Queries

### Useful NRQL Queries

#### Average Response Time by Endpoint

```sql
SELECT average(duration) FROM Transaction 
WHERE appName = 'Fullstack Checkout Service' 
FACET name SINCE 1 hour ago
```

#### Error Rate

```sql
SELECT percentage(count(*), WHERE error IS true) 
FROM Transaction 
WHERE appName = 'Fullstack Checkout Service' 
SINCE 1 day ago
```

#### Throughput by Endpoint

```sql
SELECT rate(count(*), 1 minute) 
FROM Transaction 
WHERE appName = 'Fullstack Checkout Service' 
FACET name SINCE 1 hour ago
```

## 🐛 Troubleshooting

### Agent Not Reporting Data

1. **Check Environment Variables**: Ensure `NEW_RELIC_LICENSE_KEY` is set
2. **Check Logs**: Look at `/tmp/newrelic-python-agent.log`
3. **Verify Network**: Ensure your server can reach New Relic's collector (collector.newrelic.com)
4. **Check Agent Status**: The agent logs initialization status

### Common Issues

#### "License key is not set"
- Set the `NEW_RELIC_LICENSE_KEY` environment variable
- Verify the key is correct (no extra spaces or quotes)

#### "No data appearing in New Relic"
- Wait 1-2 minutes for initial data
- Generate traffic to your application
- Check that `monitor_mode = true` in newrelic.ini

#### "Import errors"
- Ensure `newrelic` package is installed: `pip install newrelic`
- Check Python version compatibility (3.7+)

## 🔐 Security Considerations

### License Key Protection

- **Never commit** your license key to version control
- Use environment variables or secure secret management
- Rotate keys regularly
- Use different keys for different environments

### Data Privacy

Configure what data is sent to New Relic:

```ini
# Disable request parameters capture
attributes.include = request.headers.*
attributes.exclude = request.parameters.*

# Enable high security mode (restricts certain features)
high_security = true
```

## 📚 Additional Resources

- [New Relic Python Agent Documentation](https://docs.newrelic.com/docs/apm/agents/python-agent/)
- [FastAPI Integration Guide](https://docs.newrelic.com/docs/apm/agents/python-agent/hosting-services/python-agent-fastapi/)
- [New Relic University](https://learn.newrelic.com/)
- [New Relic API Documentation](https://docs.newrelic.com/docs/apis/intro-apis/introduction-new-relic-apis/)
- [Best Practices Guide](https://docs.newrelic.com/docs/new-relic-solutions/best-practices-guides/)

## 🆘 Support

- **New Relic Support**: https://support.newrelic.com
- **Community Forum**: https://discuss.newrelic.com
- **GitHub Issues**: Report issues specific to this integration

## ✅ Checklist for Production

- [ ] New Relic license key configured
- [ ] Application name set appropriately
- [ ] Environment variables secured (not in code)
- [ ] Alerts configured for critical metrics
- [ ] Dashboard created for key business metrics
- [ ] Team members have access to New Relic account
- [ ] Deployment markers configured
- [ ] Logs integrated (if applicable)
- [ ] Synthetic monitoring configured (for web frontend)
- [ ] Tested in staging environment

---

**Last Updated**: 2025-10-17
**New Relic Agent Version**: 11.0.1
