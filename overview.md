# IAM Authorization Model

---

## 1. Overview

This [model](model.fga) defines a **hierarchical role-based access control (RBAC)** authorization system designed for a **multi-tenant environment**.  It governs how **principals** (users, groups, service accounts) gain access to **resources** (tenant, organization, projects, and project assets) through **roles** that define permissions.  Access is **inherited** through scope (tenant → organization → project → resource) and **expanded** through membership (groups and roles).

---

## 2. Principals

A **principal** is any identity that can be authenticated and authorized. This model defines three kinds of principals:

### a. User

A human account that signs in interactively.
**Example:**
`alice@example.com` — an employee logging into a dashboard.

### b. Service Account

A non-human account used by applications or automation.
**Example:**
`ci-deploy-bot@tenant-a` — a CI/CD pipeline identity that deploys code.

### c. Group

A collection of users, service accounts, or other groups (supports nesting).
**Example:**
`dev-team` includes Alice, Bob, and the service account for the CI pipeline.

**Key Point:**
Permissions assigned to a group automatically apply to all of its members.

---

## 3. Resources and Hierarchy

Resources are structured hierarchically, and permissions can be inherited downward.

| Level                 | Resource Type                              | Example                | Inherits Admin From |
| --------------------- | ------------------------------------------ | ---------------------- | ------------------- |
| **Tenant**            | Tenant                                     | `tenant:acme-corp`     | —                   |
| **Organization**      | Organization                               | `org:engineering`      | Tenant Admin        |
| **Project**           | Project                                    | `project:mobile-app`   | Organization Admin  |
| **Project Resources** | Tag, Application, Device, Operating System | `device:ios-test-unit` | Project Admin       |

### Example:

If *Alice* is a **Tenant Admin** for `tenant:acme-corp`, she automatically becomes:

* Admin for all Organizations under that tenant,
* Admin for all Projects within those organizations,
* Admin for all Devices, Applications, Tags, and OSes in those projects.

---

## 4. Roles and Permission Levels

Each Tenant defines **three built-in roles**:

| Role       | Typical Permissions                                | Example Actions                                             |
| ---------- | -------------------------------------------------- | ----------------------------------------------------------- |
| **Admin**  | Full control of resources and IAM policies.        | Add/remove members, delete projects, update access control. |
| **Writer** | Modify or create resources, no IAM policy control. | Create users, update configurations, deploy code.           |
| **Reader** | Read-only access to view resources.                | View project info, list devices, read logs.                 |

These roles are hierarchical:

```
Admin ⊃ Writer ⊃ Reader
```

**Example:**
If *Bob* is a **Writer**, he can automatically perform all Reader actions without needing a separate Reader assignment.

---

## 5. Role Assignments and Delegation

Roles define what actions are permitted; **assignments** determine who has them.

Principals can receive roles in three ways:

### a. Direct Assignment

A role is explicitly granted to a principal.

> Example: Grant Alice the “Project Admin” role on `project:mobile-app`.

### b. Group Membership

A principal inherits the role by being a member of a group that holds the role.

> Example: The `dev-team` group is assigned “Writer” on the project. Alice, as a member, automatically gains Writer permissions.

### c. Role Inclusion

A role can include assignees of another role (role nesting).

> Example: The “Lead Developer” role includes all members of “Writer,” granting them additional privileges like approving deployments.

---

## 6. Inheritance and Expansion

### a. Scope Inheritance

Permissions flow down the hierarchy:

* Tenant Admin → Organization Admin → Project Admin → Project Resources Admin

**Example:**
A Tenant Admin doesn’t need explicit access to each Project; their rights apply automatically.

### b. Group Expansion

Memberships are recursive.

> Example: Alice is in `dev-team`, which is in `engineering-group`.
> If `engineering-group` has “Reader” on all projects, Alice automatically inherits that permission.

### c. Role Expansion

Roles can include other roles’ assignees.

> Example: “Security Reviewer” includes all “Reader” assignees so that everyone who can view configurations can also perform compliance checks.

---

## 7. Policy Administration Model

Every resource type supports the same IAM management pattern.

| Permission Level   | Description                                          | Example                                  |
| ------------------ | ---------------------------------------------------- | ---------------------------------------- |
| **Admin**          | Full control of the resource and its IAM policy.     | Add/remove members, delete the resource. |
| **Policy Writer**  | Modify IAM policies but not other resource settings. | Add a new group to a policy.             |
| **Set IAM Policy** | Assign or revoke roles.                              | Grant a role to a user.                  |
| **Get IAM Policy** | View the current access configuration.               | Read who has access to the project.      |

Relationship:

```
Admin ⊃ Policy Writer ⊃ Set IAM Policy ⊃ Get IAM Policy
```

**Example:**
If Carol is a Project Admin, she can both change policies and view them.
If Dave only has “Get IAM Policy,” he can see who has access but not change anything.

---

## 8. Group Management

### a. Membership Control

Group membership can be modified by:

* Roles that allow membership management,
* Group Admins,
* Group Writers.

**Example:**
The `group-admin` role allows Sarah to add or remove members from the `dev-team` group.

### b. Membership Expansion

When a user is in a group, they automatically inherit that group’s roles and permissions.
Nested groups expand recursively.

**Example:**
If `dev-team` belongs to `engineering-group`, and `engineering-group` is a Project Writer, every `dev-team` member can modify project resources.

---

## 9. Role Management

Roles themselves are managed resources with their own IAM controls.

* **Assignees:** Can be users, service accounts, groups, or other roles.
* **Delegation:** A higher role can include the members of a lower role to extend privileges.
* **Policy Control:** Roles can have Admin, Policy Writer, Set/Get IAM permissions like other resources.

**Example:**
A “Support Operator” role might include all members of “Reader” and grant them additional permissions to reset devices or restart applications.

---

## 10. Organizations and Projects

### a. Organization

* Belongs to a Tenant.
* Inherits Tenant Admin privileges.
* Organization Admins can manage all Projects within that organization.

**Example:**
If Alice is an Admin for `tenant:acme-corp`, she’s automatically an Admin for `org:engineering` and all its projects.

### b. Project

* Belongs to an Organization.
* Inherits Admin from Organization Admins.
* Provides the local scope for applications, devices, and related resources.

**Example:**
If Bob is an Organization Admin, he’s automatically Admin for `project:mobile-app` and can assign roles there.

---

## 11. Project-Scoped Resources

Each project may contain:

* **Tags**
* **Applications**
* **Devices**
* **Operating Systems**

Each follows the same policy structure (Admin, Policy Writer, Set/Get IAM) and inherits permissions from its Project.

### a. Tags and Applications

Tags can reference Applications, grouping them for shared use.

**Example:**
The tag `frontend` may reference two Applications: `ui-service` and `auth-portal`.
Assigning `frontend` to a Device links it to both Applications.

### b. Devices

Devices can have:

* A **Tag** association (defining common behavior or access),
* Explicit **Application** links,
* Implicit access to any Application linked to their Tag.

**Example:**
If a Device is tagged with `frontend`, it automatically gains access to all Applications associated with that tag.

---

## 12. Access Propagation Summary

| Source             | Target                      | Result                           |
| ------------------ | --------------------------- | -------------------------------- |
| Tenant Admin       | All Organizations and below | Full inherited admin control     |
| Organization Admin | Projects and below          | Admin control over all projects  |
| Project Admin      | Project resources           | Admin on Tags, Apps, Devices, OS |
| Group Membership   | Group’s roles               | Same access as the group         |
| Role Inclusion     | Included role’s assignees   | Extended or delegated privileges |

---

## 13. Summary

This model defines a **multi-level, inheritance-based IAM system** that uses **standard RBAC principles**:

* **Principals:** Users, Groups, Service Accounts.
* **Roles:** Admin, Writer, Reader (plus dynamic/custom roles).
* **Resources:** Organized from Tenant down to Devices and Applications.
* **Inheritance:** Access flows downward from parent scopes.
* **Delegation:** Achieved through group nesting and role inclusion.
* **Policy Management:** Consistent across all resources (Admin → Policy Writer → Set → Get).
* **Examples:**

  * Tenant Admins automatically control everything below.
  * Group membership and role inclusion expand permissions logically.
  * Devices inherit Application access via Tags.

**Result:**
A consistent, scalable IAM model that supports complex enterprise authorization needs while staying compatible with common cloud IAM patterns (like those in AWS, GCP, or Azure).
