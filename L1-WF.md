
Solarium Green Energy: Channel Partner Application Workflow
1. User Personas
1.1 Four Primary User Roles

Channel Partner (CP): Primarily uses mobile app for lead management and quotation generation
Admin: Central authority with full system access via web portal
Key Account Manager (KAM): Territory-based manager with limited admin privileges via web portal
Customer: End-user who requests services and receives quotations via mobile app

2. Objectives and Goals by Persona
2.1 Channel Partner (CP)

Manage leads effectively via mobile application
Generate and share quotations with customers
Track lead statuses
Add new leads to the system
Communicate with assigned KAM

2.2 Admin

Approve and manage Channel Partners
Oversee lead management and assignment
Configure system master data
Manage Key Account Managers
Access reporting and analytics

2.3 Key Account Manager (KAM)

Monitor assigned Channel Partners' performance
Support lead management for assigned territory
View reporting on lead funnel and conversions
Reassign leads between within Channel Partners assigned to the KAM

2.4 Customer

Self-register through the mobile app
Request solar services through catalog selection
View and share quotations for requested services
Upload required KYC documents
Raise support tickets for executed projects



# The Project has 4 Components - CPAPP , WEBPRT , CUSTAP AND BCKND

CPAPP : Channel Partner Mobile App 
WEBPRT : Web based interface for Admin and KAM 
CUSTAP : Mobile App for end customers or consumers
BCKND : Backend infrastructure for the solution



# Detailed Channel Partner (CP)  User Workflow
## 1. Getting Started: Installation to Login
### 1.1 Downloading and Installing the App
	1	Channel Partner (CP) visits the app store (Google Play Store or Apple App Store)
	2	Searches for "Solarium Green Energy- Channel Partner" app
	3	Taps "Install" and waits for the download and installation to complete
	4	Once installed, CP taps "Open" to launch the app
### 1.2 Self-Registration Process
	1	CP is presented with the Solarium splash screen, followed by the Login page
	2	On the Login page, CP finds a "Register" link at the bottom and taps it
	3	CP fills out the registration form with the following details:
	◦	Name (required)
	◦	Phone number (required) - this will serve as the unique identifier
	◦	Address (required)
	◦	State (dropdown)
	◦	PIN code (required)
	◦	Optional fields: Email, Organization name, GSTIN
	4	After completing the form, CP taps "Submit"
	5	A 6-digit OTP is sent to the provided phone number
	6	CP enters the OTP (valid for 2 minutes)
	◦	Note: 5 incorrect attempts will lock the phone for 15 minutes
	7	Upon successful OTP verification, the app displays an "Awaiting Admin Approval" banner
	8	In the background, a new CP record is created in the database with status = "Pending Verification"
### 1.3 Waiting for Admin Approval
	1	CP must wait for the Solarium admin to approve the registration
	2	Once approved, CP receives an SMS notification: "Your Solarium registration is approved."
### 1.4 First Login After Approval
	1	CP opens the app again
	2	On the Login page, enters their registered phone number
	3	Taps "Get OTP"
	4	Enters the 6-digit OTP received via SMS
	5	Upon successful verification, the server issues a JWT token
	◦	Token auto-refreshes every 24 hours
	◦	Hard logout occurs after 30 days of inactivity
	6	CP is directed to the Home Dashboard
## 2. Home Dashboard - Main Interface
### 2.1 Exploring the Dashboard
	1	CP sees the main dashboard with several UI zones:
	◦	Top Bar: Contains Sync button (↻), Notifications (🔔), and Profile (👤)
	▪	Sync button appears grey when data is up-to-date
	▪	Notifications icon shows an unread badge when there are new notifications
	▪	Long-pressing the Sync button forces a full delta pull from the server
	◦	Widget Strip: Shows three key metrics
	▪	"Today's Pending" - leads with follow-up date set to today
	▪	"Overdue" - leads with follow-up dates before today
	▪	"Total Leads" - all leads assigned to the CP
	▪	Tapping any widget navigates to a pre-filtered Lead List
	◦	Floating Action Button (FAB): "Add New Lead" button (visible unless offline)
	◦	Bottom Navigation Bar: Home | My Leads | Quotation | Customers
	▪	"Leads" tab shows a red badge with the count of overdue leads
### 2.2 Using the Top Bar Functions
	1	Sync Button: CP taps to manually synchronize data with the server
	◦	Long-press for a full refresh when needed
	2	Notifications: CP taps to view push notification history
	◦	Notifications include OTPs, status changes, quotation approvals, commission updates
	3	Profile: CP taps to access profile settings and options
## 3. Leads Management - Core Functionality
### 3.1 Adding a New Lead
	1	From any screen, CP taps the FAB (Add New Lead)
	2	A form appears with the following fields:
	◦	Name (required)
	◦	Phone (required)
	◦	Email (optional)
	◦	Address (required)
	◦	State (dropdown)
	◦	PIN code (required)
	◦	Requirement selection from  services catalog (optional)
	◦	Upload Lead documents (optional)
	3	CP fills out the form and taps "Save"
	4	The system creates a new lead record with:
	◦	Status = "New Lead"
	◦	Follow-up date = today + 1 day
	◦	Origin = CP
	5	A toast message appears: "Lead created"
	6	The app automatically navigates to the Lead Detail screen for the new lead
### 3.2 Browsing the Lead List
	1	CP taps "My Leads" in the bottom navigation
	2	The Lead List screen appears showing all leads
	◦	List has endless scroll functionality
	◦	Default sorting: next follow-up date (oldest to newest)
	3	CP can search by Name, Phone, Status, or Date using the search bar
	4	CP can filter the list by tapping filter options
### 3.3 Managing Lead Details
	1	CP taps on any lead in the list to open Lead Detail
	2	The Lead Detail screen has five sections : ( Info / Status / Quotations / Lead Docs / Customer / Timeline ) 
#### 3.3.1 Info section : 
	1	CP can view and edit:
	◦	Name : 
	◦	Phone (must remain unique in the system)
	◦	Email
	◦	City 
	◦	State : 
	◦	PIN code
#### 3.3.2 Status section
	1	CP manages the lead's status through a dropdown menu after the Info Section
	2	The dropdown only shows valid next states based on the status matrix:
	◦	From New Lead → In Discussion / Physical Meeting Assigned / Not Responding / Not Interested / Other Territory
	◦	From In Discussion → Physical Meeting Assigned / Won / Not Responding / Not Interested / Other Territory
	◦	From Physical Meeting Assigned → Won / Not Responding / Not Interested / Other Territory
	◦	From Won → Pending at Solarium
	◦	From Pending at Solarium → Under Execution
	◦	From Under Execution → Executed
	◦	Terminal states: Not Responding, Not Interested, Executed, Other Territory
	3	Required fields depend on the selected status:
	◦	Next Follow-up Date (required for non-terminal states, must be between today and today+30 days)
                Quotation Reference number  (mandatory when changing to "Won"). User selects from the dropdown of Generated Quotations mapped to the lead while moving to "Won". 
                Quotation reference number is the system generated quotation unique id. 
	◦	Token No. (mandatory when changing to "Under Execution")
	◦	Remarks (minimum 10 characters, always required for any status change ) 
	5	CP enters all required information and taps "Save" to update the status
        6	The Status section always displays Quotation Ref Number once the status has transitioned to "Won".
        7	The Status section always displays Token Number once the status has transitioned to Under Execution.
        8       CP cannot edit the Quotation Ref Number and token Number.
        10 	The Status section always displays the last saved Remark. 

       

#### 3.3.3 Quotations section 
	1	CP sees a chronological list of quotations with thumbnails, dates, and status (Valid/Rejected)
	2	For each quotation, CP can:
                See the Status of the Quote (Draft / Created / Shared ) 
	◦	Download the quotation PDF
	◦	Share the quotation with a "Share with Customer" Button that marks the Flag as - Yes.
	3	CP can tap "Generate New" to open the Quotation Wizard (detailed in section 6)

#### 3.3.4 Lead Docs section
	1	CP can upload, replace, or delete site-level documents
                These documents are different from KYC documents
                These docs are tagged to a lead and are just a library of documents for the lead. like a storage space for a lead to store docs like lead site related images or some other document. 
	◦	Maximum 7 documents per lead. 
	◦	Each document must be ≤10 MB
	◦	Supported formats: PDF, JPG, PNG
	3	CP taps "Upload" to add new documents
	4	CP can delete any uploaded document under lead Docs.


	◦	
