# **OpCon Agent for IBM i:** 
## Automating Export/Import Using Self Service

**Table of Contents**

[OpCon IBM i Agent Data Export/Import Overview](#opcon-ibm-i-agent-data-exportimport-overview)

[Process Guide](#_Toc40790155)

[Configuration Requirements](#_Toc40790156)

[IBM i LSAM Configuration](#_Toc40790157)

[Dynamic Variables](#dynamic-variables)

[EXIBATCH -- User Provided Exp/Imp Batch Name](#exibatch-user-provided-expimp-batch-name)

[EXIGROUP -- Returns Exp/Imp Group](#exigroup-returns-expimp-group)

[EXISAVF -- Returns Exp/Imp Save File](#exisavf-returns-expimp-save-file)

[OpCon Enterprise Manager Configuration](#opcon-enterprise-manager-configuration)

[Global Properties](#global-properties)

[IBMEXI_API_PASSWORD](#ibmexi_api_password)

[IBMEXI_API_USER](#ibmexi_api_user)

[IBMEXI_JOBD](#ibmexi_jobd)

[IBMEXI_JOBDLIB](#ibmexi_jobdlib)

[IBMEXI_SAMID](#ibmexi_samid)

[IBMEXI_SMALOG](#ibmexi_smalog)

[IBMEXI_SAVF](#ibmexi_savf)

[Machine Group](#machine-group)

[OpCon IBM LSAM Export From Test to Production Schedules](#opcon-ibm-lsam-export-from-test-to-production-schedules)

[Configure OpCon Jobs](#configure-opcon-jobs)

[Initialization Job](#initialization-job)

[LSA_EXPDTA -- Export LSAM Batch from Test Job](#lsa_expdta-export-lsam-batch-from-test-job)

[Web Services Connector](#web-services-connector)

[CPYTOMSGIN - IBMEXI FETCH SAVF NAME](#cpytomsgin---ibmexi-fetch-savf-name)

[5 Sec Delay for Event Processing](#sec-delay-for-event-processing)

[CRTIMPSAVF - Create save file on Production Machines](#crtimpsavf---create-save-file-on-production-machines)

[FTP Transfer from Test to Production](#ftp-transfer-from-test-to-production)

[LSA_IMPGET - IBM IMP GETSAVF](#lsa_impget---ibm-imp-getsavf)

[Solution Manager Self Service](#solution-manager-self-service)

[Create a Self Service Button to Export IBM i batches from TEST to PROD.](#create-a-self-service-button-to-export-ibm-i-batches-from-test-to-prod.)

[Self Service Job Execution](#self-service-job-execution)

[Appendix](#appendix)

[Workflow Designer Diagram](#workflow-designer-diagram)


OpCon IBM i Agent Data Export/Import Overview
=============================================

SMA offers its clients a PDF document and accompanying data files that
define a tested model of how to automate the IBM i LSAM Data
Export-Import process.

To implement this type of automated solution, read and follow the
instructions in this illustrated documentation. Please understand this
is a guide, and it is possible that some details might not match every
client site.

This documentation explains basic and advanced configuration of Self
Service, OpCon, and the IBM i LSAM, for automating the process of
exporting LSAM automation tool configurations from an IBM i LSAM source
partition (a test or master LSAM), and then distributing and installing
the configuration data into one or many remote IBM i LSAM environments.

Accompanying the illustrated instructions are two data files that can be
used as indicated in the instructions:

-   The .XML file is data that defines the model of an OpCon Schedule of
    jobs, as it was tested by an SMA team.

-   The .JSON file is data that can be imported into the version 20.0.3
    of the Webservices Connector. (This data is believed to be
    incompatible with older versions of the Webservices Connector
    application.)

These resources are being offered as-is and the document must be
understood as a general guidance document. SMA recognizes that each
client may have different network configurations that may not conform
exactly to the environment that SMA used to test the proposed automation
plan.

Please study the document before executing its instructions. Clients may
need to acquire and install additional components of the OpCon suite of
applications to implement the automated export-import process in the way
that is illustrated in the document.

If you need addition help, please contact SMA Technologies Client
Support. SMA supports the software components that it sells according to
the Support agreement in effect at each client site. Additional
assistance with automation analysis, such as working with this example
of automating LSAM data export-import can be request, although this type
of assistance is usually subject to a charge for hourly services, or
previously purchased service hours may be applied to this type of
assistance.

[]{#_Toc40790155 .anchor}

Process Guide
=============

The OpCon IBM i Agent Data Export/Import process allows the client to
Export LSAM master records groups from a test machine that will be
transferred to one or many production machines based on the
configuration of the OpCon Machine Group, and imported on each machine
using the LSAM Import process.

The list of LSAM master record groups supported includes:

-   OPRRPY = Operator Reply scripts, steps and related files

-   TRPMSG = Message Management Parameter records and related files

-   SCANSPLF = Scan Spool File rules and related records

-   CAPJOB = Captured Job definitions and related files

-   TRKJOB = Job Tracking and Queuing definitions and related files

-   RSTMOD = Restricted Mode script records

-   DYNVAR = LSAM Dynamic Variable table records (these will also appear
    as a related file to most of the other Groups)

**Configuration Pre-requisites**

The instruction in this document include a list of prerequisites that
must be completed before automation of the LSAM Export/Import batches
can be performed. The automation tools that must be configured include:

-   IBM i LSAM Dynamic Variables

-   OpCon Global Properties

-   OpCon Machine Groups

-   OpCon Schedule

-   OpCon Jobs

-   Self Service Button

To utilize the OpCon IBM i Agent Data Export-Import automation process,
you are required to build LSAM Export batch control records per the IBM
i LSAM Administration Chapter 17: Reference Information on How to Export
and Import Data. Once these Export batches have been created, the
Solutions Manager Self Service web application button will be used to
process each named batch by clicking the Self Service button, typing the
batch name in the LSAM Batch Name field and click the Submit button. In
the Enterprise Manager or Solutions Manager, you can observe a new
schedule being run with the name of the schedule you create in the OpCon
IBM i LSAM Export From Test to Production Schedules section of this
document.

[]{#_Toc40790156 .anchor}

Configuration Requirements
==========================

1.  IBM i LSAM Dynamic Variables -- dynamic variables are used to return
    the Export/Import Group Name and Save File Name for a LSAM Batch
    Name given in the Solution Manager Self Service web application.

2.  OpCon Global Properties -- global properties are used to configure
    the variables used by multiple jobs.

3.  OpCon Machine Groups -- machine groups are used to control one to
    many production machines as targets for FTP and Import processing.

4.  OpCon Schedule -- the OpCon schedule is a single schedule required
    to contain all 8 jobs for Export, FTP and Import to the production
    machine(s).

5.  OpCon Jobs -- 8 jobs in the schedule are used to initialize
    variables, LSAM Export to a save file, Web Connector to retrieve the
    username and password of the Export machine, retrieve the Save File
    name, delay 5 seconds for event processing to complete, create save
    files on target machine(s) in the machine group, FTP save file to
    the target machine(s) in the machine group and LSAM Import save file
    on each machine(s) in the machine group.

6.  Self Service Button -- the Solution Manager Self Service web
    application is the location to enter the LSAM Batch Name to be
    processed by the schedule. The button will process a \$SCHEDULE:ADD
    to create the new schedule for processing.

Exported Information
====================

**Schedule Extract XML**
SMADDI Schedule Extract XML file is
provided as an extract of the IBM LSAM Export From Test to Production
Schedules and can be imported per the instruction in the Enterprise
Manager User Guide Opening Import Export section. 

File Name: 
> SMADDI_20200508144159_IBM LSAM Export From Test to Production.xml

**WS Connector Template**
Web Services Connector exported
templated is provided to assist in the configuration. 

File Name:
> Web_Services_Connector_20200508144159_IBM LSAM Export From Test to Production.json

[]{#_Toc40790157 .anchor}

IBM i LSAM Configuration
========================

Dynamic Variables
-----------------

From the IBM i LSAM Main Menu, sub-menu 3, option 6, to define these
variables. The LSAM menu system must be used for Dynamic Variables that
use Function Codes depending on the second page of Dynamic Variables
maintenance.

Configure Dynamic Variables with the Agent

1.  In the command line, type SMAGPL/STRSMA to proceed to the LSAM menu
    system.

2.  Type 3 to choose the Event Managements and Utilities menu in the
    LSAM Main Menu.

3.  Type 6 to choose the Maintain dynamic variables option in the Event
    and Utilities Menu.

4.  Press F6 to add a new Dynamic Variable.

5.  Configure EXIBATCH, EXIGROUP and EXISAVF per the screen shots
    provided below.

### EXIBATCH -- User Provided Exp/Imp Batch Name

EXIBATCH Dynamic Variable is populated by the Schedule Instance variable
on the variable tab of job LSA_EXPDTA - Export LSAM Batch from Test
Machine.

![](.//media/EXIBATCH.png)


### EXIGROUP -- Returns Exp/Imp Group

EXIGROUP Dynamic Variable populates and return the Export/Import Group
from the DB2 database table for the EXIBATCH Dynamic Variable entered on
Self Service web application.

![](.//media/EXIGROUP.png)

Press the Enter to proceed to page 2.

![](.//media/EXIGROUP2.png)


### EXISAVF -- Returns Exp/Imp Save File

EXISAVF Dynamic Variable populates and return the Export/Import Save
File from the DB2 database table for the EXIBATCH Dynamic Variable
entered on Self Service web application.

![](.//media/EXISAVF.png)

Press the Enter Key to proceed to page 2.

![](.//media/EXISAVF2.png)


OpCon Enterprise Manager Configuration
======================================

Global Properties
-----------------

**Adding Global Properties**

To add a global property:

1.  Double-click on Global Properties under the Administration topic.
    The Global Properties screen displays.

2.  Click Add on the Global Properties toolbar.

3.  Enter an alphanumeric property name in the Name text box.

4.  (Optional) Enter the documentation in the Documentation text box.

5.  (Optional) Select the Encrypted checkbox.

6.  Enter an alphanumeric property value in the Value text box.

7.  Click Save icon Save on the Global Properties toolbar.

8.  Click Close ☒ (to the right of the Global Properties tab) to close
    the Global Properties screen.

**IBMEXI_API_PASSWORD**

-   Name Text Box: EBMEXI_API_PASSWORD

-   Documentation Text Box: OpCon API user ID password encrypted for access via RESTful connector

-   Encrypted Checkbox: Click to check

-   Value Text Box: Type API Password

**IBMEXI_API_USER**

-   Name Text Box: EBMEXI_API_USER

-   Documentation Text Box: OpCon API User name to obtain RESTful access token

-   Value Text Box: Type IBMIEVENT

**IBMEXI_JOBD**

-   Name Text Box: EBMEXI_API_JOBD

-   Documentation Text Box: IBM Export/Import Job Description

-   Value Text Box: Type IBM i job description

**IBMEXI_JOBDLIB**

-   Name Text Box: EBMEXI_API_JOBDLIB

-   Documentation Text Box: IBM Export/Import Job Description Library

-   Value Text Box: Type IBM i job description library

**IBMEXI_SAMID**

-   Name Text Box: EBMEXI_API_SAMID

-   Documentation Text Box: URL name of the SAM server hosting the OpCon API access.

-   Value Text Box: Type the machine name of the host OpCon server

**IBMEXI_SMALOG**

-   Name Text Box: IBMEXI_SMALOG

-   Documentation Text Box: SMALOG Library

-   Value Text Box: Type the SMALOG library name

**IBMEXI_SAVF**

-   Name Text Box: EBMEXI_SAVF

-   Documentation Text Box: IBM LSAM Export/Import SAVF name (20)

-   Value Text Box: Value will be initialized during the Initialization Null Job

Machine Group
-------------

Create a Machine Group for one to many production machine(s).

To add a machine group:

1.  Double-click on the Machine Groups under the Administration topic.
    The Machine Group screen displays.

2.  Click Add on the Machine Group tool bar.

3.  Type in the Machine Group name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

5.  Click the Machine Type drop-down and select IBM i.

6.  Select a production machine by clicking the machine in the Machine
    Assignment section Unassigned Machines.

7.  Click the green arrow between the Unassigned Machine and the
    Assigned Machines to add to the Machine Group.

8.  Click the Green check in the upper right hand to save the Machine
    Group.

-   Name Text Box: Name the Machine Group

-   Documentation: Type the document Machine Group information

-   Machine Type Drop-down: Click IBM i from the Machine Type drop-down

-   Machine Assignment: Select a production machine by clicking the machine in the Machine Assignment section Unassigned Machines.

OpCon Schedule: IBM LSAM Export From Test to Production
-------------------------------------------------------

Appendix: Attached is the OpCon Workflow Diagram for IBM LSAM Export
From Test to Production Schedules

To add a master schedule:

1.  Double-click on Schedule Master under the Administration topic. The
    Schedule Master screen displays.

2.  Click Add on the Schedule Master tool bar.

3.  Type a schedule name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

\*\* Note in the documentation section that this schedule is designed to
be run from the Solutions Manager Self Service button.

-   Name Text Box: IBM i LSAM Export From Test to Production

-   Documentation Text Box: This a schedule is designed to be run from Self
Service. It requires the LSAM export Batch Name in an OpCon Global
Property.

-   Workdays per Week: click all days

Configure OpCon Jobs
--------------------

To add a job:

1.  Double-click on Job Master under the Administration topic. The Job
    Master screen displays.

2.  Click Add on the Job Master toolbar.

3.  Type a job name in the Name textbox.


### Initialization Job

The Initialization Job initializes all global variables values from the
previous run to INZ.

**Selection**

-   Name Text Box: Initilization Job

-   Job Properties/Job Details Tab

-   Job Type: Null Job

-   Job Properties/Frequency Tab

**Click the Add Frequency button to add a frequency configured like below:**

![](.//media/image2.png)

**Job Properties/Events Tab**

-   Click the Add button

-   Select the Job Status toggle

-   Select Finished OK from the Job Status drop-down selection

-   Select \$PROPERTY:SET,\<property name\>,\<initial value\> from the Event Template drop-down selection

-   Replace the Event Parameters with:

>> IBMEXI_SAVF,INZ


### LSA_EXPDTA -- Export LSAM Batch from Test Job

The LSA_EXPDTA export job can export batches into save files that can be
transported to another LSAM environment. The list below shows the
completed list data that can be exported:

> **OPRRPY** = Operator Reply scripts, steps and related files

> **TRPMSG** = Message Management Parameter records and related files

> **SCANSPLF** = Scan Spool File rules and related records

> **CAPJOB** = Captured Job definitions and related files

> **TRKJOB** = Job Tracking and Queuing definitions and related files

> **RSTMOD** = Restricted Mode script records

> **DYNVAR** = LSAM Dynamic Variable table records (these will also appear
    as a related file to most of the other Groups) Show all = remove
    Group ID filtering of the control records list

**Selection**

-   Name Text Box: LSA_EXPDTA -- Export LSAM Batch from Test Machine

-   Job Properties/Job Details Tab

-   Job Type: IBM i

-   Primary Machine: select your test machine from the drop-down.

**IBM i Definition/Job Information**

-   Job Type: Select Batch Job from the drop-down selection.

-   User ID: Select your configured User ID from the drop-down selection.

-   Job Description Name: [[IBMEXI_JOBD]] Global Property

-   Job Description Library: [[IBMEXI_JOBDLIB]] Global Property

-   Call: LSAEXPDTA GROUP({EXIGROUP}) BATCH([[SI.IBMEXIBATCH]]) REPORT(1)

*Dynamic Variables*

> [EXIGROUP](#x) -- The dynamic variable uses
> the \*DB2 function code to derive its value at run time from the LSAM's
> database tables that defines the returns Export/Import Group.


Schedule Instance Variable

> [IBMEXIBATCH](#x) -- Configuration of the Schedule Instance Input Variable
> happens during the configuration of the Self Service button and is
> passed to the schedule using the predefined External Event command
> \$SCHEDULE:BUILD.

**IBM i Definition/Variables**

-   Variable Name: EXIBATCH

-   Value: \[\[SI.IBMEXIBATCH\]\]

*Click the ADD button*


Schedule Instance Variable

> [IBMEXIBATCH](#x) -- Configuration of the Schedule Instance Input Variable
> happens during the configuration of the Self Service button and is
> passed to the schedile using the predefined External Event command
> \$SCHEDULE:BUILD.


**Job Properties/Frequency Tab**

Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Events Tab**

-   Click the Add button

-   Select the Job Status toggle

-   Select Finished OK from the Job Status drop-down selection

-   Select \$PROPERTY:ADD,\<property name\>,\<initial value\> from the Event Template drop-down selection

-   Replace the Event Paramerters with SI.IBMEXI_SRC_MACHINE,\[\[\$MACHINE NAME\]\]


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select Initialization Job from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


### Web Services Connector

The Web Services Connector job uses the OpCon API web services to fetch
the username and password of the test machine from the OpCon database.

**Selection**

-   Name Text Box: Web Services Connector

-   Job Properties/Job Details Tab

-   Job Type: select Webservices from the drop-down.

-   Primary Machine: select your OpCon machine from the drop-down.

**Webservices Definition**

-   User Id: Use Service Account

-   Connector Location: \[\[WS_PATH\]\]

-   Template ID: OpConAPI-GETIBMEXISRCIP

**Webservices Definition/Variables Tab**

Type Variables in the Variables Text Box and click the Add Button:

-   \$User=\[\[IBMEXI_API_USER\]\]

-   \$Password=\[\[IBMEXI_API_PASSWORD\]\]

-   \$Machine=\[\[SI.IBMEXI_SRC_MACHINE\]\]

Type Properties in the Properties Text Box and click the Add Button:

-   SI.IBMEXISRCIP.\[\[\$SCHEDULE DATE\]\].\[\[\$SCHEDULE NAME\]\]=\$PropertyValue

**Webservices Definition/Steps Tab**

__Step 1 Tab__

POST: Select POST from the drop-down selection

-   Text Box:
> [https://\[\[IBMEXI_SAMID\]\]:9010/api/tokens](https://[[IBMEXI_SAMID]]:9010/api/tokens)

Request Tab

-   Message Body Text Box:
> {\"id\":null,\"user\":{\"id\":-1,\"loginName\":\"\$User\",\"password\":\"\$Password\"},\"tokenType\":{\"id\":null,\"type\":\"User\"}}

Response Tab

-   Type Variables in the Variables Text Box and click the Add Button:
> \$Token=\$.id


__Step 2 Tab__

GET: Select GET from the drop-down selection

-   Text Box:
>  [https://\[\[IBMEXI_SAMID\]\]:9010/api/machines?name=\$Machine&extendedProperties=true](https://[[IBMEXI_SAMID]]:9010/api/machines?name=$Machine&extendedProperties=true)

Webservices Definition/Failure Criteria Tab

-   Compairson Operation: Select Not Equal To from the drop-down selection

-   Value Text Box: Type 200

-   Result: Select Fail from the drop-down selection

Job Properties/Dependencies Tab

-   Click the Add button to add a job dependency

-   Select LSA_EXPDTA - Export LSAM Batch from Test Machine from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


### CPYTOMSGIN - IBMEXI FETCH SAVF NAME

The CPYTOMSGIN - IBMEXI FETCH SAVF NAME job fetches the save file name
from the LSAM Dynamic Variable and sets the global property IBMEXI_SAVF.

**Selection**

-   Name Text Box: CPYTOMSGIN - IBMEXI FETCH SAVF NAME

-   Job Properties/Job Details Tab

-   Job Type: select IBM i from the drop-down.

-   Primary Machine: select your test machine from the drop-down.

**IBM i Definition/Job Information Tab**

-   Job Type: Select Batch Job from the drop-down.

-   User ID: Select your configured User ID from the drop-down selection.

-   Job Description Name: \[\[IBMEXI_JOBD\]\] Global Property

-   Job Description Library: \[\[IBMEXI_JOBDLIB\]\] Global Property

-   Call: CPYTOMSGIN CPYMSGIN(\'\$PROPERTY:SET,IBMEXISAVF,{EXISAVF}\')

*Dynamic Variables*

> [EXISAVF](#x) -- This dynamic variable uses the \*DB2 function code to derive
> its value at run time from the LSAM's database table that define the
> Exort batch, based on the Export Batch Name IBMEXISAVF that was typed
> into the Self Service data entry field.

Schedule Instance Variable

> [IBMEXIBATCH](#x) -- Configuration of the Schedule Instance Input Variable
> happens during the configuration of the Self Service button and is
> passed to the schedile using the predefined External Event command
> \$SCHEDULE:BUILD.

__Configure the Job Properties/Frequency tab__

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select Web Services Connector from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


### 5 Sec Delay for Event Processing

The 5 Sec Delay for event processing job delays for 5 seconds to wait
for the events to be process in the previous job CPYTOMSGIN - IBMEXI
FETCH SAVF NAME.

**Selection**

-   Name Text Box: 5 Sec Delay for Event Processing

-   Job Properties/Job Details Tab

-   Job Type: select Windows from the drop-down.

-   Primary Machine: select your OpCon machine from the drop-down.

**Windows Definition/Job Information Tab**

-   Job Action: select Run Program from the drop-down selection

-   User ID: select User Service Account from the drop-down selection

-   Command Line: Type GENERICP.exe -t5 in the command line text box

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select CPYTOMSGIN - IBMEXI FETCH SAVF NAME from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


### CRTIMPSAVF - Create save file on Production Machines

The CRTIMPSAVF - Create save file on Production Machines job creates an
empty save file on each target machine(s) in the machine group.

**Selection**

-   Name Text Box: CRTIMPSAVF -- Create save file on Production Machines

-   Job Properties/Job Details Tab

-   Job Type: select IBM i from the drop-down.

-   Machine Group: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   Job Type: Select Batch Job from the drop-down.

-   User ID: Select your configured User ID from the drop-down selection.

-   Job Description Name: \[\[IBMEXI_JOBD\]\] Global Property

-   Job Description Library: \[\[IBMEXI_JOBDLIB\]\] Global Property

-   Call: CRTSAVF \[\[IBMEXI_SMALOG\]\]/\[\[IBMEXI_SAVF\]\]

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select 5 Sec Delay for Event Processing from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


### FTP Transfer from Test to Production 

The FTP Transfer from Test to Production job ftp the save file from the
test machine to one to many machine(s) that are configured in the
machine group.

**Selection**

-   Name Text Box: FTP Transfer from Test to Production

-   Job Properties/Job Details Tab

-   Job Type: select IBM i from the drop-down.

-   Machine Group: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   Job Type: Select FTP from the drop-down.

-   User ID: Select your configured User ID from the drop-down selection.

-   Job Description Name: \[\[IBMEXI_JOBD\]\] Global Property

-   Job Description Library: \[\[IBMEXI_JOBDLIB\]\] Global Property

-   Call Information/Transfer Information Tab

**ftp options**

-   Action Type: Select GET from the drop-down selection.

-   Transfer Type: Select BIN from the drop-down selection.

-   User: type the named user that was configured on the IBM i LSAM.

*IBM i LSAM ftp User Configuration*

> LSAM User Management must be used to store the password required by the remote system

>> \*\* From the LSAM Main Menu Navigate to the LSAM sub-menu 4, option
1: User management

>> \*\* F6=Add, Type the user profile in the User Name field, type the
password in the Password field and retype the password in the Password
(to verify) field.

>> \*\* Press the ENTER key to confirm.

**Call Information/Remote Information Tab**

-   Remote System: \[\[SI.IBMEXISRCIP\]\]

-   Remote File System: \*LCLFILNAM

-   Remote Library or Directory: \[\[IBMEXI_SMALOG\]\]

**Call Information/Location Information Tab**

-   Local File Name: \[\[IBMEXI_SAVF\]\]

-   Local Library or Directory: \[\[IBMEXI_SMALOG\]\]

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select CRTIMPSAVF -- Create save file on Production Machines from the Job drop-down selection

-   Click Requires from the Dependency Type toggle

-  Options: Select Finished OK from the options drop-down selection


### LSA_IMPGET - IBM IMP GETSAVF

The LSA_IMPGET job calls the IBM i LSAM program LSAIMPGET to import the
save file into the target machine.

**Selection**

-   Name Text Box: LSA_EXPDTA - Export LSAM Batch from Test Machine

**Job Properties/Job Details Tab**

-   Job Type: select IBM i from the drop-down.

-   Machine Group: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   Job Type: Select Batch Job from the drop-down.

-   User ID: Select your configured User ID from the drop-down selection.

-   Job Description Name: \[\[IBMEXI_JOBD\]\] Global Property

-   Job Description Library: \[\[IBMEXI_JOBDLIB\]\] Global Property

-   Call: LSAIMPGET SAVFIL(\[\[IBMEXI_SAVF\]\]) REPORT(1)

Global Property

> [IBMEXI_SAVF] -- This global property is used to store the Save File name from the CRTIMPSAVF - Create save file on Production Machines job.

**Properties/Frequency Tab**

Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select FTP Transfer from Test to Production from the Job drop-downselection

-   Click Requires from the Dependency Type toggle

-   Options: Select Finished OK from the options drop-down selection


Solution Manager Self Service
=============================

Create a Self Service Button to Export IBM i batches from TEST to PROD.
-----------------------------------------------------------------------

SMA Solution Manager is an application platform designed to host and
give access to User Interface (UI) modules called Solutions. Users will
see and have access only to the solution(s) to which they have
privileges.

Configure Self Service button with the OpCon Solution Manager

1.  Start supported web browser. (Note: Solution Manager User Guide
    provides a list of supported browsers)

2.  Type the URL of the Solution Manager (i.e.
    [https://server:port/\#!home](https://server:port/#!home))

3.  Sign into Solution Manager.

4.  Click the Self Service button.

![](.//media/StartExample.png)
![A screenshot of a cell phone Description automatically
generated](.//media/image3.jpg)
![](.//media/EndExample.png)

5.  Change your mode to Admin Mode by clicking the lock in the upper
    right of the screen.

 
> ![](.//media/image4.png)
 
> will change to 

> ![](.//media/image5.png)



6.  Create a New Category by press
 
    ![](.//media/image6.png)

![](.//media/StartExample.png)
![A screenshot of a cell phone Description automatically
generated](.//media/image7.png)
![](.//media/EndExample.png)



7.  For our purpose \"IBM i\" select a color and press the save.

![](.//media/StartExample.png)
![](.//media/image8.png)
![](.//media/EndExample.png)



8.  The following Category has been created:

![](.//media/StartExample.png)
![](.//media/image9.png)
![](.//media/EndExample.png)



9.  Click on the green "+ Create" Button:

![](.//media/StartExample.png)
![A screenshot of a cell phone Description automatically
generated](.//media/image10.png)
![](.//media/EndExample.png)


The New Service Request configuration will appear:

![](.//media/StartExample.png)
![](.//media/image11.png)
![](.//media/EndExample.png)


Fill in the following info:

-   Service Request Name : \"Export Schedule from TEST to PROD\"

-   Documentation : \"Please Enter the Export Batch Name you are working with\"

-   Pull Down Category "IBM i"

Events: Click The green "+"

![](.//media/StartExample.png)
![A screenshot of a cell phone Description automatically
generated](.//media/image12.png)
![](.//media/EndExample.png)


Event Template (Pull Down):

![](.//media/StartExample.png)
![](.//media/image13.png)
![](.//media/EndExample.png)


Event Template: select \$SCHEDULE:BUILD from the drop-down selection

Type the Schedule date, Schedule Name and Overwrite Flag values:

> Schedule date: CURRENT
>
> Schedule name: IBM LSAM EXPORT FROM TEST TO PROD
>
> Overwrite Flag: Pull down : Yes(Y)

![](.//media/StartExample.png)
![](.//media/image14.png)
![](.//media/EndExample.png)


Schedule instance property definitions:

![](.//media/StartExample.png)
![](.//media/image15.png)
![](.//media/EndExample.png)

Click The green "+"

Type:

> Name: IBMEXIBATCH
>
> Value: \${IBMEXIBATCH}

![](.//media/StartExample.png)
![](.//media/image16.png)
![](.//media/EndExample.png)

Click "OK" continue


Work with New Service Request...

![](.//media/StartExample.png)
![A screenshot of a social media post Description automatically
generated](.//media/image17.png)
![](.//media/EndExample.png)

Update the options in the red box (as above):

-   Track Event Executions: Check

-   Submit Events as OCADM: Check

-   User Inputs: Select "IBMEXIBATCH \[Text\]"

Click the edit icon (blue pencil).


Complete the following User Input Definition:

![](.//media/StartExample.png)
![](.//media/image18.png)
![](.//media/EndExample.png)

Type or select the following options:

-   User Input Caption: LSAM Batch ID

-   Required Variable: Check

-   User Input Type (Pull Down) Select: Text

-   Minimum Characters: 1

-   Maximum Characters: 20

Click "OK".


Define appropriate user authorities:

![](.//media/StartExample.png)
![](.//media/image19.png)
![](.//media/EndExample.png)

Show for Role (for example):

-   Granted permission \"Role_ocadm\"

Click "Save" Button.


Self Service Job Execution
==========================

Click the Self Service button from the Home screen.

![](.//media/StartExample.png)
![](.//media/image20.png)
![](.//media/EndExample.png)


Click the "Export Schedule from TEST to PROD" the following screen
display:

![](.//media/StartExample.png)
![](.//media/image21.png)
![](.//media/EndExample.png)


Enter the LSAM Batch ID number you are exporting: e.g., EXI0000123

(The Batch ID is often meaningful text, not to be confused with the
export file name.)

![](.//media/StartExample.png)
![](.//media/image22.png)
![](.//media/EndExample.png)


Click the Submit Button

![](.//media/StartExample.png)
![](.//media/image23.png)
![](.//media/EndExample.png)


Click the "OK" button.

![](.//media/StartExample.png)
![](.//media/image24.png)
![](.//media/EndExample.png)


Open Enterprise Manager

Open List View

Browse to today's date and you will see:

> The schedule \"IBM LSAM EXPORT FROM TEST TO PROD\" is \[RUNNING\]


Appendix
========

Workflow Designer Diagram
-------------------------


![](.//media/image25.png)

