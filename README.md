To create the "Create Appointment Report" (Label: アポ報告) button and process in Salesforce using Screen Flow, follow these steps:

### 1. **Create the "Create Appointment Report" Button**
   - Navigate to **Salesforce Setup**.
   - Go to **Object Manager** > **Opportunity**.
   - Select **Buttons, Links, and Actions**.
   - Click **New Button or Link**.
   - Set the button type to **Flow**.
   - Label: **Create Appointment Report** (Label: アポ報告).
   - In the **Flow** field, select the Screen Flow that you’ll create below.

### 2. **Create the Screen Flow**
   You'll need a Flow to handle the steps described, including the modals, picklist, and notifications.

#### 2.1. **Create New Flow**
   - Go to **Setup** > **Flows** > **New Flow**.
   - Select **Screen Flow**.

#### 2.2. **Add Variable for Opportunity Context**
   - Create a new **Variable**:
     - **API Name**: `varOpportunityId`
     - **Data Type**: **Text**
     - **Available for Input**: **Checked** (This will allow the Flow to accept the Opportunity ID from the button you created in Step 1).

#### 2.3. **Create the First Screen: Select Event**
   - Add a **Screen Element**.
   - **Screen Label**: **行動選択** (Action Selection).
   - Add a **Picklist** field:
     - **Picklist Label**: Choose Event.
     - Set the **Picklist Choices** to a dynamic choice from the related **Event** records (you’ll need to use a **Get Records** element to fetch Events related to the Opportunity).
     - **Label for Picklist Items**: `{Event's Start Date} {Event's Subject}`.

   - Add a **Next Button** for the user to proceed.

#### 2.4. **Create the Second Screen: Appointment Report Input**
   - Add a **Screen Element**.
   - **Screen Label**: **アポ報告** (Appointment Report).
   - Add the following fields:
     - **Field Type: Text**:
       - **Label**: 会社名 (Company Name).
       - **Default Value**: Use a **Get Records** element to retrieve the **Office__c** value from the Opportunity and pre-fill this field.
     - **Field Type: Number**:
       - **Label**: 年収 (Annual Income).
       - **Default Value**: Use a **Get Records** element to retrieve the **AnnualIncome__c** value from the Opportunity and pre-fill this field.
     - **Field Type: Textarea**:
       - **Label**: アポ取得数等の本文 (Appointment Details).
     - **Field Type: Textarea**:
       - **Label**: コメント (Comments).

   - Add a **Submit/Send Button** (Label: 送信).

#### 2.5. **Update Opportunity Fields**
   After the user clicks **送信**, update the Opportunity:
   - Use an **Update Records** element to update the **Office__c** and **AnnualIncome__c** fields of the Opportunity based on the input from the modal.

#### 2.6. **Send Notification**
   - Add a **Send Custom Notification** element.
     - **Color**: #2EB67D.
     - **Channel**: **Later** (Ensure that you’ve set up the Later notification channel in Salesforce if not already done).
     - **Content**: Add custom content, for example, "Appointment Report Created for Opportunity {Opportunity Name}".

#### 2.7. **Link the Flow to the Button**
   - Save and activate the flow.
   - Link it to the button created in Step 1. Now when the user clicks the button, it will launch the flow and handle the steps described.

### 3. **Testing and Deployment**
   - Test the button and flow in your sandbox environment first to ensure that the modal shows the picklist of events correctly and that the fields are populated and updated as expected.
   - Deploy to production once the testing is successful.

This setup should allow users to create an appointment report linked to the Opportunity, update relevant fields, and send the notification as specified. Let me know if you need help with any specific steps!
