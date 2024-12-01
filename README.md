# demo-prometheus-k8s

The goal of this project is to install **Prometheus** on a Kubernetes (K8s) cluster using a **Helm chart** and configure it to collect essential cluster-specific metrics. Prometheus will be deployed to monitor the Kubernetes cluster, providing insights into CPU, memory, disk space, and network usage, as well as application-specific performance.

---

## Tools

- [Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Kubernetes](https://kubernetes.io/)
- [helm](https://helm.sh/)
- [Jenkins](https://www.jenkins.io/)


---
## Install Prometheus
0. **Create Namespace**

   Run to Create the Namespace for monitoring
   ```bash
   kubectl get namespace demo-metrics || kubectl create namespace demo-metrics
   ```

1. **Add the Bitnami Repository:**

   Run to add the Bitnami charts repository to the Helm client:

   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```

2. **Update Helm Repositories:**

   After adding the repository, Run to update the local Helm chart repository cache to ensure that the latest version of the charts is used:

   ```bash
   helm repo update
   ```

3. **Install Prometheus:**

   Run to install Prometheus using the Bitnami Helm chart:

   ```bash
   helm upgrade --install  demo-prometheus bitnami/kube-prometheus -n demo-metrics
   ```

4. **Verify the Installation:**

   Use the following command to list all Helm releases to check if Prometheus is installed and running:

   ```bash
   helm list -n demo-metrics
   ```

   Also, use the following command to check the status of the pods to ensure they are running correctly:

   ```bash
   kubectl get pods -l app.kubernetes.io/instance=demo-prometheus  -n demo-metrics
   ```

---

## Install Exporters

1. **(Optional) Install Kube State Metrics:**

   > Kube State Metrics is also included in the kube-prometheus stack.
   Use the following command to install it separately.


   ```bash
   helm install kube-state-metrics bitnami/kube-state-metrics -n demo-metrics
   ```

---

## (Optional) Configure Prometheus

> The Bitnami `kube-prometheus` chart is pre-configured to scrape metrics from Kubernetes components.

Please use [Bitnami Prometheus Helm Chart README](https://github.com/bitnami/charts/blob/main/bitnami/prometheus/README.md) to customize the configuration by editing the `values.yaml` file.


1. **Edit Configuration**: If you need to customize, download the `values.yaml`:
   ```bash
   helm show values bitnami/kube-prometheus > values.yaml
   ```

2. **Modify `values.yaml`**: Edit the file to adjust scrape intervals, add additional scrape targets, etc.

3. **Apply Configuration**: Reapply the configuration with:
   ```bash
   helm upgrade --install demo-prometheus bitnami/kube-prometheus -n demo-metrics -f values.yaml
   ```

---
## Jenkins Pipeline Configuration

### Prerequisites
1. Kubernetes Cluster: The Jenkins instance should have access to the Kubernetes cluster.
> Please use the ['k8s-jenkins-helm'](https://github.com/avorakh/k8s-jenkins-helm/tree/task-7) repository to create cluster. 
2. Jenkins Configuration:
- The jenkins service account should have sufficient permissions in the cluster. PLease use the updated service account. [Link](https://github.com/avorakh/k8s-jenkins-helm/blob/task-7/jenkins-ci/templates/serviceaccount.yaml)
- [Slack Notification plugin](https://plugins.jenkins.io/slack/) should be installed.  Slack credentials (SLACK_CICD_CHANNEL and SLACK_TOKEN) must be set up in Jenkins.

### Add Pipeline to Jenkins

1. Navigate to Jenkins Dashboard.
2. Create a new multibranch pipeline project.
3. Link the repository containing the `Jenkinsfile`.

---

## Metrics Collection using Prometheus

The Prometheus queries provided below allow detailed monitoring of **Node** and **Application** performance.  
- **Node Metrics** include essential infrastructure-level insights like CPU, RAM, disk space, and network usage.
- **Application Metrics** provide visibility into the performance and health of specific namespaces and pods.

These queries can be integrated into Prometheus dashboards or used for alerting purposes to ensure system reliability and performance.

---

## Node Metrics

### 1. **CPU Usage**  
The percentage of CPU usage is calculated as:  
```prometheus
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```
- **Description**: This formula measures the CPU's active time by subtracting the idle percentage from 100.
- **Key Metric**: `node_cpu_seconds_total` tracks CPU time spent in various states (e.g., idle, user, system).
- **Filter**: `{mode="idle"}` isolates the idle state.

---

### 2. **RAM Usage**  
The percentage of RAM usage is calculated as:  
```prometheus
((node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes) * 100
```
- **Description**: This calculates the proportion of memory used versus total memory available.
- **Key Metrics**: 
  - `node_memory_MemTotal_bytes`: Total physical memory.
  - `node_memory_MemAvailable_bytes`: Available memory for use.

---

### 3. **Disk Space Usage**  
The percentage of disk space usage is calculated as:  
```prometheus
(node_filesystem_size_bytes{fstype!~"tmpfs|overlay"} - node_filesystem_free_bytes{fstype!~"tmpfs|overlay"}) 
/ node_filesystem_size_bytes{fstype!~"tmpfs|overlay"} * 100
```
- **Description**: This measures the proportion of used disk space, excluding temporary and overlay filesystems.
- **Key Metrics**: 
  - `node_filesystem_size_bytes`: Total size of the filesystem.
  - `node_filesystem_free_bytes`: Free space available.
- **Filter**: `{fstype!~"tmpfs|overlay"}` excludes temporary and overlay filesystems.

---

### 4. **Network Metrics**  
Key metrics to monitor network performance:  
- **Bytes Transmitted**:  
  ```prometheus
  container_network_transmit_bytes_total
  ```
- **Bytes Received**:  
  ```prometheus
  container_network_receive_bytes_total
  ```
- **Description**: These metrics track the total network data transmitted and received over time.

---

## Application Metrics

### 1. **CPU Usage**  
To monitor CPU usage for a specific application in the namespace `demo-app`:  
- **Total CPU Usage (Rate)**:  
  ```prometheus
  sum(rate(container_cpu_usage_seconds_total{namespace="demo-app", pod=~"demo-web-app.*"}[5m])) by (pod)
  ```
- **CPU Idle Percentage**:  
  ```prometheus
  (100 - (avg(irate(container_cpu_usage_seconds_total{namespace="demo-app", pod=~"demo-web-app.*"}[5m])) * 100))
  ```
- **Description**: These queries track CPU usage for all pods matching the regex `demo-web-app.*` in the `demo-app` namespace.

---

### 2. **Container Restarts**  
To monitor container restarts:  
```prometheus
sum(rate(kube_pod_container_status_restarts_total{namespace="demo-app",pod=~"demo-web-app.*"}[5m]))
```
- **Description**: This tracks the rate of container restarts over the last 5 minutes.

---

### 3. **RAM Usage**  
To monitor RAM usage for the application:  
```prometheus
sum(container_memory_usage_bytes{namespace="demo-app",pod=~"demo-web-app.*"})
```
- **Description**: This calculates the total memory usage across all matching pods in the `demo-app` namespace.

---

## References

- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Helm Chart for Prometheus](https://github.com/bitnami/charts/tree/main/bitnami/prometheus)
- [Bitnami Prometheus Helm Chart README](https://github.com/bitnami/charts/blob/main/bitnami/prometheus/README.md)
- [Kube state metrics](https://github.com/kubernetes/kube-state-metrics)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)