# Kubernetes GPU Nodes Health Check Scripts

## Components

**gpu-health-checker:**  
A DaemonSet responsible for monitoring and addressing AMD GPU performance issues on cluster nodes.

Main features:
- Checks for the presence and correct functioning of `rocm-smi` and `rocminfo` tools to validate GPU health.
- Searches dmesg logs for GPU performance issues such as Runlist oversubscription, evicted queue buffers, hardware exceptions, or GPU hang/memory faults.
- Assesses the network health of the node by checking download/upload bandwidth using `speedtest-cli` (customizable threshold, defaults: download ≥ 100 Mbps, upload ≥ 50 Mbps).
- When GPU health problems are detected, the script auto-labels the node (e.g., `gpu-failure`) and applies a `repair-reboot` taint. These taints/labels are removed automatically upon recovery.

**gpu-node-reboot-agent:**  
Another DaemonSet that checks for running CI workload pods (e.g., `actions-ephemeral-runner` or Jenkins agents) on each GPU node. Every 3 hours, if there are no such pods, it will reboot the node using either the platypi tool or system-level reboots.

## Usage

```bash
kubectl apply -f health_check_gpu_performance.yaml
```

## Behavior example

If all CI jobs have been cleaned up, the node will be rebooted using Platipy; if Platipy fails, a system reboot will be used as a fallback.

```
│ Checking CI pods cleaned up on node ctr-mlse-b40-05                                                                                                          │
│ Checking pod calico-node-mf7w6                                                                                                                               │
│ Checking pod node-debugger-ctr-mlse-b40-05-zqwxg                                                                                                             │
│ Pod node-debugger-ctr-mlse-b40-05-zqwxg has no labels. Skipping                                                                                              │
│ Checking pod gpu-health-checker-dvg4x                                                                                                                        │
│ Checking pod gpu-node-reboot-agent-cmb96                                                                                                                     │
│ Checking pod harbor-artifactory-cert-installer-kddhw                                                                                                         │
│ Checking pod amd-gpu-operator-node-feature-discovery-worker-9pfpq                                                                                            │
│ Checking pod default-device-plugin-9xftr                                                                                                                     │
│ Checking pod default-metrics-exporter-chstd                                                                                                                  │
│ Checking pod default-node-labeller-flkhx                                                                                                                     │
│ Checking pod coredns-monitor-pg7m9                                                                                                                           │
│ Checking pod kube-proxy-ctr-mlse-b40-05                                                                                                                      │
│ Checking pod rke2-ingress-nginx-controller-tkrps                                                                                                             │
│ All CI pods on node ctr-mlse-b40-05 are cleaned up                                                                                                           │
│ Checked node ctr-mlse-b40-05: CI pods cleaned up: True                                                                                                       │
│ All CI pods on node ctr-mlse-b40-05 are cleaned up. Rebooting node ctr-mlse-b40-05...                                                                        │
│ Rebooting node ctr-mlse-b40-05                                                                                                                               │
│ Draining node ctr-mlse-b40-05                              

......



```

If CI jobs is not cleaned up, it will skip rebooting:
```
│ Checking CI pods cleaned up on node ml-ci-internal-old                                                                                                       │
│ Checking pod calico-node-mm7c4                                                                                                                               │
│ Checking pod node-debugger-ml-ci-internal-old-hc96g                                                                                                          │
│ Pod node-debugger-ml-ci-internal-old-hc96g has no labels. Skipping                                                                                           │
│ Checking pod gpu-health-checker-lkwgt                                                                                                                        │
│ Checking pod gpu-node-reboot-agent-9hh2l                                                                                                                     │
│ Checking pod harbor-artifactory-cert-installer-v2sc7                                                                                                         │
│ Checking pod k8s-build-cluster-btk84                                                                                                                         │
│ Pod k8s-build-cluster-btk84 is a Jenkins slave pod. So, CI pods are not cleaned up.                                                                          │
│ Checked node ml-ci-internal-old: CI pods cleaned up: False  
```

If a node is already in a cordoned state, this indicates it may be under repair or reserved for other use, so the reboot will be skipped.

```
│
│ Node smci355-ccs-aus-n15-25 is in cordoned state. Skipping reboot and returning...                                                                           │
│ Node smci355-ccs-aus-n15-25 is in cordoned state. Skipping reboot and returning...                                                                           │
│ Node smci355-ccs-aus-n15-25 is in cordoned state. Skipping reboot and returning...                                                                           │
│ Node smci355-ccs-aus-n15-25 is in cordoned state. Skipping reboot and returning...                                                                           │
│ Node smci355-ccs-aus-n15-25 is in cordoned state. Skipping reboot and returning...
```


## TODO

- [ ] Implement node draining before rebooting the machine. Rebooting without draining may cause disruption to running workloads.
