# Automated Employee Onboarding with Microsoft Entra ID Lifecycle Workflows

A home lab project that automates new hire onboarding using **Microsoft Entra ID Governance (Lifecycle Workflows)**. Instead of repeating the same checklist for every new employee, a workflow enables the account, assigns licenses, adds the user to the right group, and sends a welcome email automatically.

> **Note:** This is a lab environment. User details were entered manually to stand in for an HR system, and the workflow was started with **Run on demand** for testing.

---

## Overview

| | |
|---|---|
| **Goal** | Automate the repetitive tasks of onboarding a new hire |
| **Platform** | Microsoft Entra ID Governance, Lifecycle Workflows |
| **Tenant** | evilcorpLAB (lab tenant) |
| **Licenses used** | Microsoft Entra Suite (trial), Microsoft 365 E3 |
| **Workflow template** | Onboard new hire employee |
| **Test user** | IT Level 1 new hire |

## The Flow

New hire details entered, then the workflow starts, then:

1. Enable user account
2. Assign licenses (M365 E3 + Entra Suite)
3. Add user to the IT Level 1 group
4. Send welcome email

The new hire then signs in, sets a password, and registers MFA.

## Prerequisites

- An Entra ID Governance license (included in the Entra Suite) is required for Lifecycle Workflows. Without it the page returns a **401 "You don't have access"** error.
- The user needs the attributes the workflow relies on: **Department, Job title, Employee hire date, Manager, Usage location**.
- The target group must be a regular **Assigned** group. Role-assignable groups cannot be changed by workflows.
- The license task must run before the email task, so the mailbox exists.

---

## Step-by-Step Walkthrough

### 1. Lifecycle Workflows dashboard

![Lifecycle workflows dashboard](images/Dashboard_Lifecycle_workflows.jpg)

Everything starts in **ID Governance > Lifecycle workflows**. The overview shows the workflow schedule (Entra checks for matching users **every 3 hours** by default), how many schedules are enabled, and any alerts. From here I created the workflow with **Create workflow**.

### 2. Workflow list

![Workflow list](images/IT_onboarding_workflow_view.jpg)

The **Workflows** page lists every workflow with its created date, schedule status, and enabled status. Here I have two: `Onboard new hire employee` (Sales) and `IT onboarding`. The schedule is set to **No**, so they only run when started manually. The toolbar provides **Run on demand**, **Clone**, **Enable schedule**, and **Delete**.

### 3. Choosing the tasks

![Task selector](images/task_selector_window_choose_task_to_automate.jpg)

The **Select tasks** pane shows the built-in task library, filtered to the **Joiner** category. Available tasks include:

- Add user to groups
- Enable User Account
- Generate TAP and Send Email
- Send Welcome email
- Add user to selected teams
- Run a Custom Task Extension (for external systems)
- Send onboarding reminder email
- Request user access package assignment
- Assign licenses to user
- Update user attributes

For this workflow I used four: **Enable User Account**, **Assign licenses to user**, **Add user to groups**, and **Send Welcome email**.

### 4. Selecting the licenses

![Select license](images/select__license_to_automate.jpg)

In the **Assign licenses to user** task I selected **Microsoft 365 E3** and **Microsoft Entra Suite**. The E3 license includes Exchange Online, which creates the user's mailbox. This task has to run **before** the welcome email so the email has somewhere to land.

### 5. Workflow results

![Workflow history](images/IT_onboarding_successul_task.jpg)

After running the workflow, **Workflow history** shows:

- 1 user processed, **1 successful, 0 failed**
- 4 total tasks, **0 failed tasks**
- Status: **Completed**

### 6. Account enabled

![Account enabled](images/account_enabled.jpg)

The new hire's profile shows **Account status: Enabled**, with 2 assigned licenses and group membership. No admin touched the account, and the workflow did all of it.

### 7. Added to the group

![Group membership](images/employee_added_to_group_task_successful.jpg)

The **IT Level 1** group's member list now includes the new hire. The **Add user to groups** task did this automatically.

### 8. Welcome email

![Welcome email](images/welcome_email_to_new_employees.jpg)

The new hire received the welcome email in their new Outlook mailbox. It greets them by name, links to the My Apps portal, and names their manager as the contact for next steps.

### 9. First sign-in: password update

![Password update](images/new_employee_setting_up_new_email_pass.jpg)

On first sign-in, Entra requires the new hire to **update their password**, since the account was created with a temporary one.

### 10. First sign-in: MFA registration

![MFA registration](images/user_first_login__to_her_email_with_mfa.jpg)

After the password change, the user is prompted to register **Microsoft Authenticator** by scanning a QR code, so the account is protected with MFA from day one.

---

## Troubleshooting

### Welcome email task failed

![Assigning a license manually](images/assign_license_so_email_will_push_through.jpg)

In an earlier test with Sales users, the **Send Welcome email** task failed, and the **Add user to groups** task after it never ran (it showed as *unprocessed*). The cause was a missing mailbox: those users had no Exchange license. I assigned **Microsoft 365 E3** in the Microsoft 365 admin center, and the email went through on the next run. In the final IT workflow, license assignment runs as a task before the email, which prevents the problem.

### Other issues I hit

| Issue | Cause | Fix |
|---|---|---|
| 401 on the Lifecycle workflows page | No ID Governance license | Start the Entra Suite trial and sign in again |
| Groups greyed out in the group task | Groups were role-assignable | Recreate them as normal Security groups, Assigned |
| User missing from the Run on demand list | Department text didn't match the scope rule | Make the department value identical |
| One failed task blocks the rest | Workflow stops on error by default | Order tasks carefully, or tick *Continue workflow execution on error* for non-critical tasks |

## Lessons Learned

- **Task order matters.** The license has to come before the email.
- **Clean data is essential.** Consistent department names, hire dates, and managers make the scope rules work.
- **Role-assignable groups are off-limits** to workflows, by design.
- **Test with Run on demand first**, then enable the schedule once the tasks pass.
- **Separate workflows per department** keep scope rules and group assignments simple.

## Next Steps

- [ ] **Scheduled trigger test:** use a user whose hire date is today and enable the schedule, to prove the automatic trigger
- [ ] **Offboarding (Leaver) workflow:** disable the account, remove groups and licenses, and delete after a retention period
- [ ] **Automatic ticket creation:** raise a Zendesk ticket for resources like a laptop, using a custom task extension
- [ ] **HR-driven user creation:** create the Entra user automatically from an HR source instead of entering it by hand
- [ ] **Temporary Access Pass** task for the first sign-in

## Skills Demonstrated

Microsoft Entra ID, Identity Governance, Lifecycle Workflows, license management, group management, MFA, identity lifecycle automation, troubleshooting.
