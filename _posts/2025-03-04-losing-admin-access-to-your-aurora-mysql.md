---
title: Losing access to your RDS Aurora MySQL
description: >-
  Highlighting a serious problem that can occur when managing user accounts in a database managed service.
author: Aldo
date: 2025-03-04 20:55:00 +0300
categories: [MySQL]
tags: [MySQL,Aurora]
pin: true
math: true
mermaid: true
image:
  path: /commons/losing-admin-access-to-rds-aurora.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Losing access to your RDS Aurora MySQL.
media_subpath: '/assets/'
---

## Introduction
Amazon Web Services (AWS) is the largest cloud provider with many enterprises choosing to deploy their applications and relational databases using managed services like Amazon RDS (Relational Database Service). Aurora for MySQL is a popular managed database service that is compatible with MySQL. However, certain actions can lead to critical issues, such as accidentally locking the administrative account, which can render the database inaccessible.

This issue was recently reported by a user in [mysqlcommunity.slack.com](https://mysqlcommunity.slack.com), highlighting a serious problem that can occur when managing user accounts in a database managed service.

## The Problem
Imagine a situation where a user that unintentionally executes a SQL statement that locks the admin account of an Aurora database. An example of such a command is:

```sql
ALTER USER 'admin'@'%' ACCOUNT LOCK;
```

At first, the issue might not be immediately apparent, but once the connection to the database is closed, attempts to log back in with the admin user will fail.

### Attempting to Reset Credentials
One might think that resetting the master credentials using the AWS Management Console or AWS CLI would resolve the issue. However, even after performing a credential reset, the following error persists:

```shell
ERROR 3118 (HY000): Access denied for user 'admin'@'%' because account is locked
```

### Why This Happens
When an account is explicitly locked in AWS Aurora MySQL, even a credential reset does not override the locked state. Since the admin user is typically the primary privileged account, this situation prevents users from performing further administrative actions in the environment during this period of restriction.

### Does This Affect Other Database Services?
This issue appears to be specific to **Amazon RDS Aurora for MySQL**. It does **not** occur with:
- **Amazon RDS for MySQL**
- **Amazon RDS for PostgreSQL**
- **Amazon RDS Aurora for PostgreSQL**

> In these other managed database services, resetting the master credentials restores access even if the account was previously locked.
{: .prompt-info }

## The Solution
Unfortunately, once the admin account is locked, there is no self-service way to resolve this issue. The only way to regain access is to open a support case with AWS and request assistance in unlocking the admin account. AWS support engineers can intervene and manually unlock the admin user to restore access.

## Avoid this Issue
To prevent such incidents, consider implementing the following best practices:
1. **Testing in non-production environments:** Before executing any security-related commands, test them in a non-production environment.
2. **Create an intermediate admin user:** Instead of relying solely on the admin user created during database setup, create and use additional admin accounts for your DBAs to perform operational tasks.

## Conclusion
Locking the admin account in Amazon RDS Aurora for MySQL can cause a critical outage of admin access, requiring AWS support intervention.

Understanding the risks and implementing best practices can help prevent accidental lockouts and ensure continued access to your database. Always be cautious when executing administrative SQL commands, especially those that modify user authentication settings.

Since this issue does not occur in other services, I would like to see AWS address and resolve it.