Here is the complete, consolidated Technical Specification Document, combining all the refined sections, data structures, and workflows we've developed. It is fully formatted in Markdown and ready to be copied into your company's documentation wiki or engineering tracker.

---

# Technical Specification: BuildFlow Financial Module (Progress Claims & Invoicing)

## 1. Executive Summary

**BuildFlow** is the overarching construction management platform currently under development. The platform's foundational component, the estimation module, was built first and is responsible for managing the pre-construction phase. Within this module, the Main Contractor responds to a Request for Tender (RFT) by engaging with multiple subcontractors to create a consolidated quote for a client. Subcontractors are invited to bid on a work package , which is comprised of one or more Bill of Quantities (BoQ) line items. The Main Contractor then applies a profit margin to the subcontractor's quoted price  and delivers a consolidated quote to the potential client.

The new **BuildFlow Financial Module** is a separate but loosely coupled system designed to manage the post-estimation execution phase. When the quote is accepted and terms are agreed, the quoted values become the project budget. The estimation module "passes the baton" to the financial module via an API handover. The system must lock this down as the original budget.

From this point of handover, the Financial Module takes over. It enables the managed payment of subcontractors alongside automated client progress billing, ensuring the project's finances remain clean from the start of execution through to the final retention payment release.

---

## 2. Target Personas

* 
**Project Manager / Main Contractor (MC):** Responsible for independently recording completion estimates on all budget line items , reviewing subcontractor claims, overriding percentages , managing variations , and generating the consolidated client invoice.


* 
**Subcontractor:** Engaged to perform specific work packages; responsible for submitting progress claims via an individualized input screen.


* 
**Accounts / Contract Administrator:** Handles the general ledger processing , monitors retention accounts , logs client payments , and manages bulk payment files (e.g., via Xero) for subcontractor payments.



---

## 3. Workflow / User Journey

1. 
**Project Handover:** Upon client quote acceptance, the budget is created and submitted to the finance module. The confirmation of this advances the project status to execution.


2. 
**Notification Trigger:** 1 week from the billing cycle payment date, a reminder is sent out to contractors linked to budget line items not yet completed.


3. 
**Subcontractor Submission:** The subcontractor accesses the system via an individualised input screen and inputs their progress.


4. 
**MC Assessment & Approval:** The main contractor independently records completion estimates , reviews the subcontractors claimed progress, and may choose to accept it or override it with their own estimate.


5. 
**Client Invoicing & GL Posting:** The project manager can generate the client invoice which will consolidate the progress claims for all subbies. The system needs to recognize the total value of the work completed as revenue, while splitting the amounts owed into the correct asset accounts.


6. 
**Payment & Subbie Remittance:** The user can log the payment against the issued receipt. Once client payment is received, the system must generate a bulk payment file containing all subbie payments to be made.


7. 
**Retention Release:** When the project hits practical completion (or at the end of the defects liability period), the system will generate a final "Retention Release" invoice. Retention amounts will be paid out after the defects period is up.



---

## 4. Functional Requirements & Epics

### Epic 1: Budget Integration & Project Initialization

* 
**API Handover:** The system will have an API that will accept a project budget along with key details for the project.


* 
**Budget Baselining:** The system must lock this down as the original budget.


* 
**Schedule Calculation:** The system must determine the first billing cycle dates by calculating and storing the following for the project: Claim due date and Invoice issue date .



**API Handover Payload (Estimation -> Finance):**

```json
{
  "projectId": "PRJ-2026-0042", 
  "projectName": "London Towers", 
  "clientId": "CLI-88392", 
  "status": "APPROVED", 
  "totalBudgetedIncome": 1050000.00, 
  "totalBudgetedExpense": 850000.00, 
  "billingSchedule": {
    "type": "SPECIFIC_DAY_OF_MONTH", 
    "claimDueDay": 17, 
    "invoiceIssueDay": 23, 
    "invoiceIssueWeekOfMonth": 3, 
    "paymentTermsInDays": 30, 
    "defaultRetentionPercentage": 5.00 
  },
  "workPackages": [
    {
      "packageId": "WP-ELEC-01", 
      "packageName": "Electrical & Data", 
      "subcontractorId": "SUB-9921", 
      "retentionPercentage": 5.00, 
      "lineItems": [
        {
          "lineItemId": "LI-ELEC-001", 
          "costCode": "CC-16000", 
          "description": "Rough-in: First Floor Power and Lighting", 
          "unitOfMeasure": "Item", 
          "quantity": 1.00, 
          "expenseRate": 10000.00, 
          "expenseTotal": 10000.00, 
          "incomeRate": 12000.00, 
          "incomeTotal": 12000.00 
        }
      ]
    }
  ]
}

```

