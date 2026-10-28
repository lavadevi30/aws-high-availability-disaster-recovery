# Implementation Notes

## Environment

- Primary region: Mumbai (`ap-south-1`)
- Secondary region: Sydney (`ap-southeast-2`)
- Compute: Amazon EC2
- Load balancing: one Application Load Balancer and target group per region
- DNS: Amazon Route 53 failover records using the same application DNS name
- Configuration method: AWS Management Console

## Record Your Actual Configuration

Fill in details that you can safely share:

- Application type:
- Primary ALB / target group setup:
- Secondary ALB / target group setup:
- Health-check endpoint and settings:
- Route 53 record types and failover roles:
- Relevant listener ports:
- Any differences between primary and secondary:


