# SiteMate ERP — Final Microservices Architecture Plan (Phase 1 Foundation)

Based on the BRD, DPR workflows, reporting requirements, approval workflows, dynamic RBAC requirements, and future ERP scalability, the following microservices architecture is recommended for the Construction ERP platform.

This architecture is designed for:

* Long-term ERP scalability
* Clean domain separation
* Flexible project-level workflows
* Dynamic approval hierarchies
* Role-based access control
* Independent module expansion
* Monorepo development structure

---

# 1. API Gateway Service

## Purpose

Central entry point for all frontend and external API requests.

---

## Responsibilities

* Route requests to microservices
* JWT token validation
* Request authentication
* Rate limiting
* CORS handling
* Request logging
* API aggregation (future)
* Centralized exception handling

---

## Key Features

* Secure API routing
* Public/private route separation
* Token forwarding to services
* Service discovery support (future)

---

## Notes

* All frontend requests must go through API Gateway.
* Gateway should never contain business logic.
* Recommended: Spring Cloud Gateway

---

# 2. Identity & Access Management Service (IAM Service)

## Purpose

Centralized authentication, authorization, organization users, project roles, and permissions management.

---

# Core Architecture

This service handles:

## A. Master Roles (System Roles)

Fixed system-wide roles:

| Role Code        | Description        |
| ---------------- | ------------------ |
| SUPER_ADMIN      | System Super Admin |
| ADMIN            | Organization Admin |
| PROJECT_INCHARGE | Project Incharge   |
| SITE_ENGINEER    | Site Engineer      |
| CLIENT           | Client Read Only   |

These are immutable master roles.

---

## B. Dynamic Project Roles

Each project can create custom roles based on project requirements.

Example:

* Plumbing Consultant
* Structural Reviewer
* Vendor Coordinator
* QA Inspector
* Billing Engineer

---

## Project Role Configuration

Each dynamic role contains:

* role_name
* description
* role_type

  * INTERNAL
  * EXTERNAL
* project_id
* is_active

---

## External Role Support

External users can also be assigned project roles.

Example:

* Vendor
* Client
* Consultant
* Contractor

Project creator can:

* search organization users
* search external users
* assign project-specific access

---

# Permission System

Permissions are module-based.

Example:

## Design Module Permissions

* read
* write
* upload
* comment
* approve
* freeze
* download

---

## Task Module Permissions

* create
* assign
* update
* close
* approve

---

## Report Module Permissions

* generate
* export
* approve
* share

---

# Responsibilities

## Authentication

* Login
* JWT authentication
* Refresh tokens
* Password reset

## User Management

* Organization users
* External users
* User invitations

## RBAC Management

* Master roles
* Dynamic project roles
* Permissions matrix

## Project Team Assignment

* Assign users to projects
* Assign custom project roles

---

# Notes

* Permissions must be fully dynamic.
* Avoid hardcoding module permissions.
* Every module action should be permission-driven.
* Future modules should automatically integrate into permission engine.

---

# 3. Project Management Service

## Purpose

Handles all core project configuration and project structure management.

---

# Responsibilities

## Project Management

* Create projects
* Update project details
* Archive projects

## Project Structure

* Towers
* Zones
* Floors
* Areas
* Work packages

## Dynamic Masters

Project-specific:

* Design types
* Activity types
* Categories
* Tags

---

# Important Design Requirement

Design types must NOT be predefined.

Users should create custom design categories per project.

Example:

* Plumbing Design
* Fire Fighting Layout
* Architectural Layout
* Elevation Drawings
* Electrical Shop Drawings

---

# Responsibilities

## Project Configuration

* Project metadata
* Timelines
* Settings

## Team Visibility

* Assigned users
* Active project roles

---

# Notes

* This service acts as the root business domain.
* Most services will reference project_id from here.
* Keep project hierarchy flexible.

---

# 4. Execution & Reporting Service

## Purpose

Core operational service for site execution tracking and automated reporting.

This is the heart of the ERP execution workflow.

---

# Responsibilities

## Daily Progress Updates

* Daily activity updates
* Quantity tracking
* Work remarks
* Tomorrow planning

## Daily Site Reporting

* DPR generation
* TAB generation
* Labour reports
* Manpower reports

## Bottleneck Tracking

* Delays
* Site blockers
* Dependency issues

## Machinery Tracking

* Machinery usage
* Utilization logs

## Timeline Feed

* Daily project feed
* Progress updates
* Execution history

---

# Photo-linked Progress Updates

Users can:

* upload progress photos
* attach photos to activities
* add captions
* add location references

---

# Automated Reports

Generate:

* DPR
* TAB
* Labour Reports
* Manpower Reports

Reports should be:

