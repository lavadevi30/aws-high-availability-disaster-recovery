# Multi-Region Architecture Implementation Guide

This document records the exact configuration steps and architectural parameters deployed through the AWS Management Console to provision the highly available infrastructure for `calmadosoul.online`.

---

## 🗺️ Environment Reference Blueprint

* **Production Domain:** `calmadosoul.online`
* **Primary Region:** Mumbai, India (`ap-south-1`) — **Active Status**
* **Disaster Recovery Region:** Sydney, Australia (`ap-southeast-2`) — **Standby Status**

---

## 🛠️ Step-by-Step Provisioning Registry

### Phase 1: Symmetric Network & Compute Infrastructure
1. **EC2 Application Servers:** Provisioned Amazon Linux EC2 instances in both regions to host the web application layer.
2. **Application Load Balancers (ALB):** Created an internet-facing ALB in both regions to sit in front of the application tiers.
   * **Mumbai Endpoint:** `://amazonaws.com`
   * **Sydney Endpoint:** `://amazonaws.com`
3. **Target Groups:** Standardized target groups listening on HTTP Port 80. Registered the regional instances and established the health check paths to `/`.

### Phase 2: Route 53 Health Check Configurations
To automate failure detection without manual intervention, two dedicated health checks were established inside the Amazon Route 53 Dashboard:

1. **Mumbai Health Monitor (`Mumbai health`):**
   * **Target Type:** Endpoint (Domain Name)
   * **Protocol:** HTTP (Port 80)
   * **Domain Name:** `calmadosoul.online`
   * **Request Interval:** 10 seconds (Fast monitoring)
   * **Failure Threshold:** 3 consecutive failures
2. **Sydney Health Monitor (`sydneyhealth`):**
   * **Target Type:** Endpoint (Domain Name)
   * **Protocol:** HTTP (Port 80)
   * **Domain Name:** `://amazonaws.com`
   * **Request Interval:** 30 seconds (Standard monitoring)

### Phase 3: Failover Record Topologies
Within the public hosted zone for `calmadosoul.online`, an active-passive routing policy was configured by binding both load balancers to the root apex domain using Alias records:

| Record Name | Type | Routing Policy | Set ID | Evaluate Target Health | Associated Health Check |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `calmadosoul.online` | A | **Failover (Primary)** | `mumbai-primary` | Yes | `Mumbai health` |
| `calmadosoul.online` | A | **Failover (Secondary)** | `sydney-standby` | Yes | `Syndney health` |

*Note: The Time-to-Live (TTL) routing profile is systematically managed via the AWS Alias layer to ensure rapid DNS records invalidation during failover execution.*

---

## 🔒 Security Group Control Constraints

To secure perimeter traffic while maintaining health monitoring functionality, security groups were locked down to the following rule matrix:

* **Inbound Load Balancer Rules:** Allow incoming traffic on TCP ports 80 and 443 from all public sources (`0.0.0.0/0`) to process user requests.
* **Inbound Application Instance Rules:** Restrict incoming HTTP port 80 traffic *exclusively* to the Security Group ID of the regional Application Load Balancer. This prevents public users from bypassing the web entry points.
