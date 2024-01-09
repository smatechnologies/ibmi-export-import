# **OpCon Agent for IBM i:** 
## Automating IBM i Agent (LSAM) PTF Distribution

**Document Version**

Version:  2024-01-09

This version uses the Enterprise Manager (EM) user interface as the basis for describing the model automation. 

**Table of Contents**

[OpCon IBM i Agent LSAM PTF Distribution Overview](#opcon-ibm-i-agent-lsam-ptf-distribution-overview)

[Process Guide](#process-guide)

[Configuration Requirements](#configuration-requirements)


### [OpCon Enterprise Manager Configuration](#opcon-enterprise-manager-configuration-1)

[OpCon Properties](#opcon-properties)

- IBMLSAMPTF_FILEPATH: [IBM i LSAM PTF Save File  Path name](#ibmlsamptf_filepath)

- IBMLSAMPTF_SRCMACH: [IBM i LSAM Source (Test)  Machine name](#ibmlsamptf_srcmach)

[Machine Group](#machine-group)

[Source Machine Available Property](#source-machine---available-property)

[OpCon IBM LSAM PTF Distribution from Test to Production Schedule](#opcon-schedule-ibm-lsam-ptf-distribution-from-test-to-production)

[Configure OpCon Jobs](#configure-opcon-jobs)

- [Initialization Job](#initialization-job)

- [LSCTLDTA FTP Transfer from Test to Production](#lsctldta-ftp-transfer-from-test-to-production)

- [LSCUMPTF FTP Transfer from Test to Production](#lscumptf-ftp-transfer-from-test-to-production)

- [APPLY PTFS](#apply-ptfs)

    - [APPLY PTFS User ID NOTE](#job-user-id-note)

- [Self Service Web Application (optional)](#self-service-web-app)

### [Appendix](#appendix-1)

[Workflow Designer Diagram](#workflow-designer-diagram)


OpCon IBM i Agent LSAM PTF Distribution Overview
================================================

SMA offers its clients a PDF document and an accompanying data file that define a tested model of how to automate the IBM i LSAM PTF Distribution process.

To implement this type of automated solution, read and follow the instructions in this illustrated documentation. Please understand this is a guide, and it is possible that some details might not match every client site.

**Process Overview**

This document guides the reader through two phases of operations.

(1) The first goal of this document is to walk the OpCon administrator through a one-time process required to pre-configure the various elements of the automation solution.

(2) Once the configuration work is done, the whole actual LSAM PTF Distribution process can be repeated many times with two simple manual steps:  

- The first step is to download the latest IBM i LSAM cumulative PTF save files from SMA's current network-based resource.
    - The IBM i Agent offers an option for automating the fetch of the LSAM PTF save files, but most sites do not want their IBM i partitions exposed to the internet outside of their in-house LAN, for security reasons.
    - The common practice is to manually log into the SMA file server and then download the LSAM PTF save files to a local workstation.  From there, follow the in-house security procedures for transferring those two files to the IBM i partition's IFS (root file system) directory path that was registered in the LSAM sub-menu 9, option 7: *PTF options configuration.*
- Finally, OpCon Schedule management, by Frequency, Dependencies and/or manual initiation, launches the process of distributing and installing the LSAM PTFs to every IBM i LPAR in the network, executing these four steps:
    > Initializing OpCon Schedule Instance Property values.

    > Two concurrent FTP jobs that transfer from the source (Test) LPAR to each Production Machine (Agent) in a Machine Group the two save files:  
    - **LSCTLDTA** = PTF control data.
    - **LSCUMPTF** = Cumulative collection of LSAM PTF save files, gathered within this transportation save file.

    > A final, simple IBM i Batch Job that executes the "LSAPTFINS" (LSAM PTF Install) command at each Prouction Machine (Agent).
        
**How to Use This Document**

This documentation explains the configuration of OpCon components and the IBM i LSAM, for automating the process of exporting LSAM automation tool configurations from an IBM i LSAM source partition (a Test or master LSAM), and then distributing and installing the LSAM PTFs into the LSAM environment(s) in one or many remote IBM i LPARs (referred to herein as Production LSAMs).

Accompanying the instructions is a data file that can be used as indicated in the instructions:

- An .XML file is data that defines the model of an OpCon Schedule of jobs, as it was tested by an SMA team.

These resources are being offered as-is and the document must be understood as a general guidance document. SMA recognizes that clients may have different network configurations that may not conform exactly to the environment that SMA used to test the proposed automation plan.

Please study the document before executing its instructions. Clients may need to acquire and install additional components of the OpCon suite of applications to implement the automated process in the way that is illustrated in the document.

If you need additional help, please contact SMA Technologies Client Support. SMA supports the software components that it sells according to
the Support agreement in effect at each client site. Additional assistance with automation analysis, such as working with this example of automating LSAM PTF distribution, can be requested, although this type of assistance is usually subject to a charge for hourly services, or previously purchased service hours may be applied to this type of assistance.

Process Guide
=============

The OpCon IBM i LAM PTF automation process allows the client to transmit via FTP the LSAM software patches (PTFs) from a Test machine (a/k/a master Agent) to one or many Production machines based on the configuration of the OpCon Machine Group, and then to install the patches using the existing single LSAM command that is always used to install all the latest LSAM PTFs into each Production machine.

**Configuration Pre-requisites**

The instructions in this document include a list of prerequisites that must be completed before automation of the automated distribution of LSAM PTFs can be performed. The automation tools that must be configured include:

-   OpCon Properties

-   OpCon Machine Groups

-   Source Machine - Available Property

-   OpCon Schedule

-   OpCon Jobs

    - IBM i User Profile with *SECADM authority

-   Self Service Button (optional)

To initiate the OpCon Schedule that performs the IBM i LSAM PTF automation process, the Schedule can be built manually on demand, it can be scheduled to build and execute automatically according to Frequence controls, or it can be created and launched by a Solutions Manager Self-Service web application.

Configuration Requirements
==========================

1.  OpCon Global Properties -- global properties are used to configure the variables used by multiple jobs. These will be instantiated and have their values set as Schedule Instance (SI.) properties at run time.

2.  OpCon Machine Groups -- machine groups are used to control one to many production machines as targets for FTP and Import processing.

3.  Source Machine - Available Property -- adding a user-defined property to the Source Machine record creates a convenient method for retrieving its Fully Qualified Domain Name (FQDN).  From this unique storage location the FQDN can be referenced as a Machine Instance variable (e.g., MI.FQDN,<machine name>).

4.  OpCon Schedule -- the OpCon schedule is a single schedule required to contain all 4 jobs for transferring and installing LSAM PTFs to the Production machine(s).

5.  OpCon Jobs -- 4 jobs in the schedule that is assigned to a Machine Group are used to:   
    - Initialize variables
    - FTP the LSCTLDTA save file to the Production machine(s) in a Machine Group
    - FTP the LSCUMPTF save file to the Production machine(s) in a Machine Group
    - Perform the complete LSAM PTF installation process in each Production machine of a Machine Group

6.  Self Service Button -- a Solution Manager Self Service web application can be used to collect one or more variations of the Schedule (each assigned to its own Machine Group, if necessary) into a convenient one-button appliance that will create and initiate operation of the Schedule.  
    - However, a Self Service application is not required.
    - If it is desired, an example of steps required to build a Self Service web application can be found in the [User Guide for automating the LSAM Export-Import process](UserGuide-Automate-Exp-Imp.md).
        - However, the data entry field defined in that instructional reference is not needed for this document's application.

Exported Information
====================

**Schedule Extract XML**

An SMADDI Schedule Extract XML file is provided as an extract of the *IBM i Distribute LSAM PTFs* Schedule and it can be imported per the instruction in the Enterprise Manager User Guide Opening Import Export section.

File Name: 
> IBM-LSAM-PTF-Distribution-22v10-SMADDI.xml


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

#### IBMLSAMPTF_FILEPATH

-   **Name Text Box**: IBMLSAMPTF_FILEPATH

-   **Documentation Text Box**: IBM i IFS root file system path to LSAM PTF save files.

-   **Value Text Box**: (The default path usually assigned at OpCon client sites is shown here.)
    > /SMA/IBMiLSAMptf/21.1/

    The LSAM version number, such as 21.1, will change depending on which LSAM release is installed at the site.

    *Be sure to include the trailing slash (/) character in the path value.*

-   **NOTE**:  Be sure to update the "Initialization Job" Events tab with the correct value for this variable.  The correct name is registered in the IBM i LSAM PTF and Security menu 9, option 7: PTF options configuration.  Use the name registered in the field: *Source directory or path*.

#### IBMLSAMPTF_SRCMACH

-   **Name Text Box**: IBMLSAMPTF_SRCMACH

-   **Documentation Text Box**: Test Machine (a/k/a Source Agent) where the original downloaded LSAM PTF save files are stored.

-   **Value Text Box**: *Enter the Machine Name (SM: Agent name) for the Test LSAM.*

-   **NOTE**:  Be sure to update the "Initialization Job" Events tab with the correct value for this variable.


Machine Group
-------------

Create a Machine Group for one to many Production machine(s).

*NOTE: The steps for this procedure will vary when using the SM (Solution Manager) user interface, but the elements remain the same.*

If there are LPARs with software applications that have different hourly schedules, a unique Machine Group can be specified in a separate copy of the OpCon Schedule, so that the LSAM PTF installation process which might interrupt automation operations for a short time can be scheduled or initiated at a different time from other Machine Groups.

To add a machine group:

1.  Double-click on the Machine Groups under the Administration topic. The Machine Group screen displays.

2.  Click Add on the Machine Group tool bar.

3.  Type in the Machine Group name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

5.  Click the Machine Type drop-down and select IBM i.

6.  Select a Production Machine (a/k/a Agent) by clicking the machine in the Machine Assignment section Unassigned Machines.

7.  Click the green arrow between the Unassigned Machine and the Assigned Machines to add to the Machine Group.

8.  Click the Green check in the upper right hand to save the Machine Group.

#### Field Values:

-   **Name Text Box**: Name the Machine Group

-   **Documentation**: Type the document Machine Group information

-   **Machine Type Drop-down**: Click IBM i from the Machine Type drop-down

-   **Machine Assignment**: Select a production machine by clicking the machine in the Machine Assignment section Unassigned Machines.


Source Machine - Available Property
-----------------------------------

Instead of using some form of API access to Machine records in the OpCon database, the Fully Qualified Domain Name (FQDN) of the Source Machine will be stored as a user-defined Available Property, registered in the OpCon user interface under Advanced Machine Properties.

After this new Available Property which shall be called *"FQDN"* is registered, it can be referenced anywhere that OpCon Properties are used as a Machine Instance property:  **MI.FQDN**

    NOTE:  MI. properties can be qualified using the Machine name, or using the [[$MACHINE NAME]] OpCon system property.  An example appears in the FTP job within these instructions.

The FQDN of the Source Machine is preferred over the IP Address assigned to the Source Machine because the FQDN can be associated with a different IP address very quickly and easily during a disaster recovery process, thus avoiding any requirement to change OpCon job detailed parameters that might reference IP addresses.

The Machine Instance of the FQDN assigned to the Source Machine is used by the FTP job that transfers an LSAM Export Save File to one or more target Machines.

*Following instructions are based on the Enterprise Manager (EM) user interface.  Similar steps can also be performed from the Solution Manager (SM).*

To add an Available Property to a Machine (SM: calls this an "Agent"):

1. Select "Machines" from the EM Administration menu list.
    - **SM**: From Library, under the Administration heading, select Agents. 
    - Then find the Source Machine (Agent) for LSAM PTFs in the list.

2. Before beginning maintenance to a Machine, change the Machine status to stopped.

3. Click on the option to Open Advanced Settings Panel.
    - **SM***: Use a right mouse click to open the Agent management window (titled "Agent Selection").  Click left mouse on the wrench icon that appears under the window title.

4. Click on the Administrative Machine Information tab.
    - **SM**: Click the arrow next to Administrative Machine Information.

5. Click the green plus sign (+) next to Available Property.
    - **SM**: Click the Manual Edit button.

6. In the data entry box that appears, type the name of a new property, then an equals sign (=) and then the value string that should be assigned to this new Available Machine Property.  Click the Save button to store a new or updated value.
    - **SM**: Click the blue pencil icon on the right to enter the update mode.  Type into the Name column the new Available Property name, then type into the Value column the string of data represented by the Property name.  Click the blue check box to save additions or changes.

    ``` 
    EXAMPLE:

    To match this instruction, use the name "FQDN".  Type the actual Fully Qualified Domain Name of the Source Machine (Agent) that will contain the LSAM PTF save files needing distribution.

        FQDN=SMADEV.GLOBALSMA.COM

    Note: Do not enclose the Value string within quotes.  If quotes are typed, they will become part of the Value string.
    ```

OpCon Schedule: IBM LSAM PTF Distribution from Test to Production
-------------------------------------------------------

The [Appendix](#appendix-1) shows an image of the OpCon Workflow Diagram for the *IBM i Distribute LSAM PTFs* Schedule.

To add a master schedule:

1.  Double-click on Schedule Master under the Administration topic. The Schedule Master screen displays.

2.  Click Add on the Schedule Master tool bar.

3.  Type a schedule name in the Name text box.

4.  (Optional) Type any text in the Documentation text box.

\*\* Recommended:  Include in the documentation text box information about how this schedule will be managed within the local site.

#### Schedule Details
##### Schedule tab
-   **Name Text Box**: IBM LSAM Export from Test to Production

-   **Documentation Text Box**: This schedule is designed to be run from Self Service. It requires the LSAM export Batch Name in an OpCon Global Property.

-   **Workdays per Week**: Click all days

-   **Schedule Properties**: Check the box *Multi-Instance*

##### Instance definition
-   **Definition type**: Mark the radio button for **(.)Machine Group**

-   **Machine Group**: ("Build an instance for each machine in Machine Group") Select the desired Machine Group from the drop-down menu.
    - **NOTE**: The Machine Group assigned to the Schedule will cause the Schedule to be run once for each machine in the group, and that machine will be the default machine of reference for the FTP jobs and the LSAM PTF installation job.

Configure OpCon Jobs
--------------------

To add a job:

1.  Double-click on Job Master under the Administration topic. The Job Master screen displays.

2.  Select the appropriate Schedule Master in the top line.  

3.  Click Add on the Job Master toolbar.

4.  Type a job name in the Name textbox.


### Initialization Job

The Initialization Job initializes SI. instances of the OpCon properties required by the other jobs in this Schedule.  It uses the Events tab to execute the $PROPERTY:ADD event commands that create SI. instances of the Properties for each instance of the Schedule that is instantiated by the OpCon SAM for each Machine in the Machine Group assigned to the Schedule.
    - This Initializaation Job is configured as an IBM i batch job so that a message can be logged in the IBM i System Operator message queue (QSYSOPR) documenting the instance of the Schedule that has been started.

**Selection**

-   **Name Text Box**: Initialization Job

**Job Properties/Job Details Tab**

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

-   **Call**: SNDMSG MSG('SMA OpCon Agent automated LSAM PTF distribution reports start of schedule instance: "[[$SCHEDULE NAME]]"  for schedule date: [[$SCHEDULE DATE ISO]]') TOUSR(*SYSOPR)      
    - Any command or program call can be used here.  This suggested command creates entries in the QSYSOPR message queue that could be monitored by the LSAM.  Another message queue could be used to collect just these notices of the Schedule execution.

**Job Properties/Events Tab**

-   Click the Add button

-   Select the Job Status toggle

-   Select *Finished OK* from the Job Status drop-down selection

-   Select \$PROPERTY:ADD,\<property name\>,\<initial value\> from the Event Template drop-down selection

-   Two ADD commands should be added, replace the Event Parameters in each of the commands with one of these five value pairs:

    > SI.IBMLSAMPTF_FILEPATH,/SMA/IBMiLSAMptf/21.1/

    > SI.IBMLSAMPTF_SRCMACH,IBMILSAM
  
        The value ",IBMILSAM" must be replaced with the actual name of the IBM i LSAM Machine name.  The value shown here is the default for the SMADEFAULT LSAM enrivonment after an original install of the default LSAM configuration.

### LSCTLDTA FTP Transfer from Test to Production 

The FTP Transfer from Test to Production job transfers the LSCTLDTA save file from the Test machine to one or many machine(s) that are configured in the machine group.

**Selection**

-   **Name Text Box**: LSACTLDTA FTP Transfer from Test to Production

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Machine Selection**: Check the box [x] *Use Schedule Instance Machine*.

**IBM i Definition/Job Information Tab**

-   **Job Type**: Select FTP from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.
    - This local job User ID is assigned within each Production Machine.  It can be different for each Machine, but then it would be necessary to create different copies of this Schedule and to use a different Machine Group for each set of Production Machines that can be assigned the same FTP Job User ID.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.
    - To enable a user-selected Job Description, select asterisk (\*) which refers to the LSAM Default Parameters.  The actual Job Description name can be set in each/any target Machine by using the LSAM Parameters maintenance function which is option 7 on the main LSAM menu.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.
    - If the Job Description is set to asterisk (\*) then the Job Description Library name should also be set to asterisk so that the LSAM Parameters default value for this library will match the specified Job Description.

**Call Information/Transfer Information Tab/ftp options**

-   **Action Type**: Select GET from the drop-down selection.

-   **Transfer Type**: Select BIN from the drop-down selection.

-   **User**: type the named user that was configured in the local IBM i LSAM.  *(See following instructions)*

**IBM i LSAM ftp User Configuration**

    The User specified for the Remote system must be registered in the LSAM User Management function at menu 4, option 1.  The password(s) from the Production Machine(s) must be registered using this function from with the Test partition(s) LSAM menu system.

    This Remote system user refers to the Test (Source) machine.  So all Production Machines will always use the same Remove FTP User ID.

-  From the LSAM Main Menu Navigate to the LSAM sub-menu 4, option 1: User management.

-  **F6=Add**: Type the user profile in the User Name field, type the password in the Password field and retype the password in the Password (to verify) field.

    *Press the ENTER key to confirm.*

**Call Information/Remote Information Tab**

-   Remote System: \[\[MI.FQDN\[\[SI.IBMLSAMPTF_SRCMACH\]\]\]\]

-   Remote File System: \*LCLFILNAM

-   Remote Library or Directory: \[\[SI.IBMLSAMPTF_FILEPATH\]\]

**Call Information/Location Information Tab**

-   Local File Name: LSCTLDTA

-   Local Library or Directory: \[\[SI.IBMIMP_SMALOG\]\]

**Properties/Frequency Tab**

	Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add a job dependency

-   Select *Initialization Job* from the Job drop-down selection

-   Click *Requires* from the Dependency Type toggle

-  **Options**: Select *Finished OK* from the options drop-down selection


### LSCUMPTF FTP Transfer from Test to Production 

This job is identical to the LSCTLDTA FTP Transfer job, defined above.  It may run in parallel with the LSCTLDTA FTP Transfer job.  Both jobs must be completed before the following job is allowed to execute the SMAPTFINS (PTF install) command job.

To configure the LSCUMPTF FTP Transfer job, it is possible to copy from the LSCTLDTA FTP Transfer job.  After the copy is made, with a new name, then replace the reference to "LSCTLDTA" with reference to "LSCUMPTF" at the following point:

    **Call Information/Location Information Tab**

    -   Local File Name: LSCUMPTF


### APPLY PTFS

The APPLY PTFS job executes the IBM i LSAM command SMAPTFINS to install the latest LSAM PTFs into the target machine's LSAM software environment.

#### Job User ID NOTE
As documented in the IBM i Agent User Help, the installation of the IBM i LSAM PTFs requires a user profile with the same broad authorities as the QSECOFR user profile.  The model schedule XML data file included in this project's directory under the SMA Innovations Lab names the user profile "OPCON" for this purpose, rather than naming the QSECOFR user profile itself.  It is up to the site to configure and manage access to such a powerful user profile, but this requirement cannot be overcome because of the unique built-in authority strategies of many LSAM automation tools.

The appropriate User ID must be registered in the OpCon database before attempting to import the model schedule using the OpCon SMADDI direct database input tool.  It does not have to exist in the IBM i partition during the SMADDI import process, but a matching user profile name with the required authority must be created in the IBM i partition before this "APPLY PTF" job can be successfully executed.

**Selection**

-   **Name Text Box**: APPLY PTFS

**Job Properties/Job Details Tab**

-   **Job Type**: select IBM i from the drop-down.

-   **Machine Selection**: Check the box [x] *Use Schedule Instance Machine*.

**IBM i Definition/Job Information Tab**

-   **Job Type**: Select Batch Job from the drop-down.

-   **User ID**: Select your configured User ID from the drop-down selection.  See the *Job User ID NOTE* above.

-   **Job Description Name**: SMALSAJ00
    - An alternative job description can be used as long as the four LSAM libraries are included in the initial library list.
    - To enable a user-selected Job Description, select asterisk (\*) which refers to the LSAM Default Paramters.  The actual Job Description name can be set in each/any target Machine by using the LSAM Parameters maintenance function which is option 7 on the main LSAM menu.

-   **Job Description Library**: SMADTA
    - An alternative library can be named, depending on where the Job Description object is stored.
    - If the Job Description is set to asterisk (\*) then the Job Description Library name should also be set to asterisk so that the LSAM Parameters default value for this library will match the specified Job Description.

-   **Call**: SMAPTFINS

    - The SMAPTFINS command needs no parameters because the command processor program picks up the LSAM PTF Parameters configured in LSAM menu 9, option 7.

**Properties/Frequency Tab**

Click the Add Frequency button to add a frequency configured like below:

![](.//media/image2.png)

**Job Properties/Dependencies Tab**

-   Click the Add button to add two job dependencies:

-   Select *LSCTLDTA FTP Transfer from Test to Production* from the Job drop-downselection

-   Also select *LSCUMPTF FTP Transfer from Test to Production* from the Job drop-downselection

-   Click *Requires* for each of these dependencies.

-   **Options**: Select *Finished OK* from the options drop-down selection for each dependency.


### Self Service Web App

**OPTIONAL**

A Self-Service web application, supported by the Solution Manager user interface of OpCon, is not required for managing this schedule of jobs.  However, if a site desires to use this method to manually build and initiate execution of the LSAM PTF Distribution schedule, then an example of the steps for building a Self Service web applicaxtion may be found in the [IBM LSAM Data Export-Import](UserGuide-Automate-Exp-Imp.md) instruction document.

**NOTE**: A Self-Service web application for this IBM LSAM PTF Distribution schedule does not require a schedule instance input parameter, although a site may decide to manage one or more parameters of this schedule's jobs by using that technique.

Appendix
========

Workflow Designer Diagram
-------------------------


![](.//media/image41.png)

