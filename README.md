# Hameed's-bank

You'll create a simple web-based banking app with the following stack:
Frontend: React / HTML-CSS-JS (Optional: hosted on S3 or in EC2)
Backend: Node.js / Python Flask / Java Spring Boot
Database: Amazon RDS (MySQL or PostgreSQL)
Hosting: AWS EC2 or AWS Elastic Beanstalk
Security: IAM, Security Groups, HTTPS (with ACM/ALB), optionally Cognito


Architecture Diagram

"""
   User
    ↓
Cloudflare (CDN, WAF, SSL)
    ↓
AWS Route 53 (DNS – optional)
    ↓
Application Load Balancer (ALB)
    ↓
EC2 or Elastic Beanstalk (Flask/Node API)
    ↓
Amazon RDS (PostgreSQL or MySQL)
"""