### Epic 2: Subcontractor Progress Claim Submission

* 
**Automated Notifications:** Identify projects with a claim due date of 3 working days from currents date. Select a list of subcontractors that have not yet completed one or more-line items in the work package allocated to them. Obtain email address and contact person from tender module.


* 
**Email Template:** Construct and send out an email to the subcontractor:



> Subject: ACTION REQUIRED: Progress Claim Due - [Project Name] - [Billing Month/Cycle] Dear [Subcontractor Contact Name], This is a formal reminder that the progress claim window for the current billing cycle on [Project Name - e.g., PRJ-2026-0042] is now open. To ensure timely processing, your progress claim must be submitted no later than [Insert Due Date, e.g., Monday, 23rd of March] at [Insert Time, e.g., 5:00 PM]. 
> Please be advised that our project billing schedules are strictly enforced to align with our upstream client reporting. If your claim is not submitted by this deadline, it will not be assessed in the current run and will automatically be deferred to the next scheduled pay cycle. 
> To simplify the submission process, we have set up a direct portal for your contract. You can use the link below to securely log in and update the "Percentage Complete" for your approved line items. [Insert Secure Portal Link: e.g., ] Please provide progress updates for the following approved Work Packages and Line Items: Work Package: [WP-ID] - [Package Name, e.g., WP-ELEC-01 - Electrical & Data] [Line Item ID] - [Description, e.g., LI-ELEC-001 - Rough-in: First Floor Power and Lighting] [Line Item ID] - [Description, e.g., LI-ELEC-002 - Fit-off: First Floor Fixtures] [Line Item ID] - [Description, e.g., LI-VAR001-ELEC-01 - VAR 01: Supply and install 4x extra floor boxes] Alternatively, if you prefer to submit a standard PDF payment claim from your own accounting software, please ensure it strictly matches the line items and contract values listed above, and email it directly to: [claims@BigTimeContractor.com]. 
> (Note: If emailing a PDF claim, please ensure all cumulative totals and previous claim deductions are clearly itemized to avoid rejection). 
> Thank you for your continued hard work on site. If you have any questions regarding your contract values or approved variations, please contact the site team immediately prior to submitting your claim. Sincerely, [Your Name/Automated System Name] Contract Administration [Your Company Name] 
> 
> 

* 
**Secure Portal Summary:** The link provided in the mail takes the user to a Landing Screen with Summary details. Fields include Project Name, Claim Period, Status, Days remaining to submit, Total Approved Contract Value, Total Previously Claimed, and Remaining contract Value . There should be a button labelled “Complete Claim”.


* 
**Interactive Data Tables:** The user is able to view the items total amount and the amount claimed to date. The user can now record an updated total percentage completion. The system will then calculate and display the claimed amount. The user could choose to provide the amount for the current claim and the system will calculate and display the total percentage completion.



**Table Example (Work Package A):**

| Line Item | Current Position: Item total amount | Current Position: Claimed to date | Current Claim: % Complete | Current Claim: Amount | Retention (5%): Held | Retention (5%): Current claim |
| --- | --- | --- | --- | --- | --- | --- |
| Line item 1 | 10,000.00 

 | 5,000.00 (50%) 

 | 80 

 | 3,000.00 

 | 250.00 

 | 150.00 

 |
| Line item 2 | 75,000.00 

 | 22,500.00 (30%) 

 | 50 

 | 37,500.00 

 | 1,125.00 

 | 1,875.00 

 |

* 
**Input Validation:** The system must validate the user inputs to ensure the user doesn’t put in a lower completion rate than already recorded or input a claimed amount that would exceed the items total.



