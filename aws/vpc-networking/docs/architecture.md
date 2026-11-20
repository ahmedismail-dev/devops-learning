# Architecture Overview

## Network Design

The project uses a single custom VPC with two subnets, each serving a different purpose.

| Component | Purpose |
|---|---|
| VPC | Provides an isolated network environment |
| Public subnet | Hosts resources that require direct internet connectivity |
| Private subnet | Isolates resources from direct internet access |
| Internet Gateway | Provides internet connectivity to the public subnet |
| NAT Gateway | Allows private instances to initiate outbound internet connections |
| Route tables | Determine where network traffic is sent |
| Security groups | Control inbound and outbound instance traffic |

## Traffic Flow

### Public EC2 → Internet

1. The instance sends traffic to an external destination.
2. The public route table directs traffic to the Internet Gateway.
3. The Internet Gateway provides connectivity to the internet.

### Private EC2 → Internet

1. The private instance initiates an outbound connection.
2. The private route table forwards traffic to the NAT Gateway.
3. The NAT Gateway translates the private source address using its Elastic IP.
4. Traffic leaves the VPC through the Internet Gateway.
5. Return traffic is forwarded back to the private instance.

### Public EC2 → Private EC2

The public instance can connect to the private instance over SSH because the private instance's security group allows inbound SSH traffic from the public instance's security group.

The private instance does not need a public IP address for this communication.

## Important Networking Concepts

**Internet Gateway vs. NAT Gateway**

An Internet Gateway enables internet connectivity for resources with appropriate public addressing and routing.

A NAT Gateway allows instances in private subnets to initiate outbound internet connections without requiring public IP addresses.

**Public vs. Private Subnets**

A subnet is considered public or private based on its routing configuration, not simply its name.

A public subnet has a route to an Internet Gateway. A private subnet does not have a direct route to an Internet Gateway.

**Security Groups**

Security groups act as stateful virtual firewalls attached to AWS resources. They control which traffic is permitted to reach the instances.