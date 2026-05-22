# SiteMate ERP — Database Schema Architecture Document

## 1. Introduction

SiteMate ERP is an enterprise-grade Construction Execution & Project Management platform designed to centralize project operations, automate reporting workflows, manage project execution activities, and provide real-time project visibility across organizations.

The platform follows a scalable microservices-based architecture using Spring Boot, React.js, MySQL, Docker, and Eureka Service Discovery.

This document defines the complete database schema architecture for the SiteMate ERP platform.

The database design is structured service-wise, where each microservice manages its own logical set of tables inside a centralized MySQL database.

---

# 2. Database Information

## Database Name

```sql
smts_erp
````

---

## Database Creation Query

```sql
CREATE DATABASE smts_erp
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

---

# 3. Database Architecture Strategy

The platform follows:

* Centralized MySQL Database Architecture
* Service-wise Logical Table Separation
* Normalized Relational Database Design
* Foreign Key-Based Relationships
* Indexed Query Optimization
* Audit & Tracking Columns
* Soft Activation Handling
* Cascade Deletion Management

---

# 4. Microservices Database Structure

The database schema is organized service-wise.

Each microservice manages only its own related tables while sharing the same database instance.

---

## Services Covered

### Core Services

* IAM Service
* Project Service
* Execution Service
* Workflow Service
* Media Service

### Secondary Services

* Task Service
* Notification Service

---

# 5. Table Naming Convention

Each service will maintain prefixed table names for logical separation.

Example:

```text
iam_users
iam_roles
iam_permissions

project_projects
project_members

execution_daily_reports
execution_activities

workflow_pipelines
workflow_pipeline_steps

media_documents
media_document_versions

task_tasks
task_dependencies

notification_notifications
```

This approach improves:

* maintainability
* readability
* service ownership clarity
* future scalability

---

# 6. Database Design Standards

The following standards will be followed throughout the schema design.

---

## 6.1 Normalization

The database will follow normalized relational design principles to reduce:

* duplicate data
* inconsistent records
* update anomalies

The schema will primarily follow:

* 1NF
* 2NF
* 3NF

where applicable.

---

## 6.2 Primary Keys

All tables will use:

* UUID-based primary keys

Example:

```sql
id CHAR(36) PRIMARY KEY
```

UUIDs improve:

* distributed scalability
* microservice compatibility
* future migration flexibility

---

## 6.3 Foreign Keys

Foreign keys will be used wherever relational integrity is required.

Example:

```sql
FOREIGN KEY (project_id)
REFERENCES project_projects(id)
```

This ensures:

* data consistency
* relationship validation
* referential integrity

---

## 6.4 Indexing Strategy

Indexes will be added on:

* foreign keys
* frequently searched columns
* filter columns
* unique identifiers
* status columns
* date fields

Example:

```sql
INDEX idx_project_id (project_id)
INDEX idx_status (status)
INDEX idx_created_at (created_at)
```

This improves:

* query performance
* report generation speed
* filtering efficiency
* dashboard loading speed

---
---

# Database Schemas

---

# ---------------------------------------------------------

# 🟦 **1. AUTHENTICATION & USER MANAGEMENT SERVICE (IAM SERVICE)**

# ---------------------------------------------------------

This service manages:

- system users
- organizations
- master roles
- predefined project roles
- custom project roles
- permissions
- authentication
- authorization
- project team assignments
- OTP verification
- role-based access control (RBAC)

The IAM Service acts as the centralized security and access management layer for the entire SiteMate ERP platform.

---

## **1.1 Table: `iam_organizations`**

| Column Name       | Type         | Constraints / Notes                      |
| ----------------- | ------------ | ---------------------------------------- |
| id                | CHAR(36)     | PK (UUID)                                |
| organization_name | VARCHAR(255) | NOT NULL                                 |
| organization_code | VARCHAR(100) | UNIQUE, NOT NULL                         |
| email             | VARCHAR(255) | NULL                                     |
| mobile            | VARCHAR(20)  | NULL                                     |
| address           | TEXT         | NULL                                     |
| logo_url          | VARCHAR(500) | NULL                                     |
| is_active         | TINYINT(1)   | Default 1                                |
| created_by        | CHAR(36)     | NULL                                     |
| created_at        | DATETIME     | Default CURRENT_TIMESTAMP                |
| updated_at        | DATETIME     | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_organizations (
    id CHAR(36) PRIMARY KEY,
    organization_name VARCHAR(255) NOT NULL,
    organization_code VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255),
    mobile VARCHAR(20),
    address TEXT,
    logo_url VARCHAR(500),
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

- Stores organization/company details.
- Supports multi-organization architecture.

---

## **1.2 Table: `iam_users`**

| Column Name     | Type          | Constraints / Notes                      |
| ----------------| ------------- | ---------------------------------------- |
| id              | CHAR(36)      | PK (UUID)                                |
| organization_id | CHAR(36)      | FK → iam_organizations(id)               |
| fullname        | VARCHAR(200)  | NOT NULL                                 |
| username        | VARCHAR(100)  | UNIQUE, NOT NULL                         |
| email           | VARCHAR(255)  | UNIQUE, NOT NULL                         |
| mobile          | VARCHAR(20)   | NULL                                     |
| password_hash   | VARCHAR(255)  | Hashed password                          |
| profile_photo   | VARCHAR(500)  | NULL                                     |
| otp_verified    | TINYINT(1)    | Default 0                                |
| is_active       | TINYINT(1)    | Default 1                                |
| last_login_at   | DATETIME      | NULL                                     |
| created_by      | CHAR(36)      | NULL                                     |
| created_at      | DATETIME      | Default CURRENT_TIMESTAMP                |
| updated_at      | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_users (
    id CHAR(36) PRIMARY KEY,
    organization_id CHAR(36) NOT NULL,
    fullname VARCHAR(200) NOT NULL,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    mobile VARCHAR(20),
    password_hash VARCHAR(255),
    profile_photo VARCHAR(500),
    otp_verified TINYINT(1) DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    last_login_at DATETIME NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_iam_users_organization
    FOREIGN KEY (organization_id)
    REFERENCES iam_organizations(id),

    INDEX idx_organization_id (organization_id),
    INDEX idx_email (email),
    INDEX idx_username (username)
);
```

### Notes

- Stores all platform users.
- OTP verification status is tracked using `otp_verified`.
- Passwords must always be stored in hashed format.
- Used for JWT authentication and authorization.

---

## **1.3 Table: `iam_master_roles`**

| Column Name | Type         | Constraints / Notes              |
| ------------| ------------ | -------------------------------- |
| id          | CHAR(36)     | PK (UUID)                        |
| role_code   | VARCHAR(100) | UNIQUE, NOT NULL                 |
| role_name   | VARCHAR(150) | NOT NULL                         |
| description | TEXT         | NULL                             |
| is_active   | TINYINT(1)   | Default 1                        |
| created_at  | DATETIME     | Default CURRENT_TIMESTAMP        |
| updated_at  | DATETIME     | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_master_roles (
    id CHAR(36) PRIMARY KEY,
    role_code VARCHAR(100) NOT NULL UNIQUE,
    role_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

Fixed system-level roles.

Examples:
- SUPER_ADMIN
- ADMIN
- PROJECT_INCHARGE
- SITE_ENGINEER
- CLIENT

---

## **1.4 Table: `iam_predefined_project_roles`**

| Column Name | Type         | Constraints / Notes                      |
| ------------| ------------ | ---------------------------------------- |
| id          | CHAR(36)     | PK (UUID)                                |
| role_name   | VARCHAR(150) | NOT NULL                                 |
| description | TEXT         | NULL                                     |
| is_active   | TINYINT(1)   | Default 1                                |
| created_by  | CHAR(36)     | FK → iam_users(id)                       |
| created_at  | DATETIME     | Default CURRENT_TIMESTAMP                |
| updated_at  | DATETIME     | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_predefined_project_roles (
    id CHAR(36) PRIMARY KEY,
    role_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_created_by (created_by)
);
```

### Notes

Stores reusable predefined project roles.

Examples:
- Planning Engineer
- QA Engineer
- Vendor Coordinator
- Consultant Reviewer

These roles can later be selected during project role creation.

---

## **1.5 Table: `iam_project_roles`**

| Column Name    | Type          | Constraints / Notes                      |
| -------------- | ------------- | ---------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                                |
| project_id     | CHAR(36)      | FK → project_projects(id)                |
| role_name      | VARCHAR(150)  | NOT NULL                                 |
| description    | TEXT          | NULL                                     |
| role_type      | ENUM          | PREDEFINED / CUSTOM                      |
| predefined_role_id | CHAR(36)  | FK → iam_predefined_project_roles(id)    |
| is_active      | TINYINT(1)    | Default 1                                |
| created_by     | CHAR(36)      | FK → iam_users(id)                       |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP                |
| updated_at     | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_project_roles (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    role_name VARCHAR(150) NOT NULL,
    description TEXT,
    role_type ENUM('PREDEFINED','CUSTOM') NOT NULL,
    predefined_role_id CHAR(36),
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_predefined_project_role
    FOREIGN KEY (predefined_role_id)
    REFERENCES iam_predefined_project_roles(id),

    INDEX idx_project_id (project_id),
    INDEX idx_created_by (created_by)
);
```

### Notes

Supports:
- predefined project roles
- fully custom project roles

Role creation remains fully dynamic per project.

---

## **1.6 Table: `iam_modules`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| module_code | VARCHAR(100)  | UNIQUE, NOT NULL                  |
| module_name | VARCHAR(150)  | NOT NULL                          |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_modules (
    id CHAR(36) PRIMARY KEY,
    module_code VARCHAR(100) NOT NULL UNIQUE,
    module_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

Stores all ERP modules.

Examples:
- PROJECTS
- DESIGNS
- REPORTS
- WORKFLOW
- EXECUTION

---

## **1.7 Table: `iam_permissions`**

| Column Name     | Type          | Constraints / Notes                     |
| ----------------| ------------- | --------------------------------------- |
| id              | CHAR(36)      | PK (UUID)                               |
| module_id       | CHAR(36)      | FK → iam_modules(id)                    |
| permission_code | VARCHAR(150)  | UNIQUE, NOT NULL                        |
| permission_name | VARCHAR(150)  | NOT NULL                                |
| description     | TEXT          | NULL                                    |
| is_active       | TINYINT(1)    | Default 1                               |
| created_at      | DATETIME      | Default CURRENT_TIMESTAMP               |
| updated_at      | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE iam_permissions (
    id CHAR(36) PRIMARY KEY,
    module_id CHAR(36) NOT NULL,
    permission_code VARCHAR(150) NOT NULL UNIQUE,
    permission_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_permission_module
    FOREIGN KEY (module_id)
    REFERENCES iam_modules(id)
    ON DELETE CASCADE,

    INDEX idx_module_id (module_id)
);
```

### Notes

Examples:
- DESIGN_READ
- DESIGN_WRITE
- DESIGN_APPROVE
- REPORT_EXPORT
- TASK_ASSIGN

Permissions remain fully dynamic.

---

## **1.8 Table: `iam_role_permissions`**

| Column Name   | Type      | Constraints / Notes                 |
| ------------- | --------- | ----------------------------------- |
| id            | CHAR(36)  | PK (UUID)                           |
| role_id       | CHAR(36)  | FK → iam_project_roles(id)          |
| permission_id | CHAR(36)  | FK → iam_permissions(id)            |
| created_by    | CHAR(36)  | FK → iam_users(id)                  |
| created_at    | DATETIME  | Default CURRENT_TIMESTAMP           |

### Table Creation Query

```sql
CREATE TABLE iam_role_permissions (
    id CHAR(36) PRIMARY KEY,
    role_id CHAR(36) NOT NULL,
    permission_id CHAR(36) NOT NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_role_permissions_role
    FOREIGN KEY (role_id)
    REFERENCES iam_project_roles(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_role_permissions_permission
    FOREIGN KEY (permission_id)
    REFERENCES iam_permissions(id)
    ON DELETE CASCADE,

    INDEX idx_role_id (role_id),
    INDEX idx_permission_id (permission_id)
);
```

### Notes

Maps permissions dynamically to project roles.

---

## **1.9 Table: `iam_project_members`**

| Column Name | Type       | Constraints / Notes                  |
| ------------| ---------- | ------------------------------------ |
| id          | CHAR(36)   | PK (UUID)                            |
| project_id  | CHAR(36)   | FK → project_projects(id)            |
| user_id     | CHAR(36)   | FK → iam_users(id)                   |
| role_id     | CHAR(36)   | FK → iam_project_roles(id)           |
| is_external | TINYINT(1) | Default 0                            |
| joined_at   | DATETIME   | Default CURRENT_TIMESTAMP            |
| created_by  | CHAR(36)   | FK → iam_users(id)                   |
| is_active   | TINYINT(1) | Default 1                            |

### Table Creation Query

