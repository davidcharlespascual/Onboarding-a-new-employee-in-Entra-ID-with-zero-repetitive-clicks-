# Automated Employee Onboarding with Microsoft Entra ID Lifecycle Workflows

A home lab project that automates new hire onboarding using **Microsoft Entra ID Governance (Lifecycle Workflows)**. Instead of repeating the same checklist for every new employee, one workflow enables the account, assigns licenses, adds the user to the right group, and sends a welcome email automatically.

> **Note:** This is a lab environment. User details were entered manually to stand in for an HR system, and the workflow was started with **Run on demand** for testing.

---

## Project Overview

| | |
|---|---|
| **Goal** | Automate the repetitive tasks of onboarding a new hire |
| **Platform** | Microsoft Entra ID Governance, Lifecycle Workflows |
| **Tenant** | evilcorpLAB (lab tenant) |
| **Licenses used** | Microsoft Entra Suite (trial), Microsoft 365 E3 |
| **Workflow template** | Onboard new hire employee |
| **Test user** | IT Level 1 new hire |

## The Flow

New hire details entered, then the workflow runs these tasks in order:

1. Enable user account
2. Assign licenses (Microsoft 365 E3 + Entra Suite)
3. Add user to the IT Level 1 group
4. Send welcome email

The new hire then signs in, sets a new password, and registers MFA.

## Prerequisites

Before building the workflow, make sure:

- **An Entra ID Governance license** (included in the Entra Suite) is assigned. Without it, the Lifecycle workflows page shows a **401 "You don't have access"** error.
- **The user has the attributes the workflow relies on:** Department, Job title, Employee hire date, Manager, and Usage location. Usage location is required before any license can be assigned.
- **The target group is a regular Security group with Assigned membership.** Groups with *"Microsoft Entra roles can be assigned to the group"* set to Yes (role-assignable) are greyed out and cannot be used by workflows.
- **The license task runs before the email task**, so the mailbox exists when the email is sent.

---

## Step-by-Step Walkthrough

### Step 1: Open Lifecycle Workflows

![Lifecycle workflows dashboard](Dashboard%20Lifecycle%20workflows.jpg)

1. Sign in to the **Microsoft Entra admin center** (entra.microsoft.com) as a Global Administrator.
2. Go to **ID Governance > Lifecycle workflows**.
3. The overview page shows the workflow schedule (Entra checks for matching users **every 3 hours** by default), how many schedules are enabled, and any alerts.
4. Click **Create workflow** to start.

### Step 2: Create the workflow from a template

1. On the template page, choose **Onboard new hire employee**.
2. **Basics tab:** give the workflow a name (I used `IT onboarding`). Leave the trigger as **Time based attribute**, **0 days**, **On**, **employeeHireDate**. These fields come from the template.
3. **Configure scope tab:** set the rule so only the right users are processed:
   - Property: `department`
   - Operator: `equal`
   - Value: the exact department text on the users (for example `IT Department`)

   The match must be exact, including capitalization and spaces. A user whose department text differs will not appear in the Run on demand list.

### Step 3: Choose the tasks

![Task selector](task%20selector%20window%20choose%20task%20to%20automate.jpg)

On the **Review tasks** tab, click **Add task**. The pane shows the built-in task library, filtered to the **Joiner** category:

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

I used four: **Enable User Account**, **Assign licenses to user**, **Add user to groups**, and **Send Welcome email**.

### Step 4: Configure the license task

![Select license](select%20%20license%20to%20automate.jpg)

1. Open the **Assign licenses to user** task.
2. Under **Select licenses**, tick **Microsoft 365 E3** and **Microsoft_Entra_Suite**.
3. Click **Select**, then **Save**.

The E3 license includes Exchange Online, which creates the user's mailbox. This task has to run **before** the welcome email, so the email has somewhere to land.

### Step 5: Configure the group task and set the task order

