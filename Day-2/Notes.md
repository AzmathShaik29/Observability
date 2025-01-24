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
 
**As per the steps performed in the Day-1, herein I'm continuing from there.**

- We will take our AWS account, because this Kubernetes cluster is running on AWS. Will go to the EC2 instance where we find 2 EC2 instance (which are the nodes of the Kubernetes cluster).
- When you are installing the kube Prometheus stack - along with alertmanager, Grafana, and Prometheus it will also install the node exporter and kube state metrics.
- It is very important to have the node exporter and kube state metrics on your Kubernetes cluster, without this Prometheus will not get a lot of information to pull the metrics.
- We can see 2 node exporters and 1 kube state metrics nodes are created because node exporter runs as a daemon sets.
- Access to any one of the nodes and connect through browser using 'session manager'.

**Node Exporter**
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```
curl 10.214.223.18 (IP Address of your node exporter):9100 (Port of your node exporter)/metrics

- With the above command you can see all the information that your node exporter is collecting periodically, below screenshot for your quick reference.
![image](https://github.com/user-attachments/assets/eafc510d-d9d1-4230-b452-5a9c175a3c36)

- Will take any one of the query lets say "node_cpu_seconds_total{cpu="0",mode="user"}" and hit in the Prometheus and see the result it has given in below screenshot. You can select the "Graph" to see all the data.
![image](https://github.com/user-attachments/assets/1e90fd39-9256-42ad-a58a-c7a25beecc24)

***Kube State Metrics**
curl 12.256.358.11:8080/metrics

- With the above command we can see all the information that your kube state metrics collects.
![image](https://github.com/user-attachments/assets/28b8527a-ae77-4204-86d4-e31183b706eb)

- The below screenshot shows the information related to the container status is in running state
![image](https://github.com/user-attachments/assets/17587d5f-58ed-45ab-ab30-e819decaca3d)

- If you hit *kube_pod_container_status_restarts_total{namespace="default"}* query in Prometheus dashboard you will not get any data because we don't have anything there. For this will create pod called "alpine" to see the graph.
```bash
kubectl run alpine-crash --image=alpine -- /bin/sh -c "exit 1"
```
![image](https://github.com/user-attachments/assets/bedc108d-4b61-4e51-9333-19914a63fc9d)

- Delete the pod after seeing the graph
```bash
kubectl delete pod alpine-crash
```


