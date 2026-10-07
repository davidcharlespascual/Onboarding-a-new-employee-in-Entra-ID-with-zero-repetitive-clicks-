# Automated IT Employee Onboarding with Microsoft Entra ID Lifecycle Workflows

A home lab project that automates new hire onboarding for an **IT Level 1** employee using **Microsoft Entra ID Governance (Lifecycle Workflows)**. Instead of repeating the same checklist for every new employee, one workflow enables the account, assigns licenses, adds the user to the IT Level 1 group, and sends a welcome email, all without manual steps.

> **Note:** This is a lab environment. User details were entered manually to stand in for an HR system, and the workflow was started with **Run on demand** for testing.

---

## Project Overview

| | |
|---|---|
| **Goal** | Automate the repetitive tasks of onboarding a new IT hire |
| **Platform** | Microsoft Entra ID Governance, Lifecycle Workflows |
| **Tenant** | evilcorpLAB (lab tenant) |
| **Workflow name** | IT onboarding |
| **Template** | Onboard new hire employee |
| **Scope rule** | department equals `IT Department` |
| **Licenses assigned** | Microsoft 365 E3, Microsoft Entra Suite |
| **Target group** | IT Level 1 |
| **Test user** | Carmina Galvelo (IT Level 1 new hire) |

## The Flow

New hire details entered, then the workflow runs four tasks:

1. **Enable User Account**
2. **Assign licenses to user** (Microsoft 365 E3 + Entra Suite)
3. **Add user to groups** (IT Level 1)
4. **Send Welcome email**

The new hire then signs in, sets a new password, and registers MFA.

## Prerequisites

- **An Entra ID Governance license** (included in the Entra Suite). Without it, the Lifecycle workflows page returns a **401 "You don't have access"** error.
- **User attributes the workflow relies on:** Department, Job title, Employee hire date, Manager, and Usage location. Usage location is required before a license can be assigned.
- **A regular Security group with Assigned membership** as the target. Groups with *"Microsoft Entra roles can be assigned to the group"* set to Yes (role-assignable) are greyed out and cannot be used by workflows.
- **License task before the email task**, so the mailbox exists when the email is sent.

---

## Step-by-Step Walkthrough

### Step 1: Open Lifecycle Workflows

![Lifecycle workflows dashboard](Dashboard%20Lifecycle%20workflows.jpg)

1. Sign in to the **Microsoft Entra admin center** as a Global Administrator.
2. Go to **ID Governance > Lifecycle workflows**.
3. The overview shows the **workflow schedule: every 3 hours**, the number of workflows with a schedule enabled, deleted workflows, and any alerts.
4. Click **Create workflow** to start.

### Step 2: Create the workflow from a template

1. Choose the **Onboard new hire employee** template.
2. **Basics tab:** name the workflow `IT onboarding`. Leave the trigger as **Time based attribute**, **0 days**, **On**, **employeeHireDate** (set by the template).
3. **Configure scope tab:** add a rule so only IT users are processed:
   - Property: `department`
   - Operator: `equal`
   - Value: `IT Department`

   The value must match the user's department exactly, including capitalization and spaces. A user whose department text differs will not appear in the Run on demand list.

### Step 3: Review the tasks

![Task selector](task%20selector%20window%20choose%20task%20to%20automate.jpg)

On the **Review tasks** tab, the template starts with three tasks: **Enable User Account**, **Send Welcome email**, and **Add user to groups**. Clicking **Add task** opens the built-in task library, filtered to the **Joiner** category, including:

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

I added **Assign licenses to user** to the template's defaults.

### Step 4: Configure the license task

![Select license](select%20%20license%20to%20automate.jpg)

1. Open the **Assign licenses to user** task.
2. Tick **Microsoft 365 E3** (`SPE_E3`) and **Microsoft_Entra_Suite**.
3. Click **Select**, then **Save**.

The E3 license includes Exchange Online, which creates the user's mailbox. This task has to run **before** the welcome email, so the email has somewhere to land.

### Step 5: Configure the group task

