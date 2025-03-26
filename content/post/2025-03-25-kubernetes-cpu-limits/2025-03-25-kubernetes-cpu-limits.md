---
title: How k8s CPU limits work
subtitle: What happens when a pod hits 100% CPU?
date: 2025-03-25
tags: ["kubernetes", "cpu", "stress", "linux"]
---

Everyone knows that once a pod hits 100% of memory, it will be killed by the OOM killer. But did you know what happens with CPU?

Let's say you have a pod limited to 100mCPU. What actually happens under the hood?

## Understanding CPU Limits in Kubernetes

As per [official documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/):

> cpu limits are enforced by CPU throttling. When a container approaches its cpu limit, the kernel will restrict access to the CPU corresponding to the container's limit. Thus, a cpu limit is a hard limit the kernel enforces. Containers may not use more CPU than is specified in their cpu limit.

In simple terms:

* Memory limits → Your pod gets killed(restarted) if it exceeds its limit.
* CPU limits → Your pod is throttled, meaning it slows down but doesn't get killed.

In more detail, the Linux kernel is throttling the CPU usage using cgroups. Recommended reading: [CFS Bandwidth Control](https://docs.kernel.org/scheduler/sched-bwc.html).

To make it more visible I wanted to show a simple example.

## Testing CPU Throttling in K8s
For this experiment, I created a simple nginx pod with a CPU limit of 100mCPU (0.1 CPU core). To demonstrate throttling in action, I used the `stress` tool to generate a heavy CPU load with 4 worker threads. While the pod could technically handle a single CPU thread without significant throttling, using 4 threads makes the throttling effect much more pronounced and easier to observe in our measurements.

The stressed pod is defined as follows:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-test-nginx
spec:
  containers:
  - name: nginx-stress
    image: debian
    command: ["/bin/bash", "-c"]
    args:
      - apt update && apt install -y nginx stress procps && 
        echo "Starting NGINX..." && 
        nginx && 
        echo "Starting CPU stress test..." && 
        stress --cpu 4 &
        tail -f /dev/null
    ports:
      - containerPort: 80
    resources:
      limits:
        cpu: "100m"
```        
## Measuring the Impact
To collect the effect of CPU throttling, I measured response times using this simple script from a devtools pod (Any pod with curl installed will do):

```bash
while true; do curl -s -o /dev/null -w "%{time_total}\n" 10-244-0-24.stress.pod.cluster.local:80 >> /curl.log; sleep 1; done
```
I ran two scenarios:
1. 5 min with CPU stress
2. 5 min without CPU stress

### Observations
✔️ All responses returned HTTP 200.

⚡ Response times were significantly higher when stress was applied.

To visualize the difference, I plotted the response times in...
Google Sheets. Yes, you heard that right! Improvisation at its finest!

![I have multiplied the response time by 1000 to make it more visible as curl was returning the time in seconds.](/post/2025-03-25-kubernetes-cpu-limits/cpu-throttling.png)



## Conclusion
CPU throttling is a critical feature of the Linux kernel that enforces container CPU limits through cgroups. While it prevents resource exhaustion, it can significantly impact application performance, especially for latency-sensitive workloads.

Key takeaways:
- CPU limits are enforced through throttling, not termination
- Throttling can lead to increased latency and reduced throughput
- Monitor your workloads for throttling events through metrics like response latency, p90/p95 percentiles, and CPU throttling metrics
- Consider using HPA to automatically scale based on CPU usage

Remember to continuously monitor and adjust limits based on actual usage patterns and performance requirements.