```sql
CREATE TABLE iam_project_members (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    user_id CHAR(36) NOT NULL,
    role_id CHAR(36) NOT NULL,
    is_external TINYINT(1) DEFAULT 0,
    joined_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by CHAR(36),
    is_active TINYINT(1) DEFAULT 1,

    CONSTRAINT fk_project_member_user
    FOREIGN KEY (user_id)
    REFERENCES iam_users(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_project_member_role
    FOREIGN KEY (role_id)
    REFERENCES iam_project_roles(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_user_id (user_id),
    INDEX idx_role_id (role_id)
);
```

### Notes

- Maps users to projects.
- Supports both:
  - internal organization members
  - external project members

Used in:
- project access control
- workflow approvals
- team assignments

---

## **1.10 Table: `iam_user_otps`**

| Column Name | Type         | Constraints / Notes                  |
| ------------| ------------ | ------------------------------------ |
| id          | CHAR(36)     | PK (UUID)                            |
| user_id     | CHAR(36)     | FK → iam_users(id)                   |
| otp_code    | VARCHAR(10)  | NOT NULL                             |
| otp_type    | ENUM         | VERIFY / PASSWORD_RESET              |
| expires_at  | DATETIME     | NOT NULL                             |
| is_used     | TINYINT(1)   | Default 0                            |
| created_at  | DATETIME     | Default CURRENT_TIMESTAMP            |

### Table Creation Query

```sql
CREATE TABLE iam_user_otps (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    otp_code VARCHAR(10) NOT NULL,
    otp_type ENUM('VERIFY','PASSWORD_RESET') NOT NULL,
    expires_at DATETIME NOT NULL,
    is_used TINYINT(1) DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_user_otp_user
    FOREIGN KEY (user_id)
    REFERENCES iam_users(id)
    ON DELETE CASCADE,

    INDEX idx_user_id (user_id),
    INDEX idx_otp_code (otp_code)
);
```

### Notes

Stores generated OTP records for:

- account verification
- password reset
- future login verification flows

Expired or used OTPs should not be reusable.

---
---

# ---------------------------------------------------------

# 🟩 **2. PROJECT MANAGEMENT SERVICE (PROJECT SERVICE)**

# ---------------------------------------------------------

This service manages:

- project creation
- project lifecycle management
- client management
- project hierarchy structure
- project configuration
- project-level dynamic masters
- project settings
- project metadata
- project locations
- project holidays
- customizable project fields

The Project Service acts as the central project configuration and structure management layer for the entire SiteMate ERP platform.

---

## **2.1 Table: `project_clients`**

| Column Name    | Type          | Constraints / Notes                      |
| -------------- | ------------- | ---------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                                |
| organization_id| CHAR(36)      | FK → iam_organizations(id)               |
| client_name    | VARCHAR(255)  | NOT NULL                                 |
| contact_person | VARCHAR(255)  | NULL                                     |
| email          | VARCHAR(255)  | NULL                                     |
| mobile         | VARCHAR(20)   | NULL                                     |
| address        | TEXT          | NULL                                     |
| gst_number     | VARCHAR(100)  | NULL                                     |
| pan_number     | VARCHAR(100)  | NULL                                     |
| remarks        | TEXT          | NULL                                     |
| is_active      | TINYINT(1)    | Default 1                                |
| created_by     | CHAR(36)      | FK → iam_users(id)                       |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP                |
| updated_at     | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_clients (
    id CHAR(36) PRIMARY KEY,
    organization_id CHAR(36) NOT NULL,
    client_name VARCHAR(255) NOT NULL,
    contact_person VARCHAR(255),
    email VARCHAR(255),
    mobile VARCHAR(20),
    address TEXT,
    gst_number VARCHAR(100),
    pan_number VARCHAR(100),
    remarks TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_organization_id (organization_id),
    INDEX idx_client_name (client_name)
);
```

### Notes

- Stores reusable client master data.
- One client can have multiple projects.
- Supports future CRM expansion.

---

## **2.2 Table: `project_types`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| type_name   | VARCHAR(150)  | UNIQUE, NOT NULL                  |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_types (
    id CHAR(36) PRIMARY KEY,
    type_name VARCHAR(150) NOT NULL UNIQUE,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

Examples:
- Residential
- Commercial
- Industrial
- Infrastructure
- Interior

---

## **2.3 Table: `project_statuses`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| status_name | VARCHAR(100)  | UNIQUE, NOT NULL                  |
| description | TEXT          | NULL                              |
| color_code  | VARCHAR(20)   | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_statuses (
    id CHAR(36) PRIMARY KEY,
    status_name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    color_code VARCHAR(20),
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

Recommended default statuses:
- DRAFT
- ACTIVE
- ON_HOLD
- COMPLETED
- CANCELLED
- ARCHIVED

---

## **2.4 Table: `project_projects`**

| Column Name      | Type          | Constraints / Notes                      |
| ---------------- | ------------- | ---------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                |
| organization_id  | CHAR(36)      | FK → iam_organizations(id)               |
| client_id        | CHAR(36)      | FK → project_clients(id)                 |
| project_type_id  | CHAR(36)      | FK → project_types(id)                   |
| project_status_id| CHAR(36)      | FK → project_statuses(id)                |
| project_name     | VARCHAR(255)  | NOT NULL                                 |
| project_code     | VARCHAR(100)  | UNIQUE, NOT NULL                         |
| description      | TEXT          | NULL                                     |
| start_date       | DATE          | NULL                                     |
| end_date         | DATE          | NULL                                     |
| estimated_budget | DECIMAL(18,2) | NULL                                     |
| actual_budget    | DECIMAL(18,2) | NULL                                     |
| is_active        | TINYINT(1)    | Default 1                                |
| created_by       | CHAR(36)      | FK → iam_users(id)                       |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_projects (
    id CHAR(36) PRIMARY KEY,
    organization_id CHAR(36) NOT NULL,
    client_id CHAR(36),
    project_type_id CHAR(36),
    project_status_id CHAR(36),
    project_name VARCHAR(255) NOT NULL,
    project_code VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    start_date DATE,
    end_date DATE,
    estimated_budget DECIMAL(18,2),
    actual_budget DECIMAL(18,2),
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_project_client
    FOREIGN KEY (client_id)
    REFERENCES project_clients(id),

    CONSTRAINT fk_project_type
    FOREIGN KEY (project_type_id)
    REFERENCES project_types(id),

    CONSTRAINT fk_project_status
    FOREIGN KEY (project_status_id)
    REFERENCES project_statuses(id),

    INDEX idx_organization_id (organization_id),
    INDEX idx_project_code (project_code),
    INDEX idx_project_name (project_name)
);
```

### Notes

Main project master table.

Used by:
- execution service
- workflow service
- media service
- task service
- notification service

---

## **2.5 Table: `project_structure_types`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| type_name   | VARCHAR(150)  | UNIQUE, NOT NULL                  |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_structure_types (
    id CHAR(36) PRIMARY KEY,
    type_name VARCHAR(150) NOT NULL UNIQUE,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP
);
```

### Notes

Examples:
- TOWER
- FLOOR
- BLOCK
- ZONE
- WING
- BASEMENT
- UNIT

---

## **2.6 Table: `project_structures`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | FK → project_projects(id)                   |
| structure_type_id| CHAR(36)      | FK → project_structure_types(id)            |
| parent_id        | CHAR(36)      | Self FK → project_structures(id)            |
| structure_name   | VARCHAR(255)  | NOT NULL                                    |
| structure_code   | VARCHAR(100)  | NULL                                        |
| description      | TEXT          | NULL                                        |
| sequence_no      | INT           | Default 0                                   |
| is_active        | TINYINT(1)    | Default 1                                   |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_structures (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    structure_type_id CHAR(36) NOT NULL,
    parent_id CHAR(36),
    structure_name VARCHAR(255) NOT NULL,
    structure_code VARCHAR(100),
    description TEXT,
    sequence_no INT DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_structure_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_structure_type
    FOREIGN KEY (structure_type_id)
    REFERENCES project_structure_types(id),

    CONSTRAINT fk_structure_parent
    FOREIGN KEY (parent_id)
    REFERENCES project_structures(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_parent_id (parent_id),
    INDEX idx_structure_type (structure_type_id)
);
```

### Notes

Dynamic hierarchy table.

Supports unlimited project hierarchy depth.

Example:

```text
Project
 └── Tower A
      └── Floor 1
           └── Zone A
```

---

