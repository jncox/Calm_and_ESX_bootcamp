.. _calm_iaas_windows:

---------------------------------
Calm: Windows Single VM IaaS
---------------------------------

*The estimated time to complete this lab is 20 minutes.*

Overview
++++++++

Nutanix Calm allows you to seamlessly select, provision, and manage your business applications across your infrastructure for both the private and public clouds. Nutanix Calm provides App lifecycle, monitoring and remediation to manage your heterogeneous infrastructure, for example, VMs or bare-metal servers. Nutanix Calm supports multiple platforms so that you can use a single self-service and automation interface to manage all your infrastructure.

Infrastructure-as-a-Service (IaaS) is defined as the ability to quickly provide compute resources, on-demand through a self service portal.  While many customers utilize Nutanix Calm to orchestrate complex, multi-tiered applications, a significant portion of customers also utilize Calm to provide basic IaaS for their end users.

**In this lab you'll create a "Single VM Blueprint" based on Windows, launch the blueprint, and manage the resulting application.**

Creating a Single VM Blueprint
++++++++++++++++++++++++++++++

A blueprint is the framework for every application or piece of infrastructure that you model by using Nutanix Calm.  While complex, multi-tiered applications utilize the "Multi VM/Pod Blueprint", the streamlined interface of the "Single VM Blueprint" is conducive for IaaS use cases.  You can model each type of infrastructure your company utilizes (for instance Windows, CentOS, and/or Ubuntu) in a Single VM blueprint, and end users can repeatedly launch the blueprint to create infrastructure on demand.  The resulting infrastructure (which is still referred to as an "application"), can then be managed throughout its entire lifecycle within Calm, including managing Nutanix Guest Tools (NGT), modifying resources, snapshotting, and cloning.

In this lab, you will be creating a **Windows 2016** server.

#. In **Prism Central**, select :fa:`bars` **> Services > Calm**.

   .. figure:: images/1_access_calm.png

#. Select |blueprints| **Blueprints** in the left hand toolbar to view and manage Calm blueprints.

   .. note::

     Mousing over an icon will display its title.

#. Click **+ Create Blueprint > Single VM Blueprint**.

#. Fill out the following fields:

   - **Name** - *Initials*-Win-IaaS
   - **Description** - Something descriptive of your choice
   - **Project** - *Initials*-Project

   .. figure:: images/3_windows_1.png
       :align: center
       :alt: Windows 2016 Blueprint Settings

       Windows 2016 Blueprint Settings

#. Click **VM Details** to proceed to the next step.

