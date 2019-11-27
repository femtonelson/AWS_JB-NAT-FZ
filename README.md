# AWS Tutorials
## Tutorial to setup a Jumpbox (for management) + NAT instance + private machine in AWS VPC
[Inspired from https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html]

# Step 0 : Objective
- Setup a simple AWS VPC with 2 instances in a public subnet (reachable from internet) and an EC2 instance in a private subnet, not directly accessible to the internet, its requests will be forwarded through the NAT instance.
- An EC2 instance in the public subnet called "Jumpbox", with a public IP and will serve for managing remotely all instances in the VPC
- The other instance in the public subnet called "NAT instance", with a public IP, will perform NAT translations.

# Step 1 : Create a New VPC, Internet Gateway and 2 subnets 
- Create a new VPC (Virtual Private Cloud), named "New_VPC" in the IPV4 address range : 10.0.0.0/16.
- Create a new Internet Gateway "IGW" and attach to New_VPC.
- Create a public subnet inside New_VPC, named "Public subnet" in the IPv4 address range : 10.0.4.0/24.
- Create a private subnet inside New_VPC, named "Private subnet" in the IPv4 address range : 10.0.6.0/23.

# Step 2 : Create 03 new Security Groups attached to New_VPC
- Create a new SG "Private SG": In Inbound rules set [All traffic -> All protocols -> All ports -> 10.0.0.0/16] to enable all incoming traffic from within the VPC.
  In outbound rules set [All traffic -> All protocols -> All ports -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.
- Create a new SG "Jumpbox SG": In Inbound rules set [SSH -> TCP -> 22 -> 0.0.0.0/0], [All ICMP - IPv4 -> All -> N/A -> 0.0.0.0/0] to enable incoming SSH/Pings from internet/VPC.
  In outbound rules set [All traffic -> All protocols -> All ports -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.
- Create a new SG "NAT SG": In Inbound rules set [All traffic -> All protocols -> All ports -> 10.0.0.0/16] to enable all incoming traffic from within the VPC. Add a second inbound rule [All ICMP - IPv4 -> All -> N/A -> 0.0.0.0/0] to respond to ping requests from the internet.
  In outbound rules set [All traffic -> All -> All -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.

# Step 3 : Launch 03 new instances or machines
- Launch a new EC2 instance to be named "Jumpbox" based on Linux 2 AMI -> Instance Type (t2.micro) -> Associate it to New_VPC and Public subnet.
- Launch a new EC2 instance to be named "FZ Machine" based on Linux 2 AMI -> Instance Type (t2.micro) -> Associate it to New_VPC and Private subnet.
- Launch a new instance to be named "NAT Instance" with suitable NAT AMI (Amazon Machine Image) from Community AMIs "amzn-ami-vpc-nat" -> Instance Type (t2.micro) -> Associate it to New_VPC and Public subnet.
- For this particular exercise, auto-assigned IPs of the 03 machines were as follows :
  Jumpbox (10.0.4.12), NAT Instance (10.0.4.124), FZ Machine (10.0.6.92)
  
# Step 4 : Create 02 Route Tables : A public route table and a private route table attached to New_VPC
- Create Route Table named "Public RT". Under Public RT -> Routes, ensure there are 2 entries : [Destination - 10.0.0.0/16   ->   Target - Local] for routing within VPC and [Destination - 0.0.0.0/0   ->   Target - IGW] for routing connections outside VPC, to the internet.
-and under Public RT -> Subnet associations, associate to previously created Public subnet.
- Create Route Table named "Private RT". Under Private RT -> Routes, ensure there are 2 entries : [Destination - 10.0.0.0/16   ->   Target - Local] for routing within VPC and [Destination - 0.0.0.0/0   ->   Target - NAT Instance] for routing connections to the internet, through NAT instance to be configured subsequently.

# Step 5 : 
