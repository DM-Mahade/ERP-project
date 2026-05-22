
# Welcome to SiteMate ERP

SiteMate ERP is a modern Construction Execution & Project Management ERP platform designed to centralize project operations, automate reporting workflows, and provide real-time project visibility for construction organizations.

The platform is being developed with a scalable microservices architecture using Spring Boot and React.js to support future enterprise-level expansion.

---

# 🚀 Core Features

- Project Management
- Dynamic Role & Permission Management
- Project Team Assignment
- Daily Progress Reporting (DPR)
- TAB & Bottleneck Tracking
- Task & Dependency Management
- Workflow & Approval Pipelines
- Drawing & Document Version Management
- Progress Photo Uploads
- Timeline-Based Project Updates
- Automated Report Generation
- Centralized Media Management
- Dynamic Approval Hierarchies
- Organization & External Team Collaboration

---

# 🏗️ Backend Architecture

The backend follows a microservices-based architecture using Spring Boot and Spring Cloud components.

## Core Services

- gateway-service
- iam-service
- project-service
- execution-service
- media-service
- workflow-service
- task-service
- notification-service

---

# ⚙️ Technology Stack

## Frontend
- React.js
- Tailwind CSS

## Backend
- Spring Boot
- Spring Cloud Gateway
- Spring Security
- Spring Data JPA
- Eureka Service Discovery

## Database
- MySQL

## Storage
- MinIO Object Storage

## Infrastructure
- Docker
- Docker Compose
- Nginx

## Authentication
- JWT Authentication

## API Documentation
- Swagger / OpenAPI

---

# 🧩 Architecture Highlights

- Monorepo Microservices Structure
- API Gateway Based Routing
- Eureka Service Discovery
- Centralized Swagger Aggregation
- Dynamic Project-Level RBAC
- Generic Workflow & Approval Engine
- Document Versioning & Freeze System
- Scalable Media Management
- REST-Based Inter-Service Communication
- Environment-Based Configuration (local/dev/prod)

---

# 📂 Monorepo Structure

```text
backend/

├── gateway-service/

├── iam-service/
├── project-service/
├── execution-service/
├── media-service/
├── workflow-service/
├── task-service/
├── notification-service/

├── common-libs/

├── infrastructure/

├── docs/

├── docker-compose.local.yml
├── docker-compose.dev.yml
├── docker-compose.prod.yml

└── .env
````

---

# 🌍 Environments

The platform supports:

* Local Environment
* Dev Environment
* Production Environment

Each service maintains separate environment configurations using Spring Profiles.

---

# 📦 Deployment Strategy

All services are containerized using Docker and managed using Docker Compose for simplified local, development, and production deployments.

---

# ---------------------------------------------------------

# ✨ **Credits & Ownership**

# ---------------------------------------------------------

### **🛠️ Designed & Architected By**

**👨‍💻 Vijaykumar Mehtre**
🔗 GitHub: [github.com/Vijay824669](https://github.com/Vijay824669)

**👩‍💻 Dhanashri Mahade**
🔗 GitHub: [github.com/DM-Mahade](https://github.com/DM-Mahade)

---

### **🏢 Maintained By**

**SiliconMount Tech Services Pvt. Ltd.**
🔗 GitHub Organization:
[github.com/SiliconMount-Tech-Services-Pvt-Ltd](https://github.com/SiliconMount-Tech-Services-Pvt-Ltd)

> SiteMate ERP is actively maintained, improved, and version-controlled as part of the enterprise-grade construction management solutions developed at SiliconMount Tech Services.

---

### **© 2025 — All Rights Reserved**

**SiteMate ERP — Construction Execution & Project Management Platform**

```
