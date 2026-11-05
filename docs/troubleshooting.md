# Operational Troubleshooting Guide

This guide details common edge cases, configuration anomalies, and replication issues encountered during the implementation and maintenance of the `calmadosoul.online` multi-region architecture.

---

## 🔍 Issue 1: Primary Health Check Flapping (Route 53 Oscillations)

### Symptoms
Traffic rapidly oscillates between the Mumbai (`ap-south-1`) and Sydney (`ap-southeast-2`) regions. Users experience inconsistent sessions, alternating between "Hello world PROD!" and "Welcome to Sydney" pages on refresh.

### Root Cause
The Route 53 health check `Failure Threshold` was set too low (e.g., 1 or 2 cycles), or the application server experienced short, transient CPU spikes. This caused temporary endpoint response latency, causing Route 53 to prematurely flag the region as down before it recovered naturally.

### Remediation & Fix
1. Navigate to the **Route 53 Console** -> **Health Checks**.
2. Select **Mumbai health** and click **Edit Configuration**.
3. Adjust the **Failure Threshold** from `2` to `3` or `5` to prevent false positives from brief network blips.
4. Increase the **Request Interval** to `30 seconds` if application warm-up cycles cause minor initial timeouts.

### Validation Result
The health status history graph stabilized. Transient backend spikes no longer trigger unintended global DNS failover sequences.

---

## 🔍 Issue 2: Local DNS Caching Stalling Traffic Takeover

### Symptoms
The Route 53 console shows that the **Mumbai health** check is explicitly `Unhealthy`, but local client machines continue resolving to the dead Mumbai IP addresses instead of routing to Sydney.

### Root Cause
The Time-To-Live (TTL) configuration on the Route 53 Hosted Zone record sets was set to a long duration (e.g., 300 seconds or higher). Local ISPs and OS-level DNS caches pinned the old record, delaying the failover visibility.

### Remediation & Fix
1. Open the Route 53 Hosted Zone for `calmadosoul.online`.
2. Select the Primary and Secondary **A Records**.
3. Lower the **TTL (Seconds)** value to `60` or `30` seconds. This forces downstream recursive resolvers to flush and fetch the active target endpoint quickly.
4. Force-flush the local workstation cache during testing:
   ```cmd
   ipconfig /flushdns
   ```

### Validation Result
Executing `nslookup calmadosoul.online` confirms that the client-side cutover window immediately tracks under the target 90-second RTO window upon primary region eviction.

---

## 🔍 Issue 3: Sydney ALB Displays "502 Bad Gateway" Post-Failover

### Symptoms
Route 53 correctly routes traffic to the Sydney endpoint (`://amazonaws.com`), but the browser throws an HTTP `502 Bad Gateway` error instead of rendering the "Welcome to Sydney" landing page.

### Root Cause
The Application Load Balancer in Sydney cannot establish a connection with the backend EC2 instances. This is caused by a missing or misconfigured Security Group rule on the backend instances, which blocks incoming traffic from the Sydney ALB's internal private IP space.

### Remediation & Fix
1. Navigate to **EC2** -> **Target Groups** in the Sydney region and inspect the target health status.
2. If instances show `Unhealthy` with the description *“Health check timed out”*, locate the instance Security Group.
3. Add an **Inbound Rule** allowing HTTP traffic on Port `80` (or `443` for HTTPS) where the **Source** is explicitly set to the Security Group ID of the Sydney ALB.

### Validation Result
Target group status transitioned to `Healthy`. The fallback application renders cleanly upon browser refresh during a failover drill.
