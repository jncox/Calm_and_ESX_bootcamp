.. _calm_win:

-----------------------
Calm: Windows Workloads
-----------------------

*The estimated time to complete this lab is 60 minutes.*

Overview
++++++++

**In this exercise you will explore the basics of working with Windows workloads in Nutanix Calm by building and deploying a blueprint that installs and configures a multi-tier** `bug tracker <http://bugnetproject.com/documentation/>`_ **web app using Microsoft SQL Server database & IIS webserver.
This lab assumes you are familiar with basic Calm functionality or have completed the** :ref:`calm_linux` **lab.**

Creating the Blueprint
++++++++++++++++++++++

#. Within Calm, create a new **Multi VM/Pod Blueprint**.

#. Fill out the following fields and click **Proceed** to launch the Blueprint Editor:

   - **Name** - *Initials*-CalmWindowsIntro
   - **Description** - [BugNET](\http://@@{MSIIS.address}@@/bugnet)
   - **Project** - *Initials*-Calm

   .. note::

     Using the description value provided will create a hyperlink to the BugNET application to launch once deployment has completed.

#. Click **Credentials** and create the following two credentials:

   +---------------------+---------------------+---------------------+
   | **Credential Name** | WIN_VM_CRED         | SQL_CRED            |
   +---------------------+---------------------+---------------------+
   | **Username**        | Administrator       | Administrator       |
   +---------------------+---------------------+---------------------+
   | **Secret Type**     | Password            | Password            |
   +---------------------+---------------------+---------------------+
   | **Password**        | nutanix/4u          | Str0ngSQL/4u$       |
   +---------------------+---------------------+---------------------+

   .. figure:: images/credentials.png

#. Click **Save** and return back to the Blueprint Editor.

#. Using the **Default** Application Profile, specify the following **Variables** in the **Configuration Panel**:

   +---------------------+---------------+----------------+---------------+---------------+
   | **Name**            | **Data Type** | **Value**      | **Secret**    | **Runtime**   |
   +=====================+===============+=================+==============+===============+
   | DbName              | String        | BugNET         | No            | Yes           |
   +---------------------+---------------+----------------+---------------+---------------+
   | DbUsername          | String        | BugNETUser     | No            | Yes           |
   +---------------------+---------------+----------------+---------------+---------------+
   | DbPassword          | String        | Nutanix/4u$    | Yes           | Yes           |
   +---------------------+---------------+----------------+---------------+---------------+
   | User_initials       | String        | *Leave blank*  | No            | Yes           |
   +---------------------+---------------+----------------+---------------+---------------+

   .. figure:: images/variables.png

#. Click **Save**.

Adding Services
+++++++++++++++

#. Under **Application Overview > Services**, click :fa:`plus-circle` twice to add two new Services.

   .. figure:: images/create_service.png

#. Use the table below to complete the **VM** fields for each service:

   +------------------------------+---------------------------+---------------------------+
   | **Service Name**             | **MSSQL**                 | **MSIIS**                 |
   +------------------------------+---------------------------+---------------------------+
   | **Name**                     | MSSQL2014                 | MSIIS8                    |
   +------------------------------+---------------------------+---------------------------+
   | **Account**                  | vmware                    | vmware                    |
   +------------------------------+---------------------------+---------------------------+
   | **Operating System**         | Windows                   | Windows                   |
   +------------------------------+---------------------------+---------------------------+
   | **Compute DRS Mode Cluster** | nu-cl                     | nu-cl                     |
   +------------------------------+---------------------------+---------------------------+
   | **Template**                 | Win2016-Template          | Win2016-Template          |
   +------------------------------+---------------------------+---------------------------+
   | **Storage DRS Mode Cluster** | DatastoreCluster          | DatastoreCluster          |
   +------------------------------+---------------------------+---------------------------+
   | **Instance Name**            | @@{User_initials}@@-MSSQL | @@{User_initials}@@-MSIIS |
   +------------------------------+---------------------------+---------------------------+
   | **vCPUs**                    | 2                         | 2                         |
   +------------------------------+---------------------------+---------------------------+
   | **Cores per vCPU**           | 2                         | 2                         |
   +------------------------------+---------------------------+---------------------------+
   | **Memory (GiB)**             | 6                         | 6                         |
   +------------------------------+---------------------------+---------------------------+
   | **Additional vDisks**        | 1                         | 1                         |
   +------------------------------+---------------------------+---------------------------+
   | **Device Type**              | DISK                      | DISK                      |
   +------------------------------+---------------------------+---------------------------+
   | **Adapter Type**             | SCSI                      | SCSI                      |
   +------------------------------+---------------------------+---------------------------+
   | **Size (GiB)**               | 100                       | 100                       |
   +------------------------------+---------------------------+---------------------------+
   | **Location**                 | datastore_01              | datastore_01              |
   +------------------------------+---------------------------+---------------------------+
   | **Controller**               | SCSI Controller 0         | SCSI Controller 0         |
   +------------------------------+---------------------------+---------------------------+
   | **Device Slot**              | 1                         | 1                         |
   +------------------------------+---------------------------+---------------------------+
   | **Disk Mode**                | Dependent                 | Dependent                 |
   +------------------------------+---------------------------+---------------------------+
   | **VM Guest Customization**   | Enable                    | Enable                    |
   +------------------------------+---------------------------+---------------------------+
   | **Predefined List**          | WindowsSpec               | WindowsSpec               |
   +------------------------------+---------------------------+---------------------------+
   | **Check log-in upon create** | Yes                       | Yes                       |
   +------------------------------+---------------------------+---------------------------+
   | **Credential**               | WIN_VM_CRED               | WIN_VM_CRED               |
   +------------------------------+---------------------------+---------------------------+
   | **Address**                  | Template-NIC 1            | Template-NIC 1            |
   +------------------------------+---------------------------+---------------------------+
   | **Connection Type**          | Windows (Powershell)      | Windows (Powershell)      |
   +------------------------------+---------------------------+---------------------------+
   | **Connection Port**          | 5985                      | 5985                      |
   +------------------------------+---------------------------+---------------------------+
   | **Delay (in seconds)**       | Increase to **120**       | Increase to **120**       |
   +------------------------------+---------------------------+---------------------------+

   Similar to the Task Manager application in the :ref:`calm_linux` lab, you want to ensure the database is available prior to the IIS web server setup.

#. In the Blueprint Editor, select the **MSIIS** service and create a dependency on the **MSSQL** service.

   .. figure:: images/services.png

Defining Package Install
++++++++++++++++++++++++

For **each** of the following 7 scripts (3 for MSSSQL and 4 for MSIIS), the **Type**, **Script Type**, and **Credential** fields will be the same:

- **Type** - Execute
- **Script Type** - PowerShell
- **Credential** - WIN_VM_CRED

.. note::

  If you were working with domain joined VMs, you would require a separate domain credential to execute PowerShell scripts following the VM being joined to the domain.

#. Select the **MSSQL** service and open the **Package** tab in the **Configuration Panel**.

#. Name the package and click **Configure install** to begin adding installation tasks.

   You will add multiple scripts to complete each installation. Working with multiple scripts allows for easier maintenance and application of code across multiple services or blueprints using the Calm **Task Library**. The Task Library allows you to create modularized scripts to achieve certain common functions such as joining a domain or configuring common OS settings.

#.  Under **MSSQL > Package Install**, click **+ Task** and fill out the following fields:

    - **Task Name** - InitializeDisk1
    -  **Script** -

    .. literalinclude:: InitializeDisk1.ps1
       :language: posh

    The above script simply performs an initialization and format of the extra 100GB VDisk added during VM configuration of the service.

#. Click **Publish To Library > Publish** to save this task script to the Task Library for future use.

#.  Repeat clicking **+ Task** to add the remaining two scripts:

    - **Task Name** - InstallMSSQL
    - **Script** -

    .. literalinclude:: InstallMSSQL.ps1
       :language: posh

    Reviewing the above script you can see it is performing an automated installation of SQL Server, using the SQL_CRED credential details and using the extra 100GB VDisk for the SQL data files.

    According to Nutanix best practices for production database deployments, what else would need to be added to the VM/installation?

    - **Task Name** - FirewallRules
    - **Script** -

    .. literalinclude:: FirewallRules.ps1
       :language: posh

    Reviewing the above script you can see it is allowing inbound access through the Windows Firewall for key SQL services.

    Once complete, your MSSQL service should look like this:

    .. figure:: images/mssql_package_install.png

#. Select the **MSIIS** service and open the **Package** tab in the **Configuration Panel**.

#. Name the package and click **Configure install** to begin adding installation tasks.

#. Under **MSIIS > Package Install**, click **+ Task**.

#. Similar to the first step of the MSSQL service installation, you will need to initialize and format the additional 100GB VDisk. Rather than manually specifying the same script for this task, click **Browse Library**.

#. Select the **InitializeDisk1** task you had previously published and click **Select > Copy**.

   .. figure:: images/task_library.png

   .. note::

     The Task Library also gives you the ability to provide variable definitions if there are Calm macros present in the published task.

#.  Specify the **Name** and **Credential**, then repeat clicking **+ Task** to add the remaining three scripts:

    - **Task Name** - InstallWebPI
    - **Script** -

    .. literalinclude:: InstallWebPI.ps1
       :language: posh

    The above script installs the Microsoft Web Platform Installer (WebPI), which is used to download, install, and update components of the Microsoft Web Platform, including Internet Information Services (IIS), IIS Media Platform technologies, SQL Server Express, .NET Framework, and Visual Web Developer.

    - **Task Name** - InstallNetFeatures
    - **Script** -

    .. literalinclude:: InstallNetFeatures.ps1
       :language: posh

    The above script installs .NET Framework 4.5 on the VM.

    - **Task Name** - InstallBugNetApp
    - **Script** -

    .. literalinclude:: InstallBugNetApp.ps1
       :language: posh

    The above script uses the Application Profile variables you defined at the beginning of the exercise to populate the configuration file of the Bug Tracker app. It then leverages WebPI to install the application from the `Microsoft Web App Gallery <https://webgallery.microsoft.com/gallery>`_. With minimal changes, you could leverage many popular applications from the Gallery, including apps for CMS, eCommerce, Wiki, ticketing, and more.

    Once complete, your MSIIS service should look like this:

    .. figure:: images/msiis_package_install.png

#. Click **Save**.

Launching the Blueprint
+++++++++++++++++++++++

#. From the upper toolbar in the Blueprint Editor, click **Launch**.

#. Specify a unique **Application Name** (e.g. *Initials*\ -BugNET) and your **User_initials** Runtime variable value for VM naming.

#. Click **Create**.

   The **Audit** tab can be used to monitor the deployment of the application. The application should take approximately 20 minutes to deploy.

#. Once the Create action completes, and the application is in a **Running** state, open the **BugNET** link in a new tab.

   .. figure:: images/bugnet_link.png

#. You'll be presented with an **Installation Status Report** page.  Wait for it to report **Installation Complete**, and then click the link at the bottom to access the application.

   .. figure:: images/bugnet_setup.png

   Congratulations! You now have a fully functional bug tracking application automatically provisioned leveraging Microsoft SQL Server and IIS.

   .. figure:: images/bugnet_app.png

(Optional) Scale Out IIS Tier
+++++++++++++++++++++++++++++

Leveraging the same approach from the :ref:`calm_linux` lab of having multiple web server replicas, can you add a CentOS based HAProxy service to this blueprint to allow for load balancing across multiple IIS servers?

**Hints**

- Clone your existing blueprint first!
- When registering the SQL Server source database in Era, this deployment uses the default MSSQLServer instance name. You can use Windows Authentication to access the SQL Server instance, using the WIN_VM_CRED credentials.
- When adding services in Calm, one of the **Cloud** types is using an **Existing VM**. Existing VMs only require the IP address of the VM and a credential for login.
- You could use a semi-automated approach wherein you have a **Runtime** variable for your cloned database IP. In this instance, you would create a clone of your source database, wait for it to return an IP address, and provision the blueprint with the IP specified at runtime.
- You could use a fully automated approach wherein you create a **Package Install Task** for your **Existing VM**. That task could execute an `EScript <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Calm-Admin-Operations-Guide-v240:nuc-supported-escript-modules-functions-c.html#nconcept_uxr_5dj_5bb>`_ to perform an API call to Era to initiate the DB clone operation and return the IP address.
- Don't forget about dependencies!

Takeaways
+++++++++

- Calm provides the same application deployment and lifecycle management benefits for Windows workloads as it does for Linux workloads.

- Calm can natively execute remote PowerShell scripts on Windows endpoints without the need for a Windows-based proxy.


.. |projects| image:: images/projects.png