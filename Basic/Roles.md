# 🔐 Roles in Snowflake

---

## 🧭 Overview

**Roles in Snowflake** are **used to manage access control and permissions**. They define **what a user or service account can do** within the Snowflake environment.  

Roles are part of **Snowflake’s role-based access control (RBAC)** system.

> 🧠 **In short:** Users are assigned **roles**, and roles have **privileges** to access databases, schemas, tables, warehouses, and other objects.

---

## 🧩 Key Concepts

| Concept | Description |
|---------|-------------|
| **Role** | Collection of privileges (e.g., read, write, execute) |
| **User** | Account that is assigned one or more roles |
| **Privilege** | Specific action allowed on an object (e.g., SELECT, INSERT) |
| **Object** | Database, schema, table, view, stage, warehouse, function, etc. |
| **Hierarchy** | Roles can inherit privileges from other roles |

---

## 🧱 Default Roles in Snowflake

| Role | Description |
|------|-------------|
| **ACCOUNTADMIN** | Highest level, can manage all aspects of Snowflake account |
| **SYSADMIN** | Administer databases, warehouses, and other objects |
| **SECURITYADMIN** | Manage users, roles, and grants |
| **USERADMIN** | Manage user accounts and assign roles |
| **PUBLIC** | Default role automatically granted to all users |
| **ORGADMIN** | For multi-account organizations, manage accounts |

---

## 🧩 Role Hierarchy

```

ACCOUNTADMIN
└── SECURITYADMIN
└── USERADMIN
└── Custom Roles

````

- Roles can be **nested**, inheriting privileges from other roles.
- Users can **switch roles** depending on the task.

---

## ⚙️ Creating Roles

### 🔹 Create a Custom Role
```sql
CREATE ROLE ETL_ROLE;
CREATE ROLE ANALYST_ROLE;
````

### 🔹 Grant Privileges to Roles

```sql
-- Grant access to database
GRANT USAGE ON DATABASE SALES_DB TO ROLE ANALYST_ROLE;

-- Grant access to schema
GRANT USAGE ON SCHEMA SALES_DB.PUBLIC TO ROLE ANALYST_ROLE;

-- Grant access to table
GRANT SELECT ON TABLE SALES_DB.PUBLIC.SALES TO ROLE ANALYST_ROLE;

-- Grant warehouse usage
GRANT USAGE ON WAREHOUSE ANALYTICS_WH TO ROLE ANALYST_ROLE;
```

---

## ⚙️ Assign Roles to Users

```sql
-- Create a user
CREATE USER vedant PASSWORD='StrongPass123' DEFAULT_ROLE=ANALYST_ROLE;

-- Assign additional role
GRANT ROLE ETL_ROLE TO USER vedant;
```

---

## ⚙️ Role Switching

A user with multiple roles can **switch roles** to perform different tasks.

```sql
-- Switch to ETL_ROLE
USE ROLE ETL_ROLE;

-- Switch to ANALYST_ROLE
USE ROLE ANALYST_ROLE;
```

> 🧠 Only one active role is used for the current session, but a user can have multiple roles assigned.

---

## 🧩 Granting Role to Another Role

```sql
-- Create hierarchy
GRANT ROLE ETL_ROLE TO ROLE ANALYST_ROLE;
```

* `ANALYST_ROLE` now inherits privileges from `ETL_ROLE`.

---

## ⚙️ Viewing Roles and Privileges

### 🔹 Show Roles

```sql
SHOW ROLES;
```

### 🔹 Show Grants to a Role

```sql
SHOW GRANTS TO ROLE ANALYST_ROLE;
```

### 🔹 Show Roles Assigned to a User

```sql
SHOW GRANTS TO USER vedant;
```

---

## 🧩 Common Privileges

| Object Type          | Privileges                               |
| -------------------- | ---------------------------------------- |
| Database             | USAGE, CREATE SCHEMA, MONITOR            |
| Schema               | USAGE, CREATE TABLE, CREATE VIEW         |
| Table/View           | SELECT, INSERT, UPDATE, DELETE, TRUNCATE |
| Stage                | READ, WRITE                              |
| Warehouse            | USAGE, MONITOR, OPERATE                  |
| Functions/Procedures | EXECUTE                                  |
| Roles                | OWNERSHIP, GRANT OPTION                  |

> 💡 `OWNERSHIP` allows full control, including the ability to **grant or revoke privileges**.

---

## ⚙️ Example: Role-Based Access

```
ROLE: ETL_ROLE
  - USAGE on WAREHOUSE ETL_WH
  - INSERT, UPDATE, DELETE on RAW.SALES
  - CREATE TABLE in ETL_DB.PUBLIC

ROLE: ANALYST_ROLE
  - SELECT on SALES_DB.PUBLIC.SALES
  - USE WAREHOUSE ANALYTICS_WH
  - Inherits ETL_ROLE (optional)
```

---

## 🔐 Security Best Practices

| Practice                     | Description                                       |
| ---------------------------- | ------------------------------------------------- |
| Principle of Least Privilege | Grant only necessary privileges to roles          |
| Separate Admin Roles         | ACCOUNTADMIN, SYSADMIN, SECURITYADMIN separated   |
| Role Hierarchy               | Use nested roles to simplify privilege management |
| Audit Role Usage             | Monitor `ACCOUNT_USAGE` views for role activity   |
| Revoke Unused Roles          | Clean up unnecessary roles periodically           |
| Use Default Roles Wisely     | Avoid giving PUBLIC role unnecessary privileges   |

---

## ⚡ Quick Reference SQL Commands

| Action                  | SQL Example                                            |
| ----------------------- | ------------------------------------------------------ |
| Create Role             | `CREATE ROLE analyst_role;`                            |
| Drop Role               | `DROP ROLE IF EXISTS analyst_role;`                    |
| Grant Role to User      | `GRANT ROLE analyst_role TO USER vedant;`              |
| Revoke Role from User   | `REVOKE ROLE analyst_role FROM USER vedant;`           |
| Grant Privilege to Role | `GRANT SELECT ON TABLE SALES TO ROLE analyst_role;`    |
| Revoke Privilege        | `REVOKE SELECT ON TABLE SALES FROM ROLE analyst_role;` |
| Show Roles              | `SHOW ROLES;`                                          |
| Show Grants to Role     | `SHOW GRANTS TO ROLE analyst_role;`                    |

---

## 🧠 Best Practices for Data Engineers

* ✅ Create **role hierarchy**: Admin → Power Users → Analysts → Viewers
* ✅ Assign **default roles** carefully for users
* ✅ Use **roles instead of individual privileges** for easier maintenance
* ✅ Monitor **role usage and audit logs** for security compliance
* ✅ Use **separate roles for warehouses, databases, ETL, and analytics**

---

## 🧩 Summary

| Concept            | Description                                          |
| ------------------ | ---------------------------------------------------- |
| **Role**           | Collection of privileges for access control          |
| **User**           | Assigned one or more roles                           |
| **Privilege**      | Specific action allowed on Snowflake objects         |
| **Hierarchy**      | Roles can inherit other roles                        |
| **Best Practices** | Least privilege principle, role separation, auditing |

> 💡 **Tip:** For secure and efficient Snowflake management, define clear **roles for admin, ETL, analysts, and viewers** and enforce role hierarchy.
