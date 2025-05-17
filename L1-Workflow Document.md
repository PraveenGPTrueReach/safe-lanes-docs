## L1-WF-[SafeLanes] Workflow Document for SafeLanes RH Module

#### User Roles and Access Management

###### User Roles
- There are 2 categories of users 
	- Vessel Users 
		- Vessel Super Admin (Master)
		- Vessel Admins (Senior Officers)
		- Vessel Users (Vessel Crew)
	- Office Users
		 - Office Super Admins (Senior Managers)
		 - Office Admin (Managers who manage Group of Vessels)
		 - Office User (Superintendents who manage 4-5 vessels)
		 - External (External Parties or Auditors)

###### Vessel Users Access
1. Vessel Super Admin (Master)
- Master will have access to everything including Screen V1.0a which will show all details pertaining to the assigned Vessel
- Depending upon the client policy, they may have rights to edit the form in Screen V2.0c 


2. Vessel Admin (Senior Officers)
- The senior officers will be department heads who will be involved in:
	- Editing Planning Screens V3.0a & V3.0b  
	- Viewing all the Recording screens V2.0a, V2.0b, V2.0c
	- Viewing Vessel Dashboard Screen V1.0a for their Department only
- Depending upon the client policy, they may have rights to edit the form in V2.0c 
 
3. Vessel User
- These will be all other crew members who would primarily be editing their own records i.e. in screen V2.0c and see the summary in screen V2.0b
- Basis login credentials crewmember can only view his name and details in screen V2.0b or edit his own form in screen V2.0c.  


###### Office Users Access
1. Office Super Admin 
- This user has the highest access level. They will have full access to every feature.
- They would typically be senior managers who would be focussed on:
	- Dashboard/ Analytics (1a)
	- Any changes required in Admin section (Access control).

2. Office Admin  
- These will typically be managers who are in charge of a group of vessels and would be focussed on:
	- Dashboard / Analytics (1a) for their vesselgroup
	- Viewing Recording screens (2a, 2b)
	- Depending upon the client policy, they may have rights to edit the form in 2c (though in principle the 2c form should only be filled by the vessel users)
	- Viewing planning screens 3a, 3b to see how effectively the vessels are using the planning function.

3. Office User
- These will typically be superintendents who are in charge of a few (4-5) vessels and would be closely involved in:
	- Dashboard / Analytics (1a) for vessels under their charge
	- Viewing all the Recording screens 2a, 2b, 2c.
	- Depending upon the client policy, they may have rights to edit the form in 2c (though in principle the 2c form should only be filled by the vessel users)
	- Viewing planning screens 3a, 3b to see how effectively the vessels are using the planning function.

4. External
- This would generally be external parties or auditors etc and the access will be view only. The client can provide View access to these users in SAIL application Admin panel 

###### Auth
- All users will login using their SAIL login and password
- This will be governed by the SAIL application



###### User Workflows

######## 1. Vessel User
########## 1.1 Log In to the Vessel System
- Open the SAIL application on the local device
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Access is restricted to the Vessel User’s role and assigned vessel

########## 1.2 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Vessel User can access:
	- Dash which will show Screen V2.0b for the Vessel User
	- Screen V2.0b and Recording Form (Screen V2.0c), To enter daily rest/work hours.
- Click dash on left navigation pane to open Screen V2.0b
 
########## 1.3 Check the RH Vessel Overview (Screen V2.0b)
- The user can see their Rank, name and basic info: date range on the top
- Default view is for current month
- The Dash also shows following information for the respective Vessel User – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
- In addition to information, there are 2 clickable icons - View and Edit. Clicking View and Edit icons opens up Screen V2.0c where user can view or edit the details
- User can also Filter using a date range (e.g., current week or month)
- Clicking on Total Violations or Total NCs opens up Screen V2.1b which shows the violation or NC details for the User


