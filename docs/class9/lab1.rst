Lab 1: Managing BIG-IP Advanced WAF with  **Policy Supervisor**
===============================================================

**Policy Supervisor** is an online unified configuration solution for security policies, built with the purposes of managing and converting configuration across multiple F5 Web App Firewall solutions.
It enables operators of F5 WAF technologies to easily convert policy files from *BIG-IP AWAF*, *F5 Distributed Cloud WAF*, and *NGINX NAP* formats. In the process **Policy Supervisor** generates and uses an intermediate
JSON-based common declarative format called CDP (*Common Declarative Policy*) for policy lifecycle management. After a policy is converted to CDP, it can then be deployed to any supported WAF Solution, which is referred to as a *Provider* in **Policy Supervisor** lingo.

Please refer to the Tutorial in the GitHub repo (https://github.com/f5devcentral/ps-convert) for currently supported *Provider* types.

**Policy Supervisor** provides a graphical interface for visual policy creation, editing and management for traditional SecOps personas.

Task 1: Create a new **Policy Supervisor**  *Provider*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following steps will walk you through connecting **Policy Supervisor** to your *BIG-IP WAF*.

The first step is to create a *Provider*.

A *Provider* is a generic name used by **Policy Supervisor** to indicate an F5 Web App Firewall. The supported *Provider* types are: *F5 Distributed Cloud WAF*, *BIG-IP Advanced WAF* (AWAF), and *NGINX Application Protection* (NAP). Add and connect *providers* in **Policy Supervisor** to enable the deployment of your configuration policies across endpoints and load balancers for complete WAF protection.

When you add a BIG-IP instance as a *provider*, you must first set up an *agent* and associated secret on the private network to enable a secure connection between the BIG-IP instance and **Policy Supervisor**.

- The *agent* must be connected to the same private network where the *provider* is running to ensure a secure connection between **Policy Supervisor** and the *provider*.
- The *agent* machine must also have outbound Internet access for connectivity back to **Policy Supervisor**.
- The **Policy Supervisor** *Agent* is a Linux binary that is first installed on this machine/VM and is registered using a unique token generated in the **Policy Supervisor** UI for your **Policy Supervisor** *workspace* only.
- The *Agent* is used to create *Secrets*, which are stored in your environment only and are never transmitted outside of your network.
- These *secrets* are used to connect to your *BIG-IP AWAF* or *NGINX NAP* instance to execute various policy-related functions within a Docker container environment on that machine/VM.

.. note::
   **Prerequisites:**

   Installation of the **Policy Supervisor Agent** *requires the following applications to be installed on your Linux machine/VM:*

   - Docker
   - wget

Access the F5 **Policy Supervisor** console at https://policysupervisor.io as instructed in the previous *Introduction* section of this lab guide.

.. warning::

   **Policy Supervisor** uses the Microsoft Azure AD authentication service for login. You must have a valid Azure AD account to proceed with this lab.

1. On the *Overview > Providers* page, click **Add Provider**. If this is the first provider being added,
   there are two **Add Provider** buttons on the screen. The *Add Providers* pane will appears.

+----------------------------------------------+
| |lab001|                                     |
+----------------------------------------------+

2. There are no *agents* configure yet. Choose **BIG-IP** for the *Provider Type* and click
   **+ Add new agent** that will appear below the *Select Agent* drowpdown after a *Provider Type* has been
   selected. The *Add Agent* pane will appear and a token will be automatically generated as a long text string.

+----------------------------------------------+
| |lab002|                                     |
+----------------------------------------------+

3. *Copy & paste* (save) the value of the **Token** to a text file or notepad.
   (This token will be required in *Task 2* below.)

+----------------------------------------------+
| |lab003|                                     |
+----------------------------------------------+

4. From within the *Add Agent* pane, locate and click the link to go to the **agent-install** page (step 1.).
   The corresponding GitLab *repository page* will open.

+----------------------------------------------+
| |lab004|                                     |
+----------------------------------------------+

5. At the bottom of the *Package Registry* page, **right-click** on the **agent-installer** file name and
   select **Copy Link**. (This URL will be required in *Task 2* below.)

.. note:: *The URL for the agent-installer file changes from time to time when it is updated.*

Task 2: Install a **Policy Supervisor Agent**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, we will use the **token** and the **URL** obtained in task 1 above to install the *Agent* on your UDF virtual lab environment.
For this lab, the *Agent* must be installed on your *SuperJumpHost* Linux machine, which is connected to the same management network as your BIG-IP.
The *SuperJumpHost* is pre-configured in your lab environment with permission to communicate with the **Policy Supervisor** across the Internet.

1. Browse to your lab session at https://udf.f5.com again and find the **Deployment** tab to see your virtual machines.

+----------------------------------------------+
| |lab006|                                     |
+----------------------------------------------+

2. Find the **SuperJumpHost** system and click its **ACCESS** link to see a list of access options.

+----------------------------------------------+
| |lab007|                                     |
+----------------------------------------------+

3. Select **Web Sell** to access the **SuperJumpHost** machine's command line interface in a new browser tab.
   *(You will be automatically logged in as root.)*

+----------------------------------------------+
| |lab008|                                     |
+----------------------------------------------+

4. Set your working directory to */tmp* with the **"cd /tmp"** linux command.

.. code-block:: bash

   cd /tmp

5. Use the URL copied at *step 5* above to download the installer via the command line:
   **"wget <...insert URL from above Task 1 here...>"**

.. code-block:: bash

   wget <...insert URL here...>

6. After the download completes, rename the file with this linux command:
   **"mv download agent-installer"**

.. code-block:: bash

   mv download agent-installer


7. Next, give the installer package execution rights to enable it to run:
   **"chmod +x ./agent-installer"**

.. code-block:: bash

   chmod +x ./agent-installer

8. Run the agent installer by using the following command:
   **"./agent-installer"**

.. code-block:: bash
   
   ./agent-installer

+----------------------------------------------+
| |lab009|                                     |
+----------------------------------------------+

9. Wait for the *"Enter agent token"* prompt and paste the token copied from *Task 1* above.
   *(command-V on a MAC, Ctrl-Shift-V on Windows)*
   
+----------------------------------------------+
| .. image:: _static/PSAgentToken.png          |
|    :width: 800px                             |
+----------------------------------------------+

10. Paste the value of the Token obtained in *Task 1* above.

+----------------------------------------------+
| |lab010|                                     |
+----------------------------------------------+

11. Enter the name **"udf"** when prompted for the agent name.
    Wait for registration to complete successfully (takes a few minutes). You will be prompted to *"Enter secret name"*.

+----------------------------------------------+
| |lab011|                                     |
+----------------------------------------------+

12. Select **Add Secret** and/or type **"bigip"** when prompted for the secret name.
    *If the secret already exists, you must first select **Remove Secret** and delete it before attempting
    to add it again.*

13. Type **"admin"** when prompted for the username.

14. Type **"Canada123!"** when prompted for a password.

15. Press "**Enter**" when prompted for the *ssh key path* (we're not using one in this demo).

16. Press "**Enter**" when prompted to select an option (choose the default "*Finish*" option).

Task 3: Finish adding a first *provider* in **Policy Supervisor**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The configuration of the new *Provider* can be completed now that the *Agent* is ready.

1. Go to https://policysupervisor.io again and click **Done** (return to the *Add Provider Pane* with *BIG-IP*
selected for the *Provider Type*).

+----------------------------------------------+
| .. image:: _static/PSAddProvider.png         |
|    :width: 800px                             |
+----------------------------------------------+

2. Select the new **udf** option that should now be visible on the dropdown list for the *Agent* field
(the provider that was created in the previous task).

3. Choose the new **bigip** option that should now be visible on the drop-down list for the *Secrets* field
(the secret that was created in the previous task) and click **Continue**.

4. The **Provider Name** and **Provider URL** fields will now appear.

5. Type **"bigip1"** for the *Provider Name** and type **"https://10.1.1.6"** for the **Provider URL** as shown above.

6. Click the **Test Connection** button and wait for the tests to complete successfully.

+----------------------------------------------+
| .. image:: _static/PSProviderTestConnection.png                       |
|    :width: 800px                             |
+----------------------------------------------+

Task 4: Add a 2nd BIG-IP *provider* in **Policy Supervisor**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We will re-use the same **udf** *Agent* and **bigip** *Secret* created in Task 2 above to manage the WAF policies on
your 2nd BIG-IP because they are connected to the same management network in your UDF virtual lab environment.

1. Click the **Add another Provider** button to add the second BIG-IP appliance in your virtual lab environment.

+----------------------------------------------+
| .. image:: _static/PSAddProvider2.png        |
|    :width: 800px                             |
+----------------------------------------------+

2. Select the **BIG-IP** option for the provider type.

3. Select the **udf** option for **Agent**.

4. Select the **bigip** option for **Secret** *(the two BIG-IP's have been configured with the same password)*.

5. Click **Continue**.

The **Provider Name** and **Provider URL** fields will now appear.

6. Type **"bigip2"** for the **Provider Name** and type **"https://10.1.1.7"** for the **Provider URL**.

7. Click the **Test Connection** button and wait for the tests to complete successfully.

+----------------------------------------------+
| .. image:: _static/PSProviderTestConnection.png                       |
|    :width: 800px                             |
+----------------------------------------------+

8. Click the **Go to overview** link.

+----------------------------------------------+
| .. image:: _static/PSProviderList.png        |
|    :width: 800px                             |
+----------------------------------------------+

You now have two BIG-IP providers configured in **Policy Supervisor**.

Task 5: Ingest an existing BIG-IP WAF policy in **Policy Supervisor**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BIG-IP1 is already configured with a WAF policy attached to the **web_app** virtual server.
Let's ingest this WAF policy into **Policy Supervisor**.

1. Start from the **Providers Overview** page.

+----------------------------------------------+
| .. image:: _static/PSBIGIPProvider.png       |
|    :width: 800px                             |
+----------------------------------------------+

2. Click to select **bigip1**, then click **Ingest Policies**.

+----------------------------------------------+
| .. image:: _static/PSIngest.png              |
|    :width: 800px                             |
+----------------------------------------------+

3. Select the discovered policy (i.e., **My_ASM_Rapid…**) and click **Continue**.

+----------------------------------------------+
| .. image:: _static/PSIngest2.png             |
|    :width: 800px                             |
+----------------------------------------------+

4. Click **Next**.

+----------------------------------------------+
| .. image:: _static/PSIngest2b.png            |
|    :width: 800px                             |
+----------------------------------------------+

5. Type **"Ingest from bigip1"** for the required **commit message**,

6. click **Save & Ingest Policy**, then wait for the ingestion to complete successfully.

+----------------------------------------------+
| .. image:: _static/PSIngest3.png             |
|    :width: 800px                             |
+----------------------------------------------+
| .. image:: _static/PSIngest4.png             |
|    :width: 800px                             |
+----------------------------------------------+

Task 6 (*optional*): Import an existing BIG-IP WAF policy in **Policy Supervisor**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

F5 WAF policies can be *imported* instead of *ingested*. This option is useful when the installation of 
a **Policy Supervisor** agent is not possible or when the BIG-IP appliance cannot be configured or managed as a *Provider*.

1. Browse to https://udf.f5.com again and find the **Deployment** tab to see your virtual machines.

+----------------------------------------------+
| |lab006|                                     |
+----------------------------------------------+

2. Find **bigip1** under F5 Products and click its **ACCESS** link to see a list of access options.

+----------------------------------------------+
| .. image:: _static/UDFTMUI1.png              |
|    :width: 800px                             |
+----------------------------------------------+

3. Select the **TMUI** option to opoen **bigip1**'s GUI management interface in a new browser tab.

+----------------------------------------------+
| .. image:: _static/TMUILogin.png             |
|    :width: 800px                             |
+----------------------------------------------+

4. Login with username **"admin"** and password **"Canada123!"**.

+----------------------------------------------+
| .. image:: _static/TMUIVS.png                |
|    :width: 800px                             |
+----------------------------------------------+

5. Click to the **"Security -> Application Security -> Security Policies -> Policies List"** page.

+----------------------------------------------+
| .. image:: _static/BIGIPPolicyList.png       |
|    :width: 800px                             |
+----------------------------------------------+

6. Click on your policy's name (**My_ASM_Rapid_Deployment_Policy**).

+----------------------------------------------+
| .. image:: _static/BIGIPExport.png           |
|    :width: 800px                             |
+----------------------------------------------+

7. Click the **EXPORT** button and select the **JSON Format** option.

+----------------------------------------------+
| .. image:: _static/BIGIPExport2.png          |
|    :width: 800px                             |
+----------------------------------------------+

8. Click the **OK** button and wait a few momemts for the export process to complete.

+----------------------------------------------+
| .. image:: _static/BIGIPExport3.png          |
|    :width: 800px                             |
+----------------------------------------------+

9. If prompted, click **Allow** to complete the download of the exported policy to your workstation.
   The resulting JSON file should now be in your *Downloads* folder.

10. Browse back to the **Policy Supervisor** *Policy Overview* page (*https://policysupervisor.io/).

+----------------------------------------------+
| .. image:: _static/PSImport1.png             |
|    :width: 800px                             |
+----------------------------------------------+

11. Click the **Add** button and select the **Import from File** option.

+----------------------------------------------+
| .. image:: _static/PSImport2.png             |
|    :width: 800px                             |
+----------------------------------------------+

12. Enter a name in the **Policy Name** text box (for example: *bigip1 waf imported policy*).

13. Select the **BIG-IP** option form the *Policy Type* dropdown list.

14. Click the **Upload** button, then locate and select the previously downloaded JSON file.

15. Enter a note in the **Import Notes / Summary** text box.

16. Click the **Import** button.

+----------------------------------------------+
| .. image:: _static/PSImport3.png             |
|    :width: 800px                             |
+----------------------------------------------+

17. Wait for the import process to complete.

+----------------------------------------------+
| .. image:: _static/PSImport4.png             |
|    :width: 800px                             |
+----------------------------------------------+

18. Click the **Go to Overview** button.

+----------------------------------------------+
| .. image:: _static/PSImport5.png             |
|    :width: 800px                             |
+----------------------------------------------+

The imported WAF policy will be listed on the *Policies Overview* page as shown in
the screenshot image above, which shows two WAF policies: one that was just imported
in the steps above and the other was previously imported using the *Ingest* method.

Task 7: Deploy a WAF policy to a BIG-IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. In **Policy Supervisor**, browse to the **Policies Overview** page.

+----------------------------------------------+
| .. image:: _static/PSDeploy1.png             |
|    :width: 800px                             |
+----------------------------------------------+
| .. image:: _static/PSDeploy2.png             |
|    :width: 800px                             |
+----------------------------------------------+

2. Select a policy then find and click on the **Deploy** button.

+----------------------------------------------+
| .. image:: _static/PSDeploy3.png             |
|    :width: 800px                             |
+----------------------------------------------+

3. Select **bigip2** option from the **Provider** options and type **"Deploy to bigip2"** in the mandatory commit
   message text box and click the **Conversion Summary** button.

+----------------------------------------------+
| .. image:: _static/PSDeploy4.png             |
|    :width: 800px                             |
+----------------------------------------------+

4. Wait for the Conversion Summary screen to appear.

+----------------------------------------------+
| .. image:: _static/PSDeploy5.png             |
|    :width: 800px                             |
+----------------------------------------------+

5. Click the **Save & Continue** button.

+----------------------------------------------+
| .. image:: _static/PSDeploy6.png             |
|    :width: 800px                             |
+----------------------------------------------+

6. Click the **Continue Deployment** button on the *Conversion Report* screen that appears.

+----------------------------------------------+
| .. image:: _static/PSDeploy7.png             |
|    :width: 800px                             |
+----------------------------------------------+

7. Select the **web_app** virtual server from the dropdown list and click the **Next** button.

+----------------------------------------------+
| .. image:: _static/PSDeploy7b.png            |
|    :width: 800px                             |
+----------------------------------------------+

8. Click the **Deploy** button.

+----------------------------------------------+
| .. image:: _static/PSDeploy8.png             |
|    :width: 800px                             |
+----------------------------------------------+
| .. image:: _static/PSDeploy9.png             |
|    :width: 800px                             |
+----------------------------------------------+

9. Wait for the deployment to successfully complete. and click the **Back to Overview** button.

+----------------------------------------------+
| .. image:: _static/PSImport5.png             |
|    :width: 800px                             |
+----------------------------------------------+

Task 8: Confirm successful deployment of the WAF policy on BIG-IP2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. NOTE:: The password for the admin account on your BIG-IP appliances is set to **Canada123!**


1. Browse to https://udf.f5.com again and find the **Deployment** tab to see your virtual machines.

+----------------------------------------------+
| |lab006|                                     |
+----------------------------------------------+

2. Find **bigip2** under F5 Products and click its **ACCESS** link to see a list of access options.

+----------------------------------------------+
| .. image:: _static/UDFTMUI2.png              |
|    :width: 800px                             |
+----------------------------------------------+

3. Select the **TMUI** option to opoen **bigip2**'s GUI management interface in a new browser tab.

+----------------------------------------------+
| .. image:: _static/TMUILogin.png             |
|    :width: 800px                             |
+----------------------------------------------+

4. Login with username **"admin"** and password **"Canada123!"**.

+----------------------------------------------+
| .. image:: _static/TMUIVS.png                |
|    :width: 800px                             |
+----------------------------------------------+

5. Browse to the virtual servers list page.

+----------------------------------------------+
| .. image:: _static/TMUIVS2.png               |
|    :width: 800px                             |
+----------------------------------------------+

6. Click on the **web_app** name to view the virtual sever's properties page.

+----------------------------------------------+
| .. image:: _static/TMUIVS3.png               |
|    :width: 800px                             |
+----------------------------------------------+

7. Browse to the virtual sever's **Security -> Policies** page.

+----------------------------------------------+
| .. image:: _static/TMUIVS4.png               |
|    :width: 800px                             |
+----------------------------------------------+

8. Observe that the Application Security Policy (e.g., the WAF policy) is **Enabled**.

**WELL DONE!!!**

In the next lab we will deploy a WAF policy ingested from a BIG-IP appliance to an F5 Distributed Cloud WAF.

+------------+
| |labbgn|   |
+------------+

.. |lab001| image:: _static/image9.png
   :width: 800px
.. |lab002| image:: _static/image17.png
   :width: 800px
.. |lab003| image:: _static/image18.png
   :width: 800px
.. |lab004| image:: _static/image19.png
   :width: 800px
.. |lab006| image:: _static/UDFDeploymentTab.png
   :width: 800px
.. |lab007| image:: _static/UDFWebShell.png
   :width: 800px
.. |lab008| image:: _static/UDFWebShellCLI.png
   :width: 800px
.. |lab009| image:: _static/install_agent.png
   :width: 800px
.. |lab010| image:: _static/agentsetup.png
   :width: 800px
.. |lab011| image:: _static/agentsecret.png
   :width: 800px
.. |labbgn| image:: _static/labbgn.png
   :width: 800px
