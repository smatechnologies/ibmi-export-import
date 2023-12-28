# **OpCon Agent for IBM i:** 
## Automating Export/Import Using Self Service

**Document Version**

This version from December, 2023, uses the Enterprise Manager (EM) user interface as the basis for describing most of the model automation, except for the Self Service app.  

SMA may update this document to use the Solution Manager (SM) user interface examples as the SM full-featured version is released (estimated release data of EOY 2023 or early 2024).

**Table of Contents**

[OpCon IBM i Agent Data Export/Import Overview](#opcon-ibm-i-agent-data-exportimport-overview)

[Process Guide](#process-guide)

[Configuration Requirements](#configuration-requirements)

### [IBM i LSAM Configuration](#ibm-i-lsam-configuration-1)

#### [Dynamic Variables](#dynamic-variables-1)

- EXIBATCH: [User Provided Exp/Imp Batch Name](#exibatch----user-provided-expimp-batch-name)

- EXIGROUP: [Returns Exp/Imp Group](#exigroup----returns-expimp-group)

- EXISAVF: [Returns Exp/Imp Save File](#exisavf----returns-expimp-save-file)

### [OpCon Enterprise Manager Configuration](#opcon-enterprise-manager-configuration-1)

[OpCon Properties](#opcon-properties)

- IBMEXI_SMALOG: [SMALOG export library name](#ibmexi_smalog)

- IBMIMP_SMALOG: [SMALOG import library name](#ibmimp_smalog)

- IBMEXI_SAVF: [IBM Export/Import save file name](#ibmexi_savf)

- IBMEXI_SRCMACH: [IBM LSAM Export/Import FTP Source Machine](#ibmexi_srcmach)

[Machine Group](#machine-group)

[OpCon IBM LSAM Export from Test to Production Schedules](#opcon-schedule-ibm-lsam-export-from-test-to-production)

[Configure OpCon Jobs](#configure-opcon-jobs)

- [Initialization Job](#initialization-job)

- [LSA_EXPDTA -- Export LSAM Batch from Test Job](#lsa_expdta-export-lsam-batch-from-test-job)

- [CPYTOMSGIN - IBMEXI FETCH SAVF NAME](#cpytomsgin---ibmexi-fetch-savf-name)

- [5 Sec Delay for Event Processing](#sec-delay-for-event-processing)

- [CRTIMPSAVF - Create save file on Production Machines](#crtimpsavf---create-save-file-on-production-machines)

- [FTP Transfer from Test to Production](#ftp-transfer-from-test-to-production)

- [LSA_IMPGET - IBM IMP GETSAVF](#lsa_impget---ibm-imp-getsavf)

### [Solution Manager Self Service](#solution-manager-self-service-1)

[Create a Self Service Button to Export IBM i batches from TEST to PROD](#create-a-self-service-button-to-export-ibm-i-batches-from-test-to-prod)

[Self Service Job Execution](#self-service-job-execution)

### [Appendix](#appendix-1)

[Workflow Designer Diagram](#workflow-designer-diagram)


OpCon IBM i Agent Data Export/Import Overview
=============================================

SMA offers its clients a PDF document and accompanying data files that define a tested model of how to automate the IBM i LSAM Data Export-Import process.

To implement this type of automated solution, read and follow the instructions in this illustrated documentation. Please understand this is a guide, and it is possible that some details might not match every
client site.

**Process Overview**

This document guides the reader through two phases of operations.

(1) The first goal of this document is to walk the OpCon administrator through a one-time process required to pre-configure the various elements of the automation solution.

(2) Once the configuration work is done, the whole actual Export, Transfer and Import process can be repeated many times with two simple manual steps:  

- The first step is to use an option 8=Export to mark an LSAM master record (and all its associated artifacts) for export.
    - The IBM i Agent User Help identifies the step-by-step data entry panels used to label an Export Batch, and then to stage the batch for export.
- An OpCon Self Service panel is launched by a click on a web browswer bookmark.
    - The pop-up data entry panel that appears requires only that the Export Batch name be typed or pasted into a data entry field.
    - Then a simple mouse click launches the whole process of:
        > Building and configuring an instance of the OpCon Schedule for transfering the named Batch.

        > Export from the Test LSAM

        > File Transfer from Test to one IBM i LSAM or multiple LSAMs in a Machine Group.

        > Import to each designated Production LSAM.
        
**How to Use This Document**

This documentation explains basic and advanced configuration of Self Service, OpCon, and the IBM i LSAM, for automating the process of exporting LSAM automation tool configurations from an IBM i LSAM source partition (a Test or master LSAM), and then distributing and installing the configuration data into one or many remote IBM i LSAM environments (referred to herein as Production LSAMs).

Accompanying the illustrated instructions is a data file that can be used as indicated in the instructions:

- An .XML file is data that defines the model of an OpCon Schedule of jobs, as it was tested by an SMA team.

These resources are being offered as-is and the document must be understood as a general guidance document. SMA recognizes that each client may have different network configurations that may not conform
exactly to the environment that SMA used to test the proposed automation plan.

Please study the document before executing its instructions. Clients may need to acquire and install additional components of the OpCon suite of
applications to implement the automated export/import process in the way that is illustrated in the document.

If you need additional help, please contact SMA Technologies Client Support. SMA supports the software components that it sells according to
the Support agreement in effect at each client site. Additional assistance with automation analysis, such as working with this example of automating LSAM data export-import, can be requested, although this type
of assistance is usually subject to a charge for hourly services, or previously purchased service hours may be applied to this type of assistance.

Process Guide
=============

The OpCon IBM i Agent Data Export/Import process allows the client to Export LSAM master records groups from a test machine that will be transferred to one or many production machines based on the
configuration of the OpCon Machine Group, and imported on each machine using the LSAM Import process.

The term "master record" applies to the highest level control record for each of the LSAM automation tools.  When a master record is selected, all of the associated data from other LSAM tables is included in the Export Batch.  For example, an Operator Replay Script master record will include the following required support data:

- Script master record (as selected)
- Script Step records
- Screen data Capture Application control record
- Screen data Capture Rules records
- Capture Data Response Records
- Dynamic Variable records (for all tokens in all of the tables in this list)

The list of LSAM master record groups supported includes:

-   OPRRPY = Operator Reply scripts, steps and related files

-   OPRRPYCAPT = Operator Reply screen data capture application, which can be re-used by multiple Operator Replay Scripts.  This export type distributes updates to just the shared Screen Capture Rules and any associated Response Rules and Dynamic Variables, without needing to also export any Operator Replay Scripts and Step master records.

-   TRPMSG = Message Management Parameter records and related files

-   TRPMSGCAPT = Message Management message data capture application, which can be re-used by multiple Message Management Parameter records.  This export type distributes updates to just the shared message data Capture Rules and any associated Response Rules and Dynamic Variables, without needing to also export any of the Message Management Parameters that could be linked to them.

-   SCANSPLF = Scan Spool File rules and related records

-   CAPJOB = Captured Job definitions and related files

-   TRKJOB = Job Tracking and Queuing definitions and related files

-   RSTMOD = Restricted Mode script records

-   DYNVAR = LSAM Dynamic Variable table records (these will also appear as a related file to most of the other Groups)

-   MLTJOB = Multi-Step Job Scripts and related records

**Configuration Pre-requisites**

The instructions in this document include a list of prerequisites that must be completed before automation of the LSAM Export/Import batches
can be performed. The automation tools that must be configured include:

-   IBM i LSAM Dynamic Variables

-   OpCon Properties

-   OpCon Machine Groups

-   OpCon Schedule

-   OpCon Jobs

-   Self Service Button

To utilize the OpCon IBM i Agent Data Export-Import automation process, you are required to build LSAM Export batch control records per the IBM i LSAM Administration (former Chapter 17) - Reference Information on How to Export and Import Data. Once these Export batches have been created, the
Solutions Manager Self-Service web application button will be used to process each named batch by clicking the Self-Service button, typing the batch name in the LSAM Batch Name field and click the Submit button. In
the Enterprise Manager or Solutions Manager, you can observe a new schedule being run with the name of the schedule you create in the OpCon IBM LSAM Export from Test to Production Schedules section of this
document.

Configuration Requirements
==========================

1.  IBM i LSAM Dynamic Variables -- Dynamic Variables are used to return the Export/Import Group Name and Save File Name for a LSAM Batch Name supplied by the user via the Solution Manager Self-Service web application.

2.  OpCon Global Properties -- global properties are used to configure the variables used by multiple jobs. Most of these will be constrained as Schedule Instance (SI.) properties at run time.

3.  OpCon Machine Groups -- machine groups are used to control one to many production machines as targets for FTP and Import processing.

4.  OpCon Schedule -- the OpCon schedule is a single schedule required to contain all 7 jobs for Export, FTP and Import to the production machine(s).

5.  OpCon Jobs -- 7 jobs in the schedule are used to:   
    - Initialize variables
    - LSAM Export to a save file
    - Retrieve the Save File name
    - Delay 5 seconds for event processing to complete 
    - Create save files on target machine(s) in the machine group
    - FTP save file to the target machine(s) in the machine group and LSAM Import save file on each machine(s) in the machine group.
    - Import the batch to each designated LSAM in the Machine Group.

6.  Self Service Button -- the Solution Manager Self Service web application is the location to enter the LSAM Batch Name to be processed by the schedule. The button will submit to the OpCon server a \$SCHEDULE:ADD external event command to create the new schedule for processing.

Exported Information
====================

**Schedule Extract XML**

An SMADDI Schedule Extract XML file is provided as an extract of the *IBM LSAM Export from Test to Production* Schedule and it can be imported per the instruction in the Enterprise Manager User Guide Opening Import Export section.

File Name: 
> OpConSchedule-22v10-SMADDI.xml

IBM i LSAM Configuration
========================

Dynamic Variables
-----------------

From the IBM i LSAM Main Menu, navigate to sub-menu 3, option 6, to define these variables. The LSAM menu system must be used for Dynamic Variables that use Function Codes, some of which depend on the second page of Dynamic Variables maintenance.

Configure Dynamic Variables with the Agent

1.  In the command line, type SMAGPL/STRSMA to proceed to the LSAM menu system.

2.  Type 3 to choose the Event Managements and Utilities menu in the LSAM Main Menu.

3.  Type 6 to choose the Maintain dynamic variables option in the Event and Utilities Menu.

4.  Press F6 to add a new Dynamic Variable.

5.  Configure EXIBATCH, EXIGROUP and EXISAVF according to the screen shots provided below.

### EXIBATCH -- User Provided Exp/Imp Batch Name

EXIBATCH Dynamic Variable is populated by the Schedule Instance Batch Code OpCon property registered in the variable tab of job LSA_EXPDTA - Export LSAM Batch from Test Machine.

![](.//media/EXIBATCH.png)


### EXIGROUP -- Returns Exp/Imp Group

EXIGROUP Dynamic Variable fetches and returns the Export/Import Group ID from the DB2 database table that is associated with the BATCH code entered in the Self Service web application.

![](.//media/EXIGROUP.png)

Press the Enter key to proceed to page 2.

![](.//media/EXIGROUP2.png)


### EXISAVF -- Returns Exp/Imp Save File

EXISAVF Dynamic Variable fetches and returns the Export/Import Save File from the DB2 database table for the BATCH code entered in the Self Service web application.

![](.//media/EXISAVF.png)

Press the Enter Key to proceed to page 2.

![](.//media/EXISAVF2.png)


OpCon Enterprise Manager Configuration
======================================

OpCon Properties
-----------------

**Adding OpCon Properties**

Most of the OpCon properties required by this procedure must be registered in advance as **Global Properties**.  However, within the job master records of the OpCon Schedule these properties will be instantiated as schedule instance (SI.) properties in order to enable the possibility of parallel processing of export/import batch distributions in large data centers.

To add a global property:

1.  Double-click on Global Properties under the Administration topic. The Global Properties screen displays.

2.  Click Add on the Global Properties toolbar.

3.  Enter an alphanumeric property name in the Name text box.

4.  (Optional) Enter the documentation in the Documentation text box.

5.  (Optional) Select the Encrypted checkbox.

6.  Enter an alphanumeric property value in the Value text box.

7.  Click Save icon Save on the Global Properties toolbar.

8.  Click Close ☒ (to the right of the Global Properties tab) to close the Global Properties screen.

#### IBMEXI_SMALOG

-   Name Text Box: IBMEXI_SMALOG

-   Documentation Text Box: Export SMALOG Library

-   Value Text Box: Type the Export SMALOG library name

-   NOTE:  Be sure to update the "Initialization Job" Events tab with the correct value for this variable.  The correct name for the symbolic (and default) "SMALOG" library name is the name that is registered in the IBM i LSAM Utilities menu 3, sub-menu 10, option 7: Export/Import options configuration.  Use the name registered for the Export save file library, near the top of the parameter fields.

#### IBMIMP_SMALOG

-   Name Text Box: IBMIMP_SMALOG

-   Documentation Text Box: Import SMALOG Library

-   Value Text Box: Type the Import SMALOG library name

-   NOTE:  Be sure to update the "Initialization Job" Events tab with the correct value for this variable.  The correct name for the symbolic (and default) "SMALOG" library name is the name that is registered in the IBM i LSAM Utilities menu 3, sub-menu 10, option 7: Export/Import options configuration.  Use the name registered for the Import save file library, in the bottom half of the parameter fields.

#### IBMEXI_SAVF

-   Name Text Box: IBMEXI_SAVF

-   Documentation Text Box: IBM LSAM Export/Import SAVF name (20)

-   Value Text Box: Value will be initialized during the Initialization Null Job

#### IBMEXI_SRCMACH

-   Name Text Box: IBMEXI_SRCMACH

-   Documentation Text Box: IBM LSAM Export/Import FTP Source Machine

-   Value Text Box: It is acceptable to type the Agent (Machine) name of the Test partition which is the source of the Exported data, as part of the property definition.  However, the Schedule Instance (SI.) of this property will be updated by the job LSA_EXPDTA which is running in the Export source machine.  Relying on this job to set the source machine value makes the overall Schedule easier to adapt to any different Source Machine, if that might be desired.


Machine Group
-------------

Create a Machine Group for one to many production machine(s).

Multiple Machine Groups might be useful when a large data center might have a variety of different profiles of the application database that is being automated by OpCon.  Each Machine Group would represent a collection of application database instances that have matching automation requirements.  In this case, the model of Self Service "button" demonstrated below would be created multiple times, one for each separate OpCon Schedule that corresponds to a unique Machine Group.

Sites that have only one IBM i Production partition can remove the Machine Group  and enter a single Machine Name in the appropriate data entry field of the jobs that were modeled with a Machine Group assigned.  However, it is allowed to have only one Machine assigned to the Machine Group, and that would reduce the number of changes to the provided model of OpCon Schedule Jobs.

To add a machine group:

1.  Double-click on the Machine Groups under the Administration topic. The Machine Group screen displays.

2.  Click Add on the Machine Group tool bar.

3.  Type in the Machine Group name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

5.  Click the Machine Type drop-down and select IBM i.

6.  Select a production machine by clicking the machine in the Machine Assignment section Unassigned Machines.

7.  Click the green arrow between the Unassigned Machine and the Assigned Machines to add to the Machine Group.

8.  Click the Green check in the upper right hand to save the Machine Group.

#### Field Values:

-   **Name Text Box**: Name the Machine Group

-   **Documentation**: Type the document Machine Group information

-   **Machine Type Drop-down**: Click IBM i from the Machine Type drop-down

-   **Machine Assignment**: Select a production machine by clicking the machine in the Machine Assignment section Unassigned Machines.

OpCon Schedule: IBM LSAM Export from Test to Production
-------------------------------------------------------

The [Appendix](#appendix-1) shows an image of the OpCon Workflow Diagram for the *IBM LSAM Export from Test to Production* Schedule

To add a master schedule:

1.  Double-click on Schedule Master under the Administration topic. The Schedule Master screen displays.

2.  Click Add on the Schedule Master tool bar.

3.  Type a schedule name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

\*\* Include in the documentation text box a note that this schedule is designed to be run from the Solutions Manager Self Service button.

#### Field Values:
-   **Name Text Box**: IBM LSAM Export from Test to Production

-   **Documentation Text Box**: This a schedule is designed to be run from Self Service. It requires the LSAM export Batch Name in an OpCon Global Property.

-   **Workdays per Week**: click all days

Configure OpCon Jobs
--------------------

To add a job:

1.  Double-click on Job Master under the Administration topic. The Job
    Master screen displays.

2.  Click Add on the Job Master toolbar.

3.  Type a job name in the Name textbox.


### Initialization Job

The Initialization Job initializes any global property values from the previous run to 'INZ', and it is used to set one or more SI. instances of OpCon properties.

**Selection**

-   **Name Text Box**: Initilization Job

****Job Properties/Job Details Tab**

-   **Job Type**: IBM i
    - An IBM i job is used instead of a Null job so that the Variables tab can cause the user-specified Batch ID to be stored in the LSAM database for later reference, as this schedule needs to fetch the Group ID and Save File name used by the export batch.

**Job Properties/Frequency Tab**

*Click the Add Frequency button to add a frequency configured like below:*

![](.//media/image2.png)

-   **Primary Machine**: select your test machine from the drop-down.

**IBM i Definition/Job Information**

-   **Job Type**: Select **Batch Job** from the drop-down selection.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.

-   **Call**: SNDMSG MSG('SMA OpCon Agent Export-Import reports start of automated schedule: "[[$SCHEDULE NAME]]"  for schedule date: [[$SCHEDULE DATE ISO]]') TOUSR(*SYSOPR)     
    - Any command or program call can be used here.  This suggested command creates entries in the QSYSOPR message queue that could be monitored by the LSAM.  Another message queue could be used to collect just these notices of the Schedule execution.

**IBM i Definition/Variables tab**

*Type these values then click on the **Add** button:*

-   **Variable Name**: SI.EXIBATCH
-   **Value**: [[SI.IBMEXIBATCH]]


**Job Properties/Events Tab**

-   Click the Add button

-   Select the Job Status toggle

-   Select *Finished OK* from the Job Status drop-down selection

-   Select \$PROPERTY:ADD,\<property name\>,\<initial value\> from the Event Template drop-down selection

-   Five ADD commands should be added, replace the Event Parameters in each of the commands with one of these five value pairs:

    > SI.IBMIMP_SMALOG,SMALOG

    > SI.IBMEXI_SAVF,INZ

    > SI.IBMEXI_SMALOG,SMALOG

    ```
    NOTES: 
    
    The value ",SMALOG" is the default value for the library where LSAM save files are normally stored during daily operations such as log file backups before expired records are purged.  The LSAM allows users to specify a different library for Export and/or Import save files, using the LSAM Utilities Menu (# 3), sub-menu # 10, option 7.  Be sure to replace the "EXI" and the "IMP" _SMALOG,xxxxxx parameter values with the actual values registered in the LSAM, if they have been changed from the default of "SMALOG."  For example:  SI.IBMIMP_SMALOG,MYLIB
    ```

### LSA_EXPDTA - Export LSAM Batch from Test Job

The LSA_EXPDTA export job can export batches into save files that can be transported to another LSAM environment. The list below shows the completed list data that can be exported:

> **OPRRPY** = Operator Reply scripts, steps and related files

> **TRPMSG** = Message Management Parameter records and related files

> **SCANSPLF** = Scan Spool File rules and related records

> **CAPJOB** = Captured Job definitions and related files

> **TRKJOB** = Job Tracking and Queuing definitions and related files

> **RSTMOD** = Restricted Mode script records

> **DYNVAR** = LSAM Dynamic Variable table records (these will also appear as a related file to most of the other Groups)

**Selection**

-   **Name Text Box**: LSA_EXPDTA - Export LSAM Batch from Test Machine

**Job Properties/Job Details Tab**

-   **Job Type**: IBM i

-   **Primary Machine**: select your test machine from the drop-down.

**IBM i Definition/Job Information**

-   **Job Type**: Select **Batch Job** from the drop-down selection.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.

-   **Call**: LSAEXPDTA GROUP({EXIGROUP}) BATCH([[SI.IBMEXIBATCH]]) REPORT(1)

*CALL command line Variables*

Batch Group Code:

> [EXIGROUP](#x) -- This dynamic variable appears as a token in the CALL field above.  The LSAM dynamic variable configuration uses the \*DB2 function code to derive its value at run time from the LSAM's database tables.  The {EXIGROUP} token will be replaced with its value by the LSAM Job Scheduler before the LSAEXPDTA command is executed.

Batch ID Schedule Instance Property:

> [IBMEXIBATCH](#x) -- Configuration of the Schedule Instance of this Property happens during the execution of the Self Service button and is passed to the schedule using the predefined External Event command \$SCHEDULE:BUILD.  The Self-Service function that interfaces with the OpCon Job Scheduler service manages the OpCon Property IBMIEXIBATCH and reformats it with the SI. prefix as it prepares a new instance of the LSAM Export/Import automation schedule.

**Job Properties/IBM i Definition/Variables Tab**

Use the data entry boxes to type this one pair of parameters that define how an LSAM Dynamic Variable will have its value set to the value previously stored in the OpCon Schedule Instance Property.

-   **Variable Name**: EXIBATCH

-   **Value**: \[\[SI.IBMEXIBATCH\]\]

    *Click the ADD button*

**Job Properties/Frequency Tab**

Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Events Tab**

-   Click the Add button

-   Select the Job Status toggle

-   Select *Finished OK* from the Job Status drop-down selection

-   Select \$PROPERTY:ADD,\<property name\>,\<initial value\> from the Event Template drop-down selection

-   Replace the Event Paramerters with SI.IBMEXI_SRC_MACHINE,\[\[\$MACHINE NAME\]\]


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select Initialization Job from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-   Options: Select *Finished OK* from the options drop-down selection

### CPYTOMSGIN - IBMEXI FETCH SAVF NAME

The CPYTOMSGIN - IBMEXI FETCH SAVF NAME job fetches the save file name from the LSAM database using the \*DB2 function code of a Dynamic Variable and sets the OpCon Schedule Instance of the Property SI.IBMEXI_SAVF.

**Selection**

-   **Name Text Box**: CPYTOMSGIN - IBMEXI FETCH SAVF NAME

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Primary Machine**: select the Test machine from the drop-down.

**IBM i Definition/Job Information Tab**

-   **Job Type**: Select Batch Job from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.

-   **Call**: CPYTOMSGIN CPYMSGIN(\'\$PROPERTY:SET,SI.IBMEXI_SAVF,{EXISAVF}\')

Dynamic Variable:

> [EXISAVF](#x) -- This dynamic variable uses the \*DB2 function code to derive its value at run time from the LSAM's database table that defines the Export batch, based on the Export Batch Name SI.IBMEXIBATCH that was typed into the Self Service data entry field.

Schedule Instance Property:

> [SI.IBMEX_SAVF](#x) -- This Schedule Instance Property gets loaded with the batch save file name that the LSAM fetches from its database as the CALL command text has the Dynamic Variable {TOKEN} replaced just before the CPYTOMSGIN command is submitted to run.

**Job Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *LSA_EXPDTA - Export LSAM Batch from Test Machine* from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-   **Options**: Select *Finished OK* from the options drop-down selection


### 5 Sec Delay for Event Processing

The 5 Sec Delay for event processing job delays for 5 seconds to wait for the events to be process in the previous job CPYTOMSGIN - IBMEXI FETCH SAVF NAME.

**Selection**

-   **Name Text Box**: 5 Sec Delay for Event Processing

**Job Properties/Job Details Tab**

-   **Job Type**: select Windows from the drop-down.

-   **Primary Machine**: select your OpCon machine from the drop-down.

**Windows Definition/Job Information Tab**

-   **Job Action**: select Run Program from the drop-down selection

-   **User ID**: select User Service Account from the drop-down selection

-   **Command Line**: Type the following into the command line text box..
    > GENERICP.exe -t5

**Job Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)


**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *CPYTOMSGIN - IBMEXI FETCH SAVF NAME* from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-   **Options**: Select *Finished OK* from the options drop-down selection


### CRTIMPSAVF - Create save file on Production Machines

The CRTIMPSAVF - Create save file on Production Machines job creates an empty save file on each target machine(s) in the machine group.

**Selection**

-   **Name Text Box**: CRTIMPSAVF - Create save file on Production Machines

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Machine Group**: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   **Job Type**: Select Batch Job from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.

-   **Call**: CRTSAVF \[\[SI.IBMIMP_SMALOG\]\]/\[\[SI.IBMEXI_SAVF\]\]

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *5 Sec Delay for Event Processing* from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-   **Options**: Select *Finished OK* from the options drop-down selection

### FTP Transfer from Test to Production 

The FTP Transfer from Test to Production job transfers the save file from the Test machine to one or many machine(s) that are configured in the machine group.

**Selection**

-   **Name Text Box**: FTP Transfer from Test to Production

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Machine Group**: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   **Job Type**: Select FTP from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: 
    - \*
        - The asterisk is suggested.  It means "LSAM default" which refers to the Job Description named in the LSAM Parameters (accessed from the LSAM main menu, option 7, in the bottom portion of the first display page).

            **NOTE:** When using the asterisk, the LSAM Parameters must name either the job description SMALSAJ00 or another job description that must include the four LSAM libraries in its Initial Library List.
        
    - SMALSAJ00
        - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: 
        - \*
        - The asterisk is suggested.  It means "LSAM default" which refers to the Job Description Library named in the LSAM Parameters (accessed from the LSAM main menu, option 7, in the bottom portion of the first display page).

            **NOTE:** When using the asterisk, the LSAM Parameters must name either library SMADTA or the library where the alternate job description is located.
    - SMADTA
        - An alternative library can be named, depending on where the Job Description object is stored.

**Call Information/Transfer Information Tab/ftp options**

-   **Action Type**: Select GET from the drop-down selection.

-   **Transfer Type**: Select BIN from the drop-down selection.

-   **User**: type the named user that was configured in the IBM i LSAM.  *(See following instructions)*

**IBM i LSAM ftp User Configuration**

    LSAM User Management must be used to store the password required by the remote system.

-  From the LSAM Main Menu Navigate to the LSAM sub-menu 4, option 1: User management

-  **F6=Add**: Type the user profile in the User Name field, type the password in the Password field and retype the password in the Password (to verify) field.

    *Press the ENTER key to confirm.*

**Call Information/Remote Information Tab**

-   Remote System: \[\[SI.IBMEXI_SRCMACH\]\]

-   Remote File System: \*LCLFILNAM

-   Remote Library or Directory: \[\[SI.IBMEXI_SMALOG\]\]

**Call Information/Location Information Tab**

-   Local File Name: \[\[SI.IBMEXI_SAVF\]\]

-   Local Library or Directory: \[\[SI.IBMIMP_SMALOG\]\]

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *CRTIMPSAVF - Create save file on Production Machines* from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-  **Options**: Select *Finished OK* from the options drop-down selection


### LSA_IMPGET - IBM IMP GETSAVF

The LSA_IMPGET job executes the IBM i LSAM command LSAIMPGET to import the save file into the target machine.

**Selection**

-   **Name Text Box**: LSA_EXPDTA - Export LSAM Batch from Test Machine

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Machine Group**: select your Machine Group from the drop-down.

**IBM i Definition/Job Information Tab**

-   **Job Type*8: Select Batch Job from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.

-   **Job Description Name**: 
    - \*
        - The asterisk is suggested.  It means "LSAM default" which refers to the Job Description named in the LSAM Parameters (accessed from the LSAM main menu, option 7, in the bottom portion of the first display page).

            **NOTE:** When using the asterisk, the LSAM Parameters must name either the job description SMALSAJ00 or another job description that must include the four LSAM libraries in its Initial Library List.
        
    - SMALSAJ00
        - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.

-   **Job Description Library**: 
    - \*
        - The asterisk is suggested.  It means "LSAM default" which refers to the Job Description Library named in the LSAM Parameters (accessed from the LSAM main menu, option 7, in the bottom portion of the first display page).

            **NOTE:** When using the asterisk, the LSAM Parameters must name either library SMADTA or the library where the alternate job description is located.
    - SMADTA
        - An alternative library can be named, depending on where the Job Description object is stored.

-   **Call**: LSAIMPGET SAVFIL(\[\[SI.IBMEXI_SAVF\]\]) REPORT(1)

OpCon Property

> [SI.IBMEXI_SAVF] -- This Schedule Instance of the global property is used to store the Save File name from the *CRTIMPSAVF - Create save file on Production Machines* job.

**Properties/Frequency Tab**

Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *FTP Transfer from Test to Production* from the Job drop-downselection

-   Click *Requires* from the Dependency Type toggle

-   **Options**: Select *Finished OK* from the options drop-down selection


Solution Manager Self Service
=============================

Create a Self Service Button to Export IBM i batches from TEST to PROD
-----------------------------------

SMA Solution Manager is an application platform designed to host and give access to User Interface (UI) modules called Solutions. Users will see and have access only to the solution(s) to which they have
privileges.

**Configure a Self Service button with the OpCon Solution Manager**

1.  Start supported web browser. (Note: Solution Manager User Guide provides a list of supported browsers)

2.  Type the URL of the Solution Manager (i.e.
    [https://server:port/\#!home](https://server:port/#!home))

3.  Sign into Solution Manager.

4.  Click the Self Service button.

![](.//media/StartExample.png)
![](.//media/image3.jpg)
![](.//media/EndExample.png)

5.  Change your mode to Admin Mode by clicking the lock in the upper right of the screen.

 
> ![](.//media/image4.png)
 
> will change to 

> ![](.//media/image5.png)


6.  Create a New Category by press
 
    ![](.//media/image6.png)

![](.//media/StartExample.png)
![A screenshot of a cell phone Description automatically
generated](.//media/image7.png)
![](.//media/EndExample.png)


7.  For our purpose type \"IBM i Export-Import\" (or a value of your choice), select a color and press the save button.

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

10. The New Service Request configuration will appear (shown with values already updated):

![](.//media/StartExample.png)
![](.//media/image11.png)
![](.//media/EndExample.png)

    Fill in the following info:

-   **Service Request Name** : \"IBM LSAM Export from Test to Production\"

-   **Documentation** : \"Distribute an IBM i LSAM automation rule batch from the Test LPAR to a Production Machine Group\"

-   **Pull Down Category**: "IBM i Export-Import"

-   **Events**
: Click The green "+"

![](.//media/StartExample.png)
![](.//media/image12.png)
![](.//media/EndExample.png)


Event Template (Pull Down):

![](.//media/StartExample.png)
![](.//media/image13.png)
![](.//media/EndExample.png)


**Event Template**: select \$SCHEDULE:BUILD from the drop-down selection

Type the Schedule date, Schedule Name and Overwrite Flag values:

> **Schedule date**: CURRENT
>
> **Schedule name**: \"IBM LSAM Export from Test to Production\"
>
> **Overwrite Flag**: *Pull down* : Yes(Y)

![](.//media/StartExample.png)
![](.//media/image14.png)
![](.//media/EndExample.png)

Click The green "+" to add the Schedule Instance Property definition:

>**Name**: IBMEXIBATCH
>
> **Value**: \${IBMEXIBATCH}

![](.//media/StartExample.png)
![](.//media/image16.png)
![](.//media/EndExample.png)

*Click "OK" twice to continue from New Service Request home page.*


Work with New Service Request...

![](.//media/StartExample.png)
![](.//media/image17.png)
![](.//media/EndExample.png)

Update the options in the red box (as marked above):

-   **Track Event Executions**: Check

-   **Submit Events as OCADM**: Check

-   **User Inputs**: Select "IBMEXIBATCH \[Text\]"

*Click the edit icon (blue pencil), in the red circle (as marked above).*


Complete the following User Input Definition:

![](.//media/StartExample.png)
![](.//media/image18.png)
![](.//media/EndExample.png)

Type or select the following options:

-   **User Input Caption**: IBM LSAM Export Batch ID

-   **Required Variable**: Check

-   **User Input Type** (Pull Down) Select: Text

-   **Minimum Characters**: 1

-   **Maximum Characters**: 20

*Click "OK".*


Define appropriate user authorities:

![](.//media/StartExample.png)
![](.//media/image19.png)
![](.//media/EndExample.png)

**Show for Role** (for example):

-   Granted permission \"Role_ocadm\"

*Click "Save" Button.*


Self Service Job Execution
==========================

Click the Self Service button from the Home screen.

![](.//media/StartExample.png)
![](.//media/image3.jpg)
![](.//media/EndExample.png)


Click the "IBM LSAM Export from Test to Production" the following screen display:

![](.//media/StartExample.png)
![](.//media/image21.png)
![](.//media/EndExample.png)


Enter the user-defined LSAM Batch ID you are exporting: e.g., OPRRPY0001

(The Batch ID is often meaningful text, not to be confused with the export file name.)

![](.//media/StartExample.png)
![](.//media/image22.png)

*Click the Submit Button:*

![](.//media/image23.png)
![](.//media/EndExample.png)


*Click the "OK" button:*

![](.//media/StartExample.png)
![](.//media/image24.png)
![](.//media/image24b.png)
![](.//media/EndExample.png)


Open Enterprise Manager (EM), or Solution Manager (SM).

Open List View in EM, or a schedule view in SM.

Browse to today's date and you will see:

> The schedule \"IBM LSAM Export from Test to Production\" is **\[RUNNING\]**

Appendix
========

Workflow Designer Diagram
-------------------------


![](.//media/image25.png)

