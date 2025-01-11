# ABB

**TASK 1: GitHub Repository: Branching and Workflow**

Objective
Demonstrating the creation of a GitHub repository, implementation of branching strategies, pull requests, and code reviews.

![image](https://github.com/user-attachments/assets/0ddc801c-8334-4ef2-b6e3-909a12fb24de)

Created a repo and made master as the default branch used for production deployments, and develop branch for lower environments. 
Now i will pull my code from the repo to my local system and modify the code and commit it using some feature branch 

![image](https://github.com/user-attachments/assets/0334b801-d653-43e6-a180-9dcc2f591e94)

To push the code we can authenticate using PAT token or username and password
![image](https://github.com/user-attachments/assets/b33e270f-399c-45df-8d4b-bba88bf06004)

Create a PR and assign any reviewr in organizaation
![image](https://github.com/user-attachments/assets/5af1e17f-1a1a-4a02-a070-4700b67df1fa)

once the rewiewer approves we can merge the code to the base branch 
![image](https://github.com/user-attachments/assets/ff97209f-5b7a-4c30-885b-4c598fce7481)


**TASK 2, 3, 6, 7, 8, 12, 13, 14, 15,  CI CD Pipelines, Variable groups and variables, sonarqube, Trivy, dockerbuild, Docker image scan (Trivy), dynamic update of the image tag, Azure key vault integration, AKS deployment.**
 
NOTE: Since my free trail of azure subscription is expired and my credit card is not supporting pay as you go payments, i have taken my POC pipelines and attaching the screenshots where few stages are missing but the explation is given. 

Here the demo work flow is done on a sample e-commerce application, where the entire CI/CD stage is demonstrated and deployed onto AKS. 
 
Pre-requisites:
1. Azure Service Connections:
•	SonarQube: Service connection for SonarQube integration.
•	Docker Registry: Service connection to the Docker registry (Azure Container Registry).
•	Azure Key Vault: Service connection to Azure Key Vault for secret management.
•	Azure Kubernetes Service (AKS): Service connection to AKS for deployment.
2. Trivy: Ensure that Trivy is installed and accessible on the pipeline agent for security scanning.
3. Dockerfile: A properly configured Dockerfile must exist in repository.
4. Kubernetes Manifest: A deployment.yaml file with the appropriate configurations (to be updated and deployed).
5. Email Configuration: Ensure that the email service is configured to send email reports (for the Trivy scan).

![image](https://github.com/user-attachments/assets/e7f1bae2-1e07-411f-a81e-1645817d86c6)

Then create a ENV variables in the library seccion and using the Variablegroup name in the pipeline.yaml we can retrive the variables into the piupeline
![image](https://github.com/user-attachments/assets/a677cdef-1b3c-4ca7-a3a5-a1df4bde0d08)

Here is the link to the Pipeline.yml where the detailed stages are given.
https://github.com/devopsandlearn/ABB/blob/master/worker/worker-pipeline.yaml

Here’s a concise breakdown of each stage in the pipeline. 

Build Stage:
Checkout the code, run SonarQube analysis for code quality checks, and build a Docker image using the Dockerfile.

TrivyScan Stage:
Scan the built Docker image for high and critical vulnerabilities using Trivy; fail the pipeline and send a detailed report if vulnerabilities are found.

Push Stage:
Push the Docker image to Azure Container Registry (ACR) if no vulnerabilities were found in the TrivyScan stage.

Update Stage:
Update Kubernetes deployment manifests with the new Docker image tag to ensure the latest image is used for deployment.

Deploy Stage:
Deploy the updated Docker image to Azure Kubernetes Service (AKS), using secrets from Azure Key Vault for configuration.

https://github.com/devopsandlearn/ABB/blob/master/k8s-specifications/worker-deployment.yaml

Here in this image it is shown that that docker image is built and pushed onto ACR 
Dockerfile https://github.com/devopsandlearn/ABB/blob/master/worker/Dockerfile

![image](https://github.com/user-attachments/assets/a18623d0-67bf-4e11-96c8-111de0ec21b8)

Debugging: If pipeline fails due to some reason first check the logs, and if something fro m the env variable session is missing or incorrect, then do the correction accordingly. 

**TASK 15: Kubernetes networking issue:**
1. Issue Description
The issue occurred within the AKS cluster, where the pods could not communicate with external services, such as a database. This resulted in network timeouts and connectivity failures.

2. Investigation Steps
To investigate the issue, the following steps were carried out:

2.1 Check Pod Status
The first step was to check the status of the pods to ensure they were running correctly:
kubectl get pods -n <namespace>
The output showed that the pods were in a Running state, with no immediate errors.

2.2 Describe Pod and Look for Errors
kubectl describe pod <pod-name> -n <namespace>
In the output, the following network-related error was observed in the events section:
Events:
  Type     Reason                  Age                From                       Message
  ----     ------                  ----               ----                       -------
  Normal   Starting                10m                kubelet, aks-node-1        Starting container
  Warning  FailedCreatePodSandBox  10m (x2 over 12m)  kubelet, aks-node-1        Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container for pod "myapp-pod": network plugin "kubenet" failed to set up pod "myapp-pod" network: failed to find a valid network for pod
This indicated that the network plugin (kubenet) was failing to set up the network for the pod, potentially due to misconfigurations.

2.3 Check Pod Logs
kubectl logs <pod-name> -n <namespace>
The logs contained error messages such as:
2023-12-01 10:00:01 Error: Network Timeout
2023-12-01 10:00:02 Error: Unable to reach external service: http://example.com
This confirmed that the pod was trying to access external services but was unable to establish a connection, likely due to network configuration issues.

2.4 Check Network Policies
Finally, network policies were reviewed to see if there were any restrictions on outgoing traffic:
kubectl get networkpolicy -n <namespace>
The inspection revealed that there were restrictive network policies in place, potentially blocking outbound traffic.

3. Root Cause
The issue was determined to be related to restrictive network policies that were preventing the pods from accessing external services. The kubenet network plugin was also involved, likely due to these policies blocking traffic.

4. Resolution Steps
To resolve the issue, the following steps were taken:

4.1 Modify Network Policies
A new network policy was created to allow all outbound traffic from the affected pods. The new policy looked as follows:

allow-all-outbound.yaml
https://github.com/devopsandlearn/ABB/blob/master/worker/allow-all-outbound.yaml

This policy ensured that no egress traffic from the pods would be blocked.

4.2 Apply Network Policy
kubectl apply -f allow-all-outbound.yaml

4.3 Verify the Fix
After applying the fix, connectivity was verified by executing a curl command within the pod to access an external service:
kubectl exec <pod-name> -n <namespace> -- curl http://external-service-url
The command returned a successful response, confirming that the pod was now able to reach the external service.

**Task 9: Package the Kubernetes deployment into a Helm chart and deploy it. Include parameters for customization (replicas, image tags)**

https://github.com/devopsandlearn/ABB/blob/master/Tasks/TASK-9



**TASK 11 and 12**
We have used Prometheus and grafana for metrics and monitering. 
Promql query for CPU usage in the last 5 min 
(sum(rate(container_cpu_usage_seconds_total{job="kubelet", cluster="", container!="", pod!="", namespace=~".*"}[5m])) by (namespace, pod) > 0.9

For memory usage if more than 500MB
sum(container_memory_usage_bytes{job="kubelet", cluster="", container!="", pod!="", namespace=~".*"}) by (namespace, pod) > 500000000