## **2.7 Table: `project_design_types`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| project_id  | CHAR(36)      | FK → project_projects(id)         |
| design_name | VARCHAR(150)  | NOT NULL                          |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_design_types (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    design_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_design_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Stores project-specific design categories.

Examples:
- Plumbing
- Electrical
- Architecture
- Fire Fighting

---

## **2.8 Table: `project_activity_types`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| project_id  | CHAR(36)      | FK → project_projects(id)         |
| activity_name | VARCHAR(150)| NOT NULL                          |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_activity_types (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    activity_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_activity_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Stores execution activity masters.

Examples:
- Excavation
- Slab Casting
- Plumbing
- Painting

---

## **2.9 Table: `project_tags`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| project_id  | CHAR(36)      | FK → project_projects(id)         |
| tag_name    | VARCHAR(150)  | NOT NULL                          |
| color_code  | VARCHAR(20)   | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE project_tags (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    tag_name VARCHAR(150) NOT NULL,
    color_code VARCHAR(20),
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_tag_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Reusable tagging system across modules.

---

## **2.10 Table: `project_settings`**

| Column Name | Type          | Constraints / Notes                  |
| ------------| ------------- | ------------------------------------ |
| id          | CHAR(36)      | PK (UUID)                            |
| project_id  | CHAR(36)      | FK → project_projects(id)            |
| setting_key | VARCHAR(150)  | NOT NULL                             |
| setting_value | JSON        | NULL                                 |
| created_by  | CHAR(36)      | FK → iam_users(id)                   |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP            |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE project_settings (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    setting_key VARCHAR(150) NOT NULL,
    setting_value JSON,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_setting_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_setting_key (setting_key)
);
```

### Notes

Stores dynamic project configuration settings.

Examples:
- notification settings
- reminder settings
- approval settings
- workflow configurations

```
{
  "notifications_enabled": true,
  "email_notifications": true,
  "photo_upload_required": true,
  "workflow_approval_enabled": true,
  "daily_report_reminder": true
}
```

---

## **2.11 Table: `project_locations`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| project_id  | CHAR(36)      | FK → project_projects(id)         |
| location_name | VARCHAR(255)| NOT NULL                          |
| address     | TEXT          | NULL                              |
| latitude    | DECIMAL(10,7)| NULL                              |
| longitude   | DECIMAL(10,7)| NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE project_locations (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    location_name VARCHAR(255) NOT NULL,
    address TEXT,
    latitude DECIMAL(10,7),
    longitude DECIMAL(10,7),
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_location_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Supports:
- geo-location
- multiple project sites
- future map integrations

---

## **2.12 Table: `project_holidays`**

| Column Name | Type         | Constraints / Notes               |
| ------------| ------------ | --------------------------------- |
| id          | CHAR(36)     | PK (UUID)                         |
| project_id  | CHAR(36)     | FK → project_projects(id)         |
| holiday_name| VARCHAR(255) | NOT NULL                          |
| holiday_date| DATE         | NOT NULL                          |
| remarks     | TEXT         | NULL                              |
| created_by  | CHAR(36)     | FK → iam_users(id)                |
| created_at  | DATETIME     | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE project_holidays (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    holiday_name VARCHAR(255) NOT NULL,
    holiday_date DATE NOT NULL,
    remarks TEXT,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_holiday_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_holiday_date (holiday_date)
);
```

### Notes

Useful for:
- planning calculations
- execution scheduling
- working day calculations

---

## **2.13 Table: `project_custom_fields`**

| Column Name | Type          | Constraints / Notes                 |
| ------------| ------------- | ----------------------------------- |
| id          | CHAR(36)      | PK (UUID)                           |
| project_id  | CHAR(36)      | FK → project_projects(id)           |
| field_name  | VARCHAR(150)  | NOT NULL                            |
| field_type  | VARCHAR(100)  | TEXT / NUMBER / DATE / BOOLEAN      |
| field_config| JSON          | NULL                                |
| is_required | TINYINT(1)    | Default 0                           |
| is_active   | TINYINT(1)    | Default 1                           |
| created_by  | CHAR(36)      | FK → iam_users(id)                  |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP           |

### Table Creation Query

```sql
CREATE TABLE project_custom_fields (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    field_name VARCHAR(150) NOT NULL,
    field_type VARCHAR(100) NOT NULL,
    field_config JSON,
    is_required TINYINT(1) DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_custom_field_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Supports:
- customizable project metadata
- dynamic forms
- future extensibility

---

---

# ---------------------------------------------------------

# 🟧 **3. EXECUTION & REPORTING SERVICE (EXECUTION SERVICE)**

# ---------------------------------------------------------

This service manages:

- daily progress reporting (DPR)
- execution activities
- labour tracking
- material tracking
- machinery tracking
- bottlenecks & TAB management
- tomorrow planning
- project execution timeline
- generated report snapshots
- weather tracking
- site visit tracking

The Execution Service acts as the operational execution engine of the SiteMate ERP platform.

---

## **3.1 Table: `execution_daily_reports`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | FK → project_projects(id)                   |
| report_date      | DATE          | NOT NULL                                    |
| report_no        | VARCHAR(100)  | UNIQUE                                      |
| weather_summary  | VARCHAR(255)  | NULL                                        |
| general_remark   | TEXT          | NULL                                        |
| site_condition   | TEXT          | NULL                                        |
| approval_status  | VARCHAR(50)   | NULL (future approval workflow support)     |
| approved_by      | CHAR(36)      | NULL                                        |
| approved_at      | DATETIME      | NULL                                        |
| is_locked        | TINYINT(1)    | Default 0                                   |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE execution_daily_reports (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    report_date DATE NOT NULL,
    report_no VARCHAR(100) UNIQUE,
    weather_summary VARCHAR(255),
    general_remark TEXT,
    site_condition TEXT,
    approval_status VARCHAR(50),
    approved_by CHAR(36),
    approved_at DATETIME,
    is_locked TINYINT(1) DEFAULT 0,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_execution_report_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_report_date (report_date)
);
```

### Notes

- Main DPR header table.
- Stores one daily execution report per project/day.
- Supports future report approval workflows.
- Locked reports should become immutable.

---

## **3.2 Table: `execution_daily_activities`**

| Column Name        | Type           | Constraints / Notes                        |
| ------------------ | -------------- | ------------------------------------------ |
| id                 | CHAR(36)       | PK (UUID)                                  |
| report_id          | CHAR(36)       | FK → execution_daily_reports(id)           |
| activity_type_id   | CHAR(36)       | FK → project_activity_types(id)            |
| structure_id       | CHAR(36)       | FK → project_structures(id)                |
| unit               | VARCHAR(50)    | NULL                                       |
| total_quantity     | DECIMAL(18,2)  | Default 0                                  |
| previous_quantity  | DECIMAL(18,2)  | Default 0                                  |
| today_quantity     | DECIMAL(18,2)  | Default 0                                  |
| completed_quantity | DECIMAL(18,2)  | Default 0                                  |
| balance_quantity   | DECIMAL(18,2)  | Default 0                                  |
| tomorrow_quantity  | DECIMAL(18,2)  | Default 0                                  |
| remarks            | TEXT           | NULL                                       |
| created_by         | CHAR(36)       | FK → iam_users(id)                         |
| created_at         | DATETIME       | Default CURRENT_TIMESTAMP                  |

### Table Creation Query

```sql
CREATE TABLE execution_daily_activities (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    activity_type_id CHAR(36),
    structure_id CHAR(36),
    unit VARCHAR(50),
    total_quantity DECIMAL(18,2) DEFAULT 0,
    previous_quantity DECIMAL(18,2) DEFAULT 0,
    today_quantity DECIMAL(18,2) DEFAULT 0,
    completed_quantity DECIMAL(18,2) DEFAULT 0,
    balance_quantity DECIMAL(18,2) DEFAULT 0,
    tomorrow_quantity DECIMAL(18,2) DEFAULT 0,
    remarks TEXT,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_execution_activity_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id),
    INDEX idx_activity_type_id (activity_type_id),
    INDEX idx_structure_id (structure_id)
);
```

### Notes

Supports DPR activity progress tracking.

Examples:
- Piling
- Excavation
- Slab Casting
- Plumbing

---

## **3.3 Table: `execution_report_custom_sections`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| report_id        | CHAR(36)      | FK → execution_daily_reports(id)            |
| section_title    | VARCHAR(255)  | NOT NULL                                    |
| section_type     | VARCHAR(100)  | TEXT / IMAGE / TABLE                        |
| section_content  | JSON          | Dynamic section data                        |
| sequence_no      | INT           | Default 0                                   |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE execution_report_custom_sections (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    section_title VARCHAR(255) NOT NULL,
    section_type VARCHAR(100),
    section_content JSON,
    sequence_no INT DEFAULT 0,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_custom_section_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id)
);
```

### Notes

Supports:
- custom images
- additional remarks
- custom report sections
- organization-specific DPR layouts

---

## **3.4 Table: `execution_labour_entries`**

| Column Name | Type         | Constraints / Notes                       |
| ------------| ------------ | ----------------------------------------- |
| id          | CHAR(36)     | PK (UUID)                                 |
| report_id   | CHAR(36)     | FK → execution_daily_reports(id)          |
| total_labour| INT          | Default 0                                 |
| remarks     | TEXT         | NULL                                      |
| created_by  | CHAR(36)     | FK → iam_users(id)                        |
| created_at  | DATETIME     | Default CURRENT_TIMESTAMP                 |

### Table Creation Query

```sql
CREATE TABLE execution_labour_entries (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    total_labour INT DEFAULT 0,
    remarks TEXT,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_labour_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id)
);
```

### Notes

Stores labour summary for DPR generation.

---

## **3.5 Table: `execution_labour_details`**

| Column Name   | Type         | Constraints / Notes                        |
| ------------- | ------------ | ------------------------------------------ |
| id            | CHAR(36)     | PK (UUID)                                  |
| labour_entry_id | CHAR(36)   | FK → execution_labour_entries(id)          |
| contractor_name | VARCHAR(255)| NULL                                       |
| labour_type   | VARCHAR(100) | Skilled / Unskilled                        |
| trade_name    | VARCHAR(150) | NULL                                       |
| labour_count  | INT          | Default 0                                  |
| remarks       | TEXT         | NULL                                       |
| created_at    | DATETIME     | Default CURRENT_TIMESTAMP                  |

### Table Creation Query

```sql
CREATE TABLE execution_labour_details (
    id CHAR(36) PRIMARY KEY,
    labour_entry_id CHAR(36) NOT NULL,
    contractor_name VARCHAR(255),
    labour_type VARCHAR(100),
    trade_name VARCHAR(150),
    labour_count INT DEFAULT 0,
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_labour_detail_entry
    FOREIGN KEY (labour_entry_id)
    REFERENCES execution_labour_entries(id)
    ON DELETE CASCADE,

    INDEX idx_labour_entry_id (labour_entry_id)
);
```

### Notes

Supports:
- contractor-wise labour tracking
- future labour analytics
- vendor-based reporting

---

## **3.6 Table: `execution_material_consumptions`**

| Column Name          | Type           | Constraints / Notes                  |
| -------------------- | -------------- | ------------------------------------ |
| id                   | CHAR(36)       | PK (UUID)                            |
| report_id            | CHAR(36)       | FK → execution_daily_reports(id)     |
| material_name        | VARCHAR(255)   | NOT NULL                             |
| unit                 | VARCHAR(50)    | NULL                                 |
| total_received       | DECIMAL(18,2)  | Default 0                            |
| total_consumed       | DECIMAL(18,2)  | Default 0                            |
| today_received       | DECIMAL(18,2)  | Default 0                            |
| today_consumed       | DECIMAL(18,2)  | Default 0                            |
| balance_quantity     | DECIMAL(18,2)  | Default 0                            |
| created_at           | DATETIME       | Default CURRENT_TIMESTAMP            |

### Table Creation Query

```sql
CREATE TABLE execution_material_consumptions (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    material_name VARCHAR(255) NOT NULL,
    unit VARCHAR(50),
    total_received DECIMAL(18,2) DEFAULT 0,
    total_consumed DECIMAL(18,2) DEFAULT 0,
    today_received DECIMAL(18,2) DEFAULT 0,
    today_consumed DECIMAL(18,2) DEFAULT 0,
    balance_quantity DECIMAL(18,2) DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_material_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id)
);
```

### Notes

Supports DPR material reporting.

Examples:
- Cement
- Steel
- Concrete
- Sand

---

## **3.7 Table: `execution_machinery_entries`**

| Column Name    | Type          | Constraints / Notes                    |
| -------------- | ------------- | -------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                              |
| report_id      | CHAR(36)      | FK → execution_daily_reports(id)       |
| agency_name    | VARCHAR(255)  | NULL                                   |
| machine_type   | VARCHAR(150)  | NULL                                   |
| vehicle_number | VARCHAR(100)  | NULL                                   |
| start_time     | TIME          | NULL                                   |
| end_time       | TIME          | NULL                                   |
| total_hours    | DECIMAL(10,2) | Default 0                              |
| remarks        | TEXT          | NULL                                   |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_machinery_entries (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    agency_name VARCHAR(255),
    machine_type VARCHAR(150),
    vehicle_number VARCHAR(100),
    start_time TIME,
    end_time TIME,
    total_hours DECIMAL(10,2) DEFAULT 0,
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_machine_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id)
);
```

### Notes

Supports machinery utilization reporting.

Examples:
- Cranes
- Excavators
- Poclains
- Dumpers

---

## **3.8 Table: `execution_bottlenecks`**

| Column Name      | Type          | Constraints / Notes                    |
| ---------------- | ------------- | -------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                              |
| project_id       | CHAR(36)      | FK → project_projects(id)              |
| title            | VARCHAR(255)  | NOT NULL                               |
| description      | TEXT          | NULL                                   |
| severity         | VARCHAR(50)   | LOW / MEDIUM / HIGH / CRITICAL         |
| status           | VARCHAR(50)   | OPEN / IN_PROGRESS / RESOLVED / CLOSED |
| reported_by      | CHAR(36)      | FK → iam_users(id)                     |
| assigned_to      | CHAR(36)      | FK → iam_users(id)                     |
| target_date      | DATE          | NULL                                   |
| resolved_at      | DATETIME      | NULL                                   |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP              |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE execution_bottlenecks (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    severity VARCHAR(50),
    status VARCHAR(50),
    reported_by CHAR(36),
    assigned_to CHAR(36),
    target_date DATE,
    resolved_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_bottleneck_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_status (status)
);
```

### Notes

Acts as:
- TAB system
- issue tracking
- blocker management system

---

## **3.9 Table: `execution_bottleneck_comments`**

| Column Name   | Type         | Constraints / Notes                    |
| ------------- | ------------ | -------------------------------------- |
| id            | CHAR(36)     | PK (UUID)                              |
| bottleneck_id | CHAR(36)     | FK → execution_bottlenecks(id)         |
| comment_text  | TEXT         | NOT NULL                               |
| created_by    | CHAR(36)     | FK → iam_users(id)                     |
| created_at    | DATETIME     | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_bottleneck_comments (
    id CHAR(36) PRIMARY KEY,
    bottleneck_id CHAR(36) NOT NULL,
    comment_text TEXT NOT NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_bottleneck_comment
    FOREIGN KEY (bottleneck_id)
    REFERENCES execution_bottlenecks(id)
    ON DELETE CASCADE,

    INDEX idx_bottleneck_id (bottleneck_id)
);
```

### Notes

Stores discussion history for bottlenecks.

---

## **3.10 Table: `execution_tomorrow_plans`**

| Column Name      | Type           | Constraints / Notes                    |
| ---------------- | -------------- | -------------------------------------- |
| id               | CHAR(36)       | PK (UUID)                              |
| report_id        | CHAR(36)       | FK → execution_daily_reports(id)       |
| activity_type_id | CHAR(36)       | FK → project_activity_types(id)        |
| structure_id     | CHAR(36)       | FK → project_structures(id)            |
| planned_quantity | DECIMAL(18,2)  | Default 0                              |
| unit             | VARCHAR(50)    | NULL                                   |
| remarks          | TEXT           | NULL                                   |
| created_at       | DATETIME       | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_tomorrow_plans (
    id CHAR(36) PRIMARY KEY,
    report_id CHAR(36) NOT NULL,
    activity_type_id CHAR(36),
    structure_id CHAR(36),
    planned_quantity DECIMAL(18,2) DEFAULT 0,
    unit VARCHAR(50),
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_tomorrow_plan_report
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_report_id (report_id)
);
```

### Notes

Supports structured tomorrow planning.

Used later for:
- execution comparison
- planning analytics
- forecast tracking

---

## **3.11 Table: `execution_timeline_events`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| project_id    | CHAR(36)      | FK → project_projects(id)              |
| event_type    | VARCHAR(100)  | NULL                                   |
| event_title   | VARCHAR(255)  | NOT NULL                               |
| event_message | TEXT          | NULL                                   |
| reference_id  | CHAR(36)      | NULL                                   |
| created_by    | CHAR(36)      | FK → iam_users(id)                     |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_timeline_events (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    event_type VARCHAR(100),
    event_title VARCHAR(255) NOT NULL,
    event_message TEXT,
    reference_id CHAR(36),
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_timeline_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_event_type (event_type)
);
```

### Notes

Acts as centralized project activity feed.

---

## **3.12 Table: `execution_generated_reports`**

