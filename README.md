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