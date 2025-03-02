Payment Gateway Troubleshooting Scenario:
-----------------------------------------

Architecture Overview:
-----------------------

We're hosting a payment gateway running on Nginx + Java with the following architecture:

Route53 → ALB → Target Groups → 5 EC2 servers (auto-scalable) → Backend services (RDS MySQL, S3 for image storage)

The system has been running normally for 15 days

Incident Report:
----------------

Connection timeout errors have suddenly started occurring at a high frequency.

General Troubleshooting Approach:
---------------------------------

1. Clarify Architecture & Request Flow
    Understand the complete request journey
    Identify all connection points and potential failure points
    Map dependencies between components

2. Validate the Incident with Metrics
    Check CloudWatch ALB metrics to confirm 5xx errors
    Look for patterns in error timing and frequency

3. Verify Target Group Health
    Determine if backend servers are being marked as unhealthy
    Check ALB to EC2 configuration settings (correct ports, protocols)
    Verify security group rules between ALB and EC2 instances haven't changed

4. Analyze Server Metrics
    Review CloudWatch dashboard for EC2 server metrics during the incident window:
        CPU utilization
        Memory consumption
        Storage capacity
        I/O performance

5. Isolate the Problem
    Identify that a single EC2 instance is causing 20% of the connection timeouts
    Note that metrics appear normal despite the ongoing issue

6. Perform Log Analysis
    Log into all servers (all 5 EC2 instances)
    Examine multiple log files on each server
    Tail logs in real-time to catch error patterns
    Search through various application logs for error messages related to timeouts

This troubleshooting scenario demonstrates the challenges of distributed system debugging and the importance of comprehensive logging and monitoring in a production environment.


Importance of Central Logging tool:
-----------------------------------
A centralized logging system would solve this issue by aggregating logs from all 5 EC2 instances into a single searchable platform, enabling faster correlation of errors across servers and eliminating the need to manually log into each server and search through multiple log files individually.


Logging in Kubernetes:
-----------------------

Standard file handles / data streams:
-------------------------------------

    stdout (Standard Output): The default data stream where a process sends its output data. In containerized applications, this is where normal application logs are written.

    stderr (Standard Error): A separate output stream specifically for error messages. This allows errors to be processed separately from regular output.


Understanding Container Logs in Kubernetes:
-------------------------------------------

In Kubernetes, application logging works differently from traditional systems. 

Container-based applications typically write logs to standard output (stdout) and standard error (stderr) streams rather than to files.

When containers write to these streams, the container runtime captures this output and typically redirects it to a logging driver, which determines where the logs are stored.

In a standard Kubernetes setup:
    Application logs written to stdout/stderr are captured by the container runtime
    The kubectl logs command retrieves these files on demand


Log Storage Approaches in Kubernetes:
-------------------------------------

1.) Persistent Volume Approach
How it works: Applications in pods write logs directly to a mounted persistent volume.

Key points:
    Simple implementation but creates I/O overhead
    Logs survive pod restarts but limited search capabilities

2.) S3 Storage Approach
How it works: Applications write logs to S3 using AWS SDKs or logs are synced to S3.

Key points:
    Durable, cost-effective storage
    Limited real-time analysis capabilities

3.) Database Approach
How it works: Application teams store logs directly in a database.

Key points:
    Structured storage with query capabilities
    Scaling challenges with high log volumes

Why Centralized Logging Tools Are Better:
-----------------------------------------

Purpose-built: Designed specifically for log handling at scale
Kubernetes-aware: Automatic enrichment with container metadata
Advanced capabilities: Full-text search, visualization, alerting
Simplicity: Applications only need to write to stdout/stderr
Operational efficiency: Single interface for all logs across the cluster


IRSA (IAM Roles for Service Accounts):
--------------------------------------

What it is: IRSA allows Kubernetes pods to securely access AWS services without storing credentials.

How it works:
AWS side: IAM Role defines permissions for AWS service access
Kubernetes side: Service Account is created and annotated with IAM Role ARN
AWS and EKS are connected through OIDC (OpenID Connect)

Example:
CloudWatch Agent pods need to send metrics to CloudWatch
IAM Role grants permission to write to CloudWatch
Service Account in Kubernetes is linked to this IAM Role
Pods using this Service Account automatically get AWS permissions

AWS                          EKS
IAM Role            <--->    Service Account -> CloudWatch Agent Pods