
# Understanding Internet Gateway vs. NAT Gateway

1. Internet Gateway
An Internet Gateway (IGW) allows instances in your public subnets to:
- Access the internet.
- Be accessed from the internet (e.g., SSH from your computer to a public EC2 instance).

Key Characteristics:
- Direct connection between your VPC and the internet.
- Required for instances that need public IP addresses for external communication.
- Outbound and inbound traffic is supported.

When to Use:
- When you need instances in a public subnet to be accessible from the internet.
- Examples:
  - Hosting a public-facing web server.
  - SSHing into a public EC2 instance.

Steps to Configure:
1. Attach an Internet Gateway to your VPC:
   ```hcl
   resource "aws_internet_gateway" "example" {
     vpc_id = aws_vpc.example.id

     tags = {
       Name = "example-internet-gateway"
     }
   }
   ```
2. Add a route in the route table for the public subnet to direct traffic to the internet:
   ```hcl
   resource "aws_route_table" "public_route_table" {
     vpc_id = aws_vpc.example.id

     route {
       cidr_block = "0.0.0.0/0"
       gateway_id = aws_internet_gateway.example.id
     }

     tags = {
       Name = "PublicRouteTable"
     }
   }

   resource "aws_route_table_association" "public_association" {
     subnet_id      = aws_subnet.public.id
     route_table_id = aws_route_table.public_route_table.id
   }
   ```
3. Ensure the instance has a public IP and security group allowing inbound access (e.g., SSH):
   ```hcl
   associate_public_ip_address = true
   ```

2. NAT Gateway
A NAT Gateway allows instances in your private subnets to:
- Initiate outbound internet traffic (e.g., downloading updates or software).
- Prevent direct inbound access from the internet.

Key Characteristics:
- Provides internet access for private subnets.
- Outbound traffic is supported; inbound traffic is not allowed.
- Requires an Elastic IP (EIP).

When to Use:
- When you need instances in a private subnet to communicate with the internet without being exposed to the internet.
- Examples:
  - EC2 instances in a private subnet needing to download updates or connect to external APIs.
  - Database servers fetching security patches.

Steps to Configure:
1. Create a NAT Gateway in a public subnet:
   ```hcl
   resource "aws_nat_gateway" "example" {
     allocation_id = aws_eip.example.id
     subnet_id     = aws_subnet.public.id

     tags = {
       Name = "example-nat-gateway"
     }
   }
   ```
2. Add a route in the route table for the private subnet to direct traffic to the NAT Gateway:
   ```hcl
   resource "aws_route_table" "private_route_table" {
     vpc_id = aws_vpc.example.id

     route {
       cidr_block = "0.0.0.0/0"
       nat_gateway_id = aws_nat_gateway.example.id
     }

     tags = {
       Name = "PrivateRouteTable"
     }
   }

   resource "aws_route_table_association" "private_association" {
     subnet_id      = aws_subnet.private.id
     route_table_id = aws_route_table.private_route_table.id
   }
   ```

Key Differences and When to Use Each:
| Feature                | Internet Gateway (IGW)                          | NAT Gateway                          |
|------------------------|-------------------------------------------------|--------------------------------------|
| Purpose                | Connect public subnets to the internet          | Allow private subnets to access the internet |
| Inbound Traffic        | Allowed for instances with public IP            | Not allowed (private instances are not exposed) |
| Outbound Traffic       | Allowed for instances with public IP            | Allowed for instances in private subnets |
| Use Case               | Public-facing services, SSH access              | Secure private instances needing outbound access |
| Subnet Placement       | Attached to public subnets                      | Placed in public subnets to serve private subnets |

How to Remember?
- Internet Gateway = Public Access: Think of it as the "door" to the internet for instances that need to be accessible from the outside.
- NAT Gateway = Private Access: Think of it as the "proxy" that lets private instances talk to the internet but hides them from incoming traffic.

Your Situation:
For SSH access from your computer to the public EC2 instance:
- Use an Internet Gateway.
- Ensure:
  1. The EC2 instance is in a public subnet.
  2. The EC2 instance has a public IP.
  3. Security group allows SSH (port 22) access from your computer's IP address.

For private EC2 instances needing updates or internet access:
- Use a NAT Gateway.


```
	•	A NAT Gateway enables outbound internet access from private subnets. It does not itself allow inbound SSH from the internet into private subnets.
	•	To SSH from your local laptop into an instance via the public internet, that instance must be in a public subnet (with a route to the Internet Gateway) and must have a public IP or an Elastic IP.
```
