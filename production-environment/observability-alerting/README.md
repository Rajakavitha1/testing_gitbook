# Monitoring and Alerting

The monitoring and alerting features of the Seldon Enterprise Platform provide robust tools for tracking the performance and health of machine learning models in production.

## Monitoring
* Real-Time metrics: collects and displays real-time metrics from deployed models, such as response times, error rates, and resource usage.
* Model performance tracking: monitors key performance indicators (KPIs) like accuracy, drift detection, and model degradation over time.
* Custom metrics: allows you to define and track custom metrics specific to their models and use cases.
* Visualization: Provides dashboards and visualizations to easily observe the status and performance of models.

## Alerting:
* Proactive notifications: sends alerts when specific thresholds or conditions are met, such as a sudden drop in model accuracy or an increase in response latency.
* Integration with alertmanager: leverages alertmanager to manage and route alerts to appropriate channels, such as email, Slack, or other communication tools.
* Service Level Objectives (SLOs): alerts are triggered based on SLO breaches, ensuring that any critical issues in model performance or infrastructure are promptly addressed.
* Automated response: supports automated responses to alerts, such as scaling resources or triggering workflows to retrain a model.

Together, these features ensure that models running in production are performing as expected and that any issues are quickly identified and addressed to maintain the reliability and effectiveness of the machine learning deployments.

For a hands-on experience, you can explore the alerting functionality through the [alerting demo](./) after installating [monitoring](observability.md) and [alerting](alerting.md) components of Seldon Enterprise Platform.
