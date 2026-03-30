# aws-secure-web-architecture
AWS architecture for a secure web app with CloudFront, WAF, GuardDuty, EventBridge , and Lambda auto-remediation.
## Table of Content
- [Solution Overview](#solution-overview)
- [Solution Architecture Diagram](#solution-architecture-diagram)
- [Detailed Architecture Description](#detailed-architecture-description)

# Solution Overview 

This solution deploys a secure, scalable, and highly available public‑facing web application on AWS. It delivers global content distribution, intelligent threat detection, auto‑remediation, and multi‑layered security hardening.
Incoming client traffic is delivered through Amazon CloudFront, routed to an Application Load Balancer, and served by an EC2 Auto Scaling Group.
The architecture ensures:
- High scalability
- Global low‑latency delivery
- Continuous threat monitoring
- Automated remediation
- Comprehensive DDoS & application‑layer protection

This solution defends against SQL injection, XSS, bots, and volumetric threats. The system forms a fully automated security pipeline, ensuring strong defense with minimal human intervention. 

# Solution Architecture Diagram

![Architecture](./aws_arch.png)


# Detailed Architecture Description
## 1. Introduction
This architecture deploys a secure, scalable, internet‑facing web application using AWS managed services. It integrates CloudFront, ALB, EC2 Auto Scaling, AWS WAF, Shield, GuardDuty, EventBridge, Lambda, and SNS to achieve:
- Defense‑in‑depth
- Automated threat detection
- Continuous monitoring
- Real‑time remediation
- High availability and performance

## 2. High-Level Architecture Overview
Traffic is served globally through Amazon CloudFront, secured using AWS Shield and AWS WAF, then forwarded to an ALB and Auto Scaling EC2 backend to ensure availability and elasticity.
A multi‑layered security model includes:
- Edge Security: CloudFront + Shield Standard
- Application Security: WAF managed + custom rules
- Threat Detection: Amazon GuardDuty
- Automated Response: EventBridge → Lambda → WAF IPSet
- Alerting: SNS notifications

This design ensures high performance, fault tolerance, and centralized monitoring.


## 3. Architecture Components

### 3.1 Amazon CloudFront (CDN + Edge Security)
CloudFront serves as the application’s global entry point, providing:
- Low-latency content delivery from AWS edge locations
- Traffic isolation before the ALB
- Integrate with AWS Shield Standard protection
- Integrate with AWS WAF integration for Layer 7 (HTTP/S) filtering
- Support for origin failover and caching policies
CloudFront reduces load on the origin infrastructure and limits direct exposure of the ALB to the internet.

### 3.2 AWS Shield (DDoS Protection)

AWS Shield Standard automatically protects CloudFront and ALB from common Layer 3/Layer 4 DDoS attacks.
If Shield Advanced is used (optional):
- Additional detection metrics
- Cost protection
- AWS DDoS Response Team (DRT) escalation
- Deeper visibility through CloudWatch metrics

### 3.3 AWS WAF (Application Firewall)
 
AWS WAF enforces application security at the edge and at the ALB.
Rule types configured:
- AWS Managed Rules (SQLi, XSS, bot control)
- Rate-based rules (DDoS mitigation)
- Custom rules specific to application API endpoints
- Automated IP block lists populated via Lambda
WAF blocks malicious HTTP requests before they reach ALB or EC2.

### 3.4 Application Load Balancer (ALB)

The ALB routes HTTPS requests to the backend Amazon EC2 instances.
Functions include:
- SSL/TLS termination using ACM certificates
- Path-based and host-based routing
- Health checks for Auto Scaling
- Logging to Amazon S3 for audit and analytics

### 3.5 EC2 Auto Scaling Group

The web application runs on EC2 instances located in private subnets across multiple Availability Zones.
Auto Scaling provides:
- Dynamic scaling on CPU, request count, or custom CloudWatch metrics
- High availability and resilience
- Zero-downtime scaling operations
EC2 instances access the internet only through NAT gateways, ensuring controlled outbound communication.

## 4. Threat Detection & Automated Remediation

### 4.1 Amazon GuardDuty

GuardDuty continuously monitors and analyze:
- VPC Flow Logs
- DNS query logs
- CloudTrail events
- EBS anomaly activity
GuardDuty detects:
- Brute-force login attempts
- Port scanning
- Malware command-and-control traffic
- Unusual or suspicious API calls
- Traffic from known malicious IPs
Findings are categorized by severity and pushed to Amazon EventBridge.


### 4.2 Amazon EventBridge
Routes GuardDuty findings to:
EventBridge rules filter and route GuardDuty findings to Lambda (auto‑remediation function)
- Example rule:
If finding type = UnauthorizedAccess:EC2/SSHBruteForce
Trigger Lambda → Update WAF → Notify Admin Team

EventBridge enables scalable event‑driven operations.

### 4.3 AWS Lambda (Automated WAF Updater)

Lambda functions process GuardDuty events and update AWS WAF configurations.
Workflow:
1.	Lambda parses the GuardDuty finding
2.	Extracts the malicious IP or threat indicator
3.	Adds the IP to a dedicated WAF IPSet (block list)
4.	Logs the action to CloudWatch Logs
5.	Optionally sends structured logs to SIEM tools
This enables real-time automated threat mitigation.

### 4.4 Amazon SNS (Notifications)

SNS delivers notifications to:
- Security Admin team (email)
- Incident response channels (Slack, Teams via webhook)
- Operational dashboards
Notifications include:
- Finding severity
- Affected resources
- Threat source
- Automated actions taken


## 5. Defense-in-Depth Security Layers
| Layer  | Description |
|----------|----------|
| Edge Protection		| CloudFront, AWS Shield 		|
| Application Layer		| AWS WAF	    		|
| Network Segmentation		|Private subnets, NACLs, •	Security groups	|
| Threat Detection		| GuardDuty			|
| Automated Response 		| EventBridge + Lambda + WAF rules		|
| Alerting 			| SNS, CloudWatch		|


This aligns with the AWS Well‑Architected Security Pillar. 

## 6. Traffic Flow Summary

1.	User sends HTTP(S) request
2.	CloudFront inspects traffic using WAF
3.	Clean traffic is forwarded to ALB
4.	ALB routes request to EC2 Auto Scaling instances
5.	GuardDuty monitors infrastructure behavior
6.	On detection: 
o	EventBridge triggers Lambda
o	Lambda updates WAF IP sets
o	SNS sends notifications
7.	Future traffic from malicious IPs is blocked automatically


## 7. Operational Considerations

- High availability: Multi AZ deployment across backend layers ( multi‑AZ EC2 + ALB )
- Scalability: Auto Scaling, CloudFront caching, ALB elasticity
- Security: Defense in depth, automated remediation ( WAF, Shield, GuardDuty )
- Cost optimization: Caching reduces origin load and bandwidth
- Monitoring: CloudWatch, GuardDuty, log analysis

## 8. Conclusion
This architecture provides a robust, secure, and scalable platform for hosting internet-facing applications. By combining AWS edge security, WAF rulesets, threat detection services, and automated mitigation workflows, the solution minimizes risk while ensuring the application remains resilient and responsive under varying load and threat conditions.