###  3.  Customer Section
        1.      In the customer section CP sees the Customer name mapped to the lead.
        2.      If the lead is from a new customer (based on Phone number of the lead) a new customer is added to customer list and Name and Phone number displayed here. 
        3.      If the lead is from an existing number the Customer name and phone number is fetched from customer  displayed here. 
        4.      CP also sees the Below mentioned KYC documents of the customer with the  option to upload KYC docs or view current status (Approved/ Pending Review/ Rejected)
                Income Proof
	◦	Light Bill (latest)
	◦	Cancelled Cheque (latest)
	◦	Aadhaar Card
	◦	PAN Card
	◦	Passport-size Photograph
	. 	Other
        5.      CP can  select and Upload any document and the KYC Document status is changed to Pending Review. 
        6.      Each KYC document is an independent upload and mutually exclusive.
        7.      CP cannot replace /delete documents if the file status is ‘Approved’
        8.      Upload button is disabled for all approved docs.
        9.      Upload button is only active for "Pending review" / "Rejected" docs or if there is no previous file for the document. 

#### 3.3.5 Timeline Tab
	1	CP views an immutable log of all actions on this lead
	2	Each log entry shows:
	◦	Who made the change
	◦	When the change was made
	◦	What was changed (before → after)
	3	This tab is read-only and provides a complete audit trail

## 4. Customers Module
### 4.1 Accessing the Customer List
	1	CP taps "Customers" in the bottom navigation
	2	The Customer List appears, showing all customers derived from phone numbers in leads
	3	CP can search or filter the list
	4	Each customer entry shows:
	◦	Customer Name 
	◦	Number of leads - "Requests"
	◦	Number of "executed" leads AS "Executed"
### 4.2 Viewing Customer Details
	1	CP taps on a customer to open the Customer Detail screen
	2	The Customer Detail screen includes:
	◦	Contact card with Call/SMS options
	◦	List of all leads associated with this customer
	◦	"KYC Documents" section for document management
### 4.3 Managing **Customer KYC** Documents
	1	CP taps "KYC Documents" to manage customer documents
	2	CP can upload seven types of documents:
	◦	Income Proof
	◦	Light Bill (latest)
	◦	Cancelled Cheque (latest)
	◦	Aadhaar Card
	◦	PAN Card
	◦	Passport-size Photograph
	.	Other
	3	For each document type:
	◦	CP can upload a new document (PDF/JPG/PNG, ≤10 MB)
                Each KYC document is an independent upload and mutually exclusive.  
                Each document has the following status if there is a file already uploaded - (Pending Review / Approved ) 
	◦	CP cannot replace/delete documents if the file status is updated to "Approved". Upload button is disabled for all approved Docs. 
	◦	CP Can can re-upload any doc by clicking on clicking on "Upload" button and attaching the file from device storage. The upload button is only active if there is no file or File is under "Pending Review"  or "Rejected" status. 
     
             
## 5. Quotation Module
### 5.1 Browsing Quotations
	1	CP taps "Quotation" in the bottom navigation
	2	A list of all quotations appears (Draft / Created / Shared ) 
	3	CP can Search by Lead ID, Customer Name, or Phone Number
	4	CP sees these action buttons.  - "Share with customer" /  Download 
        5       Clicking on "Share with customer" the flag is updated as Yes. Customer can see the Quote on his Customer app. This share button only appears if the flag is "No". 
	6.	CP can download the Quote. 
        6.      Every quotation has a flag indicating "Shared with Customer " - Yes/No.
### 5.2 Creating a New Quotation
	1	CP selects a lead first (either from Lead Detail or by using the search function) so that all quotations are properly mapped to the lead. 
	2	CP taps "Generate New Quote" to launch the 7-Step Quotation Wizard
## 6. 7-Step Quotation Wizard - Step by Step Process
### 6.1 Step 1: Location
	1	CP selects the following parameters:
	◦	State from dropdown (alphabetically sorted)
	◦	DISCOM from dropdown (filtered based on selected state)
	◦	Phase using radio buttons: Single/Three (default is Single)
	◦	Smart-Meter toggle (Yes/No). If only one option available - sets it to default for the State/Discom/Phase combination. 
	2	CP taps "Next" to proceed. 
### 6.2 Step 2: Panels
	1	CP selects:
	◦	Panel Make from dropdown (recommended makes appear at the top)
	◦	Variant from dropdown (options depend on selected Make)
	▪	The system automatically sets the DCR flag based on the variant
	◦	Number of Panels using a stepper control
	▪	Range is 4-9 panels for single-phase
	▪	Range is 7-18 panels for three-phase
	2	A live capacity calculation is displayed (capacity in kW = number of panels × panel wattage ÷ 1000)
	3	CP taps "Next" to proceed
### 6.3 Step 3: Inverter & BOM (Bill of Materials)
	1	CP selects:
	◦	Inverter from a dropdown list
	▪	List is filtered to show only options within ±15% of the calculated capacity and matching the selected phase
	▪	Recommended inverters are marked with a star (★)
	◦	Cable Type: Standard or Polycab ( Default Polycab)
	◦	Extra Structure Height: 0m (default), with 0.5m increments up to 2m
	2	CP taps "Next" to proceed
### 6.4 Step 4: Fees Lookup
	1	CP views a read-only summary of all applicable fees
	2	Fees are automatically calculated based on the selected state, DISCOM, phase, and smart meter flag
	3	The summary includes:
	◦	Discom Fee
	◦	Discom Miscellaneous charges
	◦	Installation
	◦	Meter charges
	◦	Modem+Box charges
	4	CP taps "Next" to proceed

###  6.5 Step 5 : Dealer Add on : 
	2	CP adjusts the Dealer Add-on using a slider:
	◦	Range: 0 to ₹2,000 per kW
	◦	Default: ₹1,800 per kW

### 6.6 Step 6: Pricing & Subsidy
	1	CP views:
	◦	Hardware Subtotal (sum of panels, inverter, BOM, Other Costs  which includes (transportation, Overhead , minimum company margin , dealer margin) , Additional Structure Cost , and  Discom Level Charges    ( Fees).)
	◦	Inflated Base Price (hardware subtotal × 1.10, where 10% is the default inflation percentage)
	1	The screen displays a live preview of the System Price (inflated base + dealer add-on total)
	2	CP taps "Next" to proceed to Subsidies calculation. 

	 	Subsidies
	1	CP views the automatically calculated subsidies:

	     ◦Central Subsidy (S₀) calculated according to rules:

	▪	Less than 1 kW: ₹0
	▪	Up to 2 kW: capacity × ₹30,000
	▪	Between 2-3 kW: ₹60,000 + (capacity-2) × ₹18,000
	▪	3 kW or more: capped at ₹78,000

	     ◦State Subsidy looked up from master data (S₁/S₂/S₃ depending on state  and DISCOM rules) and Panel Type for the requirement., 
                Condition:  State Subsidy applies only for DCR panels.
                S1 – No Subsidy

                S2 – Tiered Subsidy
	•	If Capacity ≤ 5 kW → ₹2,000 * kW
	•	If Capacity > 5 kW → Flat ₹10,000

                S3 – Higher Tiered Subsidy
	•	If Capacity ≤ 2 kW → ₹15,000 * kW
	•	If Capacity > 2 kW → Flat ₹30,000

	3	CP taps "Next" to proceed.

### 6.7 Step 7 : Review & Generate
	1	CP scrolls through a comprehensive summary:
	◦	Configuration details
	◦	Cost table with all components
	◦	Commission preview (equal to the dealer add-on total)
	2	CP taps "Save & Share" which:
                Saves all unsaved changes for the lead. 
	◦	Creates an immutable quotation record (JSON + PDF)
	◦	Opens the device native share sheet with the PDF . 
                CP can cancel the native share option and sees a Home Button in the screen. 



## 7. Profile & Settings Management
### 7.1 Accessing Profile
	1	CP taps the Profile icon  in the top bar
	2	The Profile & Settings screen appears
### 7.2 Editing Profile Information
	1	In "My Profile" section, CP can edit:
	◦	Name
	◦	Email
	◦	Address
	◦	PIN code
	2	Phone number is displayed but is read-only (cannot be changed)
	3	CP taps "Save" to update profile information
