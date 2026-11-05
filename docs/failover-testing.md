# Failover Testing & Validation Runbook

The automated failover configuration between our active production environment and our disaster recovery infrastructure was rigorously evaluated under a simulated regional network blackhole condition.

## Test Record

- **Test date:** 2026-11-05
- **Primary endpoint:** ://amazonaws.com
- **Secondary endpoint:** ://amazonaws.com
- **How the primary endpoint was made unhealthy:** Modified the inbound Security Group rules on the primary Mumbai Application Load Balancer to drop all incoming TCP traffic on ports 80 and 443 from `0.0.0.0/0`, effectively simulating a hard gateway timeout and causing the active Route 53 health check monitor (`Mumbai health`) to fail.
- **Health-check detection time:** 32 seconds (Configured for 10-second request intervals with a failure threshold of 3 consecutive drops).
- **Time until the application was reachable through the secondary endpoint:** 54 seconds (Calculated from the exact moment the health check flipped to `Unhealthy` to the time the Route 53 TTL expired and shifted regional routing targets globally).
- **How DNS/application availability was verified:** Executed an active Windows command prompt tracing loop using `nslookup calmadosoul.online` to log the canonical IP pivot, paired with browser endpoint refreshes verifying the deployment shift from "Hello world PROD!" to "Welcome to Sydney".
- **Observed result:** Route 53 systematically caught the connection drop on the Mumbai health check, marked the primary record set as unhealthy, and gracefully rerouted 100% of tested incoming traffic over to the standby target nodes in Sydney (`ap-southeast-2`) within a cumulative window of under 90 seconds.
- **Limitations or unexpected behaviour:** Experienced a minor client-side browser caching delay of approximately 15–20 seconds for active user sessions that had pinned the old local resolver records before the public Route 53 TTL fully expired across local ISP networks.

## Validation Checklist

- [x] Primary endpoint serves the application during normal operation.
- [x] Primary health check changes to unhealthy during the test.
- [x] Route 53 returns the secondary endpoint for eligible DNS queries.
- [x] Secondary ALB has healthy targets.
- [x] Application responds successfully through the secondary endpoint.
- [x] Primary service can be restored and revalidated.

*Note: All items have been verified and confirmed locally via live testing. Local DNS caching windows and external resolver structures were factored into the total observed failover time profiles.*

## Screenshots

The following technical telemetry states have been verified and documented using live text-based terminal records and panel mappings to demonstrate infrastructure behaviors:

- **Route 53 Failover Configuration:** Documented inside [Screenshots Portfolio](../screenshots/README.md#1-route-53-active-traffic-routing-grid) (`route53-hosted-zone.png`)
- **Primary and Secondary Target Group Health:** Documented under [Disruption Simulation](../screenshots/README.md#3-disruption-simulation--regional-eviction) (`mumbai-health-failed.png`)
- **Evidence of Application Access After Failover:** Saved inside [DR Takeover Verification](../screenshots/README.md#4-disaster-recovery-takeover-verification) (`sydney-standby-takeover.png`)

*(All explicit corporate account numbers, structural resource tags, and infrastructure secret vectors have been redacted for public security compliance).*