1. Open **Add user to groups** and select the **IT Level 1** group. If the group is greyed out, it is role-assignable and has to be recreated as a normal Security group.
2. Leave **Continue workflow execution on error** unchecked, so failures are obvious while testing.
3. Make sure **Assign licenses to user** sits above **Send Welcome email** in the task order.

### Step 6: Create the workflow

![Workflow list](IT%20onboarding%20workflow%20view.jpg)

Click **Review + create**, then **Create**. The **Workflows** page now lists `IT onboarding` (created 10/7/2026) next to my earlier Sales workflow. **Schedule** is **No**, so it only runs when started manually, and **Enabled** is **Yes**. The toolbar provides **Run on demand**, **Clone**, **Enable schedule**, and **Delete**.

### Step 7: Run the workflow on demand

1. Tick the checkbox next to `IT onboarding` and click **Run on demand**.
2. Click **Select users** and choose the new hire. If the user is missing, their department doesn't match the scope rule.
3. Click **Run workflow**. Run on demand skips the schedule and the hire date check, so it is ideal for testing.

### Step 8: Check the results

![Workflow history](IT%20onboarding%20successul%20task.jpg)

In **Workflow history**, the Users summary shows:

- **1** user processed, **1 successful**, **0 failed**
- **4** total tasks, **0 failed tasks**
- Carmina Galvelo, started and completed at 5:16 PM, status **Completed**

The **Users**, **Runs**, and **Tasks** tabs show results per user, per run, and per task, and any failed task shows its error.

### Step 9: Verify the account was enabled

![Account enabled](account%20enabled.jpg)

Carmina's profile in **Entra ID > Users** shows **Account status: Enabled**, with **2 group memberships** and **2 assigned licenses**. No admin touched the account, and the workflow did all of it.

### Step 10: Verify the group membership

![Group membership](employee%20added%20to%20group%20task%20successful.jpg)

Opening **Groups > IT Level 1 > Members** shows Carmina Galvelo (`Cgalvelo@evilcorpLAB.onmicrosoft.com`) in the member list, added by the **Add user to groups** task. The other members were already in the group.

### Step 11: Check the welcome email

![Welcome email](welcome%20email%20to%20new%20employees.jpg)

Carmina received the welcome email in her new Outlook mailbox at 5:16 PM, the same minute the workflow ran. It greets her by name, links to the **My Apps portal**, and names her manager (ardy pascual) as the contact for next steps.

### Step 12: First sign-in, password update

![Password update](new%20employee%20setting%20up%20new%20email%20pass.jpg)

On first sign-in, Entra requires the new hire to **update her password**, because the account was created with a temporary one.

### Step 13: First sign-in, MFA registration

![MFA registration](user%20first%20login%20%20to%20her%20email%20with%20mfa.jpg)

After the password change, the user is prompted to register **Microsoft Authenticator** by scanning a QR code, so the account has MFA from day one.

---

## Troubleshooting

### The welcome email task failed (Sales test users)

![Assigning a license manually](assign%20license%20so%20email%20will%20push%20through.jpg)

Before building the IT workflow, I tested a Sales workflow with two users (screenshot: Sales user Meanne Cauilan). The **Send Welcome email** task failed, and the **Add user to groups** task after it never ran, so it showed as *unprocessed*. The cause was a missing mailbox, because those users had no Exchange license. I assigned **Microsoft 365 E3** and **Microsoft Entra Suite** in the Microsoft 365 admin center, and the email went through on the next run. That is why the IT workflow assigns the license as its own task before the email.

### Other issues I hit

| Issue | Cause | Fix |
|---|---|---|
| 401 on the Lifecycle workflows page | No ID Governance license | Start the Entra Suite trial and sign in again |
| Groups greyed out in the group task | Groups were role-assignable | Recreate them as normal Security groups with Assigned membership |
| User missing from the Run on demand list | Department text didn't match the scope rule | Make the department value identical |
| One failed task blocks the rest | The workflow stops on error by default | Order tasks carefully, or tick *Continue workflow execution on error* for non-critical tasks |
| Run on demand says "successful" but history is empty | History takes a few minutes to update | Wait, then click Refresh |

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
