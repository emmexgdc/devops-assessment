Q1: Infrastructure as Code (IaC)
"How do you ensure a new staging environment on GCP matches production exactly using Terraform? Talk through your approach to managing environment-specific variables and avoiding drift."
Ans:
I enforce consistency between different environments by making use of modules. Since modules are reusable, I have similar infrastructure for all environments, only things that are unique or different are the values.

For environment-specific variables, I make use of tfvars for each environment while for sensitive values, I make use of a secret manager, in this instance that would be GCP secret manager, which terraform uses during its runtime. Workspaces can also be implemeneted for more isolation, but directories and tfvars for each environment makes it simple or less complex.

For avoiding or managing drifts: I would make use of the "terraform plan" to validate changes in every branch, or periodically to ensure what is in the pre-prod environments match what I have in production. I would also make use of state locking and remote backends with GCS buckets to prevent concurrent runs and have a single source of truth for every one.
Tagging resources are also helpful, though its a small effect but it helps know what is managed by th ecode or not.



Q2: Kubernetes Observability
"A pod is in a CrashLoopBackOff state. Walk me through the specific kubectl commands you would use to diagnose the root cause and how you distinguish between application errors and resource constraints."
Ans:
My approach would be checking events and exit codes, then logs and resource metrics.
- first I get the state of the pod: kubectl get pods -n (namespace)
  to get restart counts or how long its been crashing.
- to get more details: kubectl describe pod (podname) -n (namespace)
  from here i can get the exit codes, events and container sections like requests/limits status. actual errors can be gotten from the exit codes also.
- I can then go ahead to check the logs so as to get the actual error:    kubectl logs <pod-name> -n <namespace>

I can simply say Application errors can refer to errors on the log that are connected to config issues, failed dependencies, file permissions, missing volumes, db connection failures due to wrong credentials, wrong image tag or a configmap/secret mounted wrongly.
While Resource Constraints have to do with resoource limitations like OOMKilled which is due to pod exceeding the memory limit. sometimes the node may be out of resources and evicting pods.
For deeper debugging I can exec into the running pod using the kubectl exec -it command.


Q3: Secret Management
"How do you securely pass a database password to a GKE deployment? What are the 'anti-patterns' you avoid when handling sensitive credentials in a CI/CD pipeline?"
Ans:
using the GCP secret manager or a proper secret manager, then link the GKE service accont to a GCP service acct.
then I use the GKE Secrets Store CSI Driver to mount secrets directly from Secret Manager into the pod as files or environment variables.

Anti-Patterns I avoid:
- not hardcoding secrets to codes, workflow or dockerfiles.
- printing secrets in logs (during CI/Cd).
- commiting secrets to git.
- also overly prmissive IAM.

Q4: Reliability (Redis)
"How do you monitor Redis health for our session management? Which specific metrics would trigger an alert for you, and at what thresholds?"

For memory usage, I would monitor the used_memory/maxmemory ratio setting thresholds at 75%-80% for warnings and 85%-90% as critical requiring immediate action. Also I wold monitor cpu usage with used_cpu_sys and used_cpu_user. I would focus more on memory, latency, replication and eviction of keys.
- If using GCP Redis Memorystore I would make use of the built-in dashboards as they cover most of the metrics, and I can set alerts to my email through the dashboards.
- Can also make use of redis exporter for prometheus, Alertmanager for notifications and Grafana for visualisation.


