# Deployment Notes

## Deployment Order

The infrastructure was created in the following order:

1. Created a custom VPC.
2. Created public and private subnets.
3. Created and attached an Internet Gateway.
4. Allocated an Elastic IP.
5. Created a NAT Gateway in the public subnet.
6. Created public and private route tables.
7. Added the appropriate routes.
8. Associated each route table with its subnet.
9. Created security groups.
10. Launched the public and private EC2 instances.
11. Tested connectivity and network isolation.

## Connectivity Tests

| Test | Expected result |
|---|---|
| SSH from my IP to public EC2 | Allowed |
| HTTP from my IP to public EC2 | Allowed |
| SSH from public EC2 to private EC2 | Allowed |
| Direct SSH from internet to private EC2 | Blocked |
| Outbound internet from private EC2 | Allowed through NAT |

Only mark a test as successful if you actually verified it.

## Cleanup

To avoid unnecessary AWS charges, remove the resources after completing the assignment.

Recommended cleanup:

1. Terminate both EC2 instances.
2. Delete the NAT Gateway.
3. Release the Elastic IP after the NAT Gateway is deleted.
4. Detach and delete the Internet Gateway.
5. Delete the custom route tables.
6. Delete the subnets.
7. Delete the VPC.

Check that no billable resources remain in the AWS account.