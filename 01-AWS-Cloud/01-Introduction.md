---
tags: [aws, cloud, fundamentals, saa-c03]
topic: "Introduction to AWS & Global Infrastructure"
date: 2026-11-24
---

# 01 - Introduction to AWS & Global Infrastructure

## What is AWS?
- **AWS (Amazon Web Services):** Leading Cloud Service Provider.
- **Core Value Proposition:** Provides on-demand servers, storage, networking, and managed services that can scale easily based on workload demand.
- **Pay-as-you-go Pricing:** You only pay for the resources you consume, with no upfront investment in physical hardware (*CapEx* $\rightarrow$ *OpEx*).

> **Real-World Use Case (Netflix):** Its entire streaming platform runs on AWS, enabling automatic scaling for millions of concurrent users worldwide without relying on a single server or on-premises data center.
> **Other Real-World Examples:** Dropbox, Airbnb, NASA, McDonald's, Activision, and FOX rely on AWS cloud services for global operations.

---

## AWS Global Infrastructure
To ensure low latency and high availability, AWS organizes its physical infrastructure across four key components (interactive map available at [infrastructure.aws](https://infrastructure.aws)):

1. **Regions:**
   - Clusters of data centers located in distinct geographical areas worldwide (e.g., `us-east-1` in N. Virginia, `eu-west-1` in Ireland, `eu-south-2` in Spain).
   - **Key Rule:** Most AWS services are **Region-scoped**, though a few are **Global** (such as **IAM**, **Route 53**, **CloudFront**, and **WAF**).
   - **How to choose a Region:** Based on compliance/data governance laws, proximity to end users (latency), available AWS services, and pricing.

2. **Availability Zones (AZs):**
   - Each Region has multiple isolated AZs (minimum 3 (usual), maximum 6; e.g., `us-east-1a`, `us-east-1b`, `us-east-1c`).
   - Each AZ consists of one or more discrete data centers with redundant power, networking, and connectivity, linked together via ultra-low-latency fiber-optic networks.
   - If one AZ experiences an outage (flood, power failure), the remaining AZs in the same Region continue operating without service disruption.

3. **Data Centers:**
   - The physical facilities housing the actual servers, storage, and networking hardware that make up each Availability Zone.
   - Completely invisible to customers and strictly secured by AWS (biometric access, 24/7 surveillance, undisclosed locations).

4. **Edge Locations (Points of Presence - PoPs):**
   - Over 400 locations distributed globally to deliver cached content to end users with the lowest possible latency (used by **Amazon CloudFront**).

---

## Shared Responsibility Model
| Responsibility | Managed By | Examples |
| :--- | :--- | :--- |
| **Security OF the Cloud** | **AWS** | Physical security of data centers, hardware, global network, virtualization, and underlying infrastructure. |
| **Security IN the Cloud** | **Customer (You)** | IAM configuration, data encryption, firewall rules (*Security Groups*), OS patching on EC2 instances. |

---

## Common AWS Cloud Use Cases
- **Sophisticated, Scalable Applications:** Deployed across a diverse set of industries (fintech, healthcare, media).
- **Enterprise IT, Backup & Storage:** Disaster recovery, archiving, and hybrid cloud extensions.
- **Big Data Analytics:** Processing massive datasets at scale without managing physical clusters.
- **Website Hosting, Mobile & Social Apps:** High-traffic backends with automatic elasticity.
- **Gaming:** Entire multiplayer game servers and matchmaking backends running globally in the cloud.
	

