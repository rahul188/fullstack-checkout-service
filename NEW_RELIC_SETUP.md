# New Relic Monitoring Setup

This document explains how New Relic monitoring has been integrated into the fullstack-checkout-service application.

## Overview

The application is now instrumented with New Relic APM (Application Performance Monitoring) for Python, providing:

- **Application Performance Monitoring**: Track response times, throughput, and error rates
- **Distributed Tracing**: Trace requests across services
- **Error Tracking**: Automatically capture and report exceptions
- **Custom Business Metrics**: Track order values, payment methods, and customer activity
- **Log Integration**: Correlate logs with transactions

## Setup Instructions

### 1. Install Dependencies

The New Relic Python agent is included in `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 2. Configure Environment Variables

Set the following environment variables:

```bash
export NEW_RELIC_LICENSE_KEY="your_license_key_here"
export NEW_RELIC_APP_NAME="fullstack-checkout-service"
```

To get your New Relic license key:
1. Log in to [New Relic One](https://one.newrelic.com/)
2. Go to **Account settings** → **API keys**
3. Create or copy your **License key** (INGEST - LICENSE type)

You can also copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
# Edit .env with your license key
```

### 3. Run the Application

The application will automatically initialize New Relic if the `NEW_RELIC_LICENSE_KEY` environment variable is set:

```bash
# With environment variables
NEW_RELIC_LICENSE_KEY="your_key" NEW_RELIC_APP_NAME="fullstack-checkout-service" python main.py
```

Or with uvicorn:

```bash
NEW_RELIC_LICENSE_KEY="your_key" NEW_RELIC_APP_NAME="fullstack-checkout-service" uvicorn main:app --host 0.0.0.0 --port 8000
```

### 4. Alternative: Use newrelic-admin

You can also run the application using the `newrelic-admin` wrapper:

```bash
NEW_RELIC_LICENSE_KEY="your_key" NEW_RELIC_APP_NAME="fullstack-checkout-service" newrelic-admin run-program python main.py
```

## What's Being Monitored

### Automatic Instrumentation

The New Relic agent automatically instruments:

- **FastAPI endpoints**: All API routes are automatically traced
- **HTTP requests**: External API calls (if any)
- **Database queries**: When database integration is added
- **Exceptions**: Unhandled errors are automatically reported

### Custom Instrumentation

The checkout endpoint includes custom attributes to track business metrics:

- `order_id`: Unique order identifier
- `order_total`: Total order value
- `item_count`: Number of items in the order
- `payment_method`: Payment method used
- `customer_email`: Customer email (for error correlation)

These attributes can be used to:
- Create custom dashboards in New Relic
- Set up alerts based on business metrics
- Analyze customer behavior and order patterns

## Configuration

The application uses `newrelic.ini` for configuration. Key settings include:

- **Distributed Tracing**: Enabled by default
- **Error Collection**: Enabled by default
- **Transaction Tracing**: Enabled by default
- **Log Output**: Set to stdout for container environments

To customize the configuration, edit `newrelic.ini`. See the [New Relic Python agent configuration documentation](https://docs.newrelic.com/docs/apm/agents/python-agent/configuration/python-agent-configuration/) for all available options.

## Running Without New Relic

The application will run normally even if New Relic is not configured. Simply don't set the `NEW_RELIC_LICENSE_KEY` environment variable:

```bash
python main.py
```

The New Relic initialization code checks for the license key and only initializes the agent if it's present.

## Viewing Data in New Relic

After running the application with New Relic configured:

1. Log in to [New Relic One](https://one.newrelic.com/)
2. Navigate to **APM & Services**
3. Find your application (e.g., "fullstack-checkout-service")
4. Explore:
   - **Summary**: Overview of app performance
   - **Transactions**: Detailed breakdown of endpoint performance
   - **Errors**: Error rates and details
   - **Distributed Tracing**: Request traces across services
   - **Logs**: Application logs (if log forwarding is enabled)

## Setting Up Alerts

You can create alerts for key metrics:

1. In New Relic, go to your application
2. Click **Alerts & AI** → **Alert conditions**
3. Create conditions for:
   - Error rate > threshold
   - Response time > threshold
   - Throughput drops
   - Custom metrics (e.g., order total spikes)

## Best Practices

1. **Use Different App Names for Different Environments**:
   ```bash
   NEW_RELIC_APP_NAME="checkout-service-production"  # Production
   NEW_RELIC_APP_NAME="checkout-service-staging"     # Staging
   NEW_RELIC_APP_NAME="checkout-service-dev"         # Development
   ```

2. **Disable Monitoring in Development**: The configuration automatically disables monitoring in development environment (see `newrelic.ini`)

3. **Add Deployment Markers**: Track deployments in New Relic:
   ```bash
   newrelic-admin record-deploy newrelic.ini "Deployment description"
   ```

4. **Monitor Business Metrics**: Use custom attributes to track business KPIs alongside technical metrics

## Troubleshooting

### Agent Not Connecting

1. Verify the license key is correct
2. Check that the environment variable is set: `echo $NEW_RELIC_LICENSE_KEY`
3. Check the logs for initialization messages
4. Ensure network connectivity to New Relic servers

### Data Not Appearing

1. Wait 2-3 minutes after starting the app (data is sent periodically)
2. Generate traffic to the application
3. Check New Relic logs in stdout
4. Set `log_level = debug` in `newrelic.ini` for more details

### Performance Impact

The New Relic agent has minimal performance overhead:
- CPU: < 1% in most cases
- Memory: 20-40 MB
- Network: Periodic data transmission (compressed)

## Resources

- [New Relic Python Agent Documentation](https://docs.newrelic.com/docs/apm/agents/python-agent/)
- [FastAPI Integration Guide](https://docs.newrelic.com/docs/apm/agents/python-agent/hosting-services/python-agent-fastapi/)
- [Custom Instrumentation](https://docs.newrelic.com/docs/apm/agents/python-agent/custom-instrumentation/)
- [Alert Conditions](https://docs.newrelic.com/docs/alerts-applied-intelligence/new-relic-alerts/alert-conditions/)

## Support

For issues or questions:
- New Relic Support: https://support.newrelic.com
- New Relic Community: https://discuss.newrelic.com
- Documentation: https://docs.newrelic.com
