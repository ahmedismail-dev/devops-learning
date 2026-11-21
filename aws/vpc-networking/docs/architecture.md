# Architecture Design

## 1. Network Overview

The infrastructure uses a single Amazon VPC with two subnets:

- A public subnet for internet-facing infrastructure.
- A private subnet for resources that should not be directly accessible from the internet.

Both subnets use the VPC's local route for internal communication.

## 2. Network Addressing

| Network | CIDR | Purpose |
|---|---|---|
| VPC | `10.0.0.0/16` | Main network |
| Public subnet | `10.0.0.0/24` | Public EC2 and NAT Gateway |
| Private subnet | `10.0.1.0/24` | Private EC2 |

The two `/24` subnets provide separate address ranges within the larger `/16` VPC.

## 3. Routing Design

Each subnet is associated with its own route table.

### Public route table

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Internal VPC traffic |
| `0.0.0.0/0` | Internet Gateway | External traffic |

The Internet Gateway provides the internet path for resources with appropriate public addressing and security rules.

### Private route table

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Internal VPC traffic |
| `0.0.0.0/0` | NAT Gateway | Outbound internet traffic |

The private subnet does not have a direct route to the Internet Gateway.

## 4. Internet Gateway

The Internet Gateway is attached to the VPC.

It enables communication between the VPC and the internet when the relevant routing, public addressing and security configuration are in place.

An Internet Gateway does not automatically make every resource inside the VPC publicly accessible.

## 5. NAT Gateway

The NAT Gateway is deployed in the public subnet and associated with an Elastic IP.

Private instances send internet-bound traffic to the NAT Gateway through their route table.

The NAT Gateway performs source network address translation and forwards traffic through the Internet Gateway.

Return traffic is translated back to the originating private instance.

The NAT Gateway provides outbound connectivity. It does not allow unsolicited inbound internet connections to private instances.

## 6. Instance Communication

### Public instance

The public EC2 instance has a public IPv4 address and uses the public route table.

SSH and HTTP inbound access are restricted to my public IP address.

### Private instance

The private EC2 instance has no public IPv4 address.

Its security group permits SSH from the public EC2 security group.

Communication between the two instances uses the VPC's local route, not the NAT Gateway.

## 7. Security Design

Security groups are attached to the EC2 instances.

| Resource | Inbound traffic | Source |
|---|---|---|
| Public EC2 | TCP 22 | My public IP |
| Public EC2 | TCP 80 | My public IP |
| Private EC2 | TCP 22 | Public EC2 security group |

Security groups are stateful, so response traffic for permitted connections is automatically allowed.

The private instance has no direct public internet entry point.

## 8. Design Limitations

This design is intended to demonstrate foundational networking concepts.

It uses one public subnet, one private subnet and a single NAT Gateway.

For a more resilient architecture, I would consider:

- Deploying public and private subnets in multiple Availability Zones.
- Adding a NAT Gateway in each Availability Zone.
- Using Systems Manager Session Manager for administration.
- Enabling VPC Flow Logs.
- Monitoring network traffic and NAT Gateway usage.