# Salesforce Sales & Service Cloud Implementation Guide (Detailed)
## Project: EliMax Group Solar Companies

This guide provides a granular, click-by-click walkthrough to build the solution in a **Salesforce Developer Edition Org**. These steps are designed for you to follow exactly as you build your portfolio.

> **⚠️ Dev Edition Limitations:** This guide has been tailored for Developer Edition orgs. Where features are limited or unavailable, workarounds are provided. Items marked with `[DEV ORG NOTE]` require special attention.

---

### **Phase 1: Org Preparation & Feature Activation**

Before building specific objects, we must enable the core capabilities required by the project.

#### **1. Enable Multi-Currency**
*Context: The requirement states EliMax operates in different regions.*

`[DEV ORG NOTE]` Multi-Currency works in Dev Edition but **cannot be disabled once enabled**. This is permanent.

1.  Log in to Salesforce.
2.  Click the **Gear Icon** (top right) and select **Setup**.
3.  In the "Quick Find" box on the top left, type **Company Information**.
4.  Click **Company Information** in the sidebar.
5.  Click the **Edit** button.
6.  Find the checkbox labeled **Activate Multiple Currencies** and check it.
7.  Click **Save**.
8.  Now, in the Quick Find box, type **Manage Currencies**.
9.  Click **Manage Currencies**.
10. Click **New** to add currencies:
    *   **EUR - Euro**: Conversion rate `0.92`
    *   *(Optional)* **GBP - British Pound**: Conversion rate `0.79`
11. Click **Save** after each.

*If certain currencies aren't available, just use USD and EUR. The concept is what matters for the demo.*

#### **2. Enable Lightning Knowledge**
*Context: Required for the "Self-Service" and "Knowledge Base" deliverables.*
1.  In Setup, type **Knowledge** in the Quick Find box.
2.  Click **Knowledge Settings**.
3.  Check the box **Enable Lightning Knowledge**.
4.  If prompted, a wizard will start. Accept the defaults and complete it.
5.  Click **Save** or **Finish**.

#### **3. Enable Entitlement Management**
*Context: Required for SLAs (Service Level Agreements).*
1.  In Setup, type **Entitlement** in the Quick Find box.
2.  Click **Entitlement Settings**.
3.  Check the box **Enable Entitlement Management**.
4.  Click **Save**.

#### **4. Enable Digital Experiences (Community)**
*Context: Required for the "Customer Community Website".*

`[DEV ORG NOTE]` Experience Cloud is available in Dev Edition with limitations. You get one free site with restricted features. This is sufficient for demo purposes.