| Column Name      | Type          | Constraints / Notes                     |
| ---------------- | ------------- | --------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                               |
| project_id       | CHAR(36)      | FK → project_projects(id)               |
| report_id        | CHAR(36)      | FK → execution_daily_reports(id)        |
| report_type      | VARCHAR(100)  | DPR / TAB / LABOUR_REPORT               |
| file_name        | VARCHAR(255)  | NULL                                    |
| object_path      | VARCHAR(1000) | Object storage file path                |
| generated_by     | CHAR(36)      | FK → iam_users(id)                      |
| generated_at     | DATETIME      | Default CURRENT_TIMESTAMP               |

### Table Creation Query

```sql
CREATE TABLE execution_generated_reports (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    report_id CHAR(36),
    report_type VARCHAR(100),
    file_name VARCHAR(255),
    object_path VARCHAR(1000),
    generated_by CHAR(36),
    generated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_generated_report_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_generated_report_execution
    FOREIGN KEY (report_id)
    REFERENCES execution_daily_reports(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_report_id (report_id)
);
```

### Notes

Stores:
- generated PDF references
- report snapshots
- object storage paths

Actual files stored in MinIO object storage.

---

## **3.13 Table: `execution_activity_progress`**

| Column Name       | Type           | Constraints / Notes                    |
| ----------------- | -------------- | -------------------------------------- |
| id                | CHAR(36)       | PK (UUID)                              |
| project_id        | CHAR(36)       | FK → project_projects(id)              |
| activity_type_id  | CHAR(36)       | FK → project_activity_types(id)        |
| structure_id      | CHAR(36)       | FK → project_structures(id)            |
| total_quantity    | DECIMAL(18,2)  | Default 0                              |
| completed_quantity| DECIMAL(18,2)  | Default 0                              |
| balance_quantity  | DECIMAL(18,2)  | Default 0                              |
| last_updated_at   | DATETIME       | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_activity_progress (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    activity_type_id CHAR(36),
    structure_id CHAR(36),
    total_quantity DECIMAL(18,2) DEFAULT 0,
    completed_quantity DECIMAL(18,2) DEFAULT 0,
    balance_quantity DECIMAL(18,2) DEFAULT 0,
    last_updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_project_id (project_id)
);
```

### Notes

Stores cumulative activity progress for dashboards and analytics.

---

## **3.14 Table: `execution_weather_logs`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| project_id    | CHAR(36)      | FK → project_projects(id)              |
| weather_date  | DATE          | NOT NULL                               |
| weather_type  | VARCHAR(100)  | Sunny / Rainy / Cloudy                 |
| temperature   | DECIMAL(5,2)  | NULL                                   |
| remarks       | TEXT          | NULL                                   |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_weather_logs (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    weather_date DATE NOT NULL,
    weather_type VARCHAR(100),
    temperature DECIMAL(5,2),
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_weather_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_weather_date (weather_date)
);
```

### Notes

Supports:
- weather tracking
- delay analysis
- reporting integration

---

## **3.15 Table: `execution_site_visits`**

| Column Name    | Type          | Constraints / Notes                    |
| -------------- | ------------- | -------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                              |
| project_id     | CHAR(36)      | FK → project_projects(id)              |
| visitor_name   | VARCHAR(255)  | NOT NULL                               |
| visitor_type   | VARCHAR(100)  | Client / Consultant / Auditor          |
| visit_purpose  | TEXT          | NULL                                   |
| visit_remark   | TEXT          | NULL                                   |
| visit_date     | DATETIME      | NOT NULL                               |
| created_by     | CHAR(36)      | FK → iam_users(id)                     |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE execution_site_visits (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    visitor_name VARCHAR(255) NOT NULL,
    visitor_type VARCHAR(100),
    visit_purpose TEXT,
    visit_remark TEXT,
    visit_date DATETIME NOT NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_site_visit_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_visit_date (visit_date)
);
```

### Notes

Tracks:
- consultant visits
- client inspections
- audits
- quality inspections

---
---

# ---------------------------------------------------------

# 🟪 **4. WORKFLOW & APPROVAL ENGINE SERVICE (WORKFLOW SERVICE)**

# ---------------------------------------------------------

This service manages:

- approval workflows
- approval pipelines
- workflow hierarchy
- approval routing
- workflow transactions
- approval actions
- approval logs
- workflow audit history
- escalation support
- reusable workflow templates
- module-independent approval processing

The Workflow Service acts as the centralized approval and business process engine for the entire SiteMate ERP platform.

This service is intentionally designed as a generic reusable engine so that future ERP modules can use the same workflow infrastructure without redesigning approval logic.

---

# 🔶 Important Architectural Decision

The Workflow Service is designed to be:

- module-independent
- reusable
- scalable
- configurable
- future-proof

The workflow engine should NEVER directly depend on:
- drawing schema
- invoice schema
- DPR schema
- purchase schema

Instead, workflows will always work using:

```text
module_id
reference_id
reference_version
```

Example:

```text
module = DRAWINGS
reference_id = DRAWING_ID
reference_version = V3
```

This architecture ensures that all ERP modules can reuse the same workflow engine.

---

# 🔶 Workflow Architecture Strategy

The workflow system is divided into two major layers:

---

## 1. Workflow Definition Layer

Defines:
- workflow templates
- approval hierarchy
- approvers
- workflow logic

Tables:
- workflow_modules
- workflow_definitions
- workflow_steps
- workflow_templates
- workflow_conditions

---

## 2. Workflow Execution Layer

Tracks:
- actual approvals
- comments
- status changes
- approval actions
- escalation logs
- approval history

Tables:
- workflow_transactions
- workflow_transaction_steps
- workflow_transaction_logs
- workflow_escalations

---

# 🔶 Supported Workflow Types

The system supports:

- Global Workflows
- Project-Level Workflows

---

## Global Workflow Example

```text
Standard Invoice Approval
```

Reusable across projects.

---

## Project-Level Workflow Example

```text
High Value Purchase Approval - Project A
```

Specific to one project.

---

# 🔶 Approval Assignment Types

The system supports:

## Role-Based Approval

Example:

```text
SITE_ENGINEER
   ↓
PROJECT_INCHARGE
   ↓
ADMIN
```

---

## Direct User-Based Approval

Example:

```text
Rahul Sharma
   ↓
Vijay Patil
   ↓
Amit Joshi
```

---

# 🔶 Approval Modes

The system supports:

## Sequential Approval

Step-by-step approval flow.

---

## Parallel Approval

Multiple approvers can approve simultaneously.

---

# 🔶 Workflow Logs Strategy

All approval activities are stored in:

```text
workflow_transaction_logs
```

This acts as the centralized audit history table for the entire ERP.

---

## Example Logs

- drawing submitted
- changes requested
- version updated
- approval completed
- rejection comments
- reassignment
- escalation actions

---

# 🔶 Supported Workflow Action Types

Examples:

```text
SUBMITTED
APPROVED
REJECTED
COMMENTED
CHANGES_REQUESTED
REASSIGNED
ESCALATED
VERSION_UPDATED
```

These action types provide complete workflow traceability.

---

## **4.1 Table: `workflow_modules`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| module_code | VARCHAR(100)  | UNIQUE, NOT NULL                  |
| module_name | VARCHAR(150)  | NOT NULL                          |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |
| updated_at  | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE workflow_modules (
    id CHAR(36) PRIMARY KEY,
    module_code VARCHAR(100) NOT NULL UNIQUE,
    module_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_module_code (module_code)
);
```

### Notes

Stores workflow-enabled ERP modules dynamically.

Examples:
- DRAWINGS
- DPR
- PURCHASES
- TASKS
- EXPENSES

Future modules can be added without schema changes.

---

## **4.2 Table: `workflow_definitions`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | NULL for GLOBAL workflows                   |
| module_id        | CHAR(36)      | FK → workflow_modules(id)                   |
| workflow_name    | VARCHAR(255)  | NOT NULL                                    |
| workflow_scope   | VARCHAR(50)   | GLOBAL / PROJECT                            |
| approval_mode    | VARCHAR(50)   | SEQUENTIAL / PARALLEL                       |
| description      | TEXT          | NULL                                        |
| is_active        | TINYINT(1)    | Default 1                                   |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE workflow_definitions (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36),
    module_id CHAR(36) NOT NULL,
    workflow_name VARCHAR(255) NOT NULL,
    workflow_scope VARCHAR(50) NOT NULL,
    approval_mode VARCHAR(50) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_workflow_module
    FOREIGN KEY (module_id)
    REFERENCES workflow_modules(id),

    INDEX idx_project_id (project_id),
    INDEX idx_module_id (module_id)
);
```

### Notes

Stores workflow definitions and approval pipeline structures.

---

## **4.3 Table: `workflow_steps`**

| Column Name     | Type          | Constraints / Notes                          |
| ----------------| ------------- | -------------------------------------------- |
| id              | CHAR(36)      | PK (UUID)                                    |
| workflow_id     | CHAR(36)      | FK → workflow_definitions(id)                |
| step_no         | INT           | NOT NULL                                     |
| step_name       | VARCHAR(255)  | NOT NULL                                     |
| assignment_type | VARCHAR(50)   | ROLE / USER                                  |
| role_id         | CHAR(36)      | FK → iam_project_roles(id), NULL allowed     |
| user_id         | CHAR(36)      | FK → iam_users(id), NULL allowed             |
| is_mandatory    | TINYINT(1)    | Default 1                                    |
| sla_hours       | INT           | NULL                                         |
| escalation_enabled | TINYINT(1) | Default 0                                    |
| created_at      | DATETIME      | Default CURRENT_TIMESTAMP                    |

### Table Creation Query

```sql
CREATE TABLE workflow_steps (
    id CHAR(36) PRIMARY KEY,
    workflow_id CHAR(36) NOT NULL,
    step_no INT NOT NULL,
    step_name VARCHAR(255) NOT NULL,
    assignment_type VARCHAR(50) NOT NULL,
    role_id CHAR(36),
    user_id CHAR(36),
    is_mandatory TINYINT(1) DEFAULT 1,
    sla_hours INT,
    escalation_enabled TINYINT(1) DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_workflow_step_definition
    FOREIGN KEY (workflow_id)
    REFERENCES workflow_definitions(id)
    ON DELETE CASCADE,

    INDEX idx_workflow_id (workflow_id),
    INDEX idx_step_no (step_no)
);
```

### Notes

Defines workflow approval hierarchy.

Supports:
- role-based approvals
- direct user approvals
- sequential approvals
- parallel approvals

---

## **4.4 Table: `workflow_transactions`**

| Column Name        | Type          | Constraints / Notes                         |
| ------------------ | ------------- | ------------------------------------------- |
| id                 | CHAR(36)      | PK (UUID)                                   |
| workflow_id        | CHAR(36)      | FK → workflow_definitions(id)               |
| module_id          | CHAR(36)      | FK → workflow_modules(id)                   |
| project_id         | CHAR(36)      | FK → project_projects(id)                   |
| reference_id       | CHAR(36)      | Entity reference ID                         |
| reference_version  | VARCHAR(100)  | NULL                                        |
| transaction_status | VARCHAR(50)   | DRAFT / PENDING / APPROVED / REJECTED       |
| initiated_by       | CHAR(36)      | FK → iam_users(id)                          |
| initiated_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| completed_at       | DATETIME      | NULL                                        |

### Table Creation Query

```sql
CREATE TABLE workflow_transactions (
    id CHAR(36) PRIMARY KEY,
    workflow_id CHAR(36) NOT NULL,
    module_id CHAR(36) NOT NULL,
    project_id CHAR(36),
    reference_id CHAR(36) NOT NULL,
    reference_version VARCHAR(100),
    transaction_status VARCHAR(50) DEFAULT 'PENDING',
    initiated_by CHAR(36),
    initiated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME,

    CONSTRAINT fk_transaction_workflow
    FOREIGN KEY (workflow_id)
    REFERENCES workflow_definitions(id),

    CONSTRAINT fk_transaction_module
    FOREIGN KEY (module_id)
    REFERENCES workflow_modules(id),

    INDEX idx_reference_id (reference_id),
    INDEX idx_project_id (project_id),
    INDEX idx_transaction_status (transaction_status)
);
```

### Notes

Stores actual workflow execution records.

Examples:
- Drawing V3 approval
- DPR approval
- Invoice approval

This table represents workflow instances.

---

## **4.5 Table: `workflow_transaction_steps`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| transaction_id   | CHAR(36)      | FK → workflow_transactions(id)              |
| workflow_step_id | CHAR(36)      | FK → workflow_steps(id)                     |
| assigned_user_id | CHAR(36)      | FK → iam_users(id)                          |
| step_status      | VARCHAR(50)   | PENDING / APPROVED / REJECTED               |
| action_date      | DATETIME      | NULL                                        |
| due_date         | DATETIME      | NULL                                        |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE workflow_transaction_steps (
    id CHAR(36) PRIMARY KEY,
    transaction_id CHAR(36) NOT NULL,
    workflow_step_id CHAR(36) NOT NULL,
    assigned_user_id CHAR(36),
    step_status VARCHAR(50) DEFAULT 'PENDING',
    action_date DATETIME,
    due_date DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_transaction_step_transaction
    FOREIGN KEY (transaction_id)
    REFERENCES workflow_transactions(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_transaction_step_definition
    FOREIGN KEY (workflow_step_id)
    REFERENCES workflow_steps(id),

    INDEX idx_transaction_id (transaction_id),
    INDEX idx_step_status (step_status)
);
```

