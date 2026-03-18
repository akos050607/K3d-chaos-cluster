# 🚀 Zero-Trust Hybrid Cloud & Chaos Engineering Lab

This project is a practical implementation of a highly available, self-healing, and auto-scaling Kubernetes architecture. It demonstrates modern **Systems Engineering** principles by bridging a public cloud VPS and an on-premise local machine through a secure, zero-trust mesh VPN, simulating an Edge Computing environment.

## 🎯 Project Objectives
* **Decoupled Architecture:** Separate the Control Plane (Cloud) from the Compute Nodes (Edge).
* **Zero-Trust Networking:** Ensure the Kubernetes API and all internal cluster traffic never traverse the public internet unencrypted.
* **High Availability (HA):** Prove the system's ability to survive unexpected hardware/network failures (Chaos Engineering).
* **Elasticity:** Implement metrics-based Horizontal Pod Autoscaling (HPA) to handle sudden traffic spikes dynamically.

## 🛠️ Technology Stack
* **Infrastructure:** K3s (Lightweight Kubernetes)
* **Networking:** Tailscale (WireGuard-based Mesh VPN), Flannel (CNI over VPN)
* **Compute:** Hetzner Cloud VPS (Control Plane / Master Node), Local Ubuntu Machine (Edge / Worker Node)
* **Workload:** Nginx (Stateless web servers acting as simulated streaming nodes)
* **Load Testing:** ApacheBench (`ab`)

---

## 🏗️ Architecture Overview

The system is designed to bypass the need for public IP exposure for the worker nodes. 
By utilizing Tailscale, the cluster spans across physical locations securely. The `kube-proxy` binds the `NodePort` across the Tailscale network interface, allowing the load balancer to distribute traffic seamlessly between the cloud and the on-premise machine.

---

## 🌪️ Chaos Engineering: The "Sleep" Incident

A core principle of Systems Engineering is building systems that expect failure. During the lab, an unplanned hardware failure occurred: **the local Ubuntu Edge node went into sleep mode.**

### The System's Reaction (Self-Healing)
1.  The Master Node (VPS) detected that the Edge Node became unresponsive.
2.  After the standard 5-minute Grace Period, the Master marked the Edge Node as `NotReady`.
3.  **Eviction:** To maintain the declarative state of exactly 4 replicas, the Master immediately evicted the 2 lost Pods from the Edge Node and rescheduled them onto itself.
4.  Zero downtime was achieved.

<img width="932" height="501" alt="Pod Eviction and Rescheduling" src="https://github.com/user-attachments/assets/3a5bffd8-0c9d-4533-b74b-3b4c110cf5af" />

*Note: When the Edge node woke up and reconnected, the Master intentionally did not migrate the pods back to avoid unnecessary disruption. A manual `rollout restart` was performed to rebalance the cluster.*

---

## 🚦 Load Testing & Auto-Scaling (HPA)

To test the resilience of the Load Balancer and the Tailscale tunnel, a massive DDoS-like traffic spike was simulated using `ApacheBench`.

### Phase 1: Fixed Replicas (No Auto-scaling)
* **Configuration:** 4 static Nginx replicas.
* **Attack:** `ab -n 100000 -c 100 http://100.120.50.29:30662/` (100,000 requests, 100 concurrent users over the VPN).

**Results:**
* **Failed requests:** `0` (Flawless routing)
* **Requests per second:** `1718.34 [#/sec]`
* **Mean time per request:** `58.196 [ms]`

[Watch the Phase 1 Screencast](https://github.com/user-attachments/assets/b6ef64e3-ffc7-4452-bb03-615322705d39)

### Phase 2: Horizontal Pod Autoscaler (HPA) Activated
To make the system elastic, resource limits and an autoscaler were introduced.

```bash
# Setting strict CPU limits per container (5% of a core)
sudo k3s kubectl set resources deployment mini-netflix --requests=cpu=50m --limits=cpu=100m

# Configuring the HPA to scale out if average CPU hits 50% of the allocated 50m
sudo k3s kubectl autoscale deployment mini-netflix --cpu-percent=50 --min=4 --max=20
```

**The Attack & System Reaction:**
When the exact same 100,000-request load test was executed, the CPU metrics skyrocketed. The HPA calculated the necessary capacity using the standard Kubernetes formula:
`DesiredReplicas = ceil[CurrentReplicas * (CurrentMetricValue / DesiredMetricValue)]`

The cluster dynamically scaled from 4 pods up to **10 pods** to distribute the heavy load. 

**Results (Phase 2):**
* **Failed requests:** `0`
* **Requests per second:** `1691.67 [#/sec]`
* *Observation:* The bottleneck shifted from the cluster's compute capacity to the local attacker's network/VPN encryption throughput. The cluster easily handled the load and scaled up precisely as requested.

[Watch the Auto-Scaling Screencast](https://github.com/user-attachments/assets/ca27102f-b2cb-43f9-85e2-a28be8518dae)

*After the test concluded and the cool-down period passed, the HPA gracefully terminated the unneeded pods, scaling back down to 4.*

---

## 🔮 Future Roadmap: Multi-Master HA
The current architecture uses a single Master Node (Edge pattern). To achieve true enterprise-grade High Availability for the Control Plane (Production pattern), the next phase involves:
* Deploying an odd number of Master Nodes (e.g., 3 VPS instances).
* Implementing an `etcd` distributed datastore for leader election and state synchronization to eliminate the single point of failure in the Control Plane.

---
*Built with ❤️ and Chaos.*
