After deployment with the broken manifest, I used kubectl get pods -w. which showed that the pod kept restarting:
jonjo@Emmexs-MacBook-Air task-a % kubectl get pods -w
NAME                            READY   STATUS             RESTARTS      AGE
sycamore-api-7955fd5847-rz5fj   0/1     CrashLoopBackOff   2 (19s ago)   48s
sycamore-api-7955fd5847-rz5fj   0/1     OOMKilled          3 (25s ago)   54s
sycamore-api-7955fd5847-rz5fj   0/1     CrashLoopBackOff   3 (2s ago)    56s

To get more details and events, I used the kubectl describe pod sycamore-api-7955fd5847-rz5fj
Name:             sycamore-api-7955fd5847-rz5fj
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.3
Start Time:       Thu, 05 Feb 2026 03:31:21 +0100
Labels:           app=sycamore-api
                  pod-template-hash=7955fd5847
Annotations:      <none>
Status:           Running
IP:               10.1.0.6
IPs:
  IP:           10.1.0.6
Controlled By:  ReplicaSet/sycamore-api-7955fd5847
Containers:
  api:
    Container ID:  docker://8d5ceb0847dab02e343f122075b75839c27a4bb5738803ba678b52c17c1afc41
    Image:         node:18-alpine
    Image ID:      docker-pullable://node@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e
    Port:          <none>
    Host Port:     <none>
    Command:
      node
      -e
      let arr=[]; setInterval(() => { arr.push(new Array(1000000).fill('data')) }, 100)
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Thu, 05 Feb 2026 03:34:30 +0100
      Finished:     Thu, 05 Feb 2026 03:34:31 +0100
    Ready:          False
    Restart Count:  5
    Limits:
      memory:  64Mi
    Requests:
      memory:     32Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zqcm7 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-zqcm7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m52s                default-scheduler  Successfully assigned default/sycamore-api-7955fd5847-rz5fj to docker-desktop
  Normal   Pulling    3m51s                kubelet            Pulling image "node:18-alpine"
  Normal   Pulled     3m43s                kubelet            Successfully pulled image "node:18-alpine" in 8.466s (8.466s including waiting). Image size: 44929293 bytes.
  Normal   Created    43s (x6 over 3m43s)  kubelet            Created container: api
  Normal   Started    43s (x6 over 3m42s)  kubelet            Started container api
  Normal   Pulled     43s (x5 over 3m41s)  kubelet            Container image "node:18-alpine" already present on machine
  Warning  BackOff    2s (x19 over 3m40s)  kubelet            Back-off restarting failed container api in pod sycamore-api-7955fd5847-rz5fj_default(49228867-115b-4f21-8681-ff9cf188a9b5)

  The key indicatore were the Exit code 137, state, Last state and Reasons.

  with OOMKilled it means the pod is exceeding its memory and there is a memory Leak.
  Easy fix would be to allocate more memory:
  
  resources:
  requests:
    memory: "128Mi"
  limits:
    memory: "256Mi"
  
  but this wouldn't solve the Memory Leak, though its a quick fix but it just delays the crash.

  Further Research on this code section:
  command:
            [
              "node",
              "-e",
              "let arr=[]; setInterval(() => { arr.push(new Array(1000000).fill('data')) }, 100)",
            ]

showed that this causes a countinous allocation of Memory and retains it by pushing it into arr. this causes an infinite loop or growth that causes the OOM-killed and restats.

The long-term stability solution based on my analysis would be to replace the leaking command witha stable process and add readiness/liveness probes. This approach eliminates the memory leak entirely, runs within the original 64Mi limit.
After applying the fixed-manifest.

jonjo@Emmexs-MacBook-Air task-a % kubectl apply -f fixed-manifest.yaml              
deployment.apps/sycamore-api configured
jonjo@Emmexs-MacBook-Air task-a % kubectl get pods -w                               
NAME                            READY   STATUS    RESTARTS   AGE
sycamore-api-6c6964d954-78l5h   1/1     Running   0          16s

jonjo@Emmexs-MacBook-Air task-a % kubectl describe pod sycamore-api-6c6964d954-78l5h
Name:             sycamore-api-6c6964d954-78l5h
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.3
Start Time:       Thu, 05 Feb 2026 04:04:23 +0100
Labels:           app=sycamore-api
                  pod-template-hash=6c6964d954
Annotations:      <none>
Status:           Running
IP:               10.1.0.7
IPs:
  IP:           10.1.0.7
Controlled By:  ReplicaSet/sycamore-api-6c6964d954
Containers:
  api:
    Container ID:  docker://717fb07f204bec1b1a3162c28b28dbd99a99334684915b612dcb326b9ff0545b
    Image:         node:18-alpine
    Image ID:      docker-pullable://node@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e
    Port:          8080/TCP
    Host Port:     0/TCP
    Command:
      sh
      -c
      node -e "
      const http = require('http');
      const server = http.createServer((req, res) => {
        res.writeHead(200, { 'Content-Type': 'text/plain' });
        res.end('Sycamore API - Healthy\n');
      });
      server.listen(8080, () => {
        console.log('Server running on port 8080');
      });
      "
      
    State:          Running
      Started:      Thu, 05 Feb 2026 04:04:23 +0100
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  64Mi
    Requests:
      cpu:        50m
      memory:     32Mi
    Liveness:     http-get http://:8080/ delay=5s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get http://:8080/ delay=3s timeout=1s period=5s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rpdtv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-rpdtv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  63s   default-scheduler  Successfully assigned default/sycamore-api-6c6964d954-78l5h to docker-desktop
  Normal  Pulled     62s   kubelet            Container image "node:18-alpine" already present on machine
  Normal  Created    62s   kubelet            Created container: api
  Normal  Started    62s   kubelet            Started container api