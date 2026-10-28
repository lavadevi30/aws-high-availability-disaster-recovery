# Failover Testing

The failover configuration was tested. Fill in the observations below from your actual test; avoid estimating results from memory if you did not record them.

## Test Record

- Test date:
- Primary endpoint:
- Secondary endpoint:
- How the primary endpoint was made unhealthy:
- Health-check detection time:
- Time until the application was reachable through the secondary endpoint:
- How DNS/application availability was verified:
- Observed result:
- Limitations or unexpected behaviour:

## Validation Checklist

- [ ] Primary endpoint serves the application during normal operation.
- [ ] Primary health check changes to unhealthy during the test.
- [ ] Route 53 returns the secondary endpoint for eligible DNS queries.
- [ ] Secondary ALB has healthy targets.
- [ ] Application responds successfully through the secondary endpoint.
- [ ] Primary service can be restored and revalidated.

Only check items that you personally verified. DNS caching and health-check intervals can affect observed failover time.

## Screenshots

Add relevant screenshots to `../screenshots/` and link them here, for example:

- Route 53 failover configuration
- Primary and secondary target group health
- Evidence of application access after failover

Redact account IDs or other information you do not want public.
