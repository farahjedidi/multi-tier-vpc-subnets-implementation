# [DEMO] Configuring A4L Public Subnets and Jumpbox with Multi-Tier VPC Subnets Implementation

## Overview
In this **[DEMO]**, I set up a fully functioning VPC capable of supporting internet-facing applications, with a bastion host providing a secure ingress point to manage instances in private subnets.

## Prerequisites
- An AWS account with appropriate IAM permissions for VPC management: I used the management account from my previous demos.

---

## Steps

### Step 1: Creating a Custom VPC
I started by creating a custom VPC named **a4l-vpc1-farah**

- ![Screenshot](https://imgur.com/2o11G4b.png)

### Step 2: Creating the Subnets
In this step, I manually created multiple subnets across three availability zones (AZs). In real-world scenarios, subnet creation is often automated, but I performed this step manually as part of the learning process to better understand subnet architecture and design.

I created the following subnets :

| Name            | IPv4 CIDR Block  | Availability Zone | Custom IPv6 Value |
|-----------------|------------------|-------------------|-------------------|
| `sn-reserved-A` | 10.16.0.0/20     | AZA               | 00                |
| `sn-db-A`       | 10.16.16.0/20    | AZA               | 01                |
| `sn-app-A`      | 10.16.32.0/20    | AZA               | 02                |
| `sn-web-A`      | 10.16.48.0/20    | AZA               | 03                |
| `sn-reserved-B` | 10.16.64.0/20    | AZB               | 04                |
| `sn-db-B`       | 10.16.80.0/20    | AZB               | 05                |
| `sn-app-B`      | 10.16.96.0/20    | AZB               | 06                |
| `sn-web-B`      | 10.16.112.0/20   | AZB               | 07                |
| `sn-reserved-C` | 10.16.128.0/20   | AZC               | 08                |
| `sn-db-C`       | 10.16.144.0/20   | AZC               | 09                |
| `sn-app-C`      | 10.16.160.0/20   | AZC               | 0A                |
| `sn-web-C`      | 10.16.176.0/20   | AZC               | 0B                |


- ![Screenshot](https://imgur.com/J9tfLYg.png)

- ![Screenshot](https://imgur.com/pZWs3nD.png)

- I enabled **auto-assign IPv6 addressing** for every subnet created by selecting each subnet, going to **Actions > Edit Subnet Settings**, and checking the option to auto-assign IPv6 addresses.
  
- ![Screenshot](https://imgur.com/iMovNwf.png)

### Step 3: Configuring the Web Subnets as Public
I made only the **web subnets** public. The other tiers (DB and App) remained private for security and isolation purposes. This is because only the web subnets need direct internet access to serve incoming traffic, while the app and database tiers will communicate internally within the VPC.

- I added an **Internet Gateway (IGW)** to the VPC to provide internet access to the public subnets.

- ![Screenshot](https://imgur.com/5RFCX3r.png)

- ![Screenshot](https://imgur.com/VNYcpqL.png)

- I created a new **Route Table** for the web subnets

-![Screenshot](https://imgur.com/t6WDP4M.png)

- Configured the route table with the following:
  - A route `0.0.0.0/0` for IPv4 traffic, pointing to the IGW.
  - A route `::/0` for IPv6 traffic, pointing to the IGW.
 
- ![Screenshot](https://imgur.com/bEVz2U5.png)

  - I enabled the allocation of public IPv4 addresses for all web subnets.

- ![Screenshot](https://imgur.com/G7RLR8I.png)

- Then, I associated the web subnets (`sn-web-A`, `sn-web-B`, and `sn-web-C`) with this route table.

- ![Screenshot](https://imgur.com/0MXotxU.png)

### Step 4: Creating a Bastion Host
To securely access instances inside the private subnets, I created a **bastion host** in one of the public web subnets:

1. I launched a new EC2 instance in the **sn-web-A** subnet.
   
- ![Screenshot](https://imgur.com/cxZLk8I.png)
- ![Screenshot](https://imgur.com/JL9PMKV.png)
- ![Screenshot](https://imgur.com/Dga2BGv.png)
- ![Screenshot](https://imgur.com/ZEfbcXw.png)
  
2. Connected to the EC2 Instance using EC2 Instance Connect

- ![Screenshot](https://imgur.com/MTknbLi.png)
- ![Screenshot](https://imgur.com/PGMKLaL.png)
  
_Note: While bastion hosts provide temporary ingress to private subnets, they are not considered best practice for long-term use due to security concerns. However, they are important to understand, as they can be useful in specific scenarios. Always aim to minimize jumpboxes in production environments._

---

## Cleanup
Once the demo is complete, it's important to clean up the resources created to avoid unnecessary costs. The cleanup process includes:

1. **Terminate the Bastion Host EC2 instance**
2. **Delete the Internet Gateway**
3. **Delete the Subnets and Route Table**
4. **Delete the VPC**

---

## Conclusion
By the end of this demo, I successfully implemented a multi-tier VPC with public and private subnets. The **web subnets** are now publicly accessible via the Internet Gateway, and I have established a secure bastion host to access private subnets for management purposes. This setup provides an architecture for hosting internet-facing applications while maintaining security and isolation for the application and database tiers.