### Notes

Tracks execution status of each workflow step.

---

## **4.6 Table: `workflow_transaction_logs`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| transaction_id   | CHAR(36)      | FK → workflow_transactions(id)              |
| transaction_step_id | CHAR(36)   | FK → workflow_transaction_steps(id)         |
| action_type      | VARCHAR(100)  | SUBMITTED / APPROVED / REJECTED etc.        |
| comment_text     | TEXT          | NULL                                        |
| previous_status  | VARCHAR(50)   | NULL                                        |
| new_status       | VARCHAR(50)   | NULL                                        |
| performed_by     | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE workflow_transaction_logs (
    id CHAR(36) PRIMARY KEY,
    transaction_id CHAR(36) NOT NULL,
    transaction_step_id CHAR(36),
    action_type VARCHAR(100) NOT NULL,
    comment_text TEXT,
    previous_status VARCHAR(50),
    new_status VARCHAR(50),
    performed_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_transaction_log_transaction
    FOREIGN KEY (transaction_id)
    REFERENCES workflow_transactions(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_transaction_log_step
    FOREIGN KEY (transaction_step_id)
    REFERENCES workflow_transaction_steps(id)
    ON DELETE CASCADE,

    INDEX idx_transaction_id (transaction_id),
    INDEX idx_action_type (action_type)
);
```

### Notes

This is the centralized workflow audit/history table.

Stores:
- approvals
- comments
- rejections
- version updates
- reassignment logs
- escalation logs
- change requests

Example Flow:

```text
Designer uploads drawing
        ↓
Reviewer comments
        ↓
Changes requested
        ↓
Designer uploads V2
```

All workflow actions are tracked here.

---

## **4.7 Table: `workflow_escalations`**

| Column Name      | Type          | Constraints / Notes                     |
| ---------------- | ------------- | --------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                               |
| transaction_step_id | CHAR(36)   | FK → workflow_transaction_steps(id)     |
| escalated_to     | CHAR(36)      | FK → iam_users(id)                      |
| escalation_reason| TEXT          | NULL                                    |
| escalated_at     | DATETIME      | Default CURRENT_TIMESTAMP               |

### Table Creation Query

```sql
CREATE TABLE workflow_escalations (
    id CHAR(36) PRIMARY KEY,
    transaction_step_id CHAR(36) NOT NULL,
    escalated_to CHAR(36),
    escalation_reason TEXT,
    escalated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_escalation_step
    FOREIGN KEY (transaction_step_id)
    REFERENCES workflow_transaction_steps(id)
    ON DELETE CASCADE,

    INDEX idx_transaction_step_id (transaction_step_id)
);
```

### Notes

Supports:
- SLA escalations
- overdue approval escalation
- workflow reassignment tracking

---

## **4.8 Table: `workflow_templates`**

| Column Name     | Type          | Constraints / Notes                  |
| ----------------| ------------- | ------------------------------------ |
| id              | CHAR(36)      | PK (UUID)                            |
| template_name   | VARCHAR(255)  | NOT NULL                             |
| module_id       | CHAR(36)      | FK → workflow_modules(id)            |
| template_config | JSON          | Workflow template configuration      |
| is_active       | TINYINT(1)    | Default 1                            |
| created_by      | CHAR(36)      | FK → iam_users(id)                   |
| created_at      | DATETIME      | Default CURRENT_TIMESTAMP            |

### Table Creation Query

```sql
CREATE TABLE workflow_templates (
    id CHAR(36) PRIMARY KEY,
    template_name VARCHAR(255) NOT NULL,
    module_id CHAR(36),
    template_config JSON,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_template_module
    FOREIGN KEY (module_id)
    REFERENCES workflow_modules(id),

    INDEX idx_module_id (module_id)
);
```

### Notes

Reusable workflow templates.

Example:
- Standard Drawing Approval
- Invoice Approval Template
- Purchase Approval Flow

---

## **4.9 Table: `workflow_conditions`**

| Column Name      | Type          | Constraints / Notes                  |
| ---------------- | ------------- | ------------------------------------ |
| id               | CHAR(36)      | PK (UUID)                            |
| workflow_id      | CHAR(36)      | FK → workflow_definitions(id)        |
| condition_field  | VARCHAR(255)  | NOT NULL                             |
| condition_operator | VARCHAR(50) | >, <, =, != etc.                     |
| condition_value  | VARCHAR(255)  | NULL                                 |
| action_config    | JSON          | Conditional workflow config          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP            |

### Table Creation Query

```sql
CREATE TABLE workflow_conditions (
    id CHAR(36) PRIMARY KEY,
    workflow_id CHAR(36) NOT NULL,
    condition_field VARCHAR(255) NOT NULL,
    condition_operator VARCHAR(50),
    condition_value VARCHAR(255),
    action_config JSON,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_condition_workflow
    FOREIGN KEY (workflow_id)
    REFERENCES workflow_definitions(id)
    ON DELETE CASCADE,

    INDEX idx_workflow_id (workflow_id)
);
```

### Notes

Supports future conditional workflows.

Example:

```text
If invoice amount > 10 lakh
then director approval required
```

This table makes the workflow engine highly scalable and enterprise-ready.

---
---

# ---------------------------------------------------------

# 🟫 **5. MEDIA & DOCUMENT MANAGEMENT SERVICE (MEDIA SERVICE)**

# ---------------------------------------------------------

This service manages:

- file uploads
- image uploads
- document uploads
- drawing uploads
- document versioning
- object storage integration
- media metadata
- file categorization
- folder hierarchy
- preview generation
- reusable attachment linking
- document freeze/version locking
- media tagging
- file lifecycle management

The Media Service acts as the centralized document and object storage management platform for the entire SiteMate ERP system.

This service is intentionally designed as a generic reusable media platform so that all ERP modules can use the same media infrastructure without duplicating file handling logic.

---

# 🔶 Important Architectural Decision

The Media Service is designed as:

- module-independent
- reusable
- scalable
- future-proof
- object-storage based

The Media Service should NEVER directly depend on:
- DPR schema
- drawing schema
- task schema
- workflow schema

Instead, media is linked dynamically using:

```text
module_code
reference_id
```

Example:

```text
module = DPR
reference_id = DPR_ID
```

This architecture ensures all ERP modules can reuse the same media system.

---

# 🔶 Object Storage Strategy

All actual files are stored in:

# MinIO Object Storage

The database stores ONLY:

```text
object_path
thumbnail_path
preview_path
```

This architecture improves:
- scalability
- performance
- backup strategy
- file delivery
- distributed deployments

---

# 🔶 Document Versioning Strategy

The Media Service uses enterprise-grade versioning architecture.

---

## Logical Document

Stored in:

```text
media_documents
```

Example:

```text
Plumbing Drawing
```

---

## Actual Uploaded Versions

Stored in:

```text
media_document_versions
```

Example:

```text
V1
V2
V3
```

This separation is VERY important for:
- revision tracking
- workflow integration
- approval history
- frozen versions
- rollback support

---

# 🔶 Freeze Version Strategy

Approved versions can be frozen.

Example:

```text
Drawing V3 → Approved → Frozen
```

Frozen versions become:
- immutable
- non-editable
- permanent project reference versions

---

# 🔶 Folder Hierarchy Strategy

The Media Service supports dynamic folder hierarchy using:

```text
parent_folder_id
```

Example:

```text
Architecture/
   Plumbing/
   Electrical/
```

This architecture supports unlimited folder depth.

---

# 🔶 Soft Delete Strategy

Enterprise document systems should NOT immediately hard delete files.

Instead:

```text
is_deleted
deleted_at
deleted_by
```

will be used.

Benefits:
- recovery support
- audit compliance
- accidental deletion prevention

---

# 🔶 Preview Generation Strategy

The system supports:
- image previews
- PDF thumbnails
- document preview metadata

Used for:
- faster frontend rendering
- dashboard previews
- gallery view optimization

---

# 🔶 Generic Linking Strategy

Media files are linked dynamically to ERP modules using:

```text
module_code
reference_id
```

Examples:

```text
DRAWING → DRAWING_ID
DPR → REPORT_ID
TASK → TASK_ID
```

This makes Media Service reusable across the entire ERP.

---

# 🔶 Approval Integration Strategy

Approval logic remains inside:
# Workflow Service

However, Media Service stores convenience status fields like:

```text
DRAFT
UNDER_REVIEW
APPROVED
REJECTED
FROZEN
```

for:
- filtering
- UI rendering
- dashboard visibility

---

# 🔶 Future Expansion Support

The architecture supports future features:

- external document sharing
- public links
- download tracking
- storage analytics
- watermarking
- OCR indexing
- AI document analysis

without schema redesign.

---

## **5.1 Table: `media_folders`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | FK → project_projects(id)                   |
| parent_folder_id | CHAR(36)      | Self FK → media_folders(id)                 |
| folder_name      | VARCHAR(255)  | NOT NULL                                    |
| description      | TEXT          | NULL                                        |
| is_active        | TINYINT(1)    | Default 1                                   |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE media_folders (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36),
    parent_folder_id CHAR(36),
    folder_name VARCHAR(255) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_media_folder_parent
    FOREIGN KEY (parent_folder_id)
    REFERENCES media_folders(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_parent_folder_id (parent_folder_id)
);
```

### Notes

Supports unlimited folder hierarchy.

Examples:
- Architecture
- Plumbing
- Site Photos
- DPR Attachments

---

## **5.2 Table: `media_documents`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | FK → project_projects(id)                   |
| folder_id        | CHAR(36)      | FK → media_folders(id)                      |
| document_title   | VARCHAR(255)  | NOT NULL                                    |
| document_type    | VARCHAR(100)  | DRAWING / IMAGE / PDF / REPORT etc.         |
| document_status  | VARCHAR(50)   | DRAFT / UNDER_REVIEW / APPROVED / FROZEN    |
| description      | TEXT          | NULL                                        |
| current_version  | VARCHAR(50)   | NULL                                        |
| is_active        | TINYINT(1)    | Default 1                                   |
| is_deleted       | TINYINT(1)    | Default 0                                   |
| deleted_at       | DATETIME      | NULL                                        |
| deleted_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_by       | CHAR(36)      | FK → iam_users(id)                          |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at       | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE media_documents (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36),
    folder_id CHAR(36),
    document_title VARCHAR(255) NOT NULL,
    document_type VARCHAR(100),
    document_status VARCHAR(50),
    description TEXT,
    current_version VARCHAR(50),
    is_active TINYINT(1) DEFAULT 1,
    is_deleted TINYINT(1) DEFAULT 0,
    deleted_at DATETIME,
    deleted_by CHAR(36),
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_media_document_folder
    FOREIGN KEY (folder_id)
    REFERENCES media_folders(id)
    ON DELETE SET NULL,

    INDEX idx_project_id (project_id),
    INDEX idx_folder_id (folder_id),
    INDEX idx_document_status (document_status)
);
```

### Notes

Represents logical document entity.

Examples:
- Plumbing Drawing
- Site Progress Photos
- Structural Design

Actual uploaded files are stored in:
```text
media_document_versions
```

---

## **5.3 Table: `media_document_versions`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| document_id      | CHAR(36)      | FK → media_documents(id)                    |
| version_no       | VARCHAR(50)   | NOT NULL                                    |
| file_name        | VARCHAR(255)  | NOT NULL                                    |
| original_name    | VARCHAR(255)  | NULL                                        |
| object_path      | VARCHAR(1000) | Object storage path                         |
| mime_type        | VARCHAR(255)  | NULL                                        |
| file_extension   | VARCHAR(50)   | NULL                                        |
| file_size        | BIGINT        | NULL                                        |
| checksum_hash    | VARCHAR(255)  | NULL                                        |
| preview_path     | VARCHAR(1000) | NULL                                        |
| thumbnail_path   | VARCHAR(1000) | NULL                                        |
| version_remark   | TEXT          | NULL                                        |
| is_latest        | TINYINT(1)    | Default 1                                   |
| is_frozen        | TINYINT(1)    | Default 0                                   |
| frozen_at        | DATETIME      | NULL                                        |
| frozen_by        | CHAR(36)      | FK → iam_users(id)                          |
| uploaded_by      | CHAR(36)      | FK → iam_users(id)                          |
| uploaded_at      | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE media_document_versions (
    id CHAR(36) PRIMARY KEY,
    document_id CHAR(36) NOT NULL,
    version_no VARCHAR(50) NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    original_name VARCHAR(255),
    object_path VARCHAR(1000),
    mime_type VARCHAR(255),
    file_extension VARCHAR(50),
    file_size BIGINT,
    checksum_hash VARCHAR(255),
    preview_path VARCHAR(1000),
    thumbnail_path VARCHAR(1000),
    version_remark TEXT,
    is_latest TINYINT(1) DEFAULT 1,
    is_frozen TINYINT(1) DEFAULT 0,
    frozen_at DATETIME,
    frozen_by CHAR(36),
    uploaded_by CHAR(36),
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_media_document_version
    FOREIGN KEY (document_id)
    REFERENCES media_documents(id)
    ON DELETE CASCADE,

    INDEX idx_document_id (document_id),
    INDEX idx_version_no (version_no),
    INDEX idx_is_latest (is_latest)
);
```