########## 1.4 Record Daily Rest Hours (Screen V2.0c)
- Open the Recording Form for the current date by Clicking "View" clickable icon next to his details on Screen V2.0b
- View His Rank, Name on the top and filter entries by checking or unchecking Planning/OPA/Date Range
- Enter Work/Rest Hours
	- “w” for routine watch, “a” for additional work and "d" for routine day work in hour-by-hour blocks further sub-divided into half hour blocks
	- typing "w" or "d" will change the color of box to green and typing "a" will turn the color of box to blue
	- Enter any relevant comments in the Comments section
- View Violations with relevant codes against each date, as applicable
- View total Violations or NCs as applicable
- Check for Pre-Filled Planning Data by checking "Planning" box (Grey Entries)
	- If a Vessel Admin/HOD has planned tasks), the user may see grey “planned” blocks.
	- The user can confirm these blocks by clicking checkbox to mark it “w” or "d" which changes them to green or mark it "a" which changes the box to blue color
- Modify Planned Hours (If Actual Differs)
	- The user can adjust any planned hours by clicking the checkbox to mark it “w” or "d" or “a”
- Save Entriesby clicking "Apply" button
	- Apply or Save triggers immediate validation checks for potential violations


########## 1.5 View  Violations (Screen V2.0c)
- Violation Codes
	- If the total work hours exceed allowed limits (daily or weekly), the system automatically shows a violation code.
	- If OPA is selected and relevant, it may display those additional violation codes (7 & 8).
- Resolve or Acknowledge
	- The Vessel User can modify their entries immediately to resolve any violations, or they can leave them as-is 

########## 1.6 Log Out
- End the Session by logging out of the application.
- The user can log back in at any time to update entries (for the current open period) or to address any newly flagged violations


######## 2. Vessel Admin

########## 2.1 Log In to the Vessel System
- Open the SAIL application on the local device
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Access is restricted to the Vessel Admin’s role and assigned vessel

########## 2.2 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Vessel Admin can access:
	- Dash or Vessel Overview (Screen V2.0b) details for only their own department
	- Recording Form (Screen V2.0c) (view or edit as needed)
	- Planning – Fixed Tasks (Screen V3.0a)
	- Planning – Variable Tasks (Screen V3.0b)

########## 2.3 Check Vessel Overview (Screen V2.0b)
- Navigate to Dash from left side navigation pane which shows Vessel Overview (Screen V2.0b)
	- the Vessel Admin can see details of all crew in their Department, their Rank, name and basic info
	- Default view is for current month
	- The Dash also shows following information for the respective Vessel User – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
	- Clicking on Total Violations or Total NCs against a crew member will open up Screen V2.1b which will show details of the NCs or Violations
	- In addition to information, there are 2 clickable icons - View and Edit. Clicking View and Edit icons opens up Screen V2.0c where user can view or edit the details
- Filter Options
	- Filter by date range, rank, or crew member.
	- Identify if there are any missing entries, or outstanding violations
	- The Vessel Admin may see or edit data related to all Vessel Users in his department 
- Incomplete Data
	- The Recording Status bar shows completion status of entries for the selected duration
	- View which Vessel Users have incomplete data or unresolved issues
	
########## 2.4 Record Daily Rest Hours (Screen V2.0c)
- Open the Recording Form for the current date by Clicking "View" clickable icon next to his details on Screen V2.0b
- View His Rank, Name on the top and filter entries by checking or unchecking Planning/OPA/Date Range
- Enter Work/Rest Hours
	- w” for routine watch, “a” for additional work and "d" for routine day work in hour-by-hour blocks further sub-divided into half hour blocks
	- typing "w" or "d" will change the color of box to green and typing "a" will turn the color of box to blue
	- Enter any relevant comments in the Comments section
- View Violations with relevant codes against each date, as applicable
- Check for Pre-Filled Planning Data by checking "Planning" box(Grey Entries)
	- If a Vessel Admin has planned tasks, the user may see grey “planned” blocks.
	- The user can confirm these blocks by clicking checkbox to mark it “w” or "d" which changes them to green or mark it "a" which changes the box to blue color
- Modify Planned Hours (If Actual Differs)
	- The user can adjust any planned hours by clicking the checkbox to mark it “w” or "d" or “a”
