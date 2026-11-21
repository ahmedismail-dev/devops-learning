# Deployment & Validation Notes

## 1. Deployment Sequence

The infrastructure was created in the following order.

| Step | Resource | Configuration |
|---|---|---|
| 1 | VPC | `10.0.0.0/16` |
| 2 | Public subnet | `10.0.0.0/24` |
| 3 | Private subnet | `10.0.1.0/24` |
| 4 | Internet Gateway | Created and attached to VPC |
| 5 | Public route table | Default route to IGW |
| 6 | Elastic IP | Allocated for NAT Gateway |
| 7 | NAT Gateway | Created in public subnet |
| 8 | Private route table | Default route to NAT Gateway |
| 9 | Security groups | Separate rules for each EC2 instance |
| 10 | EC2 instances | One public and one private |

The NAT Gateway must reach the internet through the public subnet's route to the Internet Gateway. The private route should point to the NAT Gateway only after it becomes available.

## 2. Connectivity Validation

The following tests are intended to verify the network configuration.

Record the actual outcome of each test before publishing.

| Test | Expected result | Actual result |
|---|---|---|
| SSH from my IP to public EC2 | Connection succeeds | To record |
| Public EC2 outbound internet | Connection succeeds | To record |
| SSH from public EC2 to private EC2 | Connection succeeds | To record |
| Direct internet SSH to private EC2 | No direct public access | To record |
| Private EC2 outbound internet | Connection succeeds through NAT | To record |
| Private EC2 public IPv4 | No public IPv4 assigned | To record |

## 3. Useful Troubleshooting Commands

### Check public instance internet connectivity

```bash
curl -I https://aws.amazon.com
```

### Check private instance internet connectivity

Run from the private instance:

```bash
curl -I https://aws.amazon.com
```

### Check DNS resolution

```bash
getent hosts aws.amazon.com
```

### Inspect the routing table on Linux

```bash
ip route
```

### Connect to the public EC2 instance

```bash
ssh -i ~/.ssh/project-key.pem ubuntu@PUBLIC_IP
```

### Connect to the private instance through the public instance

From your local machine, using SSH ProxyJump:

```bash
ssh -i ~/.ssh/project-key.pem \
  -J ubuntu@PUBLIC_IP \
  ubuntu@PRIVATE_IP
```

Replace the example addresses with the actual instance addresses.

This approach uses the public instance as a jump host without copying your private key onto it.

## 4. Troubleshooting Considerations

If the private instance cannot access the internet, investigate:

1. Is the NAT Gateway in the Available state?
2. Is the NAT Gateway deployed in the public subnet?
3. Does the public route table point to the Internet Gateway?
4. Does the private route table point to the NAT Gateway?
5. Is the private route table associated with the correct subnet?
6. Do the security group outbound rules permit the required traffic?
7. Are network ACLs and DNS configuration allowing the traffic?

If SSH to the private instance fails, check:

- The private instance security group references the public instance security group.
- The SSH daemon is running.
- The correct private IP and username are used.
- The SSH key or authentication configuration is correct.

## 5. Screenshots and Evidence

Store screenshots in the `screenshots/` directory.

| Screenshot | Evidence |
|---|---|
| `vpc-overview.png` | VPC configuration |
| `subnets.png` | Public and private subnet configuration |
| `route-tables.png` | Route destinations and targets |
| `internet-nat-gateway.png` | IGW and NAT Gateway |
| `security-groups.png` | Inbound security group rules |
| `ec2-instances.png` | Instance placement and IP configuration |
| `connectivity-tests.png` | Terminal connectivity tests |

Before uploading screenshots, remove or hide public IPs, account identifiers and other information you do not want to expose.

Never upload private keys, AWS credentials or `.pem` files.

## 6. Cost Management and Cleanup

NAT Gateways have hourly and data processing charges. EC2 instances, EBS volumes and public IPv4 addresses may also incur costs.

When the project is no longer needed, clean up the resources.

Recommended order:

1. Terminate the EC2 instances.
2. Delete the NAT Gateway.
3. Release the Elastic IP once it is no longer associated.
4. Detach and delete the Internet Gateway.
5. Delete custom route tables.
6. Delete the subnets.
7. Delete the VPC.

Check for remaining resources and charges after cleanup.

## 7. Future Improvements

- Automate deployment using Terraform.
- Add VPC Flow Logs.
- Introduce multiple Availability Zones.
- Use Systems Manager Session Manager.
- Add monitoring and automated connectivity checks.