#. Note the following fields on the **VM Details** page:

   - **Name** - The internal-to-Calm name of the VM.  Can be left as default.
   - **Account** - The cloud we're deploying the infrastructure on.  Should be set to **vmware**.
   - **Operating System** - Windows (The type of OS we're deploying)

   .. figure:: images/5_windows_2.png
       :align: center
       :alt: Windows 2016 VM Details

       Windows 2016 VM Details

#. Click **VM Configuration** to proceed to the next step.

#. On this page, we'll be specifying a variety of settings for our infrastructure.

   - **General Configuration**
   - **Compute DRS Mode** - checked
   - **Cluster** - nu-cl
   - **Template** Windows2016
   - **Storage DRS Mode** - checked
   - **Datastore Cluster** - DatastoreCluster
   - **Instance Name** - this is the name of the virtual machine according to the hypervisor/cloud.  It can be left as default.
   - **CPU Hot Add** - checked
   - **vCPUs** - 2 (Mark the field as **runtime** by clicking the running man icon so it turns blue.  This allows the end user to modify this field at launch.)
   - **Cores per vCPU** - 1
   - **Memory Hot Plug** - checked
   - **Memory (GiB)** - 4 (Mark this field as **runtime**.)
  
   .. figure:: images/7_windows_3.png
       :align: center
       :alt: Windows 2016 VM Configuration - General Configuration

       Windows 2016 VM Configuration - General Configuration

   - Select the **VM Guest Customization** and then **Sysprep** and paste in the following script. Leave Join a Domain (un-checked).

     - Windows 2016

       .. literalinclude:: sysprep.xml
          :language: xml

       .. figure:: images/9_windows_4.png
           :align: center
           :alt: Windows 2016 Sysprep

           Windows 2016 Sysprep

     .. note::
        Take note of the "@@{vm_password}@@" text.  In Calm the "@@{" and "}@@" characters represent a macro.  At runtime, Calm will automatically "patch" or substitute in the proper value(s) when it encounters a macro.  A macro could represent a system defined value, a VM property, or (as it does in this case) a runtime variable.  Later in this lab we'll create a runtime variable with the name "vm_password".

   - Under **Network Settings** press the **blue plus** and verify **Use DHCP** is checked.



#. At the bottom of the page, click the blue **Save** button.  It is expected to have a single error about an incorrect macro due to our Guest Customization containing "vm_password".  If you have additional errors, please be sure to resolve them before continuing to the next section.

   .. figure:: images/15_error.png
       :align: center
       :alt: Blueprint Save - Error

       Blueprint Save - Error


Defining Variables
++++++++++++++++++

Variables allow extensibility of Blueprints, meaning a single Blueprint can be used for multiple purposes and environments depending on the configuration of its variables.  Variables can either be static values saved as part of the Blueprint or they can be specified at **Runtime** (when the Blueprint is launched), as they will in this case.

In a Single VM blueprint, variables can be accessed by clicking the **App variables** button near the top.  By default, variables are stored as a **String**, however additional **Data Types** (Integer, Multi-line String, Date, Time, and Date Time) are all possible.  Any of these data types can be optionally set as **Secret**, which will mask its value and is ideal for variables such as passwords.  There are also more advanced **Input Types** (versus the default **Simple**), however these are outside the scope of this lab.

Variables can be used in scripts executed against objects using the **@@{variable_name}@@** construct (called a macro). Calm will expand and replace the variable with the appropriate value before sending to the VM.

#. Click the **App variables** button along the top pane to bring up the variables menu.

#. In the pop-up that appears, you should see a note stating you currently do not have any variables.  Go ahead and click the blue **+ Add Variable** button to add our variable, and fill out the following fields.

   - Along the **left column**, click the **running man** icon to mark this variable as **runtime**.
   - In the main pane, set the variable **Name** as **vm_password**.  This name must *exactly* match (including case) the value within our macro from our Guest Customization script, otherwise we'll continue to get an error when we save.
   - Leave the **Data Type** as the default, **String**.  Feel free to investigate the other options, however discussion about them are outside the scope of this lab.
   - For the **Value**, leave blank, as we want the end users to specify their own VM password.
   - **Enable** the **Secret** checkbox, as we do not want this password to be visible.
   - Click the **Show Additional Options** link at the bottom.
   - Leave the **Label** field blank.
   - In the **Description** field, paste in  **Create a password for the user "Administrator"**.
   - **Enable** the **Mark this variable mandatory** checkbox.  This will ensure that the end user enters a password, which is required since we did not provide default value.
   - Leave the other two checkboxes unselected.

     .. figure:: images/16_variable.png
         :align: center
         :alt: Variable - vm_password

         Variable - vm_password

#. Scroll to the bottom, and click the blue **Done** button.

#. Click **Save**.  It is expected to receive a **Warning** stating that the value of our secret variable is empty.  This is needed as there is not way to determine the value of a secret once you save the blueprint, so this warning alerts a user in the event they accidentally left it blank.  Warnings do not prevent users from launching or publishing the blueprint.  If you receive any other warning, or a red error, please resolve the issue before continuing on.

   .. figure:: images/17_warning.png
       :align: center
       :alt: Blueprint Save - Warning

       Blueprint Save - Warning


Launching the Blueprint
+++++++++++++++++++++++

Now that our blueprint is complete, take note of the buttons to the right of the save button:

- **Publish** - this allows us to request to publish the blueprint into the Marketplace.  Blueprints have a 1:1 mapping to a Project, meaning only other users who are members of our own Project will have the ability to launch this blueprint.  Publishing blueprints to the Marketplace allows an administrator to assign any number of Projects to the Marketplace blueprint, which enables self service for any number of end users desired.
- **Download** - this option downloads the blueprint in a JSON format, which can be checked into source control, or uploaded into another Calm instance.
- **Launch** - this launches our blueprint and deploys our application and/or infrastructure.

#. Go ahead and click the **Launch** button, and enter the following:

    - **Name of the Application** - *initials*\ -Win-IaaS
    - **vm_password** - Nutanix/4u

   .. figure:: images/18_launch.png
    :align: center
    :alt: Blueprint Launch

    Blueprint Launch

#. Click **Create**, where you'll be redirected to the application page.

Managing your Application
+++++++++++++++++++++++++

Wait several minutes for your application to change from a **Provisioning** state to a **Running** state.  If it instead changes to an **Error** state, navigate to the **Audit** tab, and expand the **Create** action to start troubleshooting your issue.

Once your application is in a **Running** state, navigate around the five tabs in the UI:

.. figure:: images/19_app_tabs.png
    :align: center
    :alt: Application Tabs

    Application Tabs

- The **Overview** tab gives you information about any variables specified, the cost incurred (showback can be configured in the Calm Settings), an application summary, and a VM summary.
- The **Manage** tab allows you to run actions against the application / infrastructure.  This includes basic lifecycle (start, restart, stop, delete), NGT management (install, manage, uninstall), and App Update, which allows for editing of basic VM resources.
- The **Snapshots** tab lists the history of VM Snapshots, and allows the user to restore the VM to any of these points.
- The **Audit** tab shows every action run against the application, the time and user that ran a given action, and in depth information on the results of that action, including script output.

Next, view the common VM tasks available in the upper right corner of the UI:

.. figure:: images/20_app_buttons.png
    :align: center
    :alt: Application Buttons

    Application Buttons

- The **Clone** button allows a user to duplicate the existing application into a new app that is manageable separately from the current application.  For a brand new application, this is equivalent to launching the blueprint again.  However, a user may have spent significant time customizing the existing application to suit their specific needs, and would like these changes to be present on the new app.
- The **Snapshot** button creates a new recovery point of the VM, which allows a user to restore the VM.
- The **Launch Console** button opens a console window to the VM.
- The **Update** button allows for the end user to modify basic VM settings (this is equivalent to the **Manage > App Update** action).
- The **Delete** button deletes the underlying VM and the Calm Application (this is equivalent to the **Manage > App Delete** action).

Now that we're familiar with the application page layout, let's modify our application by adding additional memory, but let's do it in a way that we can recover from in case something goes wrong.

#. Click the **Snapshot** button in the upper right, and enter the following in the pop-up that appears: change the name to:

   - **Snapshot Name** - before-update-@@{calm_time}@@ (leave the rest as default)

   .. figure:: images/21_snapshot.png
       :align: center
       :alt: Application Snapshot

       Application Snapshot

#. Click **Save**.

#. Take note you're re-directed to the **Audit** tab.  Expand the **Snapshot Create** action to view the tasks of the snapshot.  Once complete, navigate to the **Recovery Points** tab, and validate that our new snapshot is listed.

#. Next, click the **Launch Console** button in the upper right, and log in to your VM.

   - **Username** - Administrator
   - **Password** - Nutanix/4u

#. To view the current memory on Windows, open a **Command Prompt**, and run **systeminfo | findstr Memory**.  Take note of the current memory allocated to your VM.

   .. figure:: images/23_windows_memory_before.png
       :align: center
       :alt: Windows Memory - Before Update

       Windows Memory - Before Update

#. Navigate back to the application page of Calm, and click the **Update** button in the upper right and then select the **Update VM Configuration** button.  On the page that appears, increase the **Memory (GiB)** field by 2 GiB (For Windows, 8 GiB).

#. Click the blue **Update** button in the lower right.

#. Validate that the memory field has been increased by 2 GiB, and click **Confirm**.

   .. figure:: images/25_windows_confirm.png
       :align: center
       :alt: Windows Memory - Confirm Change

       Windows Memory - Confirm Change

#. In the **Audit** tab of Calm, wait for the **App Update** action to complete.

#. Back in the **VM Console**, run the same command from earlier to view the updated memory, and note that it has increased by 2 GiB.

   .. figure:: images/27_windows_memory_after.png
       :align: center
       :alt: Windows Memory - After Update

       Windows Memory - After Update

   .. note::

      If anything went wrong with the VM Update, navigate to the **Recovery Points** tab, click **Restore** on the **before-update** snapshot we took earlier, and click **Confirm** on the pop-up.


Takeaways
+++++++++

What are the key things you should know about **Nutanix Calm** and **Single VM Blueprints**?

- Nutanix Calm provides application and infrastructure automation natively within Prism, turning complex, week long ticketing processes, into one-click self service provisioning.

- While Multi VM blueprints enable the provisioning and lifecycle management of complex, multi-tiered applications, Single VM blueprints allows IT to provide Infrastructure-as-a-Service for their end users.

- Common day 2 operations, like snapshotting, restoring, cloning, and updating the infrastructure can all be done by end users directly within Calm.

.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: ../images/blueprints.png
.. |applications| image:: ../images/blueprints.png
.. |projects| image:: ../images/projects.png