- Save Entries by clicking "Apply" button
	- Apply or Save triggers immediate validation checks for potential violations.

########## 2.5 Manage Planning (Fixed & Variable Tasks)

- Click on Plan button in Left navigation pane to access Planning form (Screen V3.0a)
- Vessel Admin can create/edit tasks for his department personnel
- Choose Fixed Tasks or Variable Tasks through a Toggle Button
	- Fixed Tasks (Screen V3.0a)
		- Create/Edit watch schedules or Day work tasks by typing in time slots(“w” for routine Watch, “d” for routine Daywork and "a" for additional work)
		- All Planned tasks will be shown in grey color
		- Working hours can be marked against "At Sea" or "In Port"
		- Comments can be added in the comments column
		- Count of Daily Rest Hours is shown "At Sea" or "In Port" as applicable 
		- Save or Submit changes by clicking "Save" or "Submit" buttons
	- Variable Tasks (Screen V3.0b)
		- Default Screen shows list of all current Variable Tasks with Start and Finish Date/Time, Task, Status (Planned/Executed), Number of crew involved and Remarks. 
		- Next to each task there are clickable icons to attach docs/View details/Edit and Delete Task
		- Variable Tasks can be added by clicking "Add" button with start/finish times and assign crew from a pop-up menu. Option to enter remarks and mark task as Planned or Executed
		- Mark tasks “planned” initially (grey). User or admin can later mark them “w” or "d" or "a" or override if actual events differ.


########## 2.6 Review & Oversee Crew Records (Recording Form 2c)
- Vessel Admin can View records for Crew members in his department by clicking "View" icon next to crew member name in Screen V2.0b. This will open Screen V2.0c for respective crew member
- Vessel Admin can Edit records for Crew members in his department by clicking "Edit" icon next to crew member name in Screen V2.0b. This will open Screen V2.0c for respective crew member where admin can edit details if permission to do so has been provided in the SAIL application
- Vessel Admin may update rest-hour entries on behalf of a crew member 
- The system will re-check for violations upon saving.


########## 2.7 Log Out
- End the Session by logging out of the application.
- The user can log back in at any time to update entries (for the current open period) or to address any newly flagged violations



######## 3. Vessel Super Admin

########## 3.1 Log In to the Vessel System
- Open the SAIL application on the local device
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Vessel Super Admin can access all information pertaining to all Users and Admins on the assigned Vessel

########## 3.2 Access RH Vessel Dashboard
- this is the default screen that opens up when Vessel Super Admin logs in (Screen V1.0a)
	- Can view details only for own Vessel
	- Choose Filters & View Mode – Select Period, check or uncheck OPA to view or hide OPA violations
	- Watchlist Panel: Lists Events, Significant NCs (last 60 days or Predicted)
	- Performance Metrics - Total Violations, Significant NCs, Predicted NCs
		- Clicking on Total Violations, Significant NCs or Predicted NCs open Screen V1.1a with details on these
	- Periodic Analysis Chart - Timeline Graph (Yearly, Quarterly, Monthly) wit plot of Significant NCs and Violations
	- Rank & Dept. Analysis  - Displays which ranks/departments (e.g., 2/E, 3/O, C/E) have the highest violation or NC count.
	- Task Analysis  - Bar chart illustrating how many violations or NCs are tied to specific tasks or events

########## 3.3 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Vessel Super Admin can access:
	- Vessel Rest Hours Dashboard (Screen V1.0a)
	- Vessel Overview (Screen V2.0b) details for all crew
	- Recording Form (Screen V2.0c) (view or edit as needed)
	- Planning – Fixed Tasks (Screen V3.0a)
	- Planning – Variable Tasks (Screen V3.0b)

########## 3.4 Check Vessel Overview (Screen V2.0b)
- Navigate to Dash from left side navigation pane which shows Vessel Overview (Screen V2.0b)
	- the Vessel Super Admin can see details of all crew, their Rank, name and basic info
	- Default view is for current month
	- The Dash also shows following information for the respective Vessel User – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
	- Clicking on Total Violations or Total NCs against a crew member will open up Screen V2.1b which will show details of the NCs or Violations
	- In addition to information, there are 2 clickable icons - View and Edit. Clicking View and Edit icons opens up Screen V2.0c where user can view or edit the details