### Notes

Stores actual uploaded file versions.

Supports:
- version control
- frozen versions
- rollback support
- workflow integration

Example:

```text
Plumbing Drawing
   ├── V1
   ├── V2
   └── V3 (Frozen)
```

---

## **5.4 Table: `media_entity_links`**

| Column Name | Type          | Constraints / Notes                         |
| ------------| ------------- | ------------------------------------------- |
| id          | CHAR(36)      | PK (UUID)                                   |
| document_id | CHAR(36)      | FK → media_documents(id)                    |
| module_code | VARCHAR(100)  | NOT NULL                                    |
| reference_id| CHAR(36)      | NOT NULL                                    |
| created_by  | CHAR(36)      | FK → iam_users(id)                          |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE media_entity_links (
    id CHAR(36) PRIMARY KEY,
    document_id CHAR(36) NOT NULL,
    module_code VARCHAR(100) NOT NULL,
    reference_id CHAR(36) NOT NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_media_entity_document
    FOREIGN KEY (document_id)
    REFERENCES media_documents(id)
    ON DELETE CASCADE,

    INDEX idx_module_code (module_code),
    INDEX idx_reference_id (reference_id)
);
```

### Notes

Generic linking table for ERP-wide media usage.

Examples:

```text
DPR → DPR_ID
DRAWING → DRAWING_ID
TASK → TASK_ID
```

This architecture keeps Media Service module-independent.

---

## **5.5 Table: `media_tags`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| tag_name    | VARCHAR(150)  | UNIQUE, NOT NULL                  |
| color_code  | VARCHAR(20)   | NULL                              |
| description | TEXT          | NULL                              |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE media_tags (
    id CHAR(36) PRIMARY KEY,
    tag_name VARCHAR(150) NOT NULL UNIQUE,
    color_code VARCHAR(20),
    description TEXT,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_tag_name (tag_name)
);
```

### Notes

Reusable media tagging system.

Examples:
- approved
- revised
- urgent
- site-photo

---

## **5.6 Table: `media_document_tags`**

| Column Name | Type       | Constraints / Notes                  |
| ------------| ---------- | ------------------------------------ |
| id          | CHAR(36)   | PK (UUID)                            |
| document_id | CHAR(36)   | FK → media_documents(id)             |
| tag_id      | CHAR(36)   | FK → media_tags(id)                  |
| created_at  | DATETIME   | Default CURRENT_TIMESTAMP            |

### Table Creation Query

```sql
CREATE TABLE media_document_tags (
    id CHAR(36) PRIMARY KEY,
    document_id CHAR(36) NOT NULL,
    tag_id CHAR(36) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_document_tag_document
    FOREIGN KEY (document_id)
    REFERENCES media_documents(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_document_tag_tag
    FOREIGN KEY (tag_id)
    REFERENCES media_tags(id)
    ON DELETE CASCADE,

    INDEX idx_document_id (document_id),
    INDEX idx_tag_id (tag_id)
);
```

### Notes

Supports many-to-many document tagging.

---

## **5.7 Table: `media_previews`**

| Column Name   | Type          | Constraints / Notes                   |
| ------------- | ------------- | ------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                             |
| version_id    | CHAR(36)      | FK → media_document_versions(id)      |
| preview_type  | VARCHAR(100)  | THUMBNAIL / PREVIEW / LOW_RES         |
| preview_path  | VARCHAR(1000) | Object storage preview path           |
| width         | INT           | NULL                                  |
| height        | INT           | NULL                                  |
| generated_at  | DATETIME      | Default CURRENT_TIMESTAMP             |

### Table Creation Query

```sql
CREATE TABLE media_previews (
    id CHAR(36) PRIMARY KEY,
    version_id CHAR(36) NOT NULL,
    preview_type VARCHAR(100),
    preview_path VARCHAR(1000),
    width INT,
    height INT,
    generated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_preview_version
    FOREIGN KEY (version_id)
    REFERENCES media_document_versions(id)
    ON DELETE CASCADE,

    INDEX idx_version_id (version_id)
);
```

### Notes

Stores preview metadata for:
- PDFs
- images
- drawings

Used for frontend optimization.

---

## **5.8 Table: `media_access_logs`**

| Column Name | Type          | Constraints / Notes                    |
| ------------| ------------- | -------------------------------------- |
| id          | CHAR(36)      | PK (UUID)                              |
| version_id  | CHAR(36)      | FK → media_document_versions(id)       |
| access_type | VARCHAR(100)  | VIEW / DOWNLOAD / SHARE                |
| accessed_by | CHAR(36)      | FK → iam_users(id)                     |
| ip_address  | VARCHAR(100)  | NULL                                   |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE media_access_logs (
    id CHAR(36) PRIMARY KEY,
    version_id CHAR(36) NOT NULL,
    access_type VARCHAR(100),
    accessed_by CHAR(36),
    ip_address VARCHAR(100),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_access_log_version
    FOREIGN KEY (version_id)
    REFERENCES media_document_versions(id)
    ON DELETE CASCADE,

    INDEX idx_version_id (version_id),
    INDEX idx_access_type (access_type)
);
```

### Notes

Supports:
- audit tracking
- download tracking
- view analytics
- compliance monitoring

---

## **5.9 Table: `media_shares`**

| Column Name     | Type          | Constraints / Notes                    |
| ----------------| ------------- | -------------------------------------- |
| id              | CHAR(36)      | PK (UUID)                              |
| version_id      | CHAR(36)      | FK → media_document_versions(id)       |
| share_token     | VARCHAR(255)  | UNIQUE                                 |
| shared_with     | VARCHAR(255)  | Email/mobile/external user             |
| expiry_date     | DATETIME      | NULL                                   |
| is_active       | TINYINT(1)    | Default 1                              |
| created_by      | CHAR(36)      | FK → iam_users(id)                     |
| created_at      | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE media_shares (
    id CHAR(36) PRIMARY KEY,
    version_id CHAR(36) NOT NULL,
    share_token VARCHAR(255) UNIQUE,
    shared_with VARCHAR(255),
    expiry_date DATETIME,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_media_share_version
    FOREIGN KEY (version_id)
    REFERENCES media_document_versions(id)
    ON DELETE CASCADE,

    INDEX idx_share_token (share_token)
);
```

### Notes

Supports:
- external sharing
- temporary links
- client sharing
- vendor sharing

---

## **5.10 Table: `media_storage_usage`**

| Column Name   | Type           | Constraints / Notes                   |
| ------------- | -------------- | ------------------------------------- |
| id            | CHAR(36)       | PK (UUID)                             |
| project_id    | CHAR(36)       | FK → project_projects(id)             |
| total_files   | BIGINT         | Default 0                             |
| total_storage | BIGINT         | Default 0                             |
| calculated_at | DATETIME       | Default CURRENT_TIMESTAMP             |

### Table Creation Query

```sql
CREATE TABLE media_storage_usage (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36),
    total_files BIGINT DEFAULT 0,
    total_storage BIGINT DEFAULT 0,
    calculated_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_project_id (project_id)
);
```

### Notes

Supports:
- storage analytics
- usage monitoring
- billing integration
- quota management

  (in future if we store media files on cloud object storage)

---
---
# ---------------------------------------------------------

# 🟦 **6. TASK & EXECUTION COORDINATION SERVICE (TASK SERVICE)**

# ---------------------------------------------------------

This service manages:

- task creation
- task assignment
- execution coordination
- task hierarchy
- dependency tracking
- milestone tracking
- planned vs actual tracking
- task comments
- checklist management
- task activity logs
- execution scheduling
- reminder-ready task architecture

The Task Service acts as the centralized execution coordination and work management engine for the SiteMate ERP platform.

This service is intentionally designed to support construction execution workflows rather than only generic todo management.

---

# 🔶 Important Architectural Decision

The Task Service is designed as:

- execution-oriented
- scalable
- hierarchy-based
- dependency-aware
- project-integrated
- future scheduling-ready

The system supports:
- parent-child tasks
- multi-user assignments
- dependency tracking
- milestone tracking
- delay tracking
- activity logging

This architecture allows future expansion into:
- Gantt charts
- Primavera-style scheduling
- sprint/task planning
- execution analytics

without schema redesign.

---

# 🔶 Task Hierarchy Strategy

The system supports:

```text
Parent Task
   ├── Child Task
   ├── Subtask
   └── Milestone
```

using:

```text
parent_task_id
```

This architecture supports unlimited task hierarchy depth.

---

# 🔶 Dependency Tracking Strategy

Task dependencies are managed separately using:

```text
task_dependencies
```

Examples:

```text
Footing Work starts after Excavation completion
```

This is extremely important for:
- project scheduling
- delay analysis
- execution planning
- critical path tracking

---

# 🔶 Multi-User Assignment Strategy

Construction tasks often involve:
- Site Engineers
- QA Teams
- Contractors
- Supervisors

Therefore:
task assignments are stored in a separate table:

```text
task_assignments
```

This architecture supports:
- multiple assignees
- role-based execution
- collaborative task management

---

# 🔶 Planned vs Actual Tracking

The system stores:

```text
planned_start_date
planned_end_date

actual_start_date
actual_end_date
```

This enables:
- delay tracking
- productivity analytics
- execution performance reporting

This is VERY important for construction ERP systems.

---

# 🔶 Milestone Strategy

Milestones are supported using:

```text
is_milestone
```

inside:
```text
task_tasks
```

Examples:
- Basement Completed
- Slab Casting Completed
- Tower Handover

---

# 🔶 Media Integration Strategy

Task Service does NOT store actual files.

All attachments are managed by:
# Media Service

Task Service only stores:
- task metadata
- task references

This keeps architecture clean and scalable.

---

# 🔶 Workflow Integration Strategy

Task Service remains independent from Workflow Service.

However:
future task approvals can integrate with workflow engine without redesigning task schema.

This separation keeps services loosely coupled.

---

# 🔶 Activity Log Strategy

All important task actions are tracked in:

```text
task_activity_logs
```

Examples:

```text
TASK_CREATED
TASK_UPDATED
STATUS_CHANGED
ASSIGNED
COMMENTED
COMPLETED
REOPENED
```

This acts as centralized audit history for tasks.

---

# 🔶 Project Integration Strategy

Tasks are deeply integrated with:
- Project Service
- Execution Service

Each task can link to:
- project
- structure
- activity type

Example:

```text
Task:
Slab Casting

Linked To:
Tower A → Floor 2
```

This makes task execution context-aware.

---

## **6.1 Table: `task_statuses`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| status_name | VARCHAR(150)  | UNIQUE, NOT NULL                  |
| color_code  | VARCHAR(20)   | NULL                              |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_by  | CHAR(36)      | FK → iam_users(id)                |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE task_statuses (
    id CHAR(36) PRIMARY KEY,
    status_name VARCHAR(150) NOT NULL UNIQUE,
    color_code VARCHAR(20),
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_status_name (status_name)
);
```

### Notes

Dynamic task statuses.

Examples:
- OPEN
- IN_PROGRESS
- QA_PENDING
- DELAYED
- COMPLETED

---

## **6.2 Table: `task_priorities`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| priority_name | VARCHAR(100)| UNIQUE, NOT NULL                  |
| color_code  | VARCHAR(20)   | NULL                              |
| sequence_no | INT           | Default 0                         |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE task_priorities (
    id CHAR(36) PRIMARY KEY,
    priority_name VARCHAR(100) NOT NULL UNIQUE,
    color_code VARCHAR(20),
    sequence_no INT DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_priority_name (priority_name)
);
```

### Notes

Examples:
- LOW
- MEDIUM
- HIGH
- CRITICAL

---

## **6.3 Table: `task_tasks`**

