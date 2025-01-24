## 📊 Metrics in Prometheus:
- Metrics in Prometheus are the core data objects that represent measurements collected from monitored systems.
- These metrics provide insights into various aspects of **system performance, health, and behavior**.

## 🏷️ Labels:
- Metrics are paired with Labels.
- Labels are key-value pairs that allow you to differentiate between dimensions of a metric, such as different services, instances, or endpoints.

## 🛠️ What is PromQL?
- PromQL (Prometheus Query Language) is a powerful and flexible query language used to query data from Prometheus.
- It allows you to retrieve and manipulate time series data, perform mathematical operations, aggregate data, and much more.

- 🔑 Key Features of PromQL:
    - Data Aggregation: Facilitates aggregating data across multiple time series.
    - Comprehensive Functions: Offers a diverse set of functions for data analysis and manipulation.
    - Time Series Selection: Enables filtering and retrieving data from specific metrics.
    - Mathematical Operations: Supports performing calculations on metric data.

## 💡 Basic Examples of PromQL
- `container_cpu_usage_seconds_total`
    - Return all time series with the metric container_cpu_usage_seconds_total
- `container_cpu_usage_seconds_total{namespace="kube-system",pod=~"kube-proxy.*"}`
    - Return all time series with the metric `container_cpu_usage_seconds_total` and the given `namespace` and `pod` labels.
- `container_cpu_usage_seconds_total{namespace="kube-system",pod=~"kube-proxy.*"}[5m]`
    - Return a whole range of time (in this case 5 minutes up to the query time) for the same vector, making it a range vector.