- Filter Options
	- Filter by date range, rank, or crew member.
	- Identify if there are any missing entries or outstanding violations
	- The Vessel Super Admin may see or edit data related to all Vessel Users and Admins
- Incomplete Data
	- The Recording Status bar shows completion status of entries for the selected duration
	- View which Vessel Users or Admins have incomplete data or unresolved issues
	
########## 3.5 Record Daily Rest Hours (Screen V2.0c)
- Open the Recording Form for the current date by Clicking “Record” on Left Navigation Pane
- View His Rank, Name on the top and filter entries by checking or unchecking Planning/OPA/Date Range
- Enter Work/Rest Hours
	- w” for routine watch, “a” for additional work and "d" for routine day work in hour-by-hour blocks further sub-divided into half hour blocks
	- typing "w" or "d" will change the color of box to green and typing "a" will turn the color of box to blue
	- Enter any relevant comments in the Comments section
- View Violations with relevant codes against each date, as applicable
- Check for Pre-Filled Planning Data (Grey Entries)
	- If a Vessel Super Admin has planned tasks, the user may see grey “planned” blocks.
	- The user can confirm these blocks by clicking a checkbox to mark it “w” or "d" which changes them to green or mark it "a" which changes the box to blue color
- Modify Planned Hours (If Actual Differs)
	- The user can adjust any planned hours by clicking the checkbox to mark it “w” or "d" or “a”
- Save Entries by clicking "Apply" button
	- Apply or Save triggers immediate validation checks for potential violations.


########## 3.6 Manage Planning (Fixed & Variable Tasks)

- Click on Plan button in Left navigation pane to access Planning form (Screen V3.0a)
- Vessel Super Admin can create/edit tasks for all vessel users and admins
- Choose Fixed Tasks or Variable Tasks through a Toggle Button
	- Fixed Tasks (Screen V3.0a)
		- Create/Edit watch schedules or Day work tasks by typing in time slots(“w” for routine Watch, “d” for routine Daywork and "a" for additional work)
		- All Planned tasks will be shown in grey color
		- Working hours can be marked against "At Sea" or "In Port"
		- Comments can be added in the comments column
		- Count of Daily Rest Hours is shown "At Sea" or "In Port" as applicable 
		- Save or Submit changes by clicking "Save" or "Submit" buttons
	- Variable Tasks (Screen V3.0b)
		- Default Screen shows list of all current Variable Tasks with Start and Finish Date/Time, Task, Status (Planned/Executed), Number of crew involved and Remarks. 
		- Next to each task there are clickable icons to attach docs/View details/Edit and Delete Task
		- Variable Tasks can be added by clicking "Add" button with start/finish times and assign crew from a pop-up menu. Option to enter remarks and mark task as Planned or Executed
		- Mark tasks “planned” initially (grey). User or admin can later mark them “w” or "d" or "a" or override if actual events differ.



########## 3.7 Review & Oversee Crew Records (Recording Form 2c)
- Vessel Super Admin can View records for all vessel users and admins by clicking "View" icon next to crew member name in Screen V2.0b. This will open Screen V2.0c for respective crew member
- Vessel Super Admin can Edit records for all vessel users and admins by clicking "Edit" icon next to crew member name in Screen V2.0b. This will open Screen V2.0c for respective crew member where admin can edit details if permission to do so has been provided in the SAIL application
- Vessel Super Admin may update rest-hour entries on behalf of a crew member 
- The system will re-check for violations upon saving.


########## 3.8 Log Out
- End the Session by logging out of the application.
- The user can log back in at any time to update entries (for the current open period) or to address any newly flagged violations


######## 4. External

########## 4.1 Log In to the SAIL app
- Open the SAIL application on the browser
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- User will have only view access to the Rest Hours Data for the vessel Screen V1.0a
	- Permissions will be governed through Admin Panel in the SAIL app



######## 5. Office User