### 7.3 Checking Notifications
	1	CP taps "Notifications" to view a scrollable log of all push notifications:
	◦	OTP verifications
	◦	Approval notifications
	◦	Quote status updates
	◦	Commission notifications
	◦	Ticket replies
### 7.4 Logging Out
	1	CP taps "Logout" to sign out
	2	The system invalidates the JWT token on the server
	3	CP is returned to the Login screen


## 8. Commission & Earnings Tracking
### 8.1 Accessing Earnings Information
	1	CP opens the side menu and taps "Earnings"
	2	The Earnings screen appears with a list of all leads that have resulted in commissions
	3	For each entry, CP can view:
                Lead id 
                Lead name
	◦	Quotation ref number
	◦	System capacity (kW)
	◦	Commission amount (₹)
	◦	Project status ( Filter with  leads Status = Executed) 
	◦	Payment status ( Pending / Approved / Paid ) 
### 8.2 Receiving Commission Updates
	1	CP receives push notifications when Admin marks a commission as "Paid"
	2	The payment status in the Earnings screen updates accordingly

### 8.3 Commission Summary 
 	1	CP sees a LIFETIME cumulative summary of   Commissions Pending (Payment Status =  Pending) /  Commissions Approved (Payment Status =  Approved ) / Commissions Earned (payment status= paid)
	

## 9. Document Management Workflow
> **Note:** *KYC documents* belong to the Customer and have a fixed checklist, whereas *Lead documents* are unlimited and tied to the individual Lead (e.g., site photos, cheque images).

### 9.1 Document Permissions
	1	CP can upload documents at both the customer level (KYC) and lead level (site-specific)
	2	CP cannot delete customer level (KYC) documents if "Approved". CP can upload KYC  docs if the KYC doc status is "Pending Review"/ "Rejected" or if there is no previous history of doc. 
	3	CP cannot approve or reject documents (only KAM and Admin can do this)
        4.      CP can delete lead level docs for his leads only. 

### 9.2 Document Upload Process
	1	CP navigates to either:
                For KYC docs 
	◦	Customer Details > KYC Documents  or Lead Details >> Customer Section >> KYC Documents. 
                (for site-specific documents)
	◦	Lead Detail >>  Lead Docs section 
### 9.3  Lead Docs  Management Actions 
	2	CP taps "Upload" button in Lead Docs Section and selects a document from their device
	3	Documents must be:
	◦	≤10 MB in size
	◦	In PDF, JPG, or PNG format
	4	Images are automatically compressed to ≤75% quality to save space
	6	CP taps "Upload" to complete the process
                CP can upload unto 7 docs under Lead Docs section.
                These docs are not grouped or tagged and are just docs with the Lead. The purpose of these docs are lead related proofs. 

### 9.3  KYC Document Management Actions

      For each document type:
	◦	CP can upload a new document (PDF/JPG/PNG, ≤10 MB)
                Each KYC document is an independent upload and mutually exclusive.  
                Each document has the following status if there is a file already uploaded - (Pending Review / Approved / Rejected  ) 
	◦	CP cannot replace/delete documents if the file status is updated to "Approved or Pending Review ". Upload button is disabled for all Approved Docs. 
	◦	CP Can can re-upload any doc by clicking on clicking on "Upload" button and attaching the file from device storage.The upload button is only active if there is no file or doc is in "Pending Review"


## 10. Other Key Features & Workflows
### 10.1 Data Synchronization

	1	The app automatically syncs data with the backend when:
	◦	CP opens the app
	◦	CP performs any CRUD operation
	◦	CP pulls down to refresh on any list screen
	2	CP can force a manual sync by tapping the Sync button (↻) in the top bar
	
### 10.2 Offline Operation
	1	When offline, the app:
	◦	Disables the "Add New Lead" FAB and all CRUD operations. 
	◦	Shows an "Offline Mode" indicator WHEN OFFLINE
	2	CP can still view previously loaded data.
	3	When connectivity is restored, the offline mode indicator is REMOVED AND  CP can perform CRUD operations like : Add new lead , generate new quote , change lead status etc. 


### 10.3 Handling Status Changes
	1	When changing a lead's status, CP must provide:
	◦	Next Follow-up Date (for all non-terminal states transition)
	◦	Quotation Reference Number (if status is moved to  "Won")
                Token No. (if status is moved to  "Under Execution")
	◦	Remarks (always required in all transitions, minimum 10 characters)
	2	The app enforces the status matrix, only allowing valid transitions:
	◦	From "New Lead" to "In Discussion" or other valid states
	◦	From "In Discussion" to "Physical Meeting Assigned" or other valid states
	◦	And so on according to the defined matrix
	3	After a successful status change, the Timeline tab is automatically updated

### 10.4 Managing Multiple Quotations for a Lead
	1	CP can generate multiple quotations for the same lead
	2	Each quotation is timestamped and stored with the configuration used
	3	CP can view all quotations in the Lead Detail > Quotations tab
	4	Quotation which is recorded as "Quotation Reference" when the lead status changes to "Won" is to be used for Commissions calculation for the CP. 
        5       CP can mark which Quotations can be shared with the customer on his App. Only shared Quotes can be viewed by the customer.  Quote Flag ( "shared with customer") is marked as "Yes"




# Detailed Admin Portal User Workflow

## 1. Login & Authentication

1. **Access the Portal**
   - Admin navigates to the Solarium Admin Portal URL in a web browser
   - System presents the login screen with email and password fields

2. **Login Process**
   - Admin enters registered email address and password
   - Clicks "Login" button
   - System validates credentials
   - If valid, grants access to the Admin Dashboard
   - If invalid, displays appropriate error message

3. **Password Recovery (if needed)**
   - Admin clicks "Forgot Password" link on login screen
   - Enters registered email
   - System sends reset link via email
   - Admin follows link to create a new password
   - Returns to login with new credentials

## 2. Dashboard & Navigation Overview

1. **Initial Dashboard**
   - Upon successful login, Admin sees the main dashboard
   - Dashboard displays overall sales funnel visualization
   - Key metrics shown: New Leads, In Discussion, Physical Meeting Assigned, Won, Pending at Solarium, Under Execution, Executed
   - Time period filters: Today, Last 7 Days, Last 30 Days, Last Quarter, Custom Date Range
   - Geographic filters:  State-wise breakdown

2. **Main Navigation Menu**
   - Left sidebar contains main navigation options:
     - Dashboard
     - Leads Management
     - Quotation Management
     - Channel Partners
     - Customers
     - KAMs (Key Account Managers)
     - Commission & Payouts
     - Quotation Master Data
     - Services catalog
     - Global Settings



3. **Top Bar Elements**
   - User profile icon (access to profile settings)
   - Notifications bell (system alerts)
   - Quick filters (date range, region)

## 3. Manage Channel Partners

1. **Access Channel Partner Management**
   - Admin clicks "Channel Partners" in the main navigation
   - System displays a list of all Channel Partners with key details:
     - Name
     - Phone
     - Registration Date
     - Status (Pending / Activate /Deactivate )
     - Total Leads Count
     - Executed Projects Count

2. **Approve New Channel Partner Registrations**
   - Admin selects "Pending Approval" filter to view pending registrations
   - For each pending CP:
     - Reviews registration details (Name, Phone, Address, PIN, Organization, GSTIN if provided)
     - Clicks "Activate" button
   - Upon approval:
     - Status changes to "Activate"
     - System sends automated SMS: "Your Solarium registration is approved"
     - CP appears in the main CP list

3. **View Channel Partner Details**
   - Admin clicks on any CP name in the list
   - System displays CP profile with tabs:
     - Basic Info (editable name, email, address, PIN; read-only phone)
     - Assigned Leads (filterable list)
     - Quotations (all quotations generated by this CP)
     - Commission History (all commissions  - pending/ Approved / paid)
     - Activity Timeline (chronological audit of all actions)

4. **Deactivate/Re- activate Channel Partner**
   - In CP detail view, Admin clicks "Deactivate" button
   - System prompts for confirmation and reason
   - Admin provides reason and confirms
   - Status changes to "Deactivate"
   - CP can no longer login to app, but data remains preserved
   - To reactivate, Admin clicks "Activate" on a deactivated CP
   - Status returns to "Activate" and CP regains app access

5. **Assign KAM to Channel Partner**
   - In CP detail view, Admin clicks "Assign KAM" button
   - System displays dropdown of active KAMs
   - Admin selects appropriate KAM
   - Clicks "Save" to confirm assignment
   - System updates CP record and notifies KAM

- IN CP List view - Admin selects multiple CP and cLicks on "Assign KAM" and selects a KAM to assign to the selected CPs. 

## 4. Manage Key Account Managers (KAMs)

1. **Access KAM Management**
   - Admin clicks "KAMs" in main navigation
   - System displays list of all KAMs with details:
     - Name
     - Email
     - Status (Active/Inactive)
     - Assigned CPs count
     - Assigned Leads count

2. **Add New KAM**
   - Admin clicks "Add New KAM" button
   - Form appears requesting:
     - Name (required)
     - Email (required, unique)
     - Phone (required)
     - Initial Password (auto-generated or admin-set)
     - Regions/States assigned (multi-select)
     - Channel Partners assigned (multi-select)
   - Admin completes form and clicks "Create KAM"
   - System creates KAM account and sends welcome email with login credentials

3. **Edit KAM Details**
   - Admin clicks on KAM name in list
   - System displays KAM profile with editable fields
   - Admin can edit Channel Partners assignment using multi-select option
   - Admin makes necessary changes and clicks "Save"
   - System updates KAM record

4. **Manage KAM Assignments**
   - Within KAM detail view, Admin sees "Assigned CPs" tab
   - Admin can:
     - Add CPs using "Assign New CP" button and selecting from available CPs
     - Remove assignment by clicking "Remove" next to a CP
     - Bulk assign by selecting multiple CPs and clicking "Assign Selected"
   - Each change updates both KAM and CP records
   - KAM and CP are mapped in a One to many mapping. 

5. **Deactivate/Reactivate KAM**
   - In KAM detail view, Admin clicks "Inactive" button
   - System prompts for confirmation
   - Upon confirmation, KAM status changes to "Inactive"
   - KAM can no longer access portal; assigned CPs remain but are flagged for reassignment
   - To reactivate, Admin clicks "Active" for the inactive KAM and the Status changes to "Active"

## 5. Manage Customers

1. **Access Customer Management**
   - Admin clicks "Customers" in main navigation
   - System displays list of all customers with key details:
     - Name
     - Phone (primary identifier)
     - Lead Count
     - Total Projects Value
     - Registration Date

2. **View Customer Details**
   - Admin clicks on customer name in list
   - System displays customer profile with tabs:
     - Basic Info (read-only: name, phone, email, address, PIN)
     - KYC Documents checklist (all uploaded verification documents) 
     - Leads (all leads associated with this customer)
     - Quotations (all quotations generated for this customer)
     - Support Tickets (ticket history)
     - Timeline of all customer activity till date. 