1. Open **Add user to groups** and select the **IT Level 1** group. If the group is greyed out, it is role-assignable and must be recreated as a normal Security group.
2. Leave **Continue workflow execution on error** unchecked, so failures are obvious while testing.
3. Use **Reorder** to set the final order:
   1. Enable User Account
   2. Assign licenses to user
   3. Add user to groups
   4. Send Welcome email

### Step 6: Create and view the workflow

![Workflow list](IT%20onboarding%20workflow%20view.jpg)

1. Click **Review + create**, check the summary, and click **Create**. Leave the schedule off for testing.
2. The **Workflows** page lists every workflow with its created date, schedule status, and enabled status. Here I have two: `Onboard new hire employee` (Sales) and `IT onboarding`.
3. The toolbar provides **Run on demand**, **Clone**, **Enable schedule**, and **Delete**.

### Step 7: Run the workflow on demand

1. Tick the checkbox next to the workflow and click **Run on demand**.
2. Click **Select users** and choose the new hire. If the user is missing, their department or job title doesn't match the scope rule.
3. Click **Run workflow**. Run on demand skips the schedule and the hire date check, so it is ideal for testing.

### Step 8: Check the results

![Workflow history](IT%20onboarding%20successul%20task.jpg)

Open the workflow and go to **Workflow history**. The result for this run:

- 1 user processed, **1 successful, 0 failed**
- 4 total tasks, **0 failed tasks**
- Status: **Completed**

The **Users**, **Runs**, and **Tasks** tabs show results per user, per run, and per task, and a failed task shows its error.

### Step 9: Verify the account was enabled

![Account enabled](account%20enabled.jpg)

The new hire's profile in **Entra ID > Users** shows **Account status: Enabled**, with 2 assigned licenses and group membership. No admin touched the account, and the workflow did all of it.

### Step 10: Verify the group membership

![Group membership](employee%20added%20to%20group%20task%20successful.jpg)

Open **Groups > IT Level 1 > Members**. The new hire is now in the member list, added by the **Add user to groups** task.

### Step 11: Check the welcome email

![Welcome email](welcome%20email%20to%20new%20employees.jpg)

The new hire received the welcome email in their new Outlook mailbox. It greets them by name, links to the My Apps portal, and names their manager as the contact for next steps.

### Step 12: First sign-in, password update

![Password update](new%20employee%20setting%20up%20new%20email%20pass.jpg)

On first sign-in, Entra requires the new hire to **update their password**, because the account was created with a temporary one.

### Step 13: First sign-in, MFA registration

![MFA registration](user%20first%20login%20%20to%20her%20email%20with%20mfa.jpg)

After the password change, the user is prompted to register **Microsoft Authenticator** by scanning a QR code, so the account is protected with MFA from day one.

---

## Troubleshooting

### The welcome email task failed

![Assigning a license manually](assign%20license%20so%20email%20will%20push%20through.jpg)

In an earlier test with Sales users, **Send Welcome email** failed, and the **Add user to groups** task after it never ran (it showed as *unprocessed*). The cause was a missing mailbox: those users had no Exchange license. I assigned **Microsoft 365 E3** in the Microsoft 365 admin center, and the email went through on the next run. In the final IT workflow, license assignment runs as a task before the email, which prevents the problem.

### Other issues I hit

| Issue | Cause | Fix |
|---|---|---|
| 401 on the Lifecycle workflows page | No ID Governance license | Start the Entra Suite trial and sign in again |
| Groups greyed out in the group task | Groups were role-assignable | Recreate them as normal Security groups with Assigned membership |
| User missing from the Run on demand list | Department text didn't match the scope rule | Make the department value identical |
| One failed task blocks the rest | The workflow stops on error by default | Order tasks carefully, or tick *Continue workflow execution on error* for non-critical tasks |
| Run on demand shows "successful" but history is empty | History takes a few minutes to update | Wait, then click Refresh |

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