### Epic 3: Main Contractor Assessment

* 
**Story 3.1:** The system will provide a separate screen for the main contractor (MC) to enter a set of completion rates based on their own independent survey of work completed. Completion rates must be recorded for all items in the project budget for a billing cycle.


* 
**Story 3.2:** Once all the subbie estimates have been provided, the MC can go to the progress payment assessment and confirmation screen. Here the MC will be presented will all project budget items not yet completed.


* 
**Story 3.3:** It will highlight items where the subbie completion estimate is higher than that recorded by the MC.


* 
**Story 3.4:** For these items, the MC must decide what completion rate the subbie will be paid for and make the adjustment to the subbie claim by overriding the claim percentage.


* 
**Story 3.5:** The MC must record a reason for doing so. This adjustment and the reason for doing so must be included in the payment advice sent out to the subbie.



### Epic 4: Variation Management

* 
**Native Execution:** The system must cater for variations to contracted amounts. These will be handled natively in the financial module.


* 
**Variation Linking:** A variation must be agreed to by the client and subcontractor. A variation must be linked to a work package. Within the work package, a variation line item can be linked to an existing budget line item if it reduces or increases that budget. A variation line item could also be an entirely new line item within the work package. A variation can also have a time extension component if additional time is agreed to implement the variation.



### Epic 5: Invoicing, GL & Payments

* 
**Client Invoice:** The consolidated completion percentage per line item will be applied to the income amount for the line item to calculate the totals for the invoice. The system will deduct all payments received from the client to date to determine the amount to include as a progress payment.


* 
**Retention Deduction:** Retention amounts - the system must automatically deduct the retention percentage from both the subbie payment and the client invoice, storing it in a separate ledger account to be released at the end of the defects liability period.



**GL Table: Issue Client Invoice**

| Account Name | Account Type | Dr | Cr |
| --- | --- | --- | --- |
| Construction Revenue | Income |  | Total invoice amount 

 |
| Accounts Receivable | Asset | Invoice amt less retention 

 |  |
| Retention Receivable | Asset | Retention amount for invoice 

 |  |

**GL Table: Receive Client Payment**

| Account Name | Account Type | Dr | Cr |
| --- | --- | --- | --- |
| Cash in Bank | Asset | Amount received 

 |  |
| Accounts Receivable | Asset |  | Amount Received 

 |

* 
**Subcontractor Invoices:** Approving subbie invoice for payment.



**GL Table: Subcontractor Approval**

| Account Name | Account Type | Dr | Cr |
| --- | --- | --- | --- |
| Cost of Goods Sold | Expense | Approved payment amount 

 |  |
| Accounts Payable | Liability |  | Approved amount less retention 

 |
| Retention Payable | Liability |  | Retention amount 

 |

* 
**Bulk Payments:** Once client payment is received, the system must generate a bulk payment file containing all subbie payments to be made. The system will cater to standard UK formats and will utilize Xero as the primary payment facilitator, with a manual file upload option for fallback.


* 
**Closing the Loop:** After the payment file has been processed, the confirmation is to be fed back into the system to close the loop on the payment.



**GL Table: Subcontractor Payment**

| Account Name | Account Type | Dr | Cr |
| --- | --- | --- | --- |
| Accounts Payable | Liability | Approved amount less retention 

 |  |
| Cash in Bank | Asset |  | Approved amount less retention 

 |

* 
**Final Retention Release:** When the project hits practical completion (or at the end of the defects liability period), the system will generate a final "Retention Release" invoice. The finance module must provide a specific workflow to handle this without double-counting revenue. It should amount to debiting Accounts receivable and crediting Retention Receivable.



---

## 5. Out of Scope & Parallel Tasks

* 
**Alternative Billing:** The initial release of the finance module will cater for only monthly billing cycles. Milestone and fortnightly billing will be introduced in a future release based on demand.


* 
**Refactoring:** The estimation module requires refactoring in parallel . Cost codes need to be moved to the finance schema. The email service must allow multiple senders so we can have something like accounts@BigTimeContractor.com.



---

Would you like me to output this as a raw text block so you can easily copy and paste it into Jira, Confluence, or your preferred documentation tool without losing the formatting?
