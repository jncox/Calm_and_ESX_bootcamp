.. _calm_projects:

--------------
Calm: Projects
--------------

Overview
++++++++

.. note::

  Review :ref:`what_is_calm` before proceeding with the lab to familiarize yourself with the UI and common terminology used in Nutanix Calm.

  Estimated time to complete: **15 MINUTES**

In this exercise you will configure a Project to contain your Blueprints and Applications created throughout the Bootcamp.

Creating A Project
++++++++++++++++++

Projects are the logical construct that integrate Calm with Nutanix's native Self-Service Portal (SSP) capabilities, allowing an administrator to assign both infrastructure resources and the roles/permissions of Active Directory users/groups to specific Blueprints and Applications.

#. Within the Calm UI, Select |proj-icon| **Projects** from the sidebar.

   .. figure:: images/510projects1.png

#. Click + Create Project

#. Fill out the following fields:

   - **Project Name** - *initials*-Calm
   - **Description** - *initials*-Calm

#. Under **Users, Groups, and Roles**, click **+ Add Users**.

#. Fill out the following fields and click **Save**:

   - **Name** - SSP Admins
   - **Role** - Project Admin

#. Click **+ User**, fill out the following fields and click **Save**:

   - **Name** - SSP Developers
   - **Role** - Developer

#. Click **+ User**, fill out the following fields and click **Save**:

   - **Name** - SSP Power Users
   - **Role** - Consumer

#. Click **+ User**, fill out the following fields and click **Save**:

   - **Name** - SSP Basic Users
   - **Role** - Operator

   .. figure:: images/projects_name_users.png

#. Under **Accounts**, click the blue **Add Accounts** button. 

   .. figure:: images/AddAccounts.png

#. Then press **Select Account** and choose **vmware** from the drop down list.

   .. figure:: images/AddVMware.png

#. Click **Save**.

.. note::

  Click `here <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Calm-Admin-Operations-Guide-v56:nuc-roles-responsibility-matrix-c.html>`_ to view the complete matrix of default SSP roles and associated permissions.

Takeaways
+++++++++

- Nutanix Calm is a fully integrated component of the Nutanix stack. Easily enabled, highly available out of the box in a Scale Out Prism Central deployment, and takes advantage of non-disruptive One Click upgrades for new features and fixes.
- By using different projects assigned to different clusters and users, administrators can ensure that workloads are deployed the right way each time.  For example, a developer can be a Project Admin for a dev/test project, so they have full control to deploy to their development clusters or to a cloud, while having Read Only access to production projects, allowing them access to logs but no ability to alter production workloads.

.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