1.  In Setup, type **Digital Experiences** in the Quick Find box.
2.  Click **Settings**.
3.  Check the box **Enable Digital Experiences**.
4.  It will ask for a **Domain Name**. Enter something unique like `elimax-portfolio-[yourinitials]`.
5.  Click **Check Availability**.
6.  If available, click **Save**.
7.  Click **OK** on the warning (you can't change this domain name later).

---

### **Phase 2: Sales Cloud Configuration**

#### **1. Data Quality: Validation Rules**
*Context: Fix "incomplete or inaccurate contact information".*
1.  Click **Object Manager** (tab next to Home in Setup).
2.  Click **Lead**.
3.  Click **Validation Rules** in the left sidebar.
4.  Click **New**.
5.  **Rule Name**: `Require_Phone_or_Email`.
6.  **Description**: "Ensures agent enters at least one contact method."
7.  **Error Condition Formula**:
    ```text
    AND(
        ISBLANK(Phone),
        ISBLANK(Email)
    )
    ```
8.  **Error Message**: "You must provide at least a Phone Number or Email Address to save this Lead."
9.  **Error Location**: Select "Top of Page".
10. Click **Save**.

#### **2. Custom Fields: Region & Lead Score**
*Context: "Standardized approach to scoring leads" and "Region-specific pricing".*

**Create Region Field on Lead:**
1.  Still in **Object Manager > Lead**, click **Fields & Relationships**.
2.  Click **New**.
3.  Select **Picklist** and click **Next**.
4.  **Field Label**: `Region`.
5.  **Values**: Select "Enter values...". Type:
    ```
    North America
    Europe
    Asia
    ```
6.  Click **Next**, **Next**, **Save & New** (to create the next field immediately).

**Create Lead Score Field:**
1.  Select **Formula** and click **Next**.
2.  **Field Label**: `Lead Score`.
3.  **Formula Return Type**: **Number** (Decimal Places: 0).
4.  Click **Next**.
5.  **Formula Editor**: Copy and paste this logic:
    ```text
    IF( NOT(ISBLANK(Email)) , 10, 0) + 
    IF( NOT(ISBLANK(Phone)) , 10, 0) + 
    CASE(Rating, 
        "Hot", 20, 
        "Warm", 10, 
        0
    )
    ```
6.  Click **Check Syntax** to ensure no errors.
7.  Click **Next**, **Next**, **Save**.

**Create Region Field on Account (Critical for Quote Pricing):**
1.  Go to **Object Manager > Account** > **Fields & Relationships**.
2.  Click **New** > **Picklist**.
3.  **Field Label**: `Region`.
4.  **Values**: `North America`, `Europe`, `Asia`.
5.  Click **Next**, **Next**, **Save**.

*Why this matters: When a Lead converts, the Region must exist on Account so you can select the correct regional Price Book for Opportunities.*

#### **3. Sales Process & Path**
*Context: "Visual sales process with clear guidance."*
1.  In Setup Quick Find, type **Path**.
2.  Click **Path Settings**.
3.  Click **Enable Path** (if not already enabled).
4.  Click **New Path**.
5.  **Path Name**: `Solar Sales Path`.
6.  **Object**: `Lead`.
7.  **Record Type**: Master.
8.  **Picklist**: `Lead Status`.
9.  Click **Next**.
10. **Guidance for Success**:
    *   Click on the **"Open - Not Contacted"** tab in the diagram.
    *   In "Fields", add `Region` and `Lead Score`.
    *   In "Guidance", type: "Check the Lead Score. If >20, call immediately. Verify contact info before proceeding."
    *   Click on **"Working - Contacted"**. Add fields like `MobilePhone`, `Company`.
    *   In "Guidance", type: "Confirm the customer's region and energy needs. Schedule site assessment."
    *   Click on **"Closed - Converted"**.
    *   In "Guidance", type: "Create Opportunity. Select the correct regional Price Book."
11. Click **Next** and **Activate Your Path**.
12. Click **Finish**.

#### **4. Regional Pricing (Products & Price Books)**
*Context: "Quotes based on regional prices."*

**Step A: Create Products**
1.  Click the **App Launcher** (9 dots, top left). Search for **Products**.
2.  Click **New**.
3.  **Product Name**: `EliMax Solar Panel 300W`.
4.  **Product Code**: `PANEL-300W`.
5.  **Active**: Checked.
6.  Click **Save**.
7.  Go to the **Related** tab of the Product you just made.
8.  Find **Price Books** section. Click **Add Standard Price**.
9.  Enter `1000` (USD). Click **Save**.

10. Create a second product: `EliMax Inverter 5kW`.
    *   **Product Code**: `INV-5KW`.
    *   **Standard Price**: `2500` USD.

**Step B: Create Regional Price Books**

*Europe Price Book:*
1.  App Launcher > **Price Books**.
2.  Click **New**.
3.  **Name**: `Europe Price Book`.
4.  **Active**: Checked.
5.  Click **Save**.
6.  Go to the **Related** tab > **Price Book Entries** > **Add Products**.
7.  Select both products and click **Next**.
8.  Select **EUR** currency. *(If EUR not available, use USD with different prices to simulate regional pricing.)*
9.  **List Prices**: Panel = `920`, Inverter = `2300`.
10. Click **Save**.

*North America Price Book:*
1.  Repeat the above steps.
2.  **Name**: `North America Price Book`.
3.  Currency: **USD**.
4.  **List Prices**: Panel = `1000`, Inverter = `2500`.

*Asia Price Book:*
1.  **Name**: `Asia Price Book`.
2.  Currency: **USD**.
3.  **List Prices**: Panel = `950`, Inverter = `2400`.

#### **5. Enable Quotes**
*Context: Agents need to generate quotes for prospects.*

`[DEV ORG NOTE]` Quote PDF generation uses the standard template in Dev Edition. Custom quote templates require switching to Salesforce Classic to configure, which is optional.

**Enable Quotes:**
1.  Setup > Quick Find **Quote Settings**.
2.  Click **Enable Quotes**.
3.  Click **Save**.

**To Generate a Quote (Agent Process):**
1.  Navigate to an Opportunity.
2.  Select the correct **Price Book** based on Account Region.
3.  Add **Products** to the Opportunity via the Products related list.
4.  Click **Quotes** related list > **New Quote**.
5.  Fill in Quote details (Expiration Date, etc.).
6.  Click **Save**.
7.  On the Quote page, click **Create PDF**.
    *   If a template selection appears, choose the default.
    *   If no templates exist, click **Save Quote as PDF** (uses basic format).
8.  The PDF is attached to the Quote. You can email it manually.

*For demo purposes, showing the Quote record with line items is sufficient if PDF generation has issues.*

---

### **Phase 3: Service Cloud Configuration**

#### **1. Support Tiers (Account Field)**
*Context: "Basic, Standard, and Premium support."*
1.  **Object Manager** > **Account** > **Fields & Relationships**.
2.  Click **New** > **Picklist**.
3.  **Label**: `Support Tier`.
4.  **Values**: `Basic`, `Standard`, `Premium`.
5.  **Default Value**: `Basic`.
6.  Click **Next**, **Next**, **Save**.

#### **2. Support Queues**
*Context: Buckets to hold cases for each support tier.*
1.  Setup > Quick Find **Queues**.
2.  Click **New**.
3.  **Label**: `Premium Support Queue`.
4.  **Queue Email**: Leave blank.
5.  **Supported Objects**: Select **Case** and click **Add**.
6.  **Queue Members**: Add your User.
7.  Click **Save**.
8.  Repeat to create:
    *   `Standard Support Queue`
    *   `Basic Support Queue`

#### **3. Case Assignment Rules**
*Context: "Automated case routing based on customer tier."*
1.  Setup > Quick Find **Case Assignment Rules**.
2.  Click **New**. Name it `EliMax Routing`. Check **Active**. Save.
3.  Click `EliMax Routing` to open it.
4.  Click **New** (Rule Entry).
5.  **Sort Order**: 1.
6.  **Field Criteria**: `Account: Support Tier` **EQUALS** `Premium`.
7.  **Select the User or Queue**: Change dropdown to **Queue**. Search `Premium Support Queue`.
8.  Click **Save & New**.
9.  **Sort Order**: 2.
10. **Field Criteria**: `Account: Support Tier` **EQUALS** `Standard`.
11. **Select Queue**: `Standard Support Queue`.
12. Click **Save & New**.
13. **Sort Order**: 3.
14. **Field Criteria**: `Account: Support Tier` **EQUALS** `Basic`.
15. **Select Queue**: `Basic Support Queue`.
16. Click **Save**.

#### **4. Web-to-Case (Automated Case Creation)**
*Context: "Automate case creation" from customer inquiries.*

`[DEV ORG NOTE]` Web-to-Case works in Dev Edition. The generated HTML form submits directly to Salesforce and creates cases.

1.  Setup > Quick Find **Web-to-Case**.
2.  Click **Web-to-Case**.
3.  Check **Enable Web-to-Case**.
4.  **Default Case Origin**: `Web`.
5.  Click **Save**.
6.  Click **Generate HTML**.
7.  Select fields: `Subject`, `Description`, `Email`, `Phone`.
8.  Click **Generate**.
9.  Copy and save this HTML code (you can use it in your demo).

*Note: Email-to-Case requires email routing infrastructure that typically doesn't work in Dev Edition. Skip it.*

#### **5. Entitlements & SLAs**
*Context: "Ensure SLAs are adhered to."*

**A. Add Entitlement Fields to Case Layout**
1.  Object Manager > Case > Page Layouts > Case Layout.
2.  Drag **Entitlement Name** to the layout.
3.  Click **Save**.

**B. Create Milestones**
1.  Setup > Quick Find **Milestones**.
2.  Click **New Milestone**.
3.  **Name**: `First Response`.
4.  **Recurrence Type**: No Recurrence.
5.  Click **Save**.
6.  Click **New Milestone** again.
7.  **Name**: `Case Resolution`.
8.  **Recurrence Type**: No Recurrence.
9.  Click **Save**.

**C. Create Premium Entitlement Process**
1.  Setup > Quick Find **Entitlement Processes**.
2.  Click **New**.
3.  **Entitlement Process Type**: Case.
4.  **Name**: `Premium Support SLA`.
5.  Click **Save**.
6.  On the detailed screen, find **Milestones**. Click **New Milestone**.
7.  **Milestone Name**: `First Response`.
8.  **Time Trigger (Minutes)**: `60` (1 Hour for Premium).
9.  **Start Time**: `Case: Created Date`.
10. Click **Save**.
11. Click **New Milestone** again.
12. **Milestone Name**: `Case Resolution`.
13. **Time Trigger (Minutes)**: `480` (8 Hours for Premium).
14. **Start Time**: `Case: Created Date`.
15. Click **Save**.

**D. Create Standard Entitlement Process**
1.  Click **New** Entitlement Process.
2.  **Name**: `Standard Support SLA`.
3.  Add Milestones:
    *   `First Response`: 240 minutes (4 Hours).
    *   `Case Resolution`: 1440 minutes (24 Hours).

**E. Activate the Entitlement Processes**
1.  Open each Entitlement Process.
2.  Click **Activate**.

**F. Create Entitlement Records (Required for SLA to Work)**
1.  App Launcher > **Entitlements**.
2.  Click **New**.
3.  **Entitlement Name**: `Premium Support Entitlement`.
4.  **Account**: Select a test Premium Account.
5.  **Entitlement Process**: `Premium Support SLA`.
6.  **Start Date**: Today.
7.  **End Date**: 1 year from now.
8.  Click **Save**.
9.  Repeat for Standard tier if needed.

*For the demo, create one test Account with Premium tier and link an Entitlement to it.*

#### **6. Case Escalation Rules (Simplified for Dev Org)**
*Context: "Case escalation protocols" for SLA breaches.*

`[DEV ORG NOTE]` Escalation Rules work in Dev Edition, but email notifications may not send due to email deliverability limits. Focus on the auto-reassignment feature for your demo.

1.  Setup > Quick Find **Escalation Rules**.
2.  Click **New**. Name: `EliMax Escalation`. Check **Active**. Save.
3.  Click `EliMax Escalation` to open it.
4.  Click **New** (Rule Entry).
5.  **Sort Order**: 1.
6.  **Criteria**: `Case: Status` **NOT EQUAL TO** `Closed`.
7.  Click **Save**.
8.  Now add **Escalation Actions**. Click **New**.
9.  **Age Over**: `60` minutes.
10. **Business Hours**: Use default or skip.
11. **Auto-reassign cases**: Check this. Assign to yourself (as the "Manager").
12. **Notification Options**: Skip (emails may not work in Dev Org).
13. Click **Save**.

*For the demo, explain that in production, this would also send email alerts to managers.*

---

### **Phase 4: Experience Cloud (Help Center)**

`[DEV ORG NOTE]` Experience Cloud has significant limitations in Dev Edition:
- Only one site allowed
- Some templates may be restricted
- Guest user permissions can be tricky
- Knowledge visibility to guests requires specific configuration

If you encounter blockers, the workaround is to demo the Knowledge articles directly in Salesforce and explain the community concept verbally.

#### **1. Create Knowledge Articles**
*Context: "Robust self-service knowledge base."*
1.  App Launcher > **Knowledge**.
2.  Click **New**.
3.  If it asks for Record Type, choose **FAQ** or the default.
4.  **Title**: "How to Clean and Maintain Your EliMax Solar Panels".
5.  **Url Name**: Auto-populates.
6.  **Summary**: Copy from the `Knowledge_Articles_Content.md` file.
7.  Paste the Article Content into the body field.
8.  Click **Save**.
9.  **Critical**: Click **Publish** (top right). Select **Publish Now**.
10. Repeat for all 4 articles in the Knowledge_Articles_Content.md file.

#### **2. Build the Help Center Site**
1.  Setup > Quick Find **All Sites**.
2.  Click **New**.
3.  Look for **Help Center** template. *(If not available, select **Customer Service** or **Build Your Own**.)*
4.  Click **Get Started**.
5.  Enter Name: `EliMax Help`. URL: `help`. Click **Create**.
6.  **Experience Builder** opens.

**Customize the Home Page:**
7.  Click the Hero/Banner. Change the headline to "How can we help with your Solar System?".
8.  Ensure a **Search** component is on the page.

**Add a Contact Support Option:**

*Option A (Simplest): Add a Link*
9.  Add a **Rich Text** component to the page.
10. Type: "Can't find what you need? Email us at support@elimax.com or call 1-800-SOLAR."

*Option B (If Form Works): Case Submission*
9.  Click **Pages** (left panel) > **New Page** > **Standard Page**.
10. Name: `Contact Support`.
11. Drag **Create Record Form** component onto the page.
12. Set Object = **Case**.
13. Add this page to navigation if possible.

`[DEV ORG NOTE]` If the Create Record Form doesn't work for guest users (common limitation), use Option A or demonstrate the Web-to-Case HTML form separately.

**Publish the Site:**
14. Click **Publish** (Top Right).
15. If prompted about guest user access, try to enable it. If errors occur, proceed anyway.
16. Click **Publish** to confirm.

**Test the Site:**
17. Open the site URL from **All Sites** > **Builder** > Copy URL.
18. Open in an incognito browser window.
19. Test the Knowledge search.
20. If articles don't appear to guests, that's a common Dev Org limitation. For your demo, show the site logged in, or show Knowledge search from within Salesforce.

---

### **Phase 5: Security & Access (Profiles & Roles)**

*Context: Different users need different access levels.*

`[DEV ORG NOTE]` In Dev Edition, you cannot create custom profiles from scratch. You must clone an existing profile. Also, you have limited user licenses (typically 2), so you'll demo with your System Admin account.

#### **1. Create Role Hierarchy**
1.  Setup > Quick Find **Roles**.
2.  Click **Set Up Roles** (or **Add Role** if hierarchy exists).
3.  Under the top-level role, click **Add Role**.
4.  Create: `Sales Manager`.
5.  Under Sales Manager, add: `Sales Representative`.
6.  Create another branch: `Support Manager` > `Support Agent`.
7.  Click **Save**.

*This demonstrates the hierarchy concept. You don't need to assign users to roles for a solo demo.*

#### **2. Clone Profiles (Optional for Demo)**

*Skip this if you're demoing everything as System Admin. Include it if you want to discuss access control in your presentation.*

**Sales User Profile:**
1.  Setup > Quick Find **Profiles**.
2.  Find **Standard User** > Click **Clone**.
3.  **Profile Name**: `EliMax Sales User`.
4.  Click **Save**.
5.  (Optional) Edit permissions to show intent.

**Support User Profile:**
1.  Clone **Standard User** again.
2.  **Profile Name**: `EliMax Support User`.
3.  Click **Save**.

*For the demo, explain: "In production, Sales users would have this profile with access to Leads and Opportunities, while Support users would have access to Cases and Knowledge."*

---

### **Phase 6: The Demo Dashboard**
*Context: The presentation needs to show "Problem vs Solution".*

#### **1. Create Reports**

**Lead Pipeline Report:**
1.  App Launcher > **Reports** > **New Report**.
2.  **Report Type**: Leads.
3.  **Filters**: Lead Status not equal to Converted.
4.  **Group Rows By**: `Region`.
5.  **Columns**: Add `Lead Score`.
6.  Click **Save & Run**. Name: `Leads by Region`.

**Cases by Support Tier:**
1.  New Report > **Cases**.
2.  **Group Rows By**: `Account: Support Tier`.
3.  **Columns**: Case Number, Subject, Status.
4.  Save as `Cases by Tier`.

**Open Cases Report:**
1.  New Report > **Cases**.
2.  **Filter**: Status does not equal Closed.
3.  **Columns**: Case Number, Subject, Priority, Created Date.
4.  Save as `Open Cases`.

`[DEV ORG NOTE]` The "Cases with Milestones" report type may not be available in Dev Edition. If missing, skip the SLA Compliance report and use the Open Cases report instead. Explain the SLA tracking concept verbally.

#### **2. Build the Dashboard**
1.  App Launcher > **Dashboards**.
2.  Click **New Dashboard**. Name: `EliMax Executive Overview`.
3.  Click **+ Component**.
4.  Select `Leads by Region` report. Choose **Donut Chart**.
5.  Add another component: `Cases by Tier` report. Choose **Horizontal Bar Chart**.
6.  Add: `Open Cases` report. Choose **Table** or **Metric**.
7.  Arrange components:
    *   Top row: Sales metrics (Leads by Region).
    *   Bottom row: Support metrics (Cases by Tier, Open Cases).
8.  Click **Save**.
9.  Click **Done**.

---

### **Phase 7: Case Deflection Flow (Standout Feature)**

*Context: This Screen Flow guides customers through self-service before creating a case. It directly addresses the requirement to "reduce the volume of support calls" and demonstrates advanced Salesforce automation skills.*

This is your **differentiator**. Most portfolio projects stop at configuration. This Flow shows you can build process automation.

#### **1. Create the Flow**

1.  Setup > Quick Find **Flows**.
2.  Click **New Flow**.
3.  Select **Screen Flow**. Click **Create**.

#### **2. Build Screen 1: Issue Category Selection**

1.  From the toolbox on the left, drag a **Screen** element onto the canvas.
2.  **Label**: `Select Issue Type`.
3.  **API Name**: Auto-populates.
4.  Add a component to the screen:
    *   Click **+ Add Component** in the screen preview.
    *   Select **Picklist**.
    *   **Label**: `What is your issue about?`
    *   **API Name**: `Issue_Category`
    *   **Data Type**: Text.
    *   **Choice Source**: Select "New Picklist Choice Set".
        *   **Label**: `Issue Categories`
        *   **Data Type**: Text.
        *   **Configure Choices**: Add these manually:
            *   `Billing & Payments`
            *   `Technical / Inverter Issues`
            *   `Maintenance & Cleaning`
            *   `Warranty Claims`
            *   `Other`
    *   Check **Require** to make it mandatory.
5.  Click **Done** to close the screen editor.

#### **3. Build the Knowledge Search (Get Records)**

1.  Drag a **Get Records** element onto the canvas, below the first screen.
2.  Connect the first screen to this element (drag the arrow).
3.  Configure:
    *   **Label**: `Find Related Articles`.
    *   **Object**: `Knowledge__kav` (Knowledge Article Version).
    *   **Filter Conditions**:
        *   Field: `PublishStatus` | Operator: Equals | Value: `Online`
        *   *(Optional)* Add: Field: `Title` | Operator: Contains | Value: `{!Issue_Category}` *(This attempts to match articles to the category. May not work perfectly, but shows intent.)*
    *   **How Many Records**: `All Records`.
    *   **How to Store**: `Automatically store all fields`.
4.  Click **Done**.

`[DEV ORG NOTE]` If the Knowledge object isn't available or the filter doesn't work, simplify by removing the filter. The Flow will just show all published articles. The concept is what matters for the demo.

#### **4. Build Screen 2: Display Knowledge Articles**

1.  Drag another **Screen** element onto the canvas.
2.  Connect the Get Records element to this screen.
3.  **Label**: `Review These Articles`.
4.  Add components:
    *   **Display Text** component:
        *   **API Name**: `Article_Instructions`
        *   **Text**: "We found some articles that might help. Please review them before submitting a case."
    *   **Display Text** component (for article list):
        *   **API Name**: `Article_List`
        *   **Text**: 
        ```
        📄 How to Clean and Maintain Your EliMax Solar Panels
        📄 Understanding Your Solar Energy Bill and Credits
        📄 Troubleshooting Inverter Error Code 404
        📄 EliMax Product Warranty Coverage
        
        (In production, this would dynamically list matching articles)
        ```
    *   **Radio Buttons** component:
        *   **Label**: `Did these articles solve your problem?`
        *   **API Name**: `Problem_Solved`
        *   **Data Type**: Text.
        *   **Add Choices manually**:
            *   `Yes, my issue is resolved`
            *   `No, I still need help`
        *   Check **Require**.
5.  Click **Done**.

#### **5. Add Decision Element**

1.  Drag a **Decision** element onto the canvas.
2.  Connect Screen 2 to the Decision.
3.  **Label**: `Check If Resolved`.
4.  **Outcome 1**:
    *   **Label**: `Issue Resolved`
    *   **Condition**: `{!Problem_Solved}` Equals `Yes, my issue is resolved`
5.  **Default Outcome**: `Needs Case` (for when they still need help).
6.  Click **Done**.

#### **6. Build Screen 3A: Thank You (Issue Resolved Path)**

1.  Drag a **Screen** element onto the canvas.
2.  Connect the "Issue Resolved" outcome to this screen.
3.  **Label**: `Thank You`.
4.  Add a **Display Text** component:
    *   **Text**: 
    ```
    ✅ Great! We're glad the Knowledge Base helped.
    
    If you have any other questions in the future, visit our Help Center anytime.
    
    Thank you for being an EliMax customer!
    ```
5.  Click **Done**.
6.  This screen should be an **end point** (no further connections).

#### **7. Build Screen 3B: Case Submission (Needs Help Path)**

1.  Drag a **Screen** element onto the canvas.
2.  Connect the "Needs Case" (default) outcome to this screen.
3.  **Label**: `Submit Your Case`.
4.  Add components:
    *   **Text** input:
        *   **Label**: `Subject`
        *   **API Name**: `Case_Subject`
        *   **Required**: Yes
    *   **Long Text Area** input:
        *   **Label**: `Description`
        *   **API Name**: `Case_Description`
        *   **Required**: Yes
    *   **Email** input:
        *   **Label**: `Your Email`
        *   **API Name**: `Contact_Email`
        *   **Required**: Yes
    *   **Phone** input:
        *   **Label**: `Your Phone (Optional)`
        *   **API Name**: `Contact_Phone`
5.  Click **Done**.

#### **8. Create the Case Record**

1.  Drag a **Create Records** element onto the canvas.
2.  Connect Screen 3B (Submit Your Case) to this element.
3.  Configure:
    *   **Label**: `Create Case`.
    *   **How Many Records**: `One`.
    *   **How to Set Record Fields**: `Use separate resources, and literal values`.
    *   **Object**: `Case`.
    *   **Set Field Values**:
        *   `Subject` = `{!Case_Subject}`
        *   `Description` = `{!Case_Description}`
        *   `SuppliedEmail` = `{!Contact_Email}`
        *   `SuppliedPhone` = `{!Contact_Phone}`
        *   `Origin` = `Web`
        *   `Status` = `New`
4.  **Store Case ID**: Check "Manually assign variables" and create a variable `{!New_Case_Id}` to store the created Case's ID.
5.  Click **Done**.

#### **9. Build Screen 4: Confirmation**

1.  Drag a **Screen** element onto the canvas.
2.  Connect the Create Records element to this screen.
3.  **Label**: `Case Submitted`.
4.  Add a **Display Text** component:
    *   **Text**:
    ```
    ✅ Your case has been submitted successfully!
    
    Case Number: {!New_Case_Id}
    
    A support representative will contact you within:
    • Premium Customers: 1 hour
    • Standard Customers: 4 hours
    • Basic Customers: 8 hours
    
    Thank you for contacting EliMax Support.
    ```
5.  Click **Done**.

#### **10. Save and Activate**

1.  Click **Save**.
2.  **Flow Label**: `EliMax Case Deflection Flow`.
3.  **Flow API Name**: `EliMax_Case_Deflection_Flow`.
4.  Click **Save**.
5.  Click **Activate**.

#### **11. Add Flow to a Lightning Page (For Demo)**

1.  App Launcher > **Cases**.
2.  Open any Case record (or the Case Home page).
3.  Click **Gear Icon** > **Edit Page**.
4.  In the Lightning App Builder, drag a **Flow** component onto the page.
5.  Select your `EliMax Case Deflection Flow`.
6.  Click **Save** > **Activate** (for the org default if prompted).

*Alternatively, for the demo, you can run the Flow directly:*
1.  Setup > Flows > Click your Flow.
2.  Click **Run** (top right).

#### **12. Test the Flow**

1.  Run the Flow.
2.  Select an issue category. Click **Next**.
3.  Review the article suggestions.
4.  Select "No, I still need help". Click **Next**.
5.  Fill in case details. Click **Next**.
6.  Verify the confirmation screen shows.
7.  Go to the **Cases** tab and verify a new Case was created.

---

### **Final Checklist**

| Item | Required | Notes |
|------|----------|-------|
| Multi-Currency enabled | ✓ | At minimum USD + EUR |
| Region field on Lead AND Account | ✓ | Both needed for pricing flow |
| Lead validation rule active | ✓ | Test by saving Lead without phone/email |
| Lead Path activated | ✓ | Should appear on Lead records |
| All 3 Price Books created and active | ✓ | North America, Europe, Asia |
| Quotes enabled | ✓ | PDF optional if issues |
| Support Tier field on Account | ✓ | |
| 3 Support Queues created | ✓ | Premium, Standard, Basic |
| Case Assignment Rules active | ✓ | Test with a new Case |
| Web-to-Case enabled | ✓ | HTML form generated |
| Milestones created | ✓ | First Response, Case Resolution |
| Entitlement Processes activated | ✓ | Premium SLA at minimum |
| Test Entitlement record created | ✓ | Linked to test Account |
| Escalation Rules configured | ○ | Optional, explain concept if issues |
| All 4 Knowledge Articles published | ✓ | Must be Published, not Draft |
| Experience Cloud site published | ○ | Do your best; fallback to internal demo |
| Dashboard created | ✓ | At least 2-3 components |
| **Case Deflection Flow activated** | ⭐ | **Standout feature** - test end-to-end |

**Legend:** ✓ = Required, ○ = Nice to have, ⭐ = Differentiator

---

### **Demo Script Outline (15 minutes)**

**Opening (2 min):**
"EliMax Group is growing fast but struggling with four key problems: poor lead data quality, no visual sales guidance, inconsistent support response times, and too many support calls that could be deflected. Let me show you how Salesforce solves each one."

**Sales Demo (4 min):**
1.  Create a Lead. Try to save without Phone or Email. *Show validation rule blocking the save.*
2.  Add Phone, select Region = Europe, Rating = Hot. Save.
3.  *Show the Path* at the top with stage guidance. Point out the Lead Score.
4.  Click **Convert**. Create Opportunity.
5.  On Opportunity, select **Europe Price Book**.
6.  Add Products. *Show the EUR pricing.*
7.  Create a Quote. *(Show the Quote record even if PDF has issues.)*

**Service Demo (4 min):**
1.  Show an Account with Support Tier = Premium.
2.  Create a Case on that Account. Save.
3.  *Show the Case routed to Premium Support Queue.*
4.  Show the Milestone countdown (if Entitlement is linked).
5.  Open **Knowledge** sidebar. Search "inverter". Attach article.
6.  Close the Case.

**⭐ Case Deflection Flow Demo (3 min) - YOUR DIFFERENTIATOR:**
*"But what if we could prevent unnecessary cases from being created in the first place?"*
1.  Run the Case Deflection Flow.
2.  Select "Technical / Inverter Issues" as the category.
3.  Show the Knowledge articles being suggested.
4.  Select "Yes, my issue is resolved." *Show the thank you screen.*
5.  *Explain:* "That customer never created a case. In production, this could deflect 30-40% of support tickets."
6.  Run the Flow again. This time select "No, I still need help."
7.  Fill in case details and submit.
8.  Show the confirmation with Case Number.
9.  Navigate to Cases tab and show the newly created Case.

**Dashboard (1 min):**
Open the dashboard. Quick overview:
- "Leadership can see lead distribution by region and case volume by tier at a glance."

**Close (1 min):**
"By implementing validation rules, visual paths, automated routing, SLA tracking, and intelligent case deflection, EliMax can now scale their operations without scaling their headcount. The Flow alone could save them hundreds of support hours per month."

---

### **Troubleshooting Common Dev Org Issues**

| Issue | Solution |
|-------|----------|
| Multi-Currency option not visible | Check Company Information. May already be enabled. |
| Knowledge articles not showing in search | Ensure articles are **Published**, not Draft. |
| Experience Cloud site shows blank | Check guest user profile has access to Knowledge. Try viewing logged in. |
| Quote PDF button missing | Use Classic mode (switch via profile) to create templates, or skip and show Quote record. |
| Escalation emails not sending | Expected in Dev Org. Focus on auto-reassignment in demo. |
| Entitlements not tracking milestones | Ensure Case has an Entitlement selected. Create Entitlement record first. |
| Cases not routing to Queue | Verify Assignment Rule is Active. Check "Assign using active assignment rules" when saving Case. |
| Flow won't activate | Check for errors in elements. Ensure all required fields are mapped. |
| Flow "Get Records" returns nothing | Verify Knowledge articles are Published. Remove filters if needed. |
| Flow doesn't create Case | Check field mappings in Create Records element. Ensure Subject is mapped. |
| Can't find Knowledge__kav object in Flow | Search for "Knowledge" in the object picker. Object name varies by org. |

---

**You are now ready to build and demo your solution!**
