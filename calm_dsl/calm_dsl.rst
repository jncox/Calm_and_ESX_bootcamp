.. _calm_dsl:

-----------------------------------------
Calm: DSL
-----------------------------------------

Overview
++++++++

To start the DSL lab we have provided a DevWorkStation blueprint to quickly get you started. The included DevWorkstation.json builds a CentOS VM with all the necessary tools.  This blueprint can be launched directly from Calm, but we recommend publishing it to the Calm Marketpkace for Self Service.

Deploy Dev Workstation from Marketplace
+++++++++++++++++++++++++++++++++++++++

Launch DevWorkstation from Calm Marketplace
...........................................

#. Select |mktmgr-icon| **Marketplace Manager** in the left hand toolbar to view and manage Marketplace Blueprints.

#. Enter your *initials* in the search bar, and you should see your blueprint listed.

#. Select your *intials*\ **_DevWorkStation** blueprint, and click **Launch** from the Marketplace.

#. Select your *initials*\ **-Project** from the **Projects** dropdown.

#. Review the following variables **Profile Configuration**:

    - **Name of the Application** - *intials*\ **_DevWorkStation**
    - **Prism Central IP** - *Provided Prism Central IP*
    - **Prism Central Password** - *PC Password*
    - **Calm Project** - *initials*\ **-Project**

    .. figure:: images/DevLaunch.png

#. Enter the following for **Credentials**:

    - **Username** - centos
    - **Password** - Nutanix/4u

    .. figure:: images/Creds.png

#. Click **Create**

#. While waiting review the audit log to see packages being deployed.

  .. note::

    The blueprint automatically installs several utilities along with Calm DSL

#. Once the application is **Running** make note of the IP Address for the next lab.

   .. note::

     The IP address of the DevWorkstation is listed under the application overview.

   .. figure:: images/IPaddress.png

Start Using Calm DSL
++++++++++++++++++++

Start the virtual environment and connect to Prism Central
..........................................................

#. Open a Console session or SSH to your *intials*\ **_DevWorkStation**

#. Change Directories by running ```cd calm-dsl``` from the home directory

#. Run ```source venv/bin/activate``` to switch to the virtual environment. This will enable the virtual environment for Calm DSL

   .. note::

     This has already been done through the blueprint launch, but once you SSH into the DevWorkstation you can setup the connection to Prism Central by running ```calm init dsl```

#. Verify the current config settings by running ```calm show config```

    .. figure:: images/Config.png

List the current blueprints in Calm
...................................

#. Run ```calm get bps``` and we see all the blueprints in Calm with their UUID, description, application count, project, and state

    .. figure:: images/getbps.png

#. Run ```calm get bps -q``` to display quiet output with only the BP names

    .. figure:: images/calmgetbpsq.png

Review and Modify a Blueprint
.............................

Now lets review a python based blueprint, and make a modification.

#. Change to the **HelloBlueprint** directory by running ```cd HelloBlueprint``` and run ```ls``` to list the contents of the directory.

    .. note::

      This directory and it's contents were automatically created during the blueprint launch.
      As part of the DevWorkstation blueprint launch we ran ```calm init bp``` which creates a sample blueprint configured to the connected Calm instance.

#. There is a file called "blueprint.py" which is a python version of a blueprint

#. There is a "scripts" directory. This is where the bash/powershell/python scripts are stored that are referenced within the blueprint

    .. figure:: images/hellols.png

Modify blueprint.py
===================

#. Run ```vi blueprint.py``` to edit the python file.

#. Review the blueprint for familiar constructs.  To skip directly to a line enter ```:<linenumber>```

    - Credentials (line 54-60)

    - OS Image (line 62-66)

    - Under class HelloPackage(Package) you will see references to the pkg\_install\_task.sh script in the scripts directory (line 139)

    - Basic VM spec information (vCPU/memory/disks/nics) (line 153-159)

    - Guest Customization contains cloud-init (line 161-171)

#. In the blueprint.py modify the number of vCPU

    - Change the vCPU from 2 to 4 (line 154)

      .. figure:: images/vcpu.png

#. Add a unique VM name using a macro (line 185)

    - ```provider_spec.name = "<Initials>-@@{calm_unique}@@"```

      .. figure:: images/vmname.png

#. Write/quit ```:wq``` the .py blueprint file to save and close

Modify pkg\_install\_task.sh
============================

#. Change to the scripts directory and run ```ls```. We will see 2 scripts that are being referenced inside blueprint.py

#. Run ```cat pkg_install_task.sh``` to view the current contents of the install script.  What does the script do?

    .. figure:: images/more1.png

#. Run ```curl -Sks https://raw.githubusercontent.com/nutanixworkshops/prep/master/nginx > pkg_install_task.sh``` to replace the existing install script

#. Run ```cat pkg_install_task.sh``` to view the changed script.  What does the script do now?

    .. figure:: images/more2.png

Push The Modified Blueprint To Calm
+++++++++++++++++++++++++++++++++++

#. Return to the "HelloBlueprint" directory

#. Run ```calm create bp --file blueprint.py --name FromDSL-<Initials>```

    .. note::

      This converts the .py file to json and pushes it to Calm

    .. figure:: images/syncbp.png

#. **Optional:** Run ```calm compile bp -f blueprint.py``` to view the python blueprint in json format from DSL

#. Verify your new blueprint by running ```calm get bps -q | grep FromDSL-<Initials>```

    .. figure:: images/verifygrep.png

Launch The Blueprint Into An Application
++++++++++++++++++++++++++++++++++++++++

#. Run ```calm get apps``` to verify all the current applications before launching your new app

#. We can also run ```calm get apps -q``` to quiet the details like we did with blueprints earlier

Launch Your Newly Uploaded Blueprint
....................................

#. Run ```calm launch bp FromDSL-<Initials> --app_name AppFromDSL-<Initials> -i```

    .. figure:: images/launchbp.png

#. Run ```calm describe app AppFromDSL-<Initials>``` to see the application summary

#. Once the app status changes to "running" we will have a nginx server deployed from Calm DSL!

    .. figure:: images/describe.png

#. Now we need to get the VM/Application IP address.  To get this we will pull the "address" from the application json output using jq by running the following:
```calm describe app AppFromDSL-<Initials> --out json | jq '.status.resources.deployment_list[].substrate_configuration.element_list[].address'```

    .. figure:: images/jqout.png

#. Enter the IP in a web browser and this will take you to the nginx **"Welcome to DSL"** web page

    .. figure:: images/welcome2.png

Log into Prism Central to Verify
.................................

#. Check the blueprint created from DSL

#. Check the application launched from DSL

Looking Back At What We Did
+++++++++++++++++++++++++++

As you went through this lab not only did you use Calm DSL, but you also used several native Linux tools such as vi, curl, grep, cat, pipe, and redirects.  Calm DSL allows extended felxibily by combining it with these powerful tools.

Think about how you can add git to this workflow to track changes or modify blueprints with sed


Takeaways
+++++++++

You have now edited a blueprint, sent it to Calm, launched an application, and used version control all from the command line using Calm-dsl.

.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: ../images/blueprints.png
.. |applications| image:: ../images/blueprints.png
.. |projects| image:: ../images/projects.png