3. **Manage Customer Documents**
   - In customer detail view, Admin navigates to "KYC Documents" tab
   - For each document type (Income Proof, Light Bill, Cancelled Cheque, Aadhaar, PAN, Photograph):
     - Views current document (if uploaded) with Buttons Accept/reject. ( The doc status remains as "Pending Review )
   - Document statuses: Pending Review / Approved / Rejected
     Clicks "Approve"  and Status CHANGES TO  Approved. 
   - CLICKS "Reject" to change document status TO Rejected.
   - For "Reject", enters reason which is sent to the customer 
  
   


## 6. Manage Leads


1. **Access Lead Management**
   - Admin clicks "Leads Management" in main navigation
   - System displays comprehensive lead list with filters:
     - Status (All/New Lead/In Discussion/etc.)
     - Date Range (Created/Follow-up)
     - Assigned CP ( Unassigned / List of all CPs in the Db) 
     - Origin (CP/Customer/Admin)
     - State
     - Source 
     

2. **View Lead Details**
   - Admin clicks on any lead ID in the list
   - System displays lead detail with five tabs (matching CP app structure):
     - Info (basic lead information, editable, including customer mapping status)
     - Status (current status and history)
     - Quotations (all quotations generated for this lead)
     - Docs (all site-specific documents)
     - Timeline (audit log of all changes)

3. **Create New Lead**
   - Admin clicks "Add New Lead" button 
   - Form appears requesting:
     - Customer details (Name, Phone, Email, Address, State, PIN)
     - Service Requirements
     - Assigned CP (dropdown of active CPs)
     - Initial Status (default: New Lead)
     - Follow-up Date (default: today+1)
     - Automatic map to  Customer based on the lead phone number and customer phone number (to link lead with existing customer)  or create  a new customer. 
   - Admin completes form and clicks "Create Lead"
   - System creates lead record and notifies assigned CP



5. **Reassign Lead to Different CP**
   - In lead detail view, Admin clicks "Reassign" button
   - System displays dropdown of active CPs
   - Admin selects new CP and provides reason for reassignment
   - System updates lead assignment and notifies both old and new CPs

6. **Force Status Change**
   - In lead detail Status tab, Admin can override normal status flow
   - Selects target status from complete dropdown (not restricted by matrix)
   - Provides mandatory reason for override
   - System updates status and logs the admin override in Timeline

7. **Bulk Lead Operations**
   - Admin can select multiple leads using checkboxes
   - Clicks "Bulk Actions" dropdown to:
     - Reassign selected leads to a specific CP
     - Export selected leads to CSV
     - Force status update on multiple leads
   - For each bulk action, Admin must confirm and provide reason

8. **Lead Import**
   - Admin clicks "Import Leads" button
   - System provides downloadable CSV template
   - Admin uploads completed CSV file
   - System validates all records (all-or-nothing approach)
   - If valid, imports all leads and assigns according to CSV data
   - If errors found, provides detailed error report without importing

9. **Unassigned Leads**
  Leads created directly by customers or via CSV import remain here until assigned.
   - Admin clicks on "Unassigned Leads" Tab as a PRE-SET filter in Manage Leads. 
   - The tab is highlighted with a prominent color so that it is easily visible 
   - Admin can see the list of leads along with the source and other details. 
   - Admin can select multiple leads and click on "Bulk ReAssign" button and choose the CP to which these selected leads will be assigned. 
   - Admin can click on any individual lead and assign CP to that lead as an Edit feature.
 
  

## 7. Quotation Management

1. **Access Quotation Management**
   - Admin clicks "Quotation Management" in main navigation
   - System displays list of all quotations with filters:
     - Status (Valid/Rejected/All)
     - Date Range
     - CP
     - Lead ID
     - Capacity Range

2. **View Quotation Details**
   - Admin clicks on quotation in  the list
   - System displays:
     - All configuration details used to generate it
     - Associated lead information
     - Current status and timestamps
     -  Admin clicks on "Share with customer" the flag is updated and the customer can see the Quote on his Customer app.
     -  Admin  can download the Quote and share it with people outside the solution. 
     - Every quotation has a flag indicating "Shared with Customer " - Yes/No. 
     - Admin can download the quotation via Download button.



4. **Generate Quotation**
   - Admin can generate quotations directly by clicking "Generate Quotation" button
   - Selects lead from dropdown or search
   - Completes same 7-step quotation wizard as CP app:
   - 7-Step Quotation Wizard - Step by Step Process

###  Step 1: Location
	1	User selects the following parameters:
	◦	State from dropdown (alphabetically sorted)
	◦	DISCOM from dropdown (filtered based on selected state)
	◦	Phase using radio buttons: Single/Three (default is Single)
	◦	Smart-Meter toggle (Yes/No). If only one option available for "Smart Meter Flag" for the State/Discom/Phase combination it is selected as Default. 
	2	User taps "Next" to proceed
###  Step 2: Panels
	1	KAM selects:
	◦	Panel Make from dropdown (recommended makes appear at the top)
	◦	Variant from dropdown (options depend on selected Make)
	▪	The system automatically sets the DCR flag based on the variant
	◦	Number of Panels using a stepper control
	▪	Range is 4-9 panels for single-phase
	▪	Range is 7-18 panels for three-phase
	2	A live capacity calculation is displayed (capacity in kW = number of panels × panel wattage ÷ 1000)
	3	User taps "Next" to proceed
###  Step 3: Inverter & BOM (Bill of Materials)
	1	User selects:
	◦	Inverter from a dropdown list
	▪	List is filtered to show only options within ±15% of the calculated capacity and matching the selected phase
	▪	Recommended inverters are marked with a star (★)
	◦	Cable Type: Standard or Polycab
	◦	Extra Structure Height: 0m (default), with 0.5m increments up to 2m
	2	User taps "Next" to proceed
###  Step 4: Fees Lookup
	1	User views a read-only summary of all applicable fees
	2	Fees are automatically calculated based on the selected state, DISCOM, phase, and smart meter flag
	3	The summary includes:
	◦	Discom Fee
	◦	Discom Miscellaneous charges
	◦	Installation
	◦	Meter charges
	◦	Modem+Box charges
	4	User taps "Next" to proceed

###   Step 5 : Dealer Add on : 
	2	User adjusts the Dealer Add-on using a slider:
	◦	Range: 0 to ₹2,000 per kW
	◦	Default: ₹1,800 per kW

###  Step 6: Pricing & Subsidy and Subsidies
	1	Admin views:
	◦	Hardware Subtotal (sum of panels cost, inverter cost, BOM costs, Other Costs ( transportation, Overhead , minimum company margin , dealer margin) , Additional Structure Cost , and  Discom Level Charges ( Fees).)
	◦	Inflated Base Price (hardware subtotal × 1.10, where 10% is the  inflation percentage set in the "Global settings" by the Admin.  
	1	The screen displays a live preview of the System Price (inflated base + dealer add-on total)
	2	Admin taps "Next" to proceed

		Subsidies 
	2	Admin views the automatically calculated subsidies:
 
		◦Central Subsidy (S₀) calculated according to rules:

	▪	Less than 1 kW: ₹0
	▪	Up to 2 kW: capacity × ₹30,000
	▪	Between 2-3 kW: ₹60,000 + (capacity-2) × ₹18,000
	▪	3 kW or more: capped at ₹78,000

		◦State Subsidy looked up from master data (S₁/S₂/S₃ depending on state rules) and Panel Type for the requirement., 
                Condition:  State Subsidy applies only for DCR panels.
                S1 – No Subsidy

                S2 – Tiered Subsidy
	•	If Capacity ≤ 5 kW → ₹2,000 * kW
	•	If Capacity > 5 kW → Flat ₹10,000

                S3 – Higher Tiered Subsidy
	•	If Capacity ≤ 2 kW → ₹15,000 * kW
	•	If Capacity > 2 kW → Flat ₹30,000

	3	user taps "Next" to proceed

### Step 7: Review & Generate
	1	User scrolls through a comprehensive summary:
	◦	Configuration details
	◦	Cost table with all components
	◦	Commission preview (equal to the dealer add-on total)
	2	User taps "Save & Share" which:
	◦	Creates an immutable quotation record (JSON + PDF)
                User is routed back to Quotations list


## 8. Commission & Payouts

1. **Access Commission Management**
   - Admin clicks "Commission & Payouts" in main navigation
   - System displays only leads with "Executed" status, showing:
   - Tabs with Payment Status wise queues ( ( Pending / Approved  / Paid) for easy navigation. 
      Each row contains the following details 
     - Lead Id
     - Lead Name
     - Quotation Reference Number
     - Token UTR
     - CP Name 
     - Commission Amount
     - Project Amount
     - Payment Status ( Pending / Approved / Paid) 
     - Payment UTR Number (if paid)

2. **Review Commission Details**
   - Admin clicks on any commission record
   - System displays:
     - Associated quotation details
     - Project information (status, kW capacity)
     - Commission calculation breakdown
 

3. **Approve Commissions**
   - For commissions with status "Pending", Admin clicks "Approve" button on the Lead list. 
  - System prompts for "Approve" value of Commissions in INR. Default value = Commission Amount. 
   - Status changes to "Approved"
   - Commission moves to payment queue
   - CP is notified of approval.
 

4. **Process Payments**
   - Admin goes to "Approved" tab  
   - Select Executed lead for which commissions  is to be  paid by clicking on "Pay" button. 
   - System prompts for payment details:
     - UTR (Unique Transaction Reference)
     - Payment Date
     - Payment Method
     - Optional Notes
     - Admin enters details and confirms/saves. 
     -  system automatically updates the Payment status as Paid and moves it to "Paid" Tab/queue and "Pay" button is disabled.
    -  The lead moves from "Approved" to "paid"
    
  
     - CP receives automatic notification of all Payment status changes as In-App Notification. 

5. **Commission Reports**
   - Admin clicks "Reports" tab in Commission section
   - Can generate various reports:
     - Pending Commissions Summary
     - Payments Processed (date range)
     - CP-wise Commission Breakdown
     - Project-wise Commission Analysis
   - Each report can be exported as CSV or PDF


## 9. Quotation Master Data Management


### 9.0 Menu Structure
```
Quotation Master Data
├─ Quotation Master Data
│  ├─ State‑DISCOM 
│  ├─ Panels
│  ├─ Inverters
│  ├─ BOM Components
│  ├─ Additional Structure
│  ├─ DISCOM level Fees
│  ├─ State Subsidy Type
│  ├─ Other Charges
```


1. **Access Quotation Master Data**
   - Admin clicks "Master Data" in main navigation
   - System displays menu of master data categories:
     - State-DISCOM
     - Panels
     - Inverters
     - BOM Components
     - Fees & Charges
     - State Subsidy Type
     - Additional Structure
     - Other Charges

2. **Manage State-DISCOM Mapping**
   - Admin clicks "State-DISCOM Mapping"
   - System displays current mappings
   - Admin can:
     - Add new state
     - Add DISCOMs to existing states
     - Specify smart meter availability per DISCOM
     - Set phase restrictions
   - Changes immediately affect quotation wizard logic

3. **Manage Panels**
   - Admin clicks "Panels"
   - System displays current panel catalog with details:
     - Make
     - Model/Variant
     - Wattage
     - DCR Flag (Yes/No)
     - Price per Unit
     - Active Flag
   - Admin can:
     - Add new panel types
     - Edit existing details (especially price)
     - Deactivate obsolete panels
     - Mark panels as "Recommended"

4. **Manage Inverters**
   - Admin clicks "Inverters"
   - System displays current inverter catalog
   - Similar management options as panels
   - Additional fields for phase compatibility and capacity range

5. **Manage BOM Components**
   - Admin clicks "BOM Components"
   - Manages additional components (cables, structures, etc.)
   - Sets prices and availability

6. **Manage Fees & Charges**
   - Admin clicks "Fees & Charges"
   - System displays fee configuration matrix
   - Admin manages per-combination (state+discom+phase+smart_meter) fees:
     - Discom Fee
     - Discom Miscellaneous
     - Installation
     - Meter Charges
     - Modem+Box Charges
     - Transport

7. **Manage Subsidies**
   - Admin clicks "Subsidies"
   - System displays subsidy configuration
   - Admin manages:
     - State subsidy types (S1/S2/S3)
     - Values per state/discom combination
     - Effective dates for subsidy programs
   - Central subsidy (S0) logic is hardcoded but values can be adjusted

8. **Manage Services Catalog**
   - Admin clicks "Services Catalog"
   - System displays list of available services
   - Admin can:
     - Add new service types
     - Edit service descriptions and images
     - Deactivate services
     - Set service-specific fields

9. **Manage Additional Structure**
   - Admin clicks "Additional Structure"
   - System displays structure options and pricing
   - Admin can:
     - Add new structure types
     - Set height options (0.5m increments)
     - Configure pricing per structure type
     - Define availability by region/state

10. **Manage Other Charges**
    - Admin clicks "Other Charges"
    - System displays configurable charge categories:
      - Transportation
      - Fixed Dealer Margin
      - Overheads
      - Minimum Company Margin


## 10. Global Settings

1. **Access Global Settings**
   - Admin clicks "Global Settings" in main navigation
   - System displays configuration panels for system-wide settings
- Currently Blank

6. **User Management**
   - Admin accesses system user management
   - Can create/edit/deactivate other admin users (with appropriate privileges)
   - Manages admin access levels and permissions

## 11. Support Ticket Management

### 11.1 Access Support Tickets
1. Admin clicks "Support" in main navigation
2. System displays ticket list with filters:
   - Status (Open/Closed/All)
   - Date Range
   - CP
   - Customer
   - Lead ID

### 11.2 View Ticket Details
1. Admin clicks on ticket ID in list
2. System displays ticket thread with:
   - Initial request
   - All responses
   - Current status
   - Associated lead/customer information

### 11.3 Respond to Ticket
1. In ticket detail view, Admin enters response in text field
2. Attaches files if needed (optional)
3. Clicks "Send Response"
4. Response is added to ticket thread
5. Ticket status updates appropriately
6. Customer receives notification

### 11.4 Close Ticket
1. When issue is resolved, Admin clicks "Close Ticket" button
2. System prompts for closure notes
3. Admin enters reason for closure
4. System updates ticket status to "Closed"
5. Customer receives notification


## 12. Services Catalog

12.1. **Manage Services Catalog**
   - Admin clicks "Services Catalog"
   - System displays list of available services
   - Admin can:
     - Add new services - Title, Description , Image , Price Range, Type
     - Edit existing service -  descriptions ,images and Price range
     - Deactivate  Existing service.
    - Filter with service Type


## 13. Dashboard Management

1. **Analyze Sales Funnel**
   - Admin reviews complete sales funnel visualization
   - Can drill down into each stage:
     - New Leads → In Discussion → Physical Meeting Assigned → Won → Pending at Solarium → Under Execution → Executed
   - Analyzes conversion rates between stages
   - Identifies bottlenecks in the process

2. **Monitor Key Metrics**
   - Admin tracks key performance indicators:
     - Lead Generation Rate
     - Conversion Percentages
     - Average Deal Size
     - Time in Pipeline
     - CP Productivity
   - System highlights metrics outside normal ranges

3. **Geographic Distribution**
   - Admin views map-based visualization of:
     - Lead distribution by state/region
     - Conversion rates by geography
     - Average system size by location
   - Can filter by date range and zoom to specific regions

4. **Export Dashboard Data**
   - Admin can export any dashboard chart or dataset
   - Available formats: PNG (charts), CSV/Excel (data)
   - Can schedule regular dashboard exports to stakeholders

## 14. Cross-Module Activities

1. **System-wide Search**
   - Admin uses global search function in top bar
   - Can search across all entities:
     - Leads (by ID, name, phone)
     - CPs (by name, phone)
     - Customers (by name, phone)
     - Quotations (by reference number)
   - Results grouped by entity type with quick links

2. **Notification Management**
   - Admin clicks notification bell in top bar
   - Views system notifications of all critical events
   - Can mark notifications as read/unread
   - Configures personal notification preferences


# Detailed Key Account Manager (KAM) User Workflow

## 1. Login & Authentication

### 1.1 Accessing the Portal
1. KAM navigates to the Solarium Web Portal URL in a web browser
2. System presents the login screen with email and password fields

### 1.2 Login Process
1. KAM enters their registered email address and password
2. Clicks "Login" button
3. System validates credentials
   - If valid, grants access to the KAM Dashboard
   - If invalid, displays appropriate error message

### 1.3 Password Recovery (if needed)
1. KAM clicks "Forgot Password" link on login screen
2. Enters registered email
3. System sends reset link via email
4. KAM follows link to create a new password
5. Returns to login with new credentials

## 2. Dashboard & Navigation Overview

### 2.1 Initial Dashboard
1. Upon successful login, KAM sees their personalized dashboard
2. Dashboard displays:
   - Sales funnel visualization for assigned Channel Partners (CPs)
   - Key metrics: New Leads, In Discussion, Physical Meeting Assigned, Won, Pending at Solarium, Under Execution, Executed
   - Time period filters: Today, Last 7 Days, Last 30 Days, Last Quarter, Custom Date Range
   - CP-specific filters

3.  **Export Dashboard Data**
   - Admin can export any dashboard chart or dataset
   - Available formats: PNG (charts), CSV/Excel (data)
   - Can schedule regular dashboard exports to stakeholders

### 2.2 Main Navigation Menu
1. Left sidebar contains main navigation options accessible to KAM:
   - Dashboard
   - Channel Partners
   - Customers
   - Leads Management
   - Quotation Management
   - Commission & Payouts - Read Only 
   - Support

### 2.3 Top Bar Elements
1. User profile icon (access to profile settings)
2. Notifications bell (system alerts)
3. Quick filters (date range, CP)

## 3. Manage Channel Partners

### 3.1 Access Channel Partner Management
1. KAM clicks "Channel Partners" in the main navigation
2. System displays a list of CPs assigned to this KAM with key details:
   - Name
   - Phone
   - Registration Date
   - Status (Registered/Deactivated)
   - Total Leads Count
   - Executed Projects Count

### 3.2 View Channel Partner Details
1. KAM clicks on any CP name in the list
2. System displays CP profile with tabs:
   - Basic Info (view-only: name, email, address, PIN, phone)
   - Assigned Leads (filterable list)
   - Quotations (all quotations generated by this CP)
   - Commission History (all commissions earned/pending/paid for each CP). Read - only. 
   - Activity Timeline (chronological audit of all actions)

## 4. Manage Customers

### 4.1 Access Customer Management
1. KAM clicks "Customers" in main navigation
2. System displays list of all customers associated with KAM's assigned CPs:
   - Name
   - Phone (primary identifier)
   - Lead Count
   - Total Projects Value
   - Registration Date

### 4.2 View Customer Details
1. KAM clicks on customer name in list
2. System displays customer profile with tabs:
   - Basic Info (view-only: name, phone, email, address, PIN)
   - KYC Documents checklist (all uploaded verification documents)
   - Leads (all leads associated with this customer)
   - Quotations (all quotations generated for this customer)
   - Support Tickets (ticket history)

### 4.3 Manage Customer Documents
1. In customer detail view, KAM navigates to "KYC Documents" tab
2. For each document type (Income Proof, Light Bill, Cancelled Cheque, Aadhaar, PAN, Photograph):
   - Views current document (if uploaded) with Buttons - Approve /reject . ( The doc status remains as Pending Review") 
   - Clicks "Approve"  and Status CHANGES TO  Approved. 
   - CLICKS "Reject" to change document status TO Rejected.
   - For "Reject", enters reason which is sent to the customer 
3. Document statuses: Pending Review, Approved, Rejected.

## 5. Manage Leads

All modules of Manage leads for Admin except - Bulk lead Operations / Unassigned leads 


1. **Access Lead Management**
   - KAM clicks "Leads Management" in main navigation
   - System displays comprehensive lead list with filters:
     - Status (All/New Lead/In Discussion/etc.)
     - Date Range (Created/Follow-up)
     - Assigned CP ( Unassigned / List of all CPs in the Db) 
     - Origin (CP/Customer/Admin)
     - State
     - Source 
     

2. **View Lead Details**
   - KAM clicks on any lead ID in the list
   - System displays lead detail with five tabs (matching CP app structure):
     - Info (basic lead information, editable, including customer mapping status)
     - Status (current status and history)
     - Quotations (all quotations generated for this lead)
     - Docs (all site-specific documents)
     - Timeline (audit log of all changes)

3. **Create New Lead**
   - KAM clicks "Add New Lead" button 
   - Form appears requesting:
     - Customer details (Name, Phone, Email, Address, State, PIN)
     - Service Requirements
     - Assigned CP (dropdown of active CPs)
     - Initial Status (default: New Lead)
     - Follow-up Date (default: today+1)
     - Automatic map to  Customer based on the lead phone number and customer phone number (to link lead with existing customer)  or create  a new customer. 
   - KAM completes form and clicks "Create Lead"
   - System creates lead record and notifies assigned CP



5. **Reassign Lead to Different CP**
   - In lead detail view, KAM clicks "Reassign" button
   - System displays dropdown of active CPs
   - KAM selects new CP and provides reason for reassignment
   - System updates lead assignment and notifies both old and new CPs
   - KAM can only Reassign leads within CPs assigned to themselves. 

6. **Force Status Change**
   - In lead detail Status tab, KAM can override normal status flow
   - Selects target status from complete dropdown (not restricted by matrix)
   - Provides mandatory reason for override
   - System updates status and logs the admin override in Timeline






## 6. Quotation Management

1. **Access Quotation Management**
   - KAM clicks "Quotation Management" in main navigation
   - System displays list of all quotations with filters:
     - Status (Valid/Rejected/All)
     - Date Range
     - CP
     - Lead ID
     - Capacity Range


2. **View Quotation Details**
   - KAM clicks on quotation in  the list
   - System displays:
     - All configuration details used to generate it
     - Associated lead information
     - Current status and timestamps
     -  KAM clicks on "Share with custom AND AND the flag is updated and the customer can see the Quote on his Customer app.
     -  KAM  can download the Quote and share it with people outside the solution. 
     - Every quotation has a flag indicating "Shared with Customer " - Yes/No. 
     - KAM can download the quotation via Download button.


4. **Generate Quotation**
   - KAM can generate quotations directly by clicking "Generate Quotation" button
   - Selects lead from dropdown or search
   - Completes same 7-step quotation wizard as CP app:
   - 7-Step Quotation Wizard - Step by Step Process

###  Step 1: Location
	1	User selects the following parameters:
	◦	State from dropdown (alphabetically sorted)
	◦	DISCOM from dropdown (filtered based on selected state)
	◦	Phase using radio buttons: Single/Three (default is Single)
	◦	Smart-Meter toggle (Yes/No). If only one option available for "Smart Meter Flag" for the State/Discom/Phase combination it is selected as Default. 
	2	User taps "Next" to proceed
###  Step 2: Panels
	1	KAM selects:
	◦	Panel Make from dropdown (recommended makes appear at the top)
	◦	Variant from dropdown (options depend on selected Make)
	▪	The system automatically sets the DCR flag based on the variant
	◦	Number of Panels using a stepper control
	▪	Range is 4-9 panels for single-phase
	▪	Range is 7-18 panels for three-phase
	2	A live capacity calculation is displayed (capacity in kW = number of panels × panel wattage ÷ 1000)
	3	User taps "Next" to proceed
###  Step 3: Inverter & BOM (Bill of Materials)
	1	User selects:
	◦	Inverter from a dropdown list
	▪	List is filtered to show only options within ±15% of the calculated capacity and matching the selected phase
	▪	Recommended inverters are marked with a star (★)
	◦	Cable Type: Standard or Polycab
	◦	Extra Structure Height: 0m (default), with 0.5m increments up to 2m
	2	User taps "Next" to proceed
###  Step 4: Fees Lookup
	1	User views a read-only summary of all applicable fees
	2	Fees are automatically calculated based on the selected state, DISCOM, phase, and smart meter flag
	3	The summary includes:
	◦	Discom Fee
	◦	Discom Miscellaneous charges
	◦	Installation
	◦	Meter charges
	◦	Modem+Box charges
	4	User taps "Next" to proceed

###   Step 5 : Dealer Add on : 
	2	User adjusts the Dealer Add-on using a slider:
	◦	Range: 0 to ₹2,000 per kW
	◦	Default: ₹1,800 per kW

###  Step 6: Pricing & Subsidy and Subsidies
	1	KAM views:
	◦	Hardware Subtotal (sum of panels cost, inverter cost, BOM costs, Other Costs ( transportation, Overhead , minimum company margin , dealer margin) , Additional Structure Cost , and  Discom Level Charges ( Fees).)
	◦	Inflated Base Price (hardware subtotal × 1.10, where 10% is the  inflation percentage set in the "Global settings" by the Admin.  
	1	The screen displays a live preview of the System Price (inflated base + dealer add-on total)
	2	User taps "Next" to proceed

		Subsidies 
	2	User views the automatically calculated subsidies:
 
		◦Central Subsidy (S₀) calculated according to rules:

	▪	Less than 1 kW: ₹0
	▪	Up to 2 kW: capacity × ₹30,000
	▪	Between 2-3 kW: ₹60,000 + (capacity-2) × ₹18,000
	▪	3 kW or more: capped at ₹78,000

		◦State Subsidy looked up from master data (S₁/S₂/S₃ depending on state rules) and Panel Type for the requirement., 
                Condition:  State Subsidy applies only for DCR panels.
                S1 – No Subsidy

                S2 – Tiered Subsidy
	•	If Capacity ≤ 5 kW → ₹2,000 * kW
	•	If Capacity > 5 kW → Flat ₹10,000

                S3 – Higher Tiered Subsidy
	•	If Capacity ≤ 2 kW → ₹15,000 * kW
	•	If Capacity > 2 kW → Flat ₹30,000

	3	user taps "Next" to proceed

### Step 7: Review & Generate
	1	User scrolls through a comprehensive summary:
	◦	Configuration details
	◦	Cost table with all components
	◦	Commission preview (equal to the dealer add-on total)
	2	User taps "Save & Share" which:
	◦	Creates an immutable quotation record (JSON + PDF)
	◦	User is routed back to User List. 
	◦	




## 7. Support Ticket Management

### 7.1 Access Support Tickets
1. KAM clicks "Support" in main navigation
2. System displays ticket list with filters:
   - Status (Open/Closed/All)
   - Date Range
   - CP
   - Customer
   - Lead ID

### 7.2 View Ticket Details
1. KAM clicks on ticket ID in list
2. System displays ticket thread with:
   - Initial request
   - All responses
   - Current status
   - Associated lead/customer information

### 7.3 Respond to Ticket
1. In ticket detail view, KAM enters response in text field
2. Attaches files if needed (optional)
3. Clicks "Send Response"
4. Response is added to ticket thread
5. Ticket status updates appropriately
6. Customer/CP receives notification

### 7.4 Close Ticket
1. When issue is resolved, KAM clicks "Close Ticket" button
2. System prompts for closure notes
3. KAM enters reason for closure
4. System updates ticket status to "Closed"
5. Customer/CP receives notification


## 9. User Profile & Settings

### 9.1 Access Profile Settings
1. KAM clicks profile icon in top bar
2. Selects "My Profile" from dropdown
3. System displays profile page with current information

### 9.2 Edit Profile Information
1. KAM can edit:
   - Name
   - Email
   - Phone
   - Profile Photo
2. Clicks "Save Changes" to update information

### 9.3 View Activities & Notifications
1. KAM clicks "Notifications" in profile menu
2. System displays list of all system notifications:
   - New CP assignments
   - Lead status changes
   - Quotation generations
   - Commission updates 
   - Ticket activities


# Customer Mobile App Workflow for Solarium Green Energy

## 1. Registration & Login Process

### 1.1 Installing the App
1. Customer downloads "Solarium Green Energy" app from App Store/Play Store
2. Upon opening, customer sees Solarium splash screen followed by Login page
3. Login page shows Phone field, "Get OTP" button

### 1.2 Registration & First-Time Login
1. **New Customer Flow:**
   - Customer enters phone number and taps "Get OTP"
   - Enters 6-digit OTP received via SMS (valid for 2 minutes, 5 attempts max)
   - System detects new phone number and presents Profile Form
   - Customer completes required fields: Name*, Address*, State*,PIN code* (Email optional)
   - State should come as a Dropdown selection from all Indian States listed. 
   - Submits form → account created → lands on "Our Services" screen
  -- The system checks for the uniqueness of the Phone number also. There is a 1:1 mapping of phone number to customer.

2. **Returning Customer Flow:**
   - Customer enters registered phone number and taps "Get OTP"
   - Enters 6-digit OTP received via SMS
   - System recognises phone number → directly lands on "Services" screen

## 2. Main Navigation & Dashboard

### 2.1 App Structure
- **Bottom Navigation:**  Services | Documents | My Records | Help
- **Top Bar:** Sync icon, Notifications bell, Profile icon
- **Home Screen:** "Our Services" with available services to buy/install

### 2.2 Services Screen
1. Displays catalog of all available services that customer can purchase/install
2. Services organized by categories (Solar Installation, Maintenance, Upgrades, etc.)
3. Each service card shows:
   - Service name and thumbnail image
   - Brief description
   - Price range indicator (if applicable)
   - "Add to Cart" button
4. Pull-to-refresh for syncing latest data
5. Search functionality to find specific services.
6. FAB (Cart icon) for quick access to cart.

### 2.3 My Records Screen
1. Accessible via bottom navigation "My Records" tab
2. Displays all customer's previous service requests in chronological order
3. Tabs for filtering: All / Active / Closed
4. Each record card shows:
   - Service ID and date
   - Current status with color-coded pill
   - Service type requested
   - Next follow-up date (if applicable)
5. Pull-to-refresh for syncing latest data
6. Search functionality to find specific service records


### 2.4 Documents 

### 2.5 Help



## 3. Creating New Service Requests

### 3.1 Service Catalog & Cart
1. Customer browses available services on the "Services" screen
2. Catalog grid displays available services/products:
   - Solar Installation packages
   - Maintenance services
   - Upgrade options
   - Other renewable energy solutions
3. Customer taps desired service → "Add to Cart"
4. Can add multiple services if needed
5. Cart icon shows count of selected items
6. There is a FAB (Cart Icon) that routes to cart. 

### 3.2 Checkout Process
1. Customer taps cart icon to review selections
2. Confirms or edits service selections
3. Verifies/updates installation address
4. Adds any special instructions (optional)
5. Taps "Submit Service Request"
6. System creates new lead with:
   - Status = "New Lead"
   - Follow-up date = Today + 1 day
   - Origin = Customer
7. Confirmation screen appears with service ID
8. New service appears at top of "My Records" list
9. "My Records" screen has the list of all services requested which leads to a detailed service request. 
10. All quotations shared with the Customer via CP App / Admin and KAMs also reflect here. ( Quote "shared with customer" flag is Yes.)  
11. With each quotation , there is an option to Accept or Reject the quotation. 
12. At any point in time only one quotation can be "Accepted" and all other quotation is Rejected and freezes. 
13. As soon as a Quotation is accepted by the customer the service status (lead status) automatically updates to "Won"


## 4. Service Detail & Tracking

### 4.1 Viewing Service Details
1. Customer taps any service card in "My Records" list
2. Service Detail screen opens with four tabs:
   - **Info:** Basic service details (service type, date, status, etc.)
   - **Quotes:** Quotations generated for this service
   - **Docs:** Documents related to this service
   - **Tickets:** Support tickets linked to this service
   

### 4.2 Info Tab
1. Displays read-only information:
   - Service ID and creation date
   - Current status and next follow-up date
   - Service requested
   - Installation address
   - Assigned Channel Partner (if any)

### 4.3 Quotations Tab
1. Shows list of all quotations generated for this service
2. Each quotation displays:
   - Thumbnail preview
   - Date generated
   - Status (Accepted /Rejected )
3. Customer can:
   - Download PDF quotation
   - Accept/Reject Quotation -- according Status Transitions. 
   - For a single service , customer can only accept one quotation. One to one : service to Quote mapping.


## 5. Document Management

### 5.1 Accessing Documents
1. Customer can manage  KYC documents in one way:
   - Via bottom navigation "Documents" tab (for all documents)

2. Customer can manage  lead docs in one way:
   - Via  "My Records" in the bottom navigation>> Service details  → "Docs" tab (for service-specific documents)

### 5.2 Customer-Level Documents (KYC)
1. From main "Documents" tab, customer sees KYC document checklist:
   - Income Proof
   - Light Bill (latest)
   - Cancelled Cheque (latest)
   - Aadhaar Card
   - PAN Card
   - Passport-size Photograph
   - Other
2. Each Uploaded KYC document displays status: Pending review / Approved / Rejected

### 5.3 Uploading Documents
1. Customer taps "Upload" button next to any document type
2. Can select document source (camera, gallery, files)
3. Document requirements: PDF, JPG, or PNG format, ≤10 MB
4. Images are automatically compressed to ≤75% quality
6. Taps "Upload" to submit document
7. This Upload button is only active and available for Docs where there is no File uploaded or Doc Status is "Pending review / Rejected"

### 5.4 Document Status Management
0. Customer can Re-upload a doc if the status is "Pending Review" by clicking on "Upload" button and selecting another file, which replaces the previous file. 
1. After admin/KAM review , document status updates to:
   - **Approved:**  if Document is Approved
   - **Rejected:**  if Document is Rejected

2. For rejected documents:
   - Customer receives notification
   - Rejection reason is displayed
   - "Upload" button re-appears

3. Customer can tap "Upload" to submit new version of document

## 6. Support Ticket System

### 6.1 Accessing Tickets
1. Customer taps "Help" in bottom navigation
2. Help screen shows central helpline number with call icon at the top
3. Below this, sees "Support Tickets" section with list of all support tickets (Active/Resolved)
4. Can filter by status or search by ticket ID/subject

### 6.2 Creating New Support Ticket
1. From Help screen, customer taps "Create New Ticket" button
2. OR from Service Detail → Tickets tab, taps "Create Ticket"
3. Ticket creation form appears:
   - Dropdown to select related service (mandatory if multiple services exist)
   - Subject field (required)
   - Description field (required)
   - Option to attach photos/documents (≤10 MB, up to 3 files)
4. Customer completes form and taps "Submit"
5. System creates ticket, assigns unique ID, and notifies appropriate team

### 6.3 Ticket Conversation
1. Customer taps any ticket to view conversation thread
2. All messages displayed chronologically
3. Customer can:
   - Add new replies
   - Attach additional files
   - View admin/KAM responses
4. Each message shows sender name and timestamp

### 6.4 Resolving Tickets
1. When admin marks ticket as "Resolved", customer receives notification
2. In ticket detail view, customer sees:
   - Resolution summary
   - "Re-open" button (if issue persists)
   - "Close Ticket" button (to confirm resolution)
3. Customer can tap "Close Ticket" to finalize resolution

## 7. Help & Support

### 7.1 Help Center
1. Customer taps "Help" in bottom navigation
2. At the top, sees central helpline number with prominent call icon
3. Below helpline, Help Center displays:
   - Support tickets section (list + create new)
   - FAQ sections
   - Video tutorials
   - User guides

### 7.2 Direct Support Options
1. From Help screen, customer can:
   - Call central helpline (tapping phone icon initiates call)
   - Create new support ticket
   - View ticket history
   - Email support team
   - Schedule callback request (if available)

## 8. Profile & Settings

### 8.1 Accessing Profile
1. Customer taps profile icon in top bar
2. Profile & Settings screen appears

### 8.2 Managing Profile Information
1. Customer can edit:
   - Name
   - Email
   - Address
   - PIN code
2. Phone number is displayed but cannot be changed (primary identifier)

### 8.3 Notification Preferences
1. Customer can view notification history
2. Toggle notification types:
   - Status updates
   - Document status changes
   - Quotation alerts
   - Ticket responses

### 8.4 Other Settings
1. Language preference (English) Non-mutable.Just for reference. 
2. Privacy policy and terms of service
3. App version information
4. Logout option

## 9. Service Lifecycle From Customer Perspective

### 9.1 Status Visibility
1. Customer can track service progress through status changes:
   - **New Lead:** Service requested, awaiting initial contact
   - **In Discussion:** CP has contacted customer, discussing requirements 
   - **Physical Meeting Assigned:** Site visit scheduled
   - **Won:** Customer has agreed to proceed. Customer has accepted a Quotation from the list of Quotes generated and shared.
   - **Pending at Solarium:** Project awaiting internal processes. 
   - **Under Execution:** Installation in progress. 
   - **Executed:** Project completed

### 9.2 Quotation Review & Acceptance
1. When CP generates quotation, customer receives notification
2. Customer can review quotation via My Records >> Service Detail → Quotes tab
3. Customer discusses quotation with CP
4. CP updates status based on customer decision

### 9.3 Installation & Completion
1. Once project reaches "Under Execution":
   - Customer can track progress via app
   - Upload additional documents if requested
   - Raise tickets for any concerns
2. Upon completion (status = "Executed"):
   - Customer receives notification
   - Final documentation becomes available
   - Support ticket access remains for any post-installation issues

Updated RBAC matrix — Admin vs KAM

Module / Action	Admin	KAM	Key excerpts
Dashboard	Full company‑wide view, date + geo filters, export	Territory‑scoped funnel only, export
Channel Partners	Create ∣ activate ∣ deactivate ∣ assign & bulk‑assign KAMs; edit profile	Read‑only profile & lists for own CPs	
KAM management	Full CRUD	—	
Customer profiles	View all; approve / reject KYC; edit info	View assigned customers; approve / reject KYC	
Lead — create / edit	✓ (any CP)	✓ (within own CPs)	
Lead — reassign CP	✓ (any CP)	✓ (but only within CPs assigned to KAM)	
Lead — force status override	✓ (no matrix limits)	✓ (same override, logged; still limited to owned leads)	
Bulk lead ops (reassign/export/import/force)	✓	✗	
Unassigned‑leads queue	✓	✗	

Quotation wizard	Generate & share for any lead	Generate & share for owned leads	

Quotation master‑data / pricing tables	Full CRUD	—	

Commission & Payouts	Approve ∣ mark Paid ∣ reports	Read‑only queue (menu present)	

Support tickets	View / reply / close (all)	View / reply / close for owned CPs & customers	

Global settings / master data	✓	—	

System‑wide search & exports	✓	Scoped to entities the KAM can access	

User management (admins)	Create / edit / deactivate	—

