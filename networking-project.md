# High Availability Web Infrastructure with SSL and Reverse Proxy on AWS

## Project Overview
This project sets up a **high availability web infrastructure** with SSL encryption, utilizing a **reverse proxy server as a load balancer** to distribute traffic and isolate backend infrastructure for improved security. The entire system is deployed on **AWS** and demonstrates practical cloud networking and security principles.

---

## Architecture Diagram
Designing a detailed architecture diagram allowed me to blueprint the infrastructure, making deployment more precise and efficient.  


---

## Project Breakdown

### 1. Domain and DNS Setup
- Purchased domain on **Cloudflare**.  
- Created two subdomains with DNS A records pointing to the public IPs of instances:  
  - **Reverse Proxy:** Handles traffic load balancing  
  - **Status Page:** Displays backend server health status  

---

### 2. Infrastructure Setup

#### 2.1 VPC and Subnets
VPC contains **4 subnets across two Availability Zones (AZs):**

| Subnet Type     | AZ   | Purpose        |
|-----------------|------|----------------|
| Public Subnet   | AZ-1 | Reverse Proxy  |
| Public Subnet   | AZ-2 | Status Page    |
| Private Subnet  | AZ-1 | Backend-1      |
| Private Subnet  | AZ-2 | Backend-2      |

> This design isolates backend servers, exposing only the reverse proxy and status page publicly.

#### 2.2 Route Tables
- **Public Route Table:**  
  - Destination: `0.0.0.0/0`  
  - Target: Internet Gateway (IGW)  
  - Enables inbound public access for reverse proxy and status page.  

- **Private Route Table:**  
  - Routes **local VPC traffic internally**  
  - No internet access, keeping backend servers isolated.  

#### 2.3 Security Groups
Separate security groups control traffic:

| Security Group        | Inbound Rules                                    | Outbound Rules                   |
|----------------------|-------------------------------------------------|---------------------------------|
| Reverse Proxy SG      | HTTP/HTTPS from anywhere                        | To backend servers               |
| Status Page SG        | HTTP from anywhere                              | As needed                        |
| Backend-1 SG          | HTTP only from Reverse Proxy SG                 | -                               |
| Backend-2 SG          | HTTP only from Reverse Proxy SG                 | -                               |

---

### 3. Instances Deployment
- **Backend-1 (Private Subnet, AZ-1):** Hosts a static webpage confirming server status  
- **Backend-1 Replica (Private Subnet, AZ-2):** High availability replica of Backend-1  
- **Backend-2 (Private Subnet, AZ-1):** Hosts a distinctive static page  
- **Backend-2 Replica (Private Subnet, AZ-2):** High availability replica of Backend-2  
- **Reverse Proxy (Public Subnet, AZ-1):** Load balances requests between backend servers using NGINX  

---

### 4. Reverse Proxy and Load Balancer Configuration
- **NGINX** installed and configured on the reverse proxy instance  
- **Load balancing:** Round-robin across backend servers for even traffic distribution  
- **SSL encryption:** Implemented with **Let’s Encrypt** using Certbot, securing communication on public subnets  

```bash
sudo yum install nginx -y
sudo certbot --nginx -d app.tm-an-awale.com