* PDF exportable
* Filterable
* Date-wise searchable

---

# Notes

* High-write traffic service.
* Optimize for frequent updates.
* Reports should generate from centralized activity data.
* Avoid duplicate report entries.

---

# 5. Task & Work Management Service

## Purpose

Handles project task assignment, dependency tracking, and execution workflows.

---

# Responsibilities

## Task Management

* Create task
* Assign task
* Set due dates
* Track status

## Dependency Management

* Blocked by
* Depends on
* Sequential execution

## Task Collaboration

* Comments
* Attachments
* Activity logs

## Project Workflows

* Work assignment tracking
* Escalation support

---

# Features

## Task Statuses

* TODO
* IN_PROGRESS
* BLOCKED
* COMPLETED
* CANCELLED

## Priority Levels

* LOW
* MEDIUM
* HIGH
* URGENT

---

# Notes

* Task engine should be generic and reusable.
* Future modules can create linked tasks automatically.

---

# 6. Media & Document Management Service

## Purpose

Centralized file storage, media management, document versioning, and design document workflows.

---

# Responsibilities

## File Uploads

* Progress photos
* Site documents
* Drawings
* Attachments

## Document Management

* Organize documents by project
* Group by design type

## Version Control

* Upload revisions
* Maintain version history
* Active version tracking

---

# Design Structure Model

```text
Project
 └── Design Type
      └── Document
           └── Versions
```

---

# Example

```text
Project
 └── Plumbing Design
      └── Basement Layout
           ├── V1
           ├── V2
           └── V3
```

---

# Document Lifecycle

Each version can have:

* DRAFT
* UNDER_REVIEW
* APPROVED
* REJECTED
* FROZEN

---

# Freeze Workflow

When a version is frozen:

* editing disabled
* immutable snapshot created
* only new revision allowed

This is critical for real construction workflows.

---

# Comment System

Users can:

* add review comments
* suggest changes
* reply to comments
* mark resolved

---

# Notes

* Files should NOT be stored inside business services.
* Use MinIO/S3-compatible object storage.
* Store metadata in DB.
* Store physical files separately.

---

# 7. Workflow & Approval Engine Service

## Purpose

Dynamic approval pipeline engine for the entire ERP.

This is one of the most important enterprise modules.

---

# Main Requirement

Users should be able to create custom approval pipelines at project level.

---

# Supported Modules

Example approval pipelines:

* Drawings
* Expenses
* Purchase Orders
* Invoices
* Reports
* Material Requests
* Change Requests

---

# Dynamic Approval Pipeline System

Users can configure:

* pipeline name
* module
* approval hierarchy
* step sequence

---

# Two Approval Assignment Modes

## A. Role-based Approval

Example:

```text
Step 1 → SITE_ENGINEER
Step 2 → PROJECT_INCHARGE
Step 3 → ADMIN
```

In this mode:

* approvers are selected dynamically by role

---

## B. Direct User-based Approval

Example:

```text
Step 1 → Rahul Sharma
Step 2 → Amit Patil
Step 3 → Vijay Mehtre
```

In this mode:

* specific users are assigned directly

---

# Pipeline Creation Logic

If pipeline mode is:

* ROLE_BASED
  → search only project roles

If pipeline mode is:

* USER_BASED
  → search project teammates directly

---

# Workflow Features

## Approval Actions

* approve
* reject
* send back
* comment
* escalate

## Pipeline Features

* sequential approvals
* parallel approvals (future)
* SLA support (future)
* escalation workflows (future)

---

# Notes

* Build this as a generic reusable engine.
* All future ERP approvals should use this service.
* Avoid hardcoding module-specific approval logic.

---

# 8. Notification Service

## Purpose

Centralized communication and notification engine.

---

# Responsibilities

## In-App Notifications

* task assigned
* approval pending
* report shared
* design approved

## Email Notifications

* approval requests
* reminders
* escalation alerts

## Future Expansion

* WhatsApp integration
* Push notifications
* SMS alerts

---

# Notes

* Should eventually become event-driven.
* Can later consume Kafka/RabbitMQ events.

---

# Recommended Monorepo Structure

```text
ERP-backend/

│
├── gateway-service/
│
├── iam-service/
├── project-service/
├── execution-service/
├── media-service/
├── workflow-service/
├── task-service/
├── notification-service/
│
├── common-libs/
│
│   ├── common-lib/
│   ├── security-lib/
│   ├── jwt-lib/
│   ├── response-lib/
│   ├── event-lib/
│   └── file-lib/
│
├── infrastructure/
│
│   ├── mysql/
│   ├── redis/
│   ├── minio/
│   ├── nginx/
│   └── scripts/
│
├── docs/
│
├── docker-compose.local.yml
├── docker-compose.dev.yml
├── docker-compose.prod.yml
│
└── .env
```

