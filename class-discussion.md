Sample Kubernetes and Helm Files for Production Grade Scenario

Think of this scenario:

We are hosting a payment gateway:

this payment gateway runs on nginx + java

and since it is serving a lof of live traffic:

so we have enabled auto-scaling here using aws auto-scaling groups

and min - 5 ec2 instances -

Route53 -> ALB -> Target Groups -> 5 EC2 servers (which can increase) -> RDS (Mysql)
																	  -> S3 Bucket (Images store)


running fine from 15 days, but I have recieved an incident saying that connection timeout - this error has started occruring a lot

clarity on the architecture
traverse the customer request journey
identify the hops

troushooting on all the levels

CloudWatch's ALB metrices to validate the incident - 5xx error

If the target groups say that the backend servers are un-healthy -> alb to ec2 server confs, like which port it is talking to, is there a change in the sg b/w alb and ec2


now, we will check the servers:
cloudwatch dashboard - we will see the ec2 server metrices like: cpu consumption during the incident window, memory consumptions, storage, io

then you have figured out that 1 particular ec2 instance is causing the issue, 20% of the request are getting connetion timeout

then again things were fine from the metrices side.

you will login to all the servers, open different different log file across and start tailing or viewing the logs file and then find the issue - 5 servers, then you have search through multiple logs files as well




Default Configuration for centralized logging:

stodout & stderror

always preferred to have stdout of all the application to the console or the terminal

kubectl logs <pod-name> 


why is there a requirement centralized logging solutions


kubectl logs <pod-name> -n <ns-name> -f


In persistant volume in kubernetes:
in pods the application run -> continuously accessing the persistant volume and writing logs files in it



s3:
application team while developing their code, they can use S3 directly s3 sdks
or they can ask devops to sync their logs from to s3



database with app for logs: if the app team has implemented a logic to store all the logs files in the database


resource:
    request:
        cpu:
        memory:
    limit:
        cpu:
        memory:

this where we define, what should be allocated to the containers

meaning of requests are minimum 1 vcpu and .5 gb memory will be associated
meaning of limits are max 2 vcpus and 1 gb of memory will be assigned to a container

monitoring - will help us to identify if the request consumption is reaching the max level, may be it is trying to use all the memory and cpu of the node where it is runing

liveness - health check - curl <url>:<port>/health - 200
readiness probe - working fine or not - curl <url>:<port>/ready - 200

script:
automating certain manual task - netstat -tulnp | grep <port>


IRSA:

IAM Role for Service account


AWS: 

IAM Role - Service to Service level communication
            e.g: ec2 and s3
                : cloudwatch agent pods running in EKS cluster can communicate to cloudwatch service


Kubernetes Side: RBAC (Role Based Access control)
Inside K8S, we need to create a service account


AWS                          EKS
IAM Role            <--->    Service Account -> CloudWatch Agent Pods