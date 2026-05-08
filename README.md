<div align="center">

<img src="https://img.shields.io/badge/Microsoft-Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white"/>
<img src="https://img.shields.io/badge/Microsoft_Entra_ID-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white"/>
<img src="https://img.shields.io/badge/PowerShell-7.6.1-5391FE?style=for-the-badge&logo=powershell&logoColor=white"/>
<img src="https://img.shields.io/badge/Microsoft_Graph_SDK-Hands--On-00BCF2?style=for-the-badge&logo=microsoft&logoColor=white"/>
<img src="https://img.shields.io/badge/Lab_Status-Completed-22c55e?style=for-the-badge"/>

<br/><br/>

# 🔐 SC-300 Lab 01 — Managing User Roles in Microsoft Entra ID

### A deep-dive portfolio record of what I built, observed, practiced, and learned — with concept explanations at every step

<img src="https://img.shields.io/badge/Course-SC--300_Identity_%26_Access_Admin-0078D4?style=flat-square&logo=microsoft&logoColor=white"/>
<img src="https://img.shields.io/badge/Tenant-Contoso_Demo-6366f1?style=flat-square"/>
<img src="https://img.shields.io/badge/Completed-May_2026-22c55e?style=flat-square"/>
<img src="https://img.shields.io/badge/Exercises-6_of_6_Completed-22c55e?style=flat-square"/>
<img src="https://img.shields.io/badge/Documented-First--Person-f59e0b?style=flat-square"/>

</div>

---

## 👋 Who This Is For & What This Covers

This repository is my personal, hands-on record of completing **SC-300 Lab 01** inside a live Microsoft Azure environment. It is written as both a **portfolio piece** and a **learning reference** — meaning every step is backed by a real screenshot I took, and every action is paired with a concept explanation so that anyone reading this (including my future self) understands not just *what* I did, but *why it works that way*.

Topics covered in this lab and documented here:

- How Microsoft Entra ID manages users and their identities
- What Role-Based Access Control (RBAC) means in practice
- How Privileged Identity Management (PIM) enforces secure role assignment
- How to provision users at scale using CSV bulk upload and PowerShell scripting
- How the Microsoft Graph API underpins everything in the Entra ecosystem
- How the user lifecycle (create → assign → modify → delete → restore) works end to end
- How Microsoft 365 licensing is managed and what prerequisites it requires

---

## 📋 Table of Contents

| # | Section |
|---|---|
| 1 | [🌐 Understanding the Environment](#-understanding-the-environment) |
| 2 | [💡 Core Concepts Before We Start](#-core-concepts-before-we-start) |
| 3 | [🏗 Exercise 1 — Creating a User & Observing Default Permissions](#-exercise-1--creating-a-user--observing-default-permissions) |
| 4 | [🛡 Exercise 2 — Assigning a Role & Watching Access Change Live](#-exercise-2--assigning-a-role--watching-access-change-live) |
| 5 | [🗑 Exercise 3 — Removing a Role via the Role Blade](#-exercise-3--removing-a-role-via-the-role-blade) |
| 6 | [📦 Exercise 4 — Bulk Provisioning via CSV & PowerShell](#-exercise-4--bulk-provisioning-via-csv--powershell) |
| 7 | [♻️ Exercise 5 — Deleting & Recovering a User Account](#️-exercise-5--deleting--recovering-a-user-account) |
| 8 | [🪪 Exercise 6 — Assigning a Microsoft 365 License](#-exercise-6--assigning-a-microsoft-365-license) |
| 9 | [🧠 Consolidated Takeaways — What I Learned](#-consolidated-takeaways--what-i-learned) |
| 10 | [⚡ PowerShell Command Reference](#-powershell-command-reference) |
| 11 | [📁 Repository Structure](#-repository-structure) |

---

## 🌐 Understanding the Environment

Before touching anything in the lab, I want to explain what I was working inside and why it matters to understand the landscape first.

### The Tenant — What Is It?

A **tenant** in Microsoft Entra ID (formerly Azure Active Directory) is a dedicated, isolated instance of the Microsoft identity platform that belongs to one organisation. Think of it like a private directory — it holds all the users, groups, applications, devices, and policies for that organisation and nobody else can see inside it.

In this lab, my tenant is called **Contoso** and its domain is `LODSM016355.onmicrosoft.com`. Every user account I create, every role I assign, and every license I manage lives inside this tenant boundary.

```
Tenant: Contoso
Domain: LODSM016355.onmicrosoft.com
Tenant ID: 51d70165-4842-xxxx-xxxx-xxxxxxxxxxxx
```

### The Two Portals I Used

| Portal | URL | Purpose |
|---|---|---|
| **Microsoft Entra Admin Center** | `entra.microsoft.com` | Identity management — users, roles, groups, apps, PIM |
| **Microsoft 365 Admin Center** | `admin.microsoft.com` | Billing and license management |

> These two portals are not identical. Some actions (like license assignment) can only be done in the M365 Admin Center, while role assignment and user creation live in Entra. Understanding which portal owns which function is practical knowledge.

### My Starting Position

I signed in as **MOD Administrator** — a Global Administrator account. The Global Administrator role is the highest privilege role in Entra ID. It can do everything: create users, assign any role, manage any application, and access all tenant settings. This is the role I used to set up and manage everything else.

| My Account | `admin@LODSM016355.onmicrosoft.com` |
|---|---|
| My Role | Global Administrator |
| Tenant | Contoso |

---

## 💡 Core Concepts Before We Start

These are the foundational ideas I needed to understand before the lab made full sense. I am documenting them here because they came up repeatedly across every exercise.

---

### 🔹 What Is Microsoft Entra ID?

Microsoft Entra ID is Microsoft's cloud-based **Identity and Access Management (IAM)** service. Its job is to answer two questions for every action attempted in your environment:

1. **Who are you?** (Authentication — proving identity)
2. **What are you allowed to do?** (Authorisation — enforcing permissions)

It handles sign-ins, manages user accounts, controls which applications people can access, and decides what administrative actions someone can perform. It is the backbone of access control for Microsoft 365, Azure, and thousands of third-party apps.

---

### 🔹 What Is Role-Based Access Control (RBAC)?

RBAC is the permission model Entra ID uses. Instead of assigning permissions directly to individual users (which becomes unmanageable at scale), you assign permissions to **roles**, and then assign **users to roles**.

```
Without RBAC:          With RBAC:
User A → Permission 1  Role: App Admin → Permission 1
User A → Permission 2               → Permission 2
User B → Permission 1               → Permission 3
User B → Permission 3  User A → App Admin Role
User C → Permission 2  User B → App Admin Role
(chaos at scale)       (clean, auditable, scalable)
```

In Entra ID there are over 100 built-in roles. In this lab I worked with two:

| Role | What It Can Do |
|---|---|
| **Global Administrator** | Everything — full control of the tenant |
| **Application Administrator** | Add, configure, and manage enterprise applications and app registrations — cannot manage users |

The key insight is that the **Application Administrator** role is scoped specifically. It gives exactly what's needed for app management and nothing more. This is the principle of **Least Privilege** in practice.

---

### 🔹 What Is Privileged Identity Management (PIM)?

PIM is a Microsoft Entra ID Premium P2 feature that adds a governance layer on top of standard role assignment. Without PIM, when you assign someone a role it is always active — they have those permissions permanently and continuously.

With PIM enabled, roles can be configured in two modes:

| Assignment Type | What It Means |
|---|---|
| **Eligible** | The user can request to activate the role when needed. It is not active until they activate it, and it expires after a set time window |
| **Active** | The role is permanently on, but PIM still requires a justification and creates an audit log of the assignment |

PIM also enforces:
- **Justification** — you must write a reason for assigning or activating a privileged role
- **Audit logs** — every assignment, activation, and removal is timestamped and logged
- **Time-bound assignments** — roles can expire automatically

In this lab my tenant had PIM enabled, which meant I had to go through the PIM assignment flow rather than a simple toggle. This mirrors what a real enterprise environment looks like.

---

### 🔹 What Is the Microsoft Graph API?

Every action in the Entra portal — creating a user, assigning a role, listing groups — is the portal calling the **Microsoft Graph API** on your behalf. Graph is a single unified REST API endpoint (`https://graph.microsoft.com`) that exposes the entire Microsoft 365 and Entra ID data model.

```
You (browser)  →  Entra Portal  →  Microsoft Graph API  →  Entra Directory
You (terminal) →  PowerShell    →  Microsoft Graph API  →  Entra Directory
```

When I used PowerShell with `New-MgUser`, I was calling the same API that the portal calls when you click "Create user" — just directly, without the GUI layer. This is why anything the portal can do, a script can automate.

---

### 🔹 What Is Soft Delete?

When you delete a user in Entra ID, they are not immediately and permanently gone. Instead, they enter a **soft-delete state** and are moved to a "Deleted users" bin. They remain recoverable for **30 days**. After 30 days, they are permanently and irreversibly removed.

This design exists because accidental deletions are common, and a hard-delete-by-default model would be catastrophic for IT operations.

---

Now with those concepts in place, everything in the lab will make more sense as we go through it.

---

## 🏗 Exercise 1 — Creating a User & Observing Default Permissions

### The Scenario

Contoso has hired a new employee who will serve as an Application Administrator. My job is to create their account and then verify (from their perspective) what they can and cannot do before any role is assigned.

---

### Step 1 — Sign In as Global Administrator

I opened the Microsoft Entra Admin Center at `entra.microsoft.com` and signed in with the MOD Administrator credentials.

<details>
<summary>📸 Admin sign-in screen</summary>
<br/>

![Admin Sign-In](screenshots/AZ_1_1.png)
> I used `admin@LODSM016355.onmicrosoft.com` — the Global Administrator account for the Contoso tenant. The `onmicrosoft.com` domain is the default domain Microsoft assigns to every new tenant.

</details>

After signing in, the Entra dashboard gives me an overview of the entire tenant at a glance:

<details>
<summary>📸 Microsoft Entra admin center — Contoso tenant dashboard</summary>
<br/>

![Entra Dashboard](screenshots/AZ_2.png)
> The dashboard shows: 33 users, 29 groups, 0 devices, and 6 apps registered in the tenant. My account (MOD Administrator) has 1 high-privileged role assignment — the Global Administrator role. The Identity Secure Score is 24.56%, which is a Microsoft-calculated metric reflecting how well the tenant's security settings align with best practices.

</details>

---

### Step 2 — Navigate to User Management

In the left navigation I expanded **Entra ID → Users → All Users**. This is the central directory listing of every identity in the tenant.

I clicked **`+ New user`** and selected **Create new user** from the dropdown.

<details>
<summary>📸 All Users → + New user → Create new user</summary>
<br/>

![Create New User](screenshots/AZ_3_1.png)
> Note there are two options: "Create new user" (an internal account managed by this tenant) and "Invite external user" (a guest account that links to someone's existing Microsoft account from another organisation). I chose "Create new user" because Chris Green is a new Contoso employee, not an external collaborator.

</details>

**Why does this distinction matter?**  
Internal users have accounts that live entirely within your tenant. External/guest users are linked to an identity in another directory. Their access, authentication requirements, and lifecycle management are different. Choosing the wrong one at onboarding creates problems later.

---

### Step 3 — Fill In the User Details

On the Create new user form I entered:

| Field | Value | Why It Matters |
|---|---|---|
| **User principal name** | `ChrisG` | This becomes the login username: `ChrisG@LODSM016355.onmicrosoft.com`. The UPN must be unique across the tenant. |
| **Display name** | `Chris Green` | This is the name shown in the directory, in emails, and in the portal. |
| **Auto-generate password** | ✅ Enabled | A random, compliant password is generated. The user must change it on first login. |

<details>
<summary>📸 The completed user creation form</summary>
<br/>

![User Form](screenshots/AZ_4.png)
> I copied the auto-generated password before clicking "Review + Create" — it is only shown once at this screen. If I miss it, I would need to trigger a password reset from the admin portal.

</details>

> [!IMPORTANT]
> The **User Principal Name (UPN)** is the primary login identifier in Entra ID. It follows email format (`user@domain.com`) and must be globally unique within your tenant. Once set, changing a UPN can break application integrations that rely on it as an identifier.

I clicked **Review + Create → Create**.  
✅ Chris Green now exists as a user in the Contoso directory.

---

### Step 4 — Sign In as Chris Green & Test Default Access

Now I needed to see the portal through Chris Green's eyes — not the admin's. To do this without logging out of my admin session, I opened a **separate InPrivate browser window**. This is important because browsers share session cookies across tabs, and I needed a completely isolated session.

<details>
<summary>📸 Chris Green signing in for the first time (InPrivate window)</summary>
<br/>

![Chris Green Login](screenshots/AZ_5_1.png)
> The "InPrivate" label is visible in the top bar, confirming this is a separate, isolated session from my admin browser. Chris Green's UPN is `chrisg@lodsm016355.onmicrosoft.com` — note that Entra ID treats UPNs as case-insensitive.

</details>

On first login, the portal immediately enforced a **mandatory password change**. This is a security feature — auto-generated passwords are considered temporary and the system forces the user to set their own before they can proceed.

<details>
<summary>📸 Mandatory first-login password change prompt</summary>
<br/>

![Password Update](screenshots/AZ_6.png)
> The form requires three fields: the current (temporary) password, and a new password entered twice for confirmation. Microsoft's password policy enforces complexity — the password must include uppercase, lowercase, numbers, and symbols, and cannot be something previously used or commonly known.

</details>

<details>
<summary>📸 "Stay signed in?" prompt after successful password reset</summary>
<br/>

![Stay Signed In](screenshots/AZ_7.png)
> This prompt appears after every successful authentication. Clicking "Yes" stores a Persistent Refresh Token in the browser, reducing re-authentication frequency. In a shared or public machine, "No" is the correct choice.

</details>

Now inside the portal as Chris Green, I searched for **Enterprise Applications** in the top search bar.

<details>
<summary>📸 Chris Green searching for Enterprise Applications</summary>
<br/>

![Enterprise Apps Search](screenshots/AZ_8.png)
> The search results show Enterprise Applications appears under "Services" — but notice the counters: Users (0), Groups (0), Devices (0). This confirms Entra ID's RBAC is already at work — Chris Green cannot enumerate users or groups in the directory because he has no role that grants that access.

</details>

I clicked **+ New Application** to attempt creating an enterprise app.

<details>
<summary>📸 "Create your own application" — greyed out and inaccessible</summary>
<br/>

![Create App Greyed Out](screenshots/AZ_9_1.png)
> The `+ Create your own application` option is visually greyed out and non-clickable. This is not just a UI design choice — the portal is checking Chris Green's permissions against the Entra ID directory in real time and rendering the interface based on what he is allowed to do. He has no role that grants application management rights, so the action is unavailable.

</details>

**What I observed here:**  
The UI is a direct reflection of the access model. Chris Green can navigate to the Enterprise Applications section and see it exists, but every action that requires elevated permissions is locked. This is RBAC working exactly as designed — read navigation is permitted, but write actions are gated by role.

I signed out of Chris Green's session.

<details>
<summary>📸 Signing out of Chris Green's InPrivate session</summary>
<br/>

![Sign Out](screenshots/AZ_10_2.png)

</details>

---

### What I Practiced & Demonstrated in Exercise 1

- Navigating the Entra ID admin center as a Global Administrator
- Understanding the difference between internal user creation and external user invitation
- Creating a user account with a UPN, display name, and auto-generated password
- Using an InPrivate browser window to simultaneously test a different user's session
- Observing how the portal UI enforces RBAC in real time — locked actions are not just hidden, they are access-controlled

---

## 🛡 Exercise 2 — Assigning a Role & Watching Access Change Live

### The Scenario

Chris Green needs Application Administrator rights to do his job. I need to assign the role, and then verify — from his session — that the permission change took effect.

### Understanding What I Am About to Assign

Before assigning anything, it is worth understanding the **Application Administrator** role precisely.

| Can Do | Cannot Do |
|---|---|
| ✅ Create and manage enterprise applications | ❌ Create or delete users |
| ✅ Configure single sign-on (SSO) for apps | ❌ Reset user passwords |
| ✅ Create app registrations | ❌ Manage other admin roles |
| ✅ Configure app permissions and consent | ❌ Access Azure subscription resources |
| ✅ Manage service principals | ❌ Read other users' full profiles |

This scope is intentional. The principle of least privilege says: give someone exactly what they need and nothing more. Chris Green needs to manage apps — not manage users, not manage billing, not touch security policies.

---

### Step 1 — Navigate to Chris Green's Assigned Roles

In my admin session I went to **Identity → Users → All Users → Chris Green → Assigned roles → + Add assignments**.

<details>
<summary>📸 The PIM Add Assignments panel — selecting Application Administrator</summary>
<br/>

![Select Role](screenshots/directory-role-select-role.png)
> The role dropdown lists every built-in Entra ID role alphabetically. "Application Administrator" is the second option visible here. The panel is titled "Privileged Identity Management | Microsoft Entra roles" — confirming PIM is handling this assignment rather than the basic role assignment UI. The breadcrumb at the top (Home > Users > Chris Green | Assigned roles) shows exactly where I am in the portal hierarchy.

</details>

---

### Step 2 — Configure the PIM Assignment

After selecting the role I clicked **Next** and reached the assignment settings screen.

<details>
<summary>📸 PIM assignment settings — Active, Permanent, with justification</summary>
<br/>

![PIM Assignment](screenshots/AZ_11_1.png)
> Three important fields are visible here:
> 
> **Assignment type: Active** — I chose Active rather than Eligible. If I had chosen Eligible, Chris Green would need to go into PIM himself and self-activate the role each time he needs it, within a time window. Active means it is always on.
> 
> **Permanently assigned** — The checkbox is ticked. This means there is no automatic expiry on this role assignment. In a real organisation, you might set a 90-day or 180-day expiry and require re-justification at renewal.
> 
> **Justification: "Needed for lab"** — PIM mandates a written justification for every privileged role assignment. This text is stored in the audit log. In a real environment, this would reference a ticket number, a manager approval, or a project name.
> 
> The assignment start date is shown as 05/05/2026, 5:04 PM — and the end date shows 11/01/2026 (the default range PIM pre-fills, even when "Permanently assigned" is selected for the activity window).

</details>

I clicked **Assign**.

---

### Step 3 — Verify the Change from Chris Green's Session

This was the most revealing moment of Exercise 2. I opened a new InPrivate window, signed in as Chris Green again, and navigated to Enterprise Applications → + New Application.

<details>
<summary>📸 "Create your own application" is now fully available</summary>
<br/>

![Create App Available](screenshots/AZ_12.png)
> The `+ Create your own application` text is now fully clickable (no longer greyed out) and clicking it opens a complete application creation panel on the right side of the screen. The panel asks for the app name and the type of application (proxy, development, or non-gallery integration). 
> 
> Compare this to Exercise 1 where this panel was inaccessible. The only thing that changed was the role assignment I made in the admin session — nothing else about Chris Green's account was modified.

</details>

**What I observed:**  
The permission propagated within seconds of the role being assigned. I did not need to log Chris Green out and back in. The next time his session made an authorisation check against Entra ID (when he navigated to the New Application page), the directory returned the updated access decision.

> **The key insight here:** Entra ID does not bake permissions into the session token at login time and leave them static. It checks authorisation continuously against the live directory. This means role changes (both grants and removals) take effect quickly — which is both a feature and a reason to be careful with privileged assignments.

---

### What I Practiced & Demonstrated in Exercise 2

- Understanding the difference between Eligible and Active PIM assignments
- Writing a justification for a privileged role assignment (real-world audit requirement)
- Understanding exactly what the Application Administrator role can and cannot do
- Proving that RBAC changes propagate and take effect without requiring the user to re-authenticate
- Seeing the Entra ID portal render different UI based on the live permission state of the signed-in user

---

## 🗑 Exercise 3 — Removing a Role via the Role Blade

### The Scenario

The role needs to be removed from Chris Green. But instead of going through Chris Green's user profile (the obvious path), I used the **Roles and Administrators** blade — approaching it from the role's perspective, not the user's.

### Why This Matters

In a large organisation you might have dozens of users assigned to a given role. Managing roles by going into each user's profile one by one is not scalable. The **Roles and Administrators** blade lets you see every holder of a specific role in one view — and manage them all from that single location. This is the more operationally realistic approach.

---

### Step 1 — Search for Roles

In the Entra admin center search bar I typed **Roles** and selected **Microsoft Entra roles and administrators**.

<details>
<summary>📸 Searching "Roles" in the Entra portal</summary>
<br/>

![Search Roles](screenshots/AZ_13_1.png)
> The search returns several relevant results under Services: "Microsoft Entra roles and administrators" is the first — this is the All Roles view. "Create custom Microsoft Entra roles" is also shown — this is where organisations can define roles beyond the 100+ built-in ones. I selected the first option.

</details>

---

### Step 2 — Find Application Administrator and Open Chris Green's Assignment

From the All Roles list I clicked **Application administrator**, then found Chris Green in the Assignments tab and clicked his row to open the Activity details.

<details>
<summary>📸 Chris Green's assignment activity details</summary>
<br/>

![Activity Details](screenshots/AZ_13.png)
> This panel shows the complete record of Chris Green's role assignment:
> 
> - **Member name:** Chris Green
> - **Role:** Application Administrator  
> - **Status:** Active
> - **Assignment start time:** 5/5/2026, 5:08:05 PM
> - **Assignment end time:** Permanent
> - **Scope:** Directory (meaning the role applies across the entire tenant, not just a subset)
> 
> At the bottom, the **Role activations** section shows the exact timestamp when the role was added: "Add member to role in PIM completed (permanent)" — this is the PIM audit trail. Even after I remove the role, this log entry will remain. PIM audit records are permanent.

</details>

---

### Step 3 — Remove the Role

I clicked **Remove** at the top of the Activity details panel.

<details>
<summary>📸 Confirming the role removal</summary>
<br/>

![Remove Role](screenshots/AZ_14_1.png)
> The confirmation dialog explicitly states: "Are you sure you want to remove assignment of 'Chris Green' from role 'Application Administrator'?" — I clicked Yes.
> 
> The dialog also shows the full activity detail below it, including the start time (5/5/2026, 5:08:05 PM), the Permanent end time, and the activation log. This design is intentional — it forces the administrator to see the full context before confirming a removal, reducing the chance of an accidental revocation.

</details>

✅ The Application Administrator role has been removed from Chris Green.

---

### What I Practiced & Demonstrated in Exercise 3

- Managing role assignments from the **role's perspective** rather than the user's — a more scalable real-world approach
- Reading and interpreting PIM Activity details (member name, role, status, scope, timestamps, activation log)
- Understanding that PIM audit logs are permanent — removal is tracked just as assignment was
- Understanding the difference between managing individual users vs. managing roles at scale

---

## 📦 Exercise 4 — Bulk Provisioning via CSV & PowerShell

### The Scenario

Contoso needs to onboard 10 new users across the Sales and Marketing departments. Doing this one by one through the portal is not efficient. I will use two methods: a CSV bulk upload through the portal, and then PowerShell scripting via the Microsoft Graph SDK.

---

### Task 1 — Bulk User Creation via CSV Upload

#### Understanding the CSV Template

The Entra bulk create CSV template has a specific structure. The first two rows are a version header and a column header row — these must not be modified. Data rows begin from row 3.

Required fields for each user:

| Column | Required? | Notes |
|---|---|---|
| `Name [displayName]` | ✅ Yes | Full display name |
| `User name [userPrincipalName]` | ✅ Yes | Must be `user@domain.com` format |
| `Initial password [passwordProfile]` | ✅ Yes | Must meet complexity requirements |
| `Block sign in [accountEnabled]` | ✅ Yes | `Yes` or `No` |
| `First name [givenName]` | Optional | |
| `Last name [surname]` | Optional | |
| `Department [department]` | Optional | |
| `Usage location [usageLocation]` | Optional | Required before license assignment |

---

#### Step 1 — Download and Edit the CSV Template

I navigated to **Identity → Users → All Users → Bulk operations → Bulk create** and clicked **Download** to get the template.

<details>
<summary>📸 Downloading the CSV template from the Bulk create panel</summary>
<br/>

![Download CSV](screenshots/AZ_16.png)
> The Bulk create panel has three steps: (1) Download CSV template, (2) Edit your CSV file, (3) Upload your CSV file. Step 1 is optional if you already have a correctly structured CSV, but downloading the template guarantees you have the right column order and version header.

</details>

I opened the file in Notepad and replaced the placeholder domain with the live tenant domain `LODSM016355.onmicrosoft.com`.

<details>
<summary>📸 The SC300BulkUser.csv file open in Notepad</summary>
<br/>

![CSV Content](screenshots/SC300BulkUser.png)
> The CSV is visible in plain text. Each row represents one user with comma-separated values. The first line is the version header (`version:v1.0,...`), the second line is the column headers, and data starts on line 3. You can see the 10 users I am importing — 6 in Sales (Malik Barden, Amina Hadzic, Lejla Selimagic, Omar Bennett, Jurgis Zukas, Bidisha Patowary) and 4 in Marketing (Andre Lawson, Monica Thomson, Edita Gabryte, Jelena Maric).
> 
> Each row uses `samplePassword1234$` as the initial password — this will be required to be changed on first login. Also note that `accountEnabled` is set to `No` for each row, which means the accounts are created in a disabled state. They would need to be enabled before the users can sign in.

</details>

---

#### Step 2 — Upload and Submit

Back in the Bulk create panel I clicked the folder icon, selected the edited CSV, and clicked **Submit**.

<details>
<summary>📸 CSV uploaded successfully — Submit clicked</summary>
<br/>

![CSV Uploaded](screenshots/Bulk_User_Creation1.png)
> The panel shows "SC300BulkUser.csv" in the file field and the message "File uploaded successfully" appeared — confirming the file passed Entra's format validation before submission. The Submit button is now active.

</details>

<details>
<summary>📸 Bulk create operation — In progress</summary>
<br/>

![In Progress](screenshots/Bulk_User_Creation.png)
> After clicking Submit, the status changes to "In progress" with a link to "Click here to view the status of each operation". Bulk operations run asynchronously in the background — the portal does not block while they complete. For large imports (hundreds of users), this can take several minutes.

</details>

---

#### Step 3 — Verify Results

I navigated to **Bulk operation results** to check the outcome.

<details>
<summary>📸 Bulk create results — all 10 users Status: Success</summary>
<br/>

![Bulk Success](screenshots/Bulk_User_Creation_2.png)
> Every row shows Status: **Success**, `accountEnabled: True`, and includes the **Object ID** assigned by Entra ID to each newly created user. The Object ID is the internal, immutable identifier for a directory object — it never changes even if the UPN or display name is updated. It is the identifier used by the Graph API, by PIM, and by audit logs to track an identity across its entire lifetime.

</details>

---

### Task 2 — Bulk User Creation via PowerShell (Microsoft Graph SDK)

This is where the lab moved from portal work into scripting — a fundamentally different and more powerful way to interact with Entra ID.

#### Why PowerShell and Microsoft Graph?

The portal GUI is appropriate for individual operations and one-time tasks. PowerShell via Microsoft Graph is appropriate when:

- You need to create, modify, or delete users in batches on a schedule
- You need to integrate identity operations with HR systems or other data sources
- You need to apply complex logic (conditionals, loops, error handling) that a GUI cannot express
- You want to version-control and audit your identity management operations as code

The Microsoft Graph PowerShell SDK (`Microsoft.Graph` module) is a collection of PowerShell cmdlets that are auto-generated from the Graph API's OpenAPI specification. Every cmdlet maps directly to a Graph API endpoint.

---

#### Step 1 — Check and Upgrade PowerShell Version

The Microsoft.Graph module requires PowerShell 7.2 or higher. I first checked my version:

```powershell
$PSVersionTable.PSVersion
```

This returned **Major: 5, Minor: 1** — too old. I needed to upgrade to PowerShell 7.

<details>
<summary>📸 Installing PowerShell 7.6.1 via winget</summary>
<br/>

![PS7 Install](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_1.png)
> I used `winget` — the Windows Package Manager — to search for and install PowerShell 7.6.1. The output shows: "Found PowerShell [Microsoft.PowerShell] Version 7.6.1.0", the installer hash was verified, the package installed, and "Successfully installed" is confirmed at the bottom.
> 
> Why winget over downloading manually? winget handles the download, verification, and installation in a single command, and the installation is reproducible — the same command will always install the same verified version.

</details>

---

#### Step 2 — Install the Microsoft.Graph Module

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Verbose
```

<details>
<summary>📸 Microsoft.Graph module installing from PSGallery</summary>
<br/>

![Graph Module Install](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_2.png)
> The `-Scope CurrentUser` flag installs the module only for my Windows user account, not system-wide — this avoids needing administrator privileges for the installation itself. The `-Verbose` flag shows detailed progress of what the installer is doing.
> 
> The terminal shows a warning: "Version '2.19.0' of module 'Microsoft.Graph' is already installed." This is because a previous version was present from an earlier attempt. The installer offers to install the new version (2.36.1) side-by-side.
> 
> The "Untrusted repository" prompt appears because PSGallery is not trusted by default — Microsoft's policy is to make you explicitly acknowledge you are installing from a public repository. Typing `Y` proceeds with the installation.

</details>

```powershell
Get-InstalledModule Microsoft.Graph    # verify installation
```

---

#### Step 3 — Authenticate to Microsoft Graph

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

This command opens a browser-based OAuth 2.0 authentication flow. The `-Scopes` parameter specifies which **delegated permissions** I am requesting. `User.ReadWrite.All` means: read and write all user profiles in the directory, on behalf of the signed-in admin.

<details>
<summary>📸 Microsoft Graph permissions consent dialog</summary>
<br/>

![Permissions Consent](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_3_1.png)
> The consent dialog is from the **Microsoft Graph Command Line Tools** application — this is a registered Microsoft application that the Graph PowerShell SDK uses as its OAuth client. It requests two permissions:
> 
> - "Read and write all users' full profiles" — needed for Get-MgUser and New-MgUser
> - "Maintain access to data you have given it access to" — this is the offline_access scope, needed for refresh tokens so the session can persist without re-prompting
> 
> The "Consent on behalf of your organization" checkbox — if checked — would grant these permissions to all users in the tenant. I left this unchecked to limit the consent to my admin session only.

</details>

<details>
<summary>📸 Entering admin credentials to authenticate the Graph connection</summary>
<br/>

![Graph Auth](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_3.png)
> I signed in with `admin@lodsm016355.onmicrosoft.com`. After authentication, the browser window can be closed and PowerShell will confirm the connection with "Welcome to Microsoft Graph!".

</details>

---

#### Step 4 — Verify Connection and Inspect the Tenant

```powershell
Get-MgUser
```

<details>
<summary>📸 "Welcome to Microsoft Graph!" — Get-MgUser listing all tenant users</summary>
<br/>

![Get-MgUser](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_4.png)
> The terminal confirms: "Welcome to Microsoft Graph!" and shows the connection details including the delegated access token ID. The `Get-MgUser` output returns every user in the tenant with three columns by default: `DisplayName`, `Id` (Object ID), and `Mail`.
> 
> I can see familiar names from the portal — Conf Room Adams, Adele Vance, MOD Administrator, Alex Wilber, Chris Green — confirming the PowerShell session is reading from the same live directory I was managing through the portal.

</details>

---

#### Step 5 — Define a Password Profile

Before creating a user, I had to define a password profile object — a hashtable that PowerShell will pass to the Graph API:

```powershell
$PWProfile = @{
    Password                      = "<complex password>"
    ForceChangePasswordNextSignIn = $false
}
```

The `ForceChangePasswordNextSignIn = $false` flag means the user will not be forced to change this password on first login. I set it this way for the lab, but in production you would typically set this to `$true` for new accounts.

---

#### Step 6 — Create a New User via Script

```powershell
New-MgUser `
    -DisplayName       "New PW User" `
    -GivenName         "New" -Surname "User" `
    -MailNickname      "newuser" `
    -UsageLocation     "US" `
    -UserPrincipalName "newuser@LODSM016355.onmicrosoft.com" `
    -PasswordProfile   $PWProfile -AccountEnabled `
    -Department        "Research" -JobTitle "Trainer"
```

<details>
<summary>📸 New PW User successfully created — Object ID and UPN confirmed</summary>
<br/>

![New User PS](screenshots/Bulk_Add_Users_Using_PowerShell_in_Microsoft_Entra_ID_5.png)
> The terminal output confirms the user was created successfully and returns the key attributes:
> 
> - **DisplayName:** New PW User
> - **Id:** `22b1b5da-b82a-47b8-9060-093e56feacfe` (the Object ID assigned by Entra ID)
> - **Mail:** (blank — no mailbox provisioned)
> - **UserPrincipalName:** `newuser@LODSM016355.onmicrosoft.com`
> 
> The red error at the top of the screenshot is from an earlier failed attempt — the first `Connect-MgGraph` call failed because of a listener error. I ran the command again and it succeeded. Troubleshooting a live terminal error in real time was a valuable part of this exercise.

</details>

---

### What I Practiced & Demonstrated in Exercise 4

- Understanding the structure and required fields of the Entra bulk create CSV template
- Performing a bulk user import via CSV and verifying results in the Bulk operation results view
- Upgrading PowerShell from 5.1 to 7.6.1 using winget from the command line
- Installing and verifying the Microsoft.Graph PowerShell module from PSGallery
- Authenticating to a live Azure tenant using delegated OAuth permissions via the Graph SDK
- Understanding the difference between delegated permissions and application permissions in Graph
- Reading `Get-MgUser` output to inspect the current state of a live directory
- Constructing a `New-MgUser` command with all key properties: UPN, display name, given/surname, mail nickname, usage location, password profile, department, and job title
- Troubleshooting a live `Connect-MgGraph` listener error in the terminal

---

## ♻️ Exercise 5 — Deleting & Recovering a User Account

### The Scenario

Chris Green's account needs to be deleted — simulating an off-boarding scenario. I then need to recover it, simulating what happens when a deletion is made in error or the off-boarding decision is reversed.

### Understanding Soft Delete vs. Hard Delete

In Entra ID, the standard Delete action performs a **soft delete**:

```
Active User → Delete → Soft-Deleted (recoverable, 30 days) → Auto-purge (permanent, day 31)
```

During the 30-day window the account:
- Cannot sign in
- Does not consume a license (licenses are released on deletion)
- Retains its Object ID, group memberships, and app assignments
- Is visible in the Deleted users view
- Can be fully restored

After 30 days the account is **hard-deleted** — the Object ID is destroyed, all relationships are severed, and the account cannot be recovered. If you then create a new user with the same UPN, it gets a brand new Object ID and has none of the previous account's history.

---

### Step 1 — Select and Delete Chris Green

In **All Users** I checked the **checkbox** next to Chris Green — not clicking his name link.

<details>
<summary>📸 Chris Green selected via checkbox — Delete button in toolbar</summary>
<br/>

![Select Delete](screenshots/Remove_a_user_from_Microsoft_Entra_ID.png)
> The checkbox (blue tick) next to Chris Green is selected. This activates the bulk-action toolbar at the top, including the Delete button. If I had clicked Chris Green's name instead, I would have opened his profile page — and the Delete button there only allows individual profile deletion, without the multi-select capability the checkbox approach provides.

</details>

I clicked **Delete** and confirmed with **OK**.

<details>
<summary>📸 "Delete the selected users?" confirmation dialog</summary>
<br/>

![Delete Confirm](screenshots/Remove_a_user_from_Microsoft_Entra_ID_2.png)
> The confirmation dialog lists the user being deleted and asks for explicit confirmation. This is a soft-delete — Chris Green moves to the Deleted users bin rather than being permanently removed.

</details>

---

### Step 2 — Find and Restore from Deleted Users

I clicked **Deleted users** in the left navigation panel.

<details>
<summary>📸 The Deleted Users view — Chris Green with permanent deletion date visible</summary>
<br/>

![Deleted Users](screenshots/Remove_a_user_from_Microsoft_Entra_ID_2.png)
> The Deleted users list shows:
> - **Chris Green** — deleted May 5, 2026 at 6:31 PM
> - **Permanent deletion date:** Jun 4, 2026 (exactly 30 days later)
> - **MngEnv Admin (DO NOT DELETE)** — another account in the soft-delete bin, with a label suggesting it should not be permanently removed (likely a lab environment service account)
> 
> I checked Chris Green's checkbox and clicked **Restore users** from the top toolbar.

</details>

---

### Bonus — Admin-Initiated Password Reset

After restoration I also demonstrated how an administrator can reset a user's password directly from the portal — without knowing the user's current password:

<details>
<summary>📸 Admin password reset panel for Chris Green</summary>
<br/>

![Password Reset](screenshots/Password_Reset.png)
> The Reset password panel shows Chris Green's profile with his monitoring data (2 sign-ins visible on the Monitoring tab). The panel on the right explains: "The user ChrisG@LODSM016355.onmicrosoft.com will be assigned a temporary password that must be changed on the next sign in."
> 
> This is a common helpdesk operation. An admin can reset any user's password without ever knowing their current one — the next login forces the user to set a new password themselves. The admin never sees the user's actual password at any point.

</details>

---

### What I Practiced & Demonstrated in Exercise 5

- Understanding the difference between soft delete (recoverable) and hard delete (permanent)
- Using the checkbox selection pattern rather than the name-link to enable bulk-action toolbar operations
- Navigating to the Deleted users view and identifying both the deletion timestamp and the permanent deletion deadline
- Restoring a soft-deleted user account
- Performing an admin-initiated password reset — a common IT support operation

---

## 🪪 Exercise 6 — Assigning a Microsoft 365 License

### The Scenario

**Raul Razo** needs a Windows 10/11 Enterprise E3 license assigned to his account. Before I can do that, I need to understand why licenses are managed separately from identity, and what prerequisites must be in place.

### Why Licenses Are Separate from Identity

User identity (who you are, what role you have) is managed by **Microsoft Entra ID**.  
Product licensing (what software you have access to) is managed by **Microsoft 365 Admin Center**.

These are related but distinct systems. Entra ID can *show* you what licenses a user has, but it cannot change them. License assignment, removal, and management is exclusively done in the M365 Admin Center. This separation reflects the fact that identity is a security/access concern while licensing is a commercial/billing concern.

### Understanding the Windows 10/11 Enterprise E3 License

| Feature | Detail |
|---|---|
| **License name** | Windows 10/11 Enterprise E3 |
| **Internal SKU** | `Win10_VDA_E3` |
| **What it enables** | Windows Enterprise features: BitLocker, AppLocker, DirectAccess, Windows Defender Credential Guard, and VDA rights (Virtual Desktop Access) |
| **Available in tenant** | 15 total, 13 available at time of assignment |
| **Assignment path** | Direct (assigned to individual user, not via group) |

---

### Step 1 — Verify Raul Has No License

I searched for **Raul** in All Users and opened Raul Razo's profile.

<details>
<summary>📸 Searching for and selecting Raul Razo</summary>
<br/>

![Search Raul](screenshots/Add_a_Windows_license_to_Raul.png)
> The search filtered to show 1 user found — Raul Razo (`RaulR@LODSM01...`). His checkbox is selected (blue tick). I then clicked his name to open his profile.

</details>

Inside his profile I clicked **Licenses** in the left nav. The page showed "No license assignments found" — confirming he had no active licenses.

Before assigning any license, I verified he had a **Usage Location** set. The Usage Location is a required attribute because Microsoft must know the user's country to apply the correct regional commercial and legal terms for the license.

> [!WARNING]
> If Usage Location is not set, the license assignment will appear to succeed but the license will not actually be applied. This is one of the more confusing silent failures in Entra ID administration.

---

### Step 2 — Assign the License via M365 Admin Center

I opened a new tab and navigated to `admin.microsoft.com`.

<details>
<summary>📸 Microsoft 365 Admin Center — Billing → Licenses</summary>
<br/>

![M365 Billing](screenshots/Add_a_Windows_10_license_to_a_user_account.png)
> The M365 Admin Center home page shows "Good evening, MOD Administrator" and the Billing section is expanded in the left nav, revealing: Your products, Licenses, Bills & payments, Billing accounts, Payment methods, Billing notifications, Pay-as-you-go, and Cost Management. I clicked **Licenses** to see all available license subscriptions.

</details>

I selected **Windows 10/11 Enterprise E3** from the subscriptions list, then clicked **+ Assign licenses**.

<details>
<summary>📸 Assigning Win 10/11 E3 to Raul Razo</summary>
<br/>

![Assign License](screenshots/Add_a_Windows_10_license_to_a_user_account_3.png)
> The Assign licenses panel slides in from the right. I searched for Raul Razo in the name/email field and selected him. The subscription dropdown shows "Subscription: Apr 15, 2026 (13 of 15 available licen..." — confirming there are available licenses to assign. Below this is "Select apps and services" which allows individual services within the license bundle to be selectively enabled or disabled for this user.
> 
> I clicked **Assign licenses** at the bottom.

</details>

<details>
<summary>📸 "Your licenses are being assigned" — confirmation</summary>
<br/>

![License Assigned](screenshots/Add_a_Windows_10_license_to_a_user_account_3_1.png)
> A green checkmark panel confirms: "Your licenses are being assigned. For larger batches of licenses, this process could take a while." The subscription meter on the left now shows 2/15 assigned (Miriam Graham and Raul Razo). License assignment is asynchronous — the portal confirms the request was accepted but the backend processing may take a short time to fully propagate.

</details>

---

### Step 3 — Verify the License in Entra ID

I switched back to the Entra admin center and navigated to **Raul Razo → Licenses**.

<details>
<summary>📸 Raul Razo's Licenses page — Win10_VDA_E3 Active</summary>
<br/>

![License Active](screenshots/Add_a_Windows_license_to_Raul_4_1.png)
> The Licenses page now shows one product entry:
> 
> | Field | Value | What It Means |
> |---|---|---|
> | **Products** | Win10_VDA_E3 | The internal SKU name for Windows 10/11 Enterprise E3 |
> | **State** | Active | The license is fully applied and the services are enabled |
> | **Enabled Services** | 6/6 | All 6 services included in this license bundle are enabled for Raul |
> | **Assignment Paths** | Direct | The license was assigned directly to this user, not inherited from a group |
> 
> Also visible at the top: a yellow banner reading "Adding, removing, and reprocessing licensing assignments is only available within the M365 Admin Center. Go to M365 Admin Center." — this confirms what I mentioned earlier: Entra ID can display license state but cannot manage it.

</details>

---

### What I Practiced & Demonstrated in Exercise 6

- Understanding why license management sits in M365 Admin Center rather than Entra ID
- Identifying the Usage Location prerequisite and why it is required (regional commercial terms)
- Navigating to the correct license SKU in M365 Admin Center and performing a user assignment
- Reading the license confirmation and understanding that assignment is asynchronous
- Verifying a license assignment in Entra ID and interpreting the State, Enabled Services, and Assignment Path fields
- Understanding the difference between **Direct** (individual) and **Group-based** license assignment

---

## 🧠 Consolidated Takeaways — What I Learned

### On Identity Architecture

**Entra ID is not just a login system — it is an access decision engine.**  
Every action in the portal, every API call, every resource access attempt goes through Entra ID for authorisation. Understanding that it is making real-time access decisions (not just checking a token stored at login) changes how you think about managing identities. Role changes take effect immediately. Permission checks are continuous.

**Tenants are isolated by design — and that isolation is total.**  
Nothing crosses tenant boundaries unless you explicitly configure it (B2B collaboration, cross-tenant synchronisation). This isolation is what makes multi-tenant SaaS architectures possible, and it is what gives organisations confidence that their identity data is not shared with other Microsoft customers.

---

### On Role-Based Access Control

**Least privilege is not a configuration you set — it is a default you build from.**  
New users have nothing. Every permission must be explicitly granted through a role. This forces administrators to think carefully about what each person actually needs rather than granting broad access by default. The Application Administrator role was a precise fit for Chris Green — app management without any of the higher-privilege capabilities he does not need.

**Managing roles from the role blade rather than the user profile scales infinitely.**  
When you manage from the user's profile, you can only see that user's roles. When you manage from the role blade, you can see every holder of that role across the entire directory. In an organisation with thousands of users, the role-centric view is the only practical approach.

---

### On Privileged Identity Management

**PIM fundamentally changes the security posture of privileged access.**  
Without PIM, a role assignment is static, silent, and always-on. With PIM, every assignment requires justification, every activation creates an audit record, and time-bound expiry forces periodic re-evaluation. The justification field might seem like a bureaucratic inconvenience, but it creates a traceable chain of accountability — who assigned what role, to whom, when, and why.

**Eligible vs. Active is a spectrum of trust.**  
Eligible assignments say: "You could have this power, but you must ask for it, and it will expire." Active assignments say: "You have this power now." For break-glass accounts and emergency access, Active permanent makes sense. For day-to-day admin work, Eligible with a time window is more secure.

---

### On Bulk Operations and Automation

**The GUI and the API are two interfaces to the same directory.**  
When I used `New-MgUser` in PowerShell and then saw the user appear in the Entra portal, it was not a synchronisation — it was the same underlying data. The portal is a UI layer on top of the Graph API, and PowerShell is a scripting layer on top of the same API. Knowing this means knowing that anything visible in the portal can be automated, and anything a script does will immediately appear in the portal.

**PowerShell scripting is identity management at scale.**  
The CSV bulk create is useful for one-time imports. PowerShell is useful when you need to run the same logic repeatedly — daily new hire onboarding, weekly licence reconciliation, monthly access reviews. The `New-MgUser` command I wrote is the foundation of every HR-driven provisioning workflow in Microsoft environments.

---

### On the User Lifecycle

**The 30-day soft-delete window is not a backup — it is a recovery window.**  
It exists specifically because accidental deletions happen. In a real IT environment, knowing this window exists (and how to access it) is operationally critical. Equally critical: knowing that after 30 days, the Object ID is destroyed and recovery is impossible. The permanent deletion date is visible in the portal — there is no excuse for missing it.

**Object IDs are the true identity of a directory object.**  
UPNs can change. Display names can change. Department, job title, even the domain can change. The Object ID never changes. Every external system that integrates with Entra ID (applications, logging systems, SIEM tools) should reference Object IDs, not UPNs, for stable tracking of a user across their entire lifecycle in the directory.

---

### On Licensing

**License management and identity management are intentionally separate.**  
Entra ID handles who you are. M365 Admin Center handles what software you have. This separation keeps the identity system clean and the billing system accountable. As an administrator you need to be comfortable navigating both and understanding which portal owns which function — because they do not fully overlap.

**Usage Location is not optional even when it looks optional.**  
The Usage Location field on a user profile is required before any license can be successfully applied. This is a legal and commercial requirement — Microsoft must know the country to apply the correct regional license terms. Not setting it before attempting license assignment is a silent failure mode that is easy to miss if you do not know to look for it.

---

## ⚡ PowerShell Command Reference

```powershell
# ── Check PowerShell version ──────────────────────────────────
$PSVersionTable.PSVersion

# ── Install PowerShell 7 via winget ───────────────────────────
winget search --id Microsoft.PowerShell --exact
winget install --id Microsoft.PowerShell --source winget

# ── Install and verify Microsoft.Graph module ─────────────────
Install-Module Microsoft.Graph -Scope CurrentUser -Verbose
Get-InstalledModule Microsoft.Graph

# ── Authenticate to Microsoft Graph ───────────────────────────
Connect-MgGraph -Scopes "User.ReadWrite.All"

# ── List all users in the tenant ──────────────────────────────
Get-MgUser

# ── Get a specific user ───────────────────────────────────────
Get-MgUser -Filter "displayName eq 'Chris Green'"

# ── Define a password profile ─────────────────────────────────
$PWProfile = @{
    Password                      = "YourP@ssword123!"
    ForceChangePasswordNextSignIn = $false
}

# ── Create a new user ─────────────────────────────────────────
New-MgUser `
    -DisplayName       "New PW User" `
    -GivenName         "New" `
    -Surname           "User" `
    -MailNickname      "newuser" `
    -UsageLocation     "US" `
    -UserPrincipalName "newuser@yourtenant.onmicrosoft.com" `
    -PasswordProfile   $PWProfile `
    -AccountEnabled    $true `
    -Department        "Research" `
    -JobTitle          "Trainer"

# ── Assign a role (Application Administrator) ─────────────────
$user   = Get-MgUser -Filter "displayName eq 'Chris Green'"
$roleId = (Get-MgDirectoryRole | Where-Object {
    $_.DisplayName -eq "Application Administrator"
}).Id
New-MgDirectoryRoleMember -DirectoryRoleId $roleId `
    -OdataId "https://graph.microsoft.com/v1.0/directoryObjects/$($user.Id)"

# ── Soft-delete a user ────────────────────────────────────────
Remove-MgUser -UserId $user.Id

# ── Restore a soft-deleted user ───────────────────────────────
Restore-MgDirectoryDeletedItem -DirectoryObjectId $user.Id

# ── List soft-deleted users ───────────────────────────────────
Get-MgDirectoryDeletedItemAsUser
```

---

## ✅ Lab Completion Record

| # | Exercise | What I Did | Result |
|---|---|---|---|
| 1 | Create User | Created ChrisG, verified zero default access from his session | ✅ |
| 2 | Assign Role | Assigned Application Administrator via PIM (Active/Permanent/Justified), verified live from ChrisG's session | ✅ |
| 3 | Remove Role | Removed role via Roles & Administrators blade (role-centric approach) | ✅ |
| 4a | Bulk CSV | Imported 10 users via SC300BulkUser.csv — all Status: Success | ✅ |
| 4b | PowerShell | Upgraded PS → installed Graph module → authenticated → created New PW User via New-MgUser | ✅ |
| 5 | Delete & Restore | Soft-deleted ChrisG, found in Deleted users bin with permanent date, restored, reset password | ✅ |
| 6 | License | Assigned Win10_VDA_E3 to Raul Razo via M365 Admin Center — Active, 6/6 services, Direct | ✅ |

---

## 📁 Repository Structure

```
azure-lab-01-manage-user-roles/
│
├── 📄 README.md                            ← This document
├── 📊 SC300BulkUser.csv                    ← The bulk user import file I used in Exercise 4
│
└── 📁 screenshots/                         ← Every screenshot taken live during my lab session
    ├── AZ_1_1.png                          Admin sign-in
    ├── AZ_2.png                            Contoso Entra ID dashboard
    ├── AZ_3_1.png                          + New user → Create new user
    ├── AZ_4.png                            User creation form (ChrisG)
    ├── AZ_5_1.png                          Chris Green first login (InPrivate)
    ├── AZ_5_2.png                          Password entry screen
    ├── AZ_6.png                            Mandatory first-login password change
    ├── AZ_7.png                            Stay signed in prompt
    ├── AZ_8.png                            Enterprise applications search (as ChrisG)
    ├── AZ_9_1.png                          Create app greyed out — no role
    ├── AZ_10_2.png                         Sign-out (ChrisG)
    ├── AZ_11_1.png                         PIM assignment — Active, Permanent, Justification
    ├── AZ_12.png                           Create app now available — role confirmed
    ├── AZ_13_1.png                         Search: Roles and administrators
    ├── AZ_13.png                           ChrisG activity details in PIM
    ├── AZ_14_1.png                         Remove role confirmation
    ├── AZ_16.png                           Bulk create CSV template download
    ├── SC300BulkUser.png                   CSV file content in Notepad
    ├── Bulk_User_Creation1.png             CSV uploaded + Submit
    ├── Bulk_User_Creation.png              Bulk create: In progress
    ├── Bulk_User_Creation_2.png            Bulk create: All 10 Status Success
    ├── Bulk_Add_..._1.png                  PowerShell 7.6.1 installed via winget
    ├── Bulk_Add_..._2.png                  Microsoft.Graph module install
    ├── Bulk_Add_..._3_1.png                Graph permissions consent dialog
    ├── Bulk_Add_..._3.png                  Admin password for Graph auth
    ├── Bulk_Add_..._4.png                  Get-MgUser tenant user list
    ├── Bulk_Add_..._5.png                  New-MgUser success output
    ├── directory-role-select-role.png      Application Administrator role dropdown
    ├── Remove_a_user_..._1.png             ChrisG checkbox selected + Delete
    ├── Remove_a_user_..._2.png             Delete confirm + Deleted users restore view
    ├── Password_Reset.png                  Admin password reset panel
    ├── Add_a_Windows_..._1.png             M365 Admin: Billing → Licenses
    ├── Add_a_Windows_license_Raul.png      Search and select Raul Razo
    ├── Add_a_Windows_..._3.png             Assign license panel (Raul added)
    ├── Add_a_Windows_..._3_1.png           "Licenses are being assigned" confirmed
    └── Add_a_Windows_..._4_1.png           Win10_VDA_E3 Active on Raul's profile
```

---

## 🔗 References

<div align="center">

[![SC-300](https://img.shields.io/badge/SC--300-Exam_Overview-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/certifications/exams/sc-300)
[![Entra ID Roles](https://img.shields.io/badge/Entra_ID-Built--in_Roles_Reference-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference)
[![PIM Docs](https://img.shields.io/badge/PIM-Documentation-6366f1?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/)
[![Graph SDK](https://img.shields.io/badge/Microsoft_Graph-PowerShell_SDK-5391FE?style=for-the-badge&logo=powershell&logoColor=white)](https://aka.ms/graph/sdk/powershell)
[![Graph API](https://img.shields.io/badge/Microsoft_Graph-API_Reference-00BCF2?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/graph/api/overview)

</div>

---

<div align="center">

*Lab environment: Contoso · `LODSM016355.onmicrosoft.com` · Completed May 2026*

<br/>

<img src="https://img.shields.io/badge/SC--300-In_Progress-f59e0b?style=flat-square&logo=microsoft&logoColor=white"/>
<img src="https://img.shields.io/badge/Lab_01-Completed-22c55e?style=flat-square"/>
<img src="https://img.shields.io/badge/All_Screenshots-Live_Lab-6366f1?style=flat-square"/>
<img src="https://img.shields.io/badge/Concepts-Explained-0078D4?style=flat-square"/>

</div>