---

# Database Strategy

## Recommended Approach

### One MySQL Server

### One Database

```text
smts_erp
```

All service tables will exist inside the same database.

Example:

```text
iam_users
iam_roles
iam_permissions

project_projects
project_members
project_structures

execution_daily_reports
execution_activities
execution_bottlenecks

media_documents
media_document_versions

workflow_pipelines
workflow_pipeline_steps

task_tasks
task_dependencies

notification_notifications
```

---

# Important Database Rule

Even though all services use the same database:

## Each service must own and manage only its own tables.

Example:

* IAM Service
  → only manages `iam_*` tables

* Project Service
  → only manages `project_*` tables

* Workflow Service
  → only manages `workflow_*` tables

Avoid direct cross-service table modifications.
 allow only when its necessary

---

# Service Discovery Strategy

## Eureka Server Will Be Used

Purpose:

* automatic service registration
* dynamic internal service discovery
* load balancing support
* cleaner gateway routing

---

# Eureka Flow

```text
Gateway Service
       ↓
Eureka Discovery Server
       ↓
------------------------------------------------
|         |           |          |             |
IAM    Project    Execution   Workflow      Media
Svc      Svc         Svc         Svc          Svc
```

---

# Gateway Responsibilities

The gateway-service will handle:

* request routing
* JWT validation
* centralized API access
* Swagger aggregation
* CORS handling
* request filtering
* service forwarding

Frontend will communicate only with:

```text
gateway-service
```

---

# Swagger Documentation Strategy

Swagger aggregation will be handled inside the gateway-service.

Each microservice exposes:

```text
/v3/api-docs
```

Gateway aggregates all swagger documents into one unified Swagger UI.

Example:

```text
http://localhost/swagger-ui.html
```

This will contain:

* IAM APIs
* Project APIs
* Workflow APIs
* Media APIs
* Execution APIs

all in one place.

---

# Recommended Communication Strategy

## Initial Phase

### REST APIs Between Services

Simple and easier for development/debugging.

---

## Future Phase

Can later introduce:

* Kafka
* RabbitMQ
* Event-driven communication

only when system complexity increases.

---

# Important Architectural Notes

## 1. Use UUIDs Everywhere

Avoid auto-increment IDs across services.

Recommended:

* UUID v4
* ULID (better option for sorting)

---

## 2. Keep Files Outside Database

Store:

* file metadata in MySQL
* actual files in MinIO object storage

---

## 3. Keep Permissions Fully Dynamic

Never hardcode:

* module permissions
* actions
* approval rights

Everything should be configurable.

---

## 4. Approval Engine Must Be Generic

Workflow service should support:

* drawings
* expenses
* invoices
* reports
* future ERP modules

without module-specific approval logic.

---

## 5. Media Service Must Remain Independent

Files and transactional data scale differently.

Media service should independently manage:

* uploads
* document versions
* image storage
* preview generation
* freeze snapshots

---

## 6. Project Roles Must Be Dynamic

System supports:

### Master Roles

* SUPER_ADMIN
* ADMIN
* PROJECT_INCHARGE
* SITE_ENGINEER
* CLIENT

AND

### Custom Project Roles

created dynamically per project.

Example:

* Plumbing Reviewer
* QA Engineer
* Vendor Coordinator
* External Consultant

---

## 7. Approval Pipelines Must Be Flexible

Workflow engine supports:

### Role-Based Approvals

Example:

```text
SITE_ENGINEER
   ↓
PROJECT_INCHARGE
   ↓
ADMIN
```

OR

### Direct User-Based Approvals

Example:

```text
Rahul Sharma
   ↓
Amit Patil
   ↓
Vijay Mehtre
```

Both approaches should work dynamically.

---

# Environment Strategy

Three environments will be maintained:

## Local

Developer local machine.

## Dev

Remote development/testing server.

## Prod

Production server.

---

# Environment Configuration Strategy

Each service should contain:

```text
application-local.yml
application-dev.yml
application-prod.yml
```

Profile activation example:

```bash
SPRING_PROFILES_ACTIVE=dev
```

---

# Docker Strategy

All services will run through Docker Compose.

Files:

```text
docker-compose.local.yml
docker-compose.dev.yml
docker-compose.prod.yml
```

This enables:

* environment isolation
* easy deployments
* easier testing
* centralized infrastructure management

---

# Final Recommended Phase 1 Services

## Core Mandatory Services

1. gateway-service
2. iam-service
3. project-service
4. execution-service
5. media-service
6. workflow-service

---

## Secondary Services

7. task-service
8. notification-service

These can initially remain lightweight and later scale independently.

---
