Onboarding a new employee in Entra ID with zero repetitive clicks 🚀

Lifecycle Workflows is one of the best features in Microsoft Entra ID, and one you should never miss.

Every new hire means the same checklist: enable the account, assign licenses, add them to the right groups, send a welcome email. Done by hand, it's slow and easy to get wrong.

So I built it as an automated workflow in my home lab using Microsoft Entra ID Governance. Here's how it works, step by step 👇

1️⃣ Lifecycle Workflows dashboard
Everything runs from ID Governance. I used the "Onboard new hire" template as the starting point.

2️⃣ Scoping the workflow
The workflow targets users by attributes (department, job title, hire date). Only people who match the rule get processed, so IT hires get an IT workflow and Sales hires get a Sales one.

3️⃣ Choosing the tasks
Entra has a built-in task library. I picked four: Enable User Account, Assign licenses, Add user to groups, and Send Welcome email.

4️⃣ License task
Microsoft 365 E3 and Entra Suite are assigned automatically. This matters because the license creates the mailbox that the welcome email needs.

5️⃣ Run results
One new hire processed, 4 tasks, 0 failures.

6️⃣ Account enabled
Account status flipped to Enabled with no admin touching it.

7️⃣ Group membership
The new hire landed in the IT Level 1 group on their own.

8️⃣ Welcome email
It arrived in their new mailbox, naming their manager and linking to their apps.

9️⃣ First sign-in
They set a new password, then registered MFA with Microsoft Authenticator. Secure from day one.

💡 What I learned
- Order matters. The license has to come before the email, or the mailbox doesn't exist yet and the email fails.
- Role-assignable groups can't be targeted by workflows. Normal security groups work, so I rebuilt mine.
- Automation is only as good as your data. Consistent department names, hire dates, and managers make or break the scope rules.
- Test with "Run on demand" first, then enable the schedule once it works.

Note: this is a lab, with user details entered by hand to stand in for an HR system.

⏭️ Next up: offboarding. A Leaver workflow that disables the account, removes groups and licenses, and keeps things clean when someone leaves. Joiners are the easy half. Leavers are where the real security risk sits.

If you work in IT or identity, this is a feature worth learning. It saves time, cuts mistakes, and makes sure every new hire starts secure.

Coming from 8+ years in IT support and telecom, automating the identity lifecycle is a skill I'm excited to keep building.

#MicrosoftEntra #EntraID #IdentityGovernance #IAM #ITSupport #Automation #HomeLab #Microsoft365 #CareerGrowth
