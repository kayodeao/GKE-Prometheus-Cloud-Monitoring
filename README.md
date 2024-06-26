# GKE Prometheus Cloud Monitoring

This project aims to provide a comprehensive guide and resources for monitoring applications deployed on Google Kubernetes Engine (GKE) using Prometheus and visualizing the metrics using Google's Cloud Monitoring.

## Introduction

As organizations increasingly adopt microservices architectures and containerized applications, effective monitoring and observability become crucial for ensuring reliability, performance, and scalability. Google Kubernetes Engine (GKE) offers a robust platform for deploying and managing containerized workloads, while Prometheus google cloud monitoring provide powerful tools for monitoring and visualizing metrics.

This project serves as a central hub for understanding and implementing monitoring solutions using Prometheus and Grafana on GKE. Whether you're new to Kubernetes monitoring or looking to optimize your existing monitoring setup, you'll find resources, tutorials, and best practices to help you monitor your applications effectively.

## Key Features

- **Managed Service for Prometheus (MSP)**: Learn how to leverage [Google's Managed Service for Prometheus](https://cloud.google.com/stackdriver/docs/managed-prometheus)
 to collect, store, and query metrics from your GKE clusters effortlessly.
- **Sample Application Deployment**: Follow step-by-step instructions to deploy a sample application on GKE and instrument it for monitoring with Prometheus.
- **Visualization with Cloud Monitoring**: When working with metric data, including data from Managed Service for Prometheus, in Cloud Monitoring, you can use the query tools provided by Cloud Monitoring such as PromQL.


## Step 1: Create a GKE Cluster
A GKE cluster consists of a cluster control plane machine and multiple worker machines called nodes. You deploy applications to clusters, and the applications run on the nodes. 
Before creating a GKE cluster, ensure that you have set your default project in the Google Cloud CLI. Run the following command, replacing PROJECT_ID with your project ID:

```bash
gcloud config set project PROJECT_ID
```

To create an Autopilot cluster named hello-cluster or any name of your choice, execute the following command:
```bash
gcloud container clusters create-auto hello-cluster --location=us-central1
```
After creating your cluster, you need to get authentication credentials to interact with it. Use the following command to configure kubectl to use the cluster you created:
```bash
gcloud container clusters get-credentials hello-cluster --location=us-central1
```
![](kubernetes_cluster.png)

## Step 2: Configure your environment
Configure the kubectl CLI to use your cluster:
```bash
kubectl config set-cluster <CLUSTER_NAME>
```
Create the NAMESPACE_NAME Kubernetes namespace for resources you create as part of the example application:
```bash
kubectl create ns <NAMESPACE_NAME>
```
## Step 3: Deploy the example application

The example application emits the ```example_requests_total``` counter metric and the ```example_random_numbers``` histogram metric (among others) on its metrics port. The manifest for the application defines three replicas.

To deploy the example application, run the following command:
```bash
kubectl -n <NAMESPACE_NAME> apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.8.2/examples/example-app.yaml
```
## Step 4: Configure a PodMonitoring resource
To ingest the metric data emitted by the example application, Managed Service for Prometheus uses target scraping. Target scraping and metrics ingestion are configured using Kubernetes custom resources. The managed service uses PodMonitoring custom resources (CRs).PodMonitoring is a Kubernetes custom resource that allows you to define which pods you want to monitor and how you want to collect metrics from them. By creating PodMonitoring resources, you can tell MSP which pods to scrape metrics from and how often to scrape them.

A PodMonitoring CR scrapes targets only in the namespace the CR is deployed in. To scrape targets in multiple namespaces, deploy the same PodMonitoring CR in each namespace. You can verify the PodMonitoring resource is installed in the intended namespace by running ```kubectl get podmonitoring -A``` 

The following manifest defines a PodMonitoring resource, ```prom-example```, in the ```NAMESPACE_NAME``` namespace. The resource uses a Kubernetes label selector to find all pods in the namespace that have the label ```app.kubernetes.io/name``` with the value ```prom-example```. The matching pods are scraped on a port named ```metrics```, every 30 seconds, on the /metrics HTTP path.
```
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: prom-example
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: prom-example
  endpoints:
  - port: metrics
    interval: 30s
```
To apply this resource, run the following command:

```bash
kubectl -n <NAMESPACE_NAME> apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.8.2/examples/pod-monitoring.yaml
```

Your [managed collector](https://cloud.google.com/stackdriver/docs/managed-prometheus#gmp-data-collection) is now scraping the matching pods. You can view the status of your scrape target by enabling the target status feature.

## Step 4: Enabling the target status feature 
Enabling the target status feature allows you to monitor the status of the targets specified in your PodMonitoring or ClusterPodMonitoring resources. These targets could be individual pods or entire clusters that are being monitored for metrics.
By setting the ```features.targetStatus.enabled``` value within the OperatorConfig resource to ```true```, you activate this feature. Once enabled, you'll be able to view the status of your targets, such as whether they are up or down, their health status, and other relevant information.
```
    apiVersion: monitoring.googleapis.com/v1
    kind: OperatorConfig
    metadata:
      namespace: gmp-public
      name: config
    features:
      targetStatus:
        enabled: true
```
run this command and copy/paste the manifest above

```
nano target_status.yaml
```
Save the file within nano by pressing Ctrl+O and then confirm by pressing Enter. Exit nano using Ctrl+X.
Apply
```
kubectl apply -f target_status.yaml
```

After a few seconds, the Status.Endpoint Statuses field appears on every valid PodMonitoring resource, when configured.

If you have a PodMonitoring resource with the name prom-example in your ```NAMESPACE_NAME``` namespace, then you can check the status by running the following command:

```
kubectl -n <NAMESPACE_NAME> describe podmonitorings/prom-example
```
The output looks like the following:
```
API Version:  monitoring.googleapis.com/v1
Kind:         PodMonitoring
...
Status:
  Conditions:
    ...
    Status:                True
    Type:                  ConfigurationCreateSuccess
  Endpoint Statuses:
    Active Targets:       3
    Collectors Fraction:  1
    Last Update Time:     2023-08-02T12:24:26Z
    Name:                 PodMonitoring/custom/prom-example/metrics
    Sample Groups:
      Count:  3
      Sample Targets:
        Health:  up
        Labels:
          Cluster:                     CLUSTER_NAME
          Container:                   prom-example
          Instance:                    prom-example-589ddf7f7f-hcnpt:metrics
          Job:                         prom-example
          Location:                    REGION
          Namespace:                   NAMESPACE_NAME
          Pod:                         prom-example-589ddf7f7f-hcnpt
          project_id:                  PROJECT_ID
        Last Scrape Duration Seconds:  0.020206416
        Health:                        up
        Labels:
          ...
        Last Scrape Duration Seconds:  0.054189485
        Health:                        up
        Labels:
          ...
        Last Scrape Duration Seconds:  0.006224887
```
## Step 5: Query the metrics using PromQL in Cloud Monitoring
The simplest way to query your Prometheus data is to use the Cloud Monitoring Metrics Explorer page in the Google Cloud console. To verify that your Prometheus data is being collected correctly, do the following:

1. In the navigation panel of the Google Cloud console, select Monitoring, and then select  Metrics explorer
2. In the toolbar of the query-builder pane, select the button whose name is either code MQL or code PromQL.
3. Ensure the Auto-run toogle switch is off.
4. Enter the following query into the editor, and then click Run query:
```
example_requests_total
```
![](visualized_metrics.png)