########## 5.1 Log In to the SAIL app
- Open the SAIL application on the office device browser
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Access is limited to vessels and data the user is authorized to manage or view

########## 5.2 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Office User can access:
	- RH Dashboard - Office (Screen O 1.0a)for the vessels under Office User's charge
	- Recording Screens O2.0a, O2.0b and O2.0c
	- Depending upon CLient policy they may have right to edit Screen V2.0c

########## 5.3 Access the Rest Hours Dashboard
- This is the default screen that opens up when the Office User logs in (Screen O1.0a)
- Default view is for current month
- Office User can also access this screen by clicking Dash from the left navigation pane
- Office User can apply filters - Period (Duration), Vessels, Fleet and any Additional Group 
- Office User can check or uncheck OPA to include/exclude OPA violations
- The dashboard displays fleet-level metrics for the vessels under the user’s purview, including:
	- Watchlist : Shows Vessel Name, Event/Inspection, Significant NCs (last 60 days and predicted)
	- Performance Metrics - Total Violations, Significant NCs, Predicted NCs
		- Clicking on Total Violations, Significant NCs or Predicted NCs open Screen O1.1a with details on these
	- NUmber of Vessels with Violation
		- Clicking on this opens Screen O1.2a which shows more details on vessel wise violations
	- Number of Vessels with Significant NCs	
		- Clicking on this opens Screen O1.3a which shows more details on vessel wise Significant NCs
	- Number of Vessels with O/Due Office Response (Status Bar in format - 80% (40/50)
		- Clicking on this opens Screen O1.4a which shows more details 
	- Number of Vessels with Rest Hours Violations (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.5a which shows more details
	- Incomplete Data (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.6a which shows more details
	- Periodic Analysis Chart - Timeline Graph (Yearly, Quarterly, Monthly) wit plot of Significant NCs and Violations per vessel	
	- Group and Vessel Analysis
		- Table with Vessels on Y axis and Months on X axis
		- Bubble showing NCs/Violations numbers for each month and vessel in red color
		- Office User can use Toggle button to switch between Violations and NCs
		- Office User can click and select Fleet (Vessel Group), Additional Group and Vessels
	- Vessel View
		- Bar chart showing Vessel wise NCs or violations
		- Office User can use toggle button to switch between NCs and Violations
		- Office User can click and select  select Fleet (Vessel Group), Additional Group, Owners and Vessels
	- Rank & Dept. Analysis  
		- Bar chart Displaying which ranks/departments (e.g., 2/E, 3/O, C/E) have the highest violation or NC count	
	- Task Analysis
		- Bar chart illustrating how many violations or NCs are tied to specific tasks or events

########## 5.4 Review Recording Screens (O2.0a, O2.0b, O2.0c)
- These Screens are similar to Vessel Screens V2.0a, V2.0b and V2.0c except from an office user perspective
- Office User can click on Record on left navigation pane to access Screen O2.0a
	- Office User can see vessel wise details - Month, Total Crew, Recording Status bar, Total Violations/Number of Crew involved, Total NCs/Number of Crew involved, Predicted Violations, Predicted NCs and Office Review (Due/Completed/Overdue)
		- Office User can click on Total Violations/Number of Crew involved which opens Screen O2.1a which shows details
		- Office User can click on Total NCs/Number of Crew involved which opens Screen O2.1a which shows details
		- In addition to information, there are 2 clickable icons - View and Edit. Clicking View and Edit icons opens up Screen V2.0c where user can view or edit the details
	- User can filter details on Screen O2.0a by Period (Duration), Vessel, Fleet (vessel Group) and Additional Group
- Office User can click on Edit icon next to Vessel details to open Screen O2.0b
	- Default view is for current month
	- The Dash also shows following information for All Vessel Users – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
	- In addition, there are 2 clickable icons - View and Edit. Clicking View and Edit icons opens up Screen V2.0c where user can view or edit the details
	- Clicking on Total Violations or Total NCs against a crew member will open up Screen O2.1a which will show details of the NCs or Violations for the respective vessel and crew member
- Filter Options
	- Filter by date range, rank, or crew member 
- Office User can Click on View Icon against each crew member on Screen O2.0b to view Screen O2.0c which shows their Work and rest hours details
	- Office User can filter details using Period (duration), Vessel, Rank
	- Office User can also click 
		- SHow Planning - this opens up Screen O3.0a
		- OPA - checking/unchecking this includes/excludes OPA violations
		- Choose Plan or Record using a toggle button 
			- Choosing Plan allows user to only view the Screen O2.0c
			- Choosing Record will enable the user to make changes in the Recording form Screen O2.0c (only if allowed by company policy and permission provided in SAIL application)
			- Office User can click on Apply to save any changes made
			- Office User can clear to Undo any changes made 
			
########## 5.5 View Planning Screens (Screens O3.0a and O3.0b)			
- Office User can click on Plan in left navigation pane to view Screens O3.0a and O3.0b
- These screens are similar to Vessel side Screens V3.0a and V3.0b
- USer will have only view access for these screens


########## 5.6 Log Out
- End the Session by logging out of the application.
- The user can log back in at any time to view rest hours module again



######## 6. Office Admin

########## 6.1 Log In to the SAIL app 
- Open the SAIL application on the office device
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Access is limited to vessels, vessel groups and data the user is authorized to manage or view

########## 6.2 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Office Admin can access:
	- RH Dashboard - Office (Screen O 1.0a)for the vessel groups and vessels under Office Admin's charge
	- Recording Screens O2.0a, O2.0b and O2.0c
	- Depending upon CLient policy they may have right to edit Screen V2.0c

########## 6.3 Access the Rest Hours Dashboard
- This is the default screen that opens up when the Office Admin logs in (Screen O1.0a)
- Default view is for current month
- Office Admin can also access this screen by clicking Dash from the left navigation pane
- Office Admin can apply filters - Period (Duration), Vessels, Fleet and any Additional Group 
- Office Admin can access details for all the Vessel Groups and Vessels that are under his charge
- Office Admin can click or unclick OPA to include/exclude OPA violations
- The dashboard displays fleet-level metrics for the vessels under the user’s purview, including:
	- Watchlist : Shows Vessel Name, Event/Inspection, Significant NCs (last 60 days and predicted)
	- Performance Metrics - Total Violations, Significant NCs, Predicted NCs
		- Clicking on Total Violations, Significant NCs or Predicted NCs open Screen O1.1a with details on these
	- NUmber of Vessels with Violation
		- Clicking on this opens Screen O1.2a which shows more details on vessel wise violations
	- Number of Vessels with Significant NCs	
		- Clicking on this opens Screen O1.3a which shows more details on vessel wise Significant NCs
	- Number of Vessels with O/Due Office Response (Status Bar in format - 80% (40/50)
		- Clicking on this opens Screen O1.4a which shows more details 
	- Number of Vessels with Rest Hours Violations (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.5a which shows more details
	- Incomplete Data (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.6a which shows more details
	- Periodic Analysis Chart - Timeline Graph (Yearly, Quarterly, Monthly) wit plot of Significant NCs and Violations per vessel	
	- Group and Vessel Analysis
		- Table with Vessels on Y axis and Months on X axis
		- Bubble showing NCs/Violations numbers for each month and vessel in red color
		- Office Admin can use Toggle button to switch between Violations and NCs
		- Office Admin can click and select Fleet (Vessel Group), Additional Group and Vessels
	- Vessel View
		- Bar chart showing Vessel wise NCs or violations
		- Office Admin can use toggle button to switch between NCs and Violations
		- Office Admin can click and select  select Fleet (Vessel Group), Additional Group, Owners and Vessels
	- Rank & Dept. Analysis  
		- Bar chart Displaying which ranks/departments (e.g., 2/E, 3/O, C/E) have the highest violation or NC count	
	- Task Analysis
		- Bar chart illustrating how many violations or NCs are tied to specific tasks or events


########## 6.4 Review Recording Screens (O2.0a, O2.0b, O2.0c)
- These Screens are similar to Vessel Screens V2.0a, V2.0b and V2.0c except from an office user perspective
- Office Admin can click on Record on left navigation pane to access Screen O2.0a
	- User can see vessel wise details - Month, Total Crew, Recording Status bar, Total Violations/Number of Crew involved, Total NCs/Number of Crew involved, Predicted Violations, Predicted NCs and Office Review (Due/Completed/Overdue)
		- Office Admin can click on Total Violations/Number of Crew involved which opens Screen O2.1a which shows details
		- Office Admin can click on Total NCs/Number of Crew involved which opens Screen O2.1a which shows details
	- In addition to information, there are 2 clickable icons - View and Edit. Clicking on any of these will open Screen O2.0b
	- User can filter details on Screen O2.0a by Period (Duration), Vessel, Fleet (Vessel Group) and Additional Group
- User can click on View or Edit icons next to Vessel details to open Screen O2.0b
	- Default view is for current month
	- The Dash also shows following information for All Vessel Users – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
	- Clicking on Total Violations or Total NCs against a crew member will open up Screen O2.1a which will show details of the NCs or Violations for the respective vessel and crew member
	- In addition to information, there are 2 clickable actions - View and Edit
- Filter Options
	- Filter by date range, rank, or crew member 
- Office Admin can Click on View Icon against each crew member on Screen O2.0b to view Screen O2.0c which shows their Work and rest hours details
- Office Admin can Click on Edit Icon against each crew member on Screen O2.0b to view Screen O2.0c which shows their Work and rest hours details. This can be edited if permissiona have been provided in the SAIL application
	- Office Admin can filter details using Period (duration), Vessel, Rank
	- Office Admin can also click 
		- SHow Planning - this opens up Screen O3.0a
		- OPA - checking/unchecking this includes/excludes OPA violations
		- Choose Plan or Record using a toggle button 
			- Choosing Plan allows user to only view the Screen O2.0c
			- Choosing Record will enable the user to make changes in the Recording form Screen O2.0c (only if allowed by company policy and permission provided in SAIL application)
			- Office Admin can click on Apply to save any changes made
			- Office Admin can clear to Undo any changes made 
			
			
########## 6.5 View Planning Screens (Screens O3.0a and O3.0b)			
- User can click on Plan in left navigation pane to view Screens O3.0a and O3.0b
- These screens are similar to Vessel side Screens V3.0a and V3.0b
- USer will have only view access for these screens



########## 6.6 Log Out
- End the Session by logging out of the application.
- The user can log back in at any time to view rest hours module again


######## 7. Office Super Admin

########## 7.1 Log In to the SAIL app 
- Open the SAIL application on the office device
- Enter credentials to login
	- The system authenticates using existing Safe Lanes login functionality.
	- Office Super Admin has full access to complete solution, all screens and data

########## 7.2 Access the Rest Hours Module
- From the main menu, select “Rest Hours”
- Office Super Admin can access:
	- RH Dashboard - Office (Screen O 1.0a)for the all vessel groups and vessels 
	- Recording Screens O2.0a, O2.0b and O2.0c
	- Office Super Admin will have access to all the features of Rest Hours module

########## 6.3 Access the Rest Hours Dashboard
- This is the default screen that opens up when the Office Super Admin logs in (Screen O1.0a)
- Default view is for current month
- Office Super Admin can also access this screen by clicking Dash from the left navigation pane
- Office Super Admin can apply filters - Period (Duration), Vessels, Fleet and any Additional Group 
- Office Super Admin can access details for all the Vessel Groups and Vessels 
- Office Super Admin can check or uncheck OPA to include/exclude OPA violations
- The dashboard displays fleet-level metrics for the vessels under the user’s purview, including:
	- Watchlist : Shows Vessel Name, Event/Inspection, Significant NCs (last 60 days and predicted)
	- Performance Metrics - Total Violations, Significant NCs, Predicted NCs
		- Clicking on Total Violations, Significant NCs or Predicted NCs open Screen O1.1a with details on these
	- NUmber of Vessels with Violation
		- Clicking on this opens Screen O1.2a which shows more details on vessel wise violations
	- Number of Vessels with Significant NCs	
		- Clicking on this opens Screen O1.3a which shows more details on vessel wise Significant NCs
	- Number of Vessels with O/Due Office Response (Status Bar in format - 80% (40/50)
		- Clicking on this opens Screen O1.4a which shows more details 
	- Number of Vessels with Rest Hours Violations (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.5a which shows more details
	- Incomplete Data (Status Bar in format - 80% (40/50)	
		- Clicking on this opens Screen O1.6a which shows more details
	- Periodic Analysis Chart - Timeline Graph (Yearly, Quarterly, Monthly) wit plot of Significant NCs and Violations per vessel	
	- Group and Vessel Analysis
		- Table with Vessels on Y axis and Months on X axis
		- Bubble showing NCs/Violations numbers for each month and vessel in red color
		- User can use Toggle button to switch between Violations and NCs
		- User can click and select Fleet (Vessel Group), Additional Group and Vessels
	- Vessel View
		- Bar chart showing Vessel wise NCs or violations
		- User can use toggle button to switch between NCs and Violations
		- User can click and select  select Fleet (Vessel Group), Additional Group, Owners and Vessels
	- Rank & Dept. Analysis  
		- Bar chart Displaying which ranks/departments (e.g., 2/E, 3/O, C/E) have the highest violation or NC count	
	- Task Analysis
		- Bar chart illustrating how many violations or NCs are tied to specific tasks or events


########## 7.4 Review Recording Screens (O2.0a, O2.0b, O2.0c)
- These Screens are similar to Vessel Screens V2.0a, V2.0b and V2.0c except from an office user perspective
- Office Super Admin can click on Record on left navigation pane to access Screen O2.0a
	- User can see vessel wise details - Month, Total Crew, Recording Status bar, Total Violations/Number of Crew involved, Total NCs/Number of Crew involved, Predicted Violations, Predicted NCs and Office Review (Due/Completed/Overdue)
		- User can click on Total Violations/Number of Crew involved which opens Screen O2.1a which shows details
		- User can click on Total NCs/Number of Crew involved which opens Screen O2.1a which shows details
		- In addition to information, there are 2 clickable icons - View and Edit. Clicking on any of these will open Screen O2.0b
	- User can filter details on Screen O2.0a by Period (Duration), Vessel, Fleet (Vessel Group) and Additional Group
- Office Super Admin can click on View or Edit icon next to Vessel details to open Screen O2.0b
	- Default view is for current month
	- The Dash also shows following information for All Vessel Users – Rank, name, Month, S.On/S.Off with date, Recording status bar, Total Violations, Total NCs, Predicted Violations and Predicted NCs
	- In addition to information, there are 2 clickable icons - View and Edit
	- Clicking on Total Violations or Total NCs against a crew member will open up Screen O2.1a which will show details of the NCs or Violations for the respective vessel and crew member
- Filter Options
	- Filter by date range, rank, or crew member 
- Office Super Admin can Click on View Icon against each crew member on Screen O2.0b to view Screen O2.0c which shows their Work and rest hours details
	- User can filter details using Period (duration), Vessel, Rank
	- User can also click 
		- SHow Planning - this opens up Screen O3.0a
		- OPA - checking/unchecking this includes/excludes OPA violations
		- Choose Plan or Record using a toggle button 
			- Choosing Plan allows user to only view the Screen O2.0c
			- Choosing Record will enable the user to make changes in the Recording form Screen O2.0c (only if allowed by company policy and permission provided in SAIL application)
			- User can click on Apply to save any changes made
			- User can clear to Undo any changes made 


########## 7.5 View Planning Screens (Screens O3.0a and O3.0b)			
- User can click on Plan in left navigation pane to view Screens O3.0a and O3.0b
- These screens are similar to Vessel side Screens V3.0a and V3.0b
- USer will have only view access for these screens

########## 7.6 Manage Access Controls
- Office Super Admin can manage access controls for all Users and Admins in the SAIL application 

########## 7.7 Log Out
- End the Session by logging out of the application.
- The Office Super Admin can log back in at any time to view rest hours module again






	