| Column Name        | Type          | Constraints / Notes                         |
| ------------------ | ------------- | ------------------------------------------- |
| id                 | CHAR(36)      | PK (UUID)                                   |
| project_id         | CHAR(36)      | FK → project_projects(id)                   |
| parent_task_id     | CHAR(36)      | Self FK → task_tasks(id)                    |
| structure_id       | CHAR(36)      | FK → project_structures(id)                 |
| activity_type_id   | CHAR(36)      | FK → project_activity_types(id)             |
| task_status_id     | CHAR(36)      | FK → task_statuses(id)                      |
| priority_id        | CHAR(36)      | FK → task_priorities(id)                    |
| task_title         | VARCHAR(255)  | NOT NULL                                    |
| task_description   | TEXT          | NULL                                        |
| planned_start_date | DATETIME      | NULL                                        |
| planned_end_date   | DATETIME      | NULL                                        |
| actual_start_date  | DATETIME      | NULL                                        |
| actual_end_date    | DATETIME      | NULL                                        |
| completion_percent | DECIMAL(5,2)  | Default 0                                   |
| estimated_hours    | DECIMAL(10,2) | NULL                                        |
| actual_hours       | DECIMAL(10,2) | NULL                                        |
| is_milestone       | TINYINT(1)    | Default 0                                   |
| is_active          | TINYINT(1)    | Default 1                                   |
| created_by         | CHAR(36)      | FK → iam_users(id)                          |
| created_at         | DATETIME      | Default CURRENT_TIMESTAMP                   |
| updated_at         | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE task_tasks (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    parent_task_id CHAR(36),
    structure_id CHAR(36),
    activity_type_id CHAR(36),
    task_status_id CHAR(36),
    priority_id CHAR(36),
    task_title VARCHAR(255) NOT NULL,
    task_description TEXT,
    planned_start_date DATETIME,
    planned_end_date DATETIME,
    actual_start_date DATETIME,
    actual_end_date DATETIME,
    completion_percent DECIMAL(5,2) DEFAULT 0,
    estimated_hours DECIMAL(10,2),
    actual_hours DECIMAL(10,2),
    is_milestone TINYINT(1) DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_task_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_task_parent
    FOREIGN KEY (parent_task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id),
    INDEX idx_parent_task_id (parent_task_id),
    INDEX idx_structure_id (structure_id),
    INDEX idx_task_status_id (task_status_id)
);
```

### Notes

Main execution task table.

Supports:
- parent-child tasks
- milestones
- planned vs actual tracking
- execution hierarchy

---

## **6.4 Table: `task_assignments`**

| Column Name | Type         | Constraints / Notes                    |
| ------------| ------------ | -------------------------------------- |
| id          | CHAR(36)     | PK (UUID)                              |
| task_id     | CHAR(36)     | FK → task_tasks(id)                    |
| user_id     | CHAR(36)     | FK → iam_users(id)                     |
| assigned_by | CHAR(36)     | FK → iam_users(id)                     |
| assigned_at | DATETIME     | Default CURRENT_TIMESTAMP              |
| is_active   | TINYINT(1)   | Default 1                              |

### Table Creation Query

```sql
CREATE TABLE task_assignments (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    user_id CHAR(36) NOT NULL,
    assigned_by CHAR(36),
    assigned_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    is_active TINYINT(1) DEFAULT 1,

    CONSTRAINT fk_assignment_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id),
    INDEX idx_user_id (user_id)
);
```

### Notes

Supports multi-user task assignments.

---

## **6.5 Table: `task_dependencies`**

| Column Name      | Type       | Constraints / Notes                    |
| ---------------- | ---------- | -------------------------------------- |
| id               | CHAR(36)   | PK (UUID)                              |
| task_id          | CHAR(36)   | FK → task_tasks(id)                    |
| depends_on_task_id | CHAR(36) | FK → task_tasks(id)                    |
| dependency_type  | VARCHAR(50)| FINISH_TO_START / START_TO_START       |
| created_at       | DATETIME   | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_dependencies (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    depends_on_task_id CHAR(36) NOT NULL,
    dependency_type VARCHAR(50),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_dependency_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_dependency_reference_task
    FOREIGN KEY (depends_on_task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id),
    INDEX idx_depends_on_task_id (depends_on_task_id)
);
```

### Notes

Supports task scheduling and execution dependency tracking.

---

## **6.6 Table: `task_comments`**

| Column Name  | Type         | Constraints / Notes                    |
| -------------| ------------ | -------------------------------------- |
| id           | CHAR(36)     | PK (UUID)                              |
| task_id      | CHAR(36)     | FK → task_tasks(id)                    |
| comment_text | TEXT         | NOT NULL                               |
| created_by   | CHAR(36)     | FK → iam_users(id)                     |
| created_at   | DATETIME     | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_comments (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    comment_text TEXT NOT NULL,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_comment_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id)
);
```

### Notes

Stores:
- discussions
- updates
- coordination comments
- execution communication

---

## **6.7 Table: `task_checklists`**

| Column Name   | Type         | Constraints / Notes                    |
| ------------- | ------------ | -------------------------------------- |
| id            | CHAR(36)     | PK (UUID)                              |
| task_id       | CHAR(36)     | FK → task_tasks(id)                    |
| checklist_text| VARCHAR(500) | NOT NULL                               |
| is_completed  | TINYINT(1)   | Default 0                              |
| completed_by  | CHAR(36)     | FK → iam_users(id)                     |
| completed_at  | DATETIME     | NULL                                   |
| created_at    | DATETIME     | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_checklists (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    checklist_text VARCHAR(500) NOT NULL,
    is_completed TINYINT(1) DEFAULT 0,
    completed_by CHAR(36),
    completed_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_checklist_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id)
);
```

### Notes

Supports execution checklist tracking.

Examples:
- QA Checked
- Safety Approved
- Material Received

---

## **6.8 Table: `task_activity_logs`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| task_id       | CHAR(36)      | FK → task_tasks(id)                    |
| action_type   | VARCHAR(100)  | TASK_CREATED / STATUS_CHANGED etc.     |
| previous_value| TEXT          | NULL                                   |
| new_value     | TEXT          | NULL                                   |
| remarks       | TEXT          | NULL                                   |
| performed_by  | CHAR(36)      | FK → iam_users(id)                     |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_activity_logs (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    action_type VARCHAR(100) NOT NULL,
    previous_value TEXT,
    new_value TEXT,
    remarks TEXT,
    performed_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_activity_log_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id),
    INDEX idx_action_type (action_type)
);
```

### Notes

Centralized task audit/history table.

Examples:
- TASK_CREATED
- ASSIGNED
- STATUS_CHANGED
- COMPLETED
- REOPENED

---

## **6.9 Table: `task_time_logs`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| task_id       | CHAR(36)      | FK → task_tasks(id)                    |
| user_id       | CHAR(36)      | FK → iam_users(id)                     |
| start_time    | DATETIME      | NOT NULL                               |
| end_time      | DATETIME      | NULL                                   |
| total_hours   | DECIMAL(10,2) | NULL                                   |
| remarks       | TEXT          | NULL                                   |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_time_logs (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    user_id CHAR(36),
    start_time DATETIME NOT NULL,
    end_time DATETIME,
    total_hours DECIMAL(10,2),
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_time_log_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id),
    INDEX idx_user_id (user_id)
);
```

### Notes

Supports:
- productivity tracking
- work-hour analysis
- execution analytics

---

## **6.10 Table: `task_reminders`**

| Column Name    | Type          | Constraints / Notes                    |
| -------------- | ------------- | -------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                              |
| task_id        | CHAR(36)      | FK → task_tasks(id)                    |
| reminder_type  | VARCHAR(100)  | EMAIL / PUSH / SMS                     |
| reminder_time  | DATETIME      | NOT NULL                               |
| is_sent        | TINYINT(1)    | Default 0                              |
| sent_at        | DATETIME      | NULL                                   |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_reminders (
    id CHAR(36) PRIMARY KEY,
    task_id CHAR(36) NOT NULL,
    reminder_type VARCHAR(100),
    reminder_time DATETIME NOT NULL,
    is_sent TINYINT(1) DEFAULT 0,
    sent_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_reminder_task
    FOREIGN KEY (task_id)
    REFERENCES task_tasks(id)
    ON DELETE CASCADE,

    INDEX idx_task_id (task_id),
    INDEX idx_reminder_time (reminder_time)
);
```

### Notes

Future-ready reminder engine.

Supports:
- due reminders
- overdue alerts
- escalation notifications

---

## **6.11 Table: `task_labels`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| label_name  | VARCHAR(150)  | UNIQUE, NOT NULL                  |
| color_code  | VARCHAR(20)   | NULL                              |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE task_labels (
    id CHAR(36) PRIMARY KEY,
    label_name VARCHAR(150) NOT NULL UNIQUE,
    color_code VARCHAR(20),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Notes

Reusable task labels/tags.

Examples:
- urgent
- blocked
- QA
- client-review

---

## **6.12 Table: `task_sprints`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| project_id    | CHAR(36)      | FK → project_projects(id)              |
| sprint_name   | VARCHAR(255)  | NOT NULL                               |
| start_date    | DATE          | NULL                                   |
| end_date      | DATE          | NULL                                   |
| description   | TEXT          | NULL                                   |
| created_by    | CHAR(36)      | FK → iam_users(id)                     |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE task_sprints (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    sprint_name VARCHAR(255) NOT NULL,
    start_date DATE,
    end_date DATE,
    description TEXT,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_sprint_project
    FOREIGN KEY (project_id)
    REFERENCES project_projects(id)
    ON DELETE CASCADE,

    INDEX idx_project_id (project_id)
);
```

### Notes

Future-ready sprint/planning support.

Can later support:
- execution phases
- planning cycles
- agile planning
- work batches

---
---

# ---------------------------------------------------------

# 🟨 **7. NOTIFICATION & COMMUNICATION SERVICE (NOTIFICATION SERVICE)**

# ---------------------------------------------------------

This service manages:

- in-app notifications
- email notifications
- WhatsApp notifications
- push notifications
- scheduled reminders
- notification queue processing
- delivery logs
- retry handling
- notification templates
- user notification preferences
- project notification preferences
- provider integrations
- communication history

The Notification Service acts as the centralized communication and notification engine for the entire SiteMate ERP platform.

This service is intentionally designed as an asynchronous event-driven system so that all ERP modules can send notifications without directly handling communication logic.

---

# 🔶 Important Architectural Decision

The Notification Service is designed as:

- asynchronous
- event-driven
- scalable
- reusable
- provider-independent
- future-ready

Other services should NEVER directly send:
- emails
- WhatsApp messages
- push notifications

Instead:

```text
Service
   ↓
Creates notification event
   ↓
Notification Queue
   ↓
Background Worker
   ↓
Notification Delivery
```

This architecture prevents:
- API blocking
- slow response times
- notification failures affecting business operations

---

# 🔶 Centralized Notification Strategy

The system uses a single centralized notification table:

```text
notification_notifications
```

This table supports:

- in-app notifications
- email notifications
- WhatsApp notifications
- push notifications

using:

```text
channel_type
```

Examples:

```text
IN_APP
EMAIL
WHATSAPP
PUSH
```

This architecture keeps the notification engine simple and scalable.

---

# 🔶 Dynamic Navigation Strategy

Notifications support frontend redirection using:

```text
module_code
entity_id
redirect_path
```

Example:

```text
module = TASK
entity_id = TASK_ID
redirect_path = /tasks/details/123
```

This allows frontend applications to dynamically open:
- task details
- drawing pages
- workflow approvals
- DPR details

directly from notifications.

---

# 🔶 Seen Tracking Strategy

The system supports:

```text
is_seen
seen_date
```

This enables:
- unread notification counters
- notification center
- read/unread tracking

which is mandatory for enterprise applications.

---

# 🔶 Notification Expiry Strategy

Notifications support:

```text
expiry_date
```

Benefits:
- temporary alerts
- auto-expiring reminders
- cleanup optimization
- better notification management

---

# 🔶 Notification Logs Strategy

Notification delivery history is stored separately in:

```text
notification_logs
```

This is VERY important because:

Notification entity
≠
delivery attempts

The logs table stores:
- sent status
- failed status
- retries
- provider responses
- delivery history

This architecture supports:
- debugging
- retry handling
- audit history
- provider troubleshooting

---

# 🔶 Queue Processing Strategy

The system uses:

```text
notification_queue
```

for asynchronous processing.

This allows:
- background delivery
- retries
- scheduled notifications
- cron processing
- scalable communication handling

This is enterprise-grade architecture.

---

# 🔶 Notification Template Strategy

Templates are stored separately using:

```text
notification_templates
```

Supports:
- dynamic placeholders
- email templates
- WhatsApp templates
- push templates

Example:

```text
Hello {{username}},
Task {{task_name}} assigned to you.
```

This avoids hardcoding messages inside services.

---

# 🔶 Notification Preferences Strategy

Separate preference tables are maintained for:

- user-level preferences
- project-level preferences

This architecture supports:

- personalized notification settings
- project-specific communication rules
- future advanced notification controls

---

# 🔶 Retry & Delivery Tracking Strategy

Notification retries are supported using:

```text
retry_count
next_retry_at
```

