# GKE Prometheus Grafana Monitoring

Welcome to the GKE Prometheus Grafana Monitoring repository! This project aims to provide a comprehensive guide and resources for monitoring applications deployed on Google Kubernetes Engine (GKE) using Prometheus and visualizing the metrics in Grafana.

## Introduction

As organizations increasingly adopt microservices architectures and containerized applications, effective monitoring and observability become crucial for ensuring reliability, performance, and scalability. Google Kubernetes Engine (GKE) offers a robust platform for deploying and managing containerized workloads, while Prometheus and Grafana provide powerful tools for monitoring and visualizing metrics.

This repository serves as a central hub for understanding and implementing monitoring solutions using Prometheus and Grafana on GKE. Whether you're new to Kubernetes monitoring or looking to optimize your existing monitoring setup, you'll find resources, tutorials, and best practices to help you monitor your applications effectively.

## Key Features

- **Managed Service for Prometheus (MSP)**: Learn how to leverage [Google's Managed Service for Prometheus](https://cloud.google.com/stackdriver/docs/managed-prometheus)
 to collect, store, and query metrics from your GKE clusters effortlessly.
- **Sample Application Deployment**: Follow step-by-step instructions to deploy a sample application on GKE and instrument it for monitoring with Prometheus.
- **Grafana Visualization**: Explore best practices for configuring Grafana dashboards to visualize metrics collected by Prometheus, including request rate metrics, latency, error rates, and more.
- **PromQL Queries**: Dive into PromQL (Prometheus Query Language) to write custom queries for analyzing and troubleshooting your application metrics.

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

The example application emits the example_requests_total counter metric and the example_random_numbers histogram metric (among others) on its metrics port. The manifest for the application defines three replicas.
