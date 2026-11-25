
# Session 04: Docker and Kubernetes on GKE

## 1. Running Kubernetes Pods

The following output shows the generator pod running successfully
on the GKE cluster.

```text
NAME                         READY   STATUS    RESTARTS   AGE     IP          NODE                             NOMINATED NODE   READINESS GATES
generator-59f565ccf4-sxh75   1/1     Running   0          9m52s   10.0.0.16   gke-freight-main-f4cb639e-18z7   <none>           <none>
```

## 2. Deployment Rollout History

The following output shows the Deployment revisions created
during the configuration update and rollback.

```text
deployment.apps/generator
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

## 3. Kubernetes Self-Healing

When I deleted the generator pod, Kubernetes automatically created
a replacement pod with a new name. This happened because the
Deployment controller continuously compares the desired state
with the actual state and creates a replacement when a pod is missing.

## 4. Rollout and Rollback

I changed the generator configuration to AIR mode with a rate of 5.
Kubernetes successfully rolled out the updated Deployment.
I then used `kubectl rollout undo` to restore the previous configuration.
Finally, I restored the original YAML settings to MODE=both and RATE=1.


## 5. GKE Standard vs Autopilot

GKE Standard gives users direct control over worker nodes, node pools,
machine types, and scaling, but requires more infrastructure management.
GKE Autopilot allows Google to manage the worker nodes and reduces
operational work, but provides less control over node-level configuration.

For this freight platform, I chose GKE Standard because it provides
greater control over infrastructure and node configuration, which is
useful for future workloads such as Kafka.