This is very important because:
- email providers may fail
- WhatsApp APIs may timeout
- push notifications may fail temporarily

---

# 🔶 Future Expansion Support

The architecture supports future features:

- Firebase Push Notifications
- SMS Gateway
- Slack Notifications
- Microsoft Teams Integration
- Notification Analytics
- Read Receipts
- Multi-language Templates
- AI-generated Messages

without schema redesign.

---

## **7.1 Table: `notification_types`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| type_code   | VARCHAR(100)  | UNIQUE, NOT NULL                  |
| type_name   | VARCHAR(150)  | NOT NULL                          |
| description | TEXT          | NULL                              |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE notification_types (
    id CHAR(36) PRIMARY KEY,
    type_code VARCHAR(100) NOT NULL UNIQUE,
    type_name VARCHAR(150) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_type_code (type_code)
);
```

### Notes

Dynamic notification type master.

Examples:
- TASK_ASSIGNED
- WORKFLOW_APPROVED
- DPR_SUBMITTED
- REMINDER
- ESCALATION

---

## **7.2 Table: `notification_templates`**

| Column Name    | Type          | Constraints / Notes                     |
| -------------- | ------------- | --------------------------------------- |
| id             | CHAR(36)      | PK (UUID)                               |
| notification_type_id | CHAR(36)| FK → notification_types(id)             |
| channel_type   | VARCHAR(100)  | EMAIL / WHATSAPP / PUSH                 |
| template_name  | VARCHAR(255)  | NOT NULL                                |
| subject        | VARCHAR(500)  | NULL                                    |
| template_body  | LONGTEXT      | NOT NULL                                |
| variables_json | JSON          | Supported placeholders                  |
| is_active      | TINYINT(1)    | Default 1                               |
| created_by     | CHAR(36)      | FK → iam_users(id)                      |
| created_at     | DATETIME      | Default CURRENT_TIMESTAMP               |
| updated_at     | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE notification_templates (
    id CHAR(36) PRIMARY KEY,
    notification_type_id CHAR(36),
    channel_type VARCHAR(100),
    template_name VARCHAR(255) NOT NULL,
    subject VARCHAR(500),
    template_body LONGTEXT NOT NULL,
    variables_json JSON,
    is_active TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_notification_template_type
    FOREIGN KEY (notification_type_id)
    REFERENCES notification_types(id),

    INDEX idx_notification_type_id (notification_type_id)
);
```

### Notes

Reusable communication templates.

Supports:
- email templates
- WhatsApp templates
- push notification templates

---

## **7.3 Table: `notification_notifications`**

| Column Name      | Type          | Constraints / Notes                         |
| ---------------- | ------------- | ------------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                                   |
| project_id       | CHAR(36)      | FK → project_projects(id)                   |
| user_id          | CHAR(36)      | FK → iam_users(id)                          |
| notification_type_id | CHAR(36)  | FK → notification_types(id)                 |
| channel_type     | VARCHAR(100)  | IN_APP / EMAIL / WHATSAPP / PUSH            |
| title            | VARCHAR(500)  | NOT NULL                                    |
| message          | LONGTEXT      | NOT NULL                                    |
| module_code      | VARCHAR(100)  | NULL                                        |
| entity_id        | CHAR(36)      | NULL                                        |
| redirect_path    | VARCHAR(1000) | NULL                                        |
| expiry_date      | DATETIME      | NULL                                        |
| is_seen          | TINYINT(1)    | Default 0                                   |
| seen_date        | DATETIME      | NULL                                        |
| notification_status | VARCHAR(100)| PENDING / SENT / FAILED                    |
| retry_count      | INT           | Default 0                                   |
| next_retry_at    | DATETIME      | NULL                                        |
| provider_message_id | VARCHAR(500)| NULL                                       |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP                   |

### Table Creation Query

```sql
CREATE TABLE notification_notifications (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36),
    user_id CHAR(36),
    notification_type_id CHAR(36),
    channel_type VARCHAR(100) NOT NULL,
    title VARCHAR(500) NOT NULL,
    message LONGTEXT NOT NULL,
    module_code VARCHAR(100),
    entity_id CHAR(36),
    redirect_path VARCHAR(1000),
    expiry_date DATETIME,
    is_seen TINYINT(1) DEFAULT 0,
    seen_date DATETIME,
    notification_status VARCHAR(100) DEFAULT 'PENDING',
    retry_count INT DEFAULT 0,
    next_retry_at DATETIME,
    provider_message_id VARCHAR(500),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_notification_type
    FOREIGN KEY (notification_type_id)
    REFERENCES notification_types(id),

    INDEX idx_project_id (project_id),
    INDEX idx_user_id (user_id),
    INDEX idx_notification_status (notification_status),
    INDEX idx_is_seen (is_seen)
);
```

### Notes

Main centralized notification table.

Supports:
- in-app notifications
- email notifications
- WhatsApp notifications
- push notifications

Also supports:
- frontend redirection
- entity linking
- seen tracking
- expiry management
- retry handling

---

## **7.4 Table: `notification_logs`**

| Column Name      | Type          | Constraints / Notes                    |
| ---------------- | ------------- | -------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                              |
| notification_id  | CHAR(36)      | FK → notification_notifications(id)    |
| log_status       | VARCHAR(100)  | SENT / FAILED / RETRY                  |
| provider_response| LONGTEXT      | NULL                                   |
| error_message    | TEXT          | NULL                                   |
| logged_at        | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE notification_logs (
    id CHAR(36) PRIMARY KEY,
    notification_id CHAR(36) NOT NULL,
    log_status VARCHAR(100),
    provider_response LONGTEXT,
    error_message TEXT,
    logged_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_notification_log
    FOREIGN KEY (notification_id)
    REFERENCES notification_notifications(id)
    ON DELETE CASCADE,

    INDEX idx_notification_id (notification_id),
    INDEX idx_log_status (log_status)
);
```

### Notes

Stores delivery history and provider responses.

Supports:
- retry debugging
- delivery tracking
- audit history
- provider troubleshooting

---

## **7.5 Table: `notification_queue`**

| Column Name      | Type          | Constraints / Notes                    |
| ---------------- | ------------- | -------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                              |
| notification_id  | CHAR(36)      | FK → notification_notifications(id)    |
| queue_status     | VARCHAR(100)  | PENDING / PROCESSING / COMPLETED       |
| scheduled_at     | DATETIME      | NULL                                   |
| processed_at     | DATETIME      | NULL                                   |
| retry_count      | INT           | Default 0                              |
| next_retry_at    | DATETIME      | NULL                                   |
| created_at       | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE notification_queue (
    id CHAR(36) PRIMARY KEY,
    notification_id CHAR(36) NOT NULL,
    queue_status VARCHAR(100) DEFAULT 'PENDING',
    scheduled_at DATETIME,
    processed_at DATETIME,
    retry_count INT DEFAULT 0,
    next_retry_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_notification_queue
    FOREIGN KEY (notification_id)
    REFERENCES notification_notifications(id)
    ON DELETE CASCADE,

    INDEX idx_notification_id (notification_id),
    INDEX idx_queue_status (queue_status)
);
```

### Notes

Supports asynchronous background notification processing.

Very important for:
- scalability
- retries
- scheduled notifications
- delayed reminders

---

## **7.6 Table: `notification_user_preferences`**

| Column Name       | Type          | Constraints / Notes                     |
| ----------------- | ------------- | --------------------------------------- |
| id                | CHAR(36)      | PK (UUID)                               |
| user_id           | CHAR(36)      | FK → iam_users(id)                      |
| notification_type_id | CHAR(36)   | FK → notification_types(id)             |
| in_app_enabled    | TINYINT(1)    | Default 1                               |
| email_enabled     | TINYINT(1)    | Default 1                               |
| whatsapp_enabled  | TINYINT(1)    | Default 1                               |
| push_enabled      | TINYINT(1)    | Default 1                               |
| created_at        | DATETIME      | Default CURRENT_TIMESTAMP               |
| updated_at        | DATETIME      | Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

### Table Creation Query

```sql
CREATE TABLE notification_user_preferences (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    notification_type_id CHAR(36),
    in_app_enabled TINYINT(1) DEFAULT 1,
    email_enabled TINYINT(1) DEFAULT 1,
    whatsapp_enabled TINYINT(1) DEFAULT 1,
    push_enabled TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_user_preference_type
    FOREIGN KEY (notification_type_id)
    REFERENCES notification_types(id),

    INDEX idx_user_id (user_id)
);
```

### Notes

Stores user-specific notification preferences.

---

## **7.7 Table: `notification_project_preferences`**

| Column Name       | Type          | Constraints / Notes                     |
| ----------------- | ------------- | --------------------------------------- |
| id                | CHAR(36)      | PK (UUID)                               |
| project_id        | CHAR(36)      | FK → project_projects(id)               |
| notification_type_id | CHAR(36)   | FK → notification_types(id)             |
| in_app_enabled    | TINYINT(1)    | Default 1                               |
| email_enabled     | TINYINT(1)    | Default 1                               |
| whatsapp_enabled  | TINYINT(1)    | Default 1                               |
| push_enabled      | TINYINT(1)    | Default 1                               |
| created_by        | CHAR(36)      | FK → iam_users(id)                      |
| created_at        | DATETIME      | Default CURRENT_TIMESTAMP               |

### Table Creation Query

```sql
CREATE TABLE notification_project_preferences (
    id CHAR(36) PRIMARY KEY,
    project_id CHAR(36) NOT NULL,
    notification_type_id CHAR(36),
    in_app_enabled TINYINT(1) DEFAULT 1,
    email_enabled TINYINT(1) DEFAULT 1,
    whatsapp_enabled TINYINT(1) DEFAULT 1,
    push_enabled TINYINT(1) DEFAULT 1,
    created_by CHAR(36),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_project_preference_type
    FOREIGN KEY (notification_type_id)
    REFERENCES notification_types(id),

    INDEX idx_project_id (project_id)
);
```

### Notes

Stores project-level notification configuration.

Supports:
- project-specific reminder rules
- communication controls
- organization-specific workflows

---

## **7.8 Table: `notification_channels`**

| Column Name | Type          | Constraints / Notes               |
| ------------| ------------- | --------------------------------- |
| id          | CHAR(36)      | PK (UUID)                         |
| channel_code| VARCHAR(100)  | UNIQUE, NOT NULL                  |
| channel_name| VARCHAR(150)  | NOT NULL                          |
| is_active   | TINYINT(1)    | Default 1                         |
| created_at  | DATETIME      | Default CURRENT_TIMESTAMP         |

### Table Creation Query

```sql
CREATE TABLE notification_channels (
    id CHAR(36) PRIMARY KEY,
    channel_code VARCHAR(100) NOT NULL UNIQUE,
    channel_name VARCHAR(150) NOT NULL,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Notes

Examples:
- EMAIL
- WHATSAPP
- PUSH
- IN_APP

---

## **7.9 Table: `notification_provider_configs`**

| Column Name   | Type          | Constraints / Notes                    |
| ------------- | ------------- | -------------------------------------- |
| id            | CHAR(36)      | PK (UUID)                              |
| channel_id    | CHAR(36)      | FK → notification_channels(id)         |
| provider_name | VARCHAR(255)  | SMTP / Firebase / WhatsApp Cloud API   |
| config_json   | JSON          | Provider configuration                 |
| is_active     | TINYINT(1)    | Default 1                              |
| created_at    | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE notification_provider_configs (
    id CHAR(36) PRIMARY KEY,
    channel_id CHAR(36),
    provider_name VARCHAR(255),
    config_json JSON,
    is_active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_provider_channel
    FOREIGN KEY (channel_id)
    REFERENCES notification_channels(id)
);
```

### Notes

Stores provider-level configuration.

Examples:
- SMTP
- WhatsApp Cloud API
- Firebase Push Notifications

---

## **7.10 Table: `notification_webhooks`**

| Column Name      | Type          | Constraints / Notes                    |
| ---------------- | ------------- | -------------------------------------- |
| id               | CHAR(36)      | PK (UUID)                              |
| notification_id  | CHAR(36)      | FK → notification_notifications(id)    |
| webhook_payload  | LONGTEXT      | Provider webhook response              |
| webhook_status   | VARCHAR(100)  | DELIVERED / READ / FAILED              |
| received_at      | DATETIME      | Default CURRENT_TIMESTAMP              |

### Table Creation Query

```sql
CREATE TABLE notification_webhooks (
    id CHAR(36) PRIMARY KEY,
    notification_id CHAR(36),
    webhook_payload LONGTEXT,
    webhook_status VARCHAR(100),
    received_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_webhook_notification
    FOREIGN KEY (notification_id)
    REFERENCES notification_notifications(id)
    ON DELETE CASCADE,

    INDEX idx_notification_id (notification_id)
);
```

### Notes

Supports:
- provider delivery callbacks
- WhatsApp read receipts
- push delivery tracking
- webhook auditing

---
---
