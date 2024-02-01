Lab 3: Leveraging Terraform
===========================

The following lab tasks will guide you through using Terraform to deploy and secure a Web based application.  
Students will start by creating an authentication certificate within Distributed Cloud that Terraform utilizes
for authenticating the API calls.  Next, a Tfvars file is created to customize the deployment to match the 
student's environment. Terraform will then be used to deploy an HTTP Health Check, Origin Pool, and HTTP Load 
Balancer. Students will then modify and apply the Terraform configuration to add a Web Application Firewall 
to their existing HTTP Load Balancer. Finally, Terraform will be used to tear down everything it created in 
this lab.

**Expected Lab Time: 20 minutes**

Task 1: Deploy a Web Application with Terraform  
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In this task, you will create an API Certificate for Terraform to authneticate to the Distributed Cloud API.  Next, 
you will create a Tfvars file to specify environment variables unique to your environment.  After the Tfvars file is 
created, you will intialize Terraform and then deploy an HTTP Health Check, Origin Pool, and HTTP Load Balancer. 

This lab will begin back in the Windows 10 client deployed as part of the UDF.

+---------------------------------------------------------------------------------------------------------------+
| **Create API Certificate from the Distributed Cloud Console**                                                 |
+===============================================================================================================+
| 1. From the Windows 10 client deployed as part of the UDF, open Chrome.                                       |
|                                                                                                               |
| |lab1-Chrome|                                                                                                 |
+---------------------------------------------------------------------------------------------------------------+
| 2. Click on the **XC Console** bookmark to be taken to the XC Console login.                                  |
|                                                                                                               |
| |lab1-XC_Bookmark|                                                                                            |
+---------------------------------------------------------------------------------------------------------------+
| 3. Enter your e-mail address in the **Email** form and password in the **Password** form and click **Sign**   |
|                                                                                                               |
|    **In**.                                                                                                    |
|                                                                                                               |
| |lab1-XC_Signin|                                                                                              |
+---------------------------------------------------------------------------------------------------------------+
| 4. In the top right corner of the Distributed Cloud Console click the **User Icon** dropdown and select       |
|                                                                                                               |
|    **Account Settings**.                                                                                      |
|                                                                                                               |
| |lab1-Account_Settings|                                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| 5. In the resulting screen click **Credentials** under the Peronal Management Heading on the left.            |
|                                                                                                               |
| |lab1-Credentials|                                                                                            |
+---------------------------------------------------------------------------------------------------------------+
| 6. Click **Add Credentials**.                                                                                 |
|                                                                                                               |
| |lab1-Add_Credentials|                                                                                        |
+---------------------------------------------------------------------------------------------------------------+
| 7. Fill in the resulting form with the following values:                                                      |
|                                                                                                               |
|    * **Credential Name ID:**  *<namespace>-api-cert*                                                          |
|    * **Credential Type: Select:** *API Certificate*                                                           |
|    * **Password:** *<some_password>*                                                                          |
|    * **Confirm Password:** *<some_password>*                                                                  |
|    * **Expiry Date: Select:** *<date two days in the future of today's date>*                                 |
|                                                                                                               |
| 8. Click **Download**.                                                                                        |
|                                                                                                               |
| |lab3-Terraform_Download_API_Cert|                                                                            |
|                                                                                                               |
| .. note::                                                                                                     |
|    *Use a password that you will remember for the certificate, if you don't remember your API cert password,* |
|    *you will need to generate a new API cert.*                                                                |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Set Windows Environment Variables for Terraform to Utilize**                                                |
+===============================================================================================================+
| 9. Minimize the Chrome Browser and double click the **Command Prompt** icon on the Windows 10 desktop.        |
|                                                                                                               |
| |lab3-Terraform_Cmd_Prompt|                                                                                   |
+---------------------------------------------------------------------------------------------------------------+
| 10. Copy the certificate you downloaded to the labuser home folder using the command:                         |
|                                                                                                               |
| .. literalinclude:: lab3-copy.txt                                                                             |
|                                                                                                               |
| |lab3-Terraform_Cert_Copy|                                                                                    |
+---------------------------------------------------------------------------------------------------------------+
| 11. Set an environment variable for the API certificate password with the following command:                  |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    setx VES_P12_PASSWORD "<some_password>"                                                                    |
|                                                                                                               |
| |lab3-Terraform_Cert_Password|                                                                                |
+---------------------------------------------------------------------------------------------------------------+
| 12. Close the command prompt window.                                                                          |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Open the Pre-Created Terraform Code in Visual Studio Code**                                                 |
+===============================================================================================================+
| 13. Double click the **Visual Studio Code** icon on the desktop to launch **Visual Studio Code**.             |
|                                                                                                               |
| |lab3-Terraform_VSC|                                                                                          |
+---------------------------------------------------------------------------------------------------------------+
| 14. When Visual Studio Code launches, click **File** and then **Open Folder...**.                             |
|                                                                                                               |
| |lab3-Terraform_VSC_Folder|                                                                                   |
+---------------------------------------------------------------------------------------------------------------+
| 15. In the resulting window, paste the below text into the location bar, click the arrow to open that         |
|                                                                                                               |
|     location, and then click **Select Folder**.                                                               |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    c:\Users\labuser\appworld-f5xc-automation\Terraform                                                        |
|                                                                                                               |
| |lab3-Terraform_VSC_Folder_Select|                                                                            |
|                                                                                                               |
| .. note::                                                                                                     |
|    *You may see a pop up window that says "Do you trust the authors of the files in this folder?" If you see* |
|    *this pop up, click "Yes, I trust the authors"*                                                            |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Create a tfvars File for Specifying Environment Specific Variables**                                        |
+===============================================================================================================+
| 16. From the **EXPLORER** frame, click the new file icon next to the TERRAFORM folder, and then enter the name|
|                                                                                                               |
|     **terraform.tfvars** for the new file that is created and press **Enter**.                                | 
|                                                                                                               |
| |lab3-Terraform_VSC_Tfvars|                                                                                   |
+---------------------------------------------------------------------------------------------------------------+
| 17. This will open the **terraform.tfvars** file in the right frame of Visual Studio Code, enter the following|
|                                                                                                               |
|     values into the file:                                                                                     |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    api_p12     = "c:/Users/labuser/xc-api-cert.p12"                                                           |
|    tenant_name = "f5-xc-lab-app"                                                                              |
|    namespace   = "<namespace>"                                                                                |
|                                                                                                               |
| |lab3-Terraform_VSC_Tfvars_Values|                                                                            |
+---------------------------------------------------------------------------------------------------------------+
| 18. Click **File** and **Save** to save the changes you made to the file.                                     |
|                                                                                                               |
| |lab3-Terraform_VSC_Tfvars_Save|                                                                              |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Initialize, Plan, and Apply Your Terraform Code**                                                           |
+===============================================================================================================+
| 19. From the Visual Studio Code menu bar, click **View**, and then click **Terminal**.                        |
|                                                                                                               |
| |lab3-Terraform_VSC_Terminal|                                                                                 |
+---------------------------------------------------------------------------------------------------------------+
| 20. In the Terminal at the bottom of Visual Studio Code, enter the following command and press Enter:         |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform init                                                                                             |
|                                                                                                               |
| |lab3-Terraform_VSC_Init|                                                                                     |
+---------------------------------------------------------------------------------------------------------------+
| 21. Review the Init Results. You should see a **Terraform has been successfully initialized!** message.       |
|                                                                                                               |
|     **DO NOT PROCEED AND ASK A LAB ASSISTANT FOR HELP IF YOU DON'T SEE THE SUCCESSFULLY INITIALIZED MESSAGE.**|
|                                                                                                               |
| |lab3-Terraform_VSC_Init_Success|                                                                             |
+---------------------------------------------------------------------------------------------------------------+
| 22. In the Terminal, enter the following command and press Enter:                                             |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform plan                                                                                             |
|                                                                                                               |
| |lab3-Terraform_VSC_Plan|                                                                                     |
+---------------------------------------------------------------------------------------------------------------+
| 23. Review the Plan results. This shows what Terraform is planning to create.                                 |
|                                                                                                               |
| |lab3-Terraform_VSC_Plan_Results|                                                                             |
+---------------------------------------------------------------------------------------------------------------+
| 24. In the Terminal, enter the following command and press Enter:                                             |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform apply                                                                                            |
|                                                                                                               |
| |lab3-Terraform_VSC_Apply|                                                                                    |
+---------------------------------------------------------------------------------------------------------------+
| 25. When prompted **Do you want to perform these actions?**, type **yes** and press Enter.                    |
|                                                                                                               |
| |lab3-Terraform_VSC_Apply_Yes|                                                                                |
+---------------------------------------------------------------------------------------------------------------+
| 26. Review the Apply results. This shows what Terraform created.                                              |
|                                                                                                               |
| |lab3-Terraform_VSC_Apply_Results|                                                                            |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Verify the Demo Shop App is Accessible Via a Web Browser**                                                  |
+===============================================================================================================+
| 27. Open a new tab in your Chrome browser and enter the following URL                                         |
|                                                                                                               |
|     **http://<namespace>-demoshop.lab-app.f5demos.com**                                                       |
|                                                                                                               |
| .. note::                                                                                                     |
|    *This illustrates that you are able to configure the delivery of an application via the Distributed Cloud* |
|    *API utilizing Terraform.*                                                                                 |
+---------------------------------------------------------------------------------------------------------------+
| |lab1-Demoshop|                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

Task 2: Create & Attach WAF Policy 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In this task, you will modify your Terraform configuration to create and apply an Application Firewall policy with
the default settings. Since Terraform tracks state, the apply command is used to modify the required existing 
objects within Distributed Cloud.

+---------------------------------------------------------------------------------------------------------------+
| **Edit Your Terraform Code to Create an Application Firewall and Add It to the Load Balancer**                |
+===============================================================================================================+
| 1. From the Visual Studio Code Explorer frame, click **main.tf**, to open the Terraform configuration.        |
|                                                                                                               |
| |lab3-Terraform_VSC_Main|                                                                                     |
+---------------------------------------------------------------------------------------------------------------+
| 2. Scroll down to the bottom of the configuration and paste in the following lines to create the Web          |
|                                                                                                               |
|    Application Firewall policy.                                                                               |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    # Create WAF Policy                                                                                        |
|    resource "volterra_app_firewall" "waf" {                                                                   |
|      name = "${var.namespace}-appfw"                                                                          |
|      namespace = var.namespace                                                                                |
|      allow_all_response_codes = true                                                                          |
|      default_anonymization = true                                                                             |
|      use_default_blocking_page = true                                                                         |
|      default_bot_setting = true                                                                               |
|      default_detection_settings = true                                                                        |
|      use_loadbalancer_setting = true                                                                          |
|      blocking = true                                                                                          |
|    }                                                                                                          |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Create|                                                                             |
+---------------------------------------------------------------------------------------------------------------+
| 3. Locate the **Create Load Balancer** configuration within **main.tf** and replace the **disable_waf = true**|
|                                                                                                               |
|    line with the following configuration:                                                                     |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    # WAF Config                                                                                               |
|    app_firewall {                                                                                             |
|      name = volterra_app_firewall.waf.name                                                                    |
|      namespace = var.namespace                                                                                |
|    }                                                                                                          |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_LB_Disable|                                                                         |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_LB_Config|                                                                          |
|                                                                                                               |
| .. note::                                                                                                     |
|    *The WAF Config should be indented two spaces under the Load Balancer configuration to maintain nesting*   |
|    *style conventions.*                                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| 4. Click **File** and **Save** to save the changes you made to **main.tf**.                                   |
|                                                                                                               |
| |lab3-Terraform_VSC_Main_Save|                                                                                |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Plan and Apply Your New Terraform Code to Create an Application Firewall and Associate It to Your LB**      |
+===============================================================================================================+
| 5. In the Terminal, enter the following command and press Enter:                                              |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform plan                                                                                             |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Plan|                                                                               |
+---------------------------------------------------------------------------------------------------------------+
| 6. Review the Plan results. This shows what Terraform is planning to create.                                  |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Plan_Results|                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| 7. In the Terminal, enter the following command and press Enter:                                              |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform apply                                                                                            |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Apply|                                                                              |
+---------------------------------------------------------------------------------------------------------------+
| 8. When prompted **Do you want to perform these actions?**, type **yes** and press Enter.                     |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Apply_Yes|                                                                          |
+---------------------------------------------------------------------------------------------------------------+
| 9. Review the Apply results. This shows what Terraform created.                                               |
|                                                                                                               |
| |lab3-Terraform_VSC_Appfw_Apply_Results|                                                                      |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **Verify the Application Firewall was Created and Applied Within the Distributed Cloud Console**              |
+===============================================================================================================+
| 10. Switch back to the Chrome Browser that is connected to the Distributed Cloud Console.                     |
+---------------------------------------------------------------------------------------------------------------+
| 11. Within the Distributed Cloud dashboard, select the **Multi-Cloud App Connect** tile.                      |
|                                                                                                               |
| |lab1-XC_App_Connect|                                                                                         |
+---------------------------------------------------------------------------------------------------------------+
| 12. In the resulting screen, expand the **Manage** menu and click **Load Balancers** and then select          |
|                                                                                                               |
|     **HTTP Load Balancers**.                                                                                  |
|                                                                                                               |
| |lab1-XC_LB|                                                                                                  |
+---------------------------------------------------------------------------------------------------------------+
| 13. From the HTTP Load Balancers page, locate the HTTP Load Balancer that you created via Terraform.  Click   |
|                                                                                                               |
|     the **ellipsis** under **Actions** and select **Manage Configuration**.                                   |
|                                                                                                               |
| |lab1-XC_LB_Manage|                                                                                           |
+---------------------------------------------------------------------------------------------------------------+
| 14. From the resulting screen, select **Web Application Firewall** under the HTTP Load Balancer frame to jump |
|                                                                                                               |
|     to the **Web Application Firewall** configuration section.                                                |
|                                                                                                               |
| |lab3-XC_Terraform_WAF|                                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| 15. Notice that the Web Application Firewall is now Enabled and the policy you created using Terraform is     |
|                                                                                                               |
|     applied.                                                                                                  |
|                                                                                                               |
| |lab3-XC_Terraform_WAF_Enable|                                                                                |
+---------------------------------------------------------------------------------------------------------------+
| 16. Click Cancel and Exit to close out of the HTTP Load Balancer configuration.                               |
|                                                                                                               |
| |lab3-XC_Terraform_WAF_Cancel|                                                                                |
+---------------------------------------------------------------------------------------------------------------+

Task 3: Destroy the Terraform Objects 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In this task, you will use Terraform to destroy the HTTP Health Check, Origin Pool, HTTP Load Balancer, and Web 
Application Firewall Policy that was created in Tasks 1 & 2.

+---------------------------------------------------------------------------------------------------------------+
| **Delete Distributed Cloud Objects Utilizing Terraform Destroy**                                              |
+===============================================================================================================+
| 1. Switch back to the Visual Studio Code application.                                                         |
+---------------------------------------------------------------------------------------------------------------+
| 2. In the Visual Studio Code Terminal, enter the following command and press Enter:                           |
|                                                                                                               |
| .. code-block:: bash                                                                                          |
|                                                                                                               |
|    terraform destroy                                                                                          |
|                                                                                                               |
| |lab3-Terraform_VSC_Destroy|                                                                                  |
+---------------------------------------------------------------------------------------------------------------+
| 3. When prompted **Do you really want to destroy all resources?** type **yes** and press Enter.               |
|                                                                                                               |
| |lab3-Terraform_VSC_Destroy_Yes|                                                                              |
+---------------------------------------------------------------------------------------------------------------+
| 4. Review the Destroy results. This shows what Terraform deleted.                                             |
|                                                                                                               |
| |lab3-Terraform_VSC_Destroy_Results|                                                                          |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| **End of Lab 3**                                                                                              |
+===============================================================================================================+
| This concludes Lab 3. In this lab, you learned how to setup Terraform to authenticate to to Distributed Cloud |
|                                                                                                               |
| utilizing an API Certificate. You then created a Tfvars file to customize the deployment to match your        |
|                                                                                                               |
| environment. After that, you used Terraform to deploy an HTTP Health Check, Origin Pool, and HTTP Load        |
|                                                                                                               |
| Balancer. The Terraform configuration was then modified to create a Web Application Firewall policy and apply |
|                                                                                                               |
| it to the HTTP Load Balancer. Finally, Terraform was used to destroy all of the objects created in this lab. A|
|                                                                                                               |
| brief presentation and demo will be shared prior to the conclusion of this class.                             |
+---------------------------------------------------------------------------------------------------------------+
| |labend|                                                                                                      |
+---------------------------------------------------------------------------------------------------------------+

.. |lab1-Chrome| image:: _static/lab1-Chrome.png
   :width: 800px
.. |lab1-XC_Bookmark| image:: _static/lab1-XC_Bookmark.png
   :width: 800px
.. |lab1-XC_Signin| image:: _static/lab1-XC_Signin.png
   :width: 800px
.. |lab1-Account_Settings| image:: _static/lab1-Account_Settings.png
   :width: 800px
.. |lab1-Credentials| image:: _static/lab1-Credentials.png
   :width: 800px
.. |lab1-Add_Credentials| image:: _static/lab1-Add_Credentials.png
   :width: 800px
.. |lab3-Terraform_Download_API_Cert| image:: _static/lab3-Terraform_Download_API_Cert.png
   :width: 800px
.. |lab3-Terraform_Cmd_Prompt| image:: _static/lab3-Terraform_Cmd_Prompt.png
   :width: 800px
.. |lab3-Terraform_Cert_Copy| image:: _static/lab3-Terraform_Cert_Copy.png
   :width: 800px
.. |lab3-Terraform_Cert_Password| image:: _static/lab3-Terraform_Cert_Password.png
   :width: 800px
.. |lab3-Terraform_VSC| image:: _static/lab3-Terraform_VSC.png
   :width: 800px
.. |lab3-Terraform_VSC_Folder| image:: _static/lab3-Terraform_VSC_Folder.png
   :width: 800px
.. |lab3-Terraform_VSC_Folder_Select| image:: _static/lab3-Terraform_VSC_Folder_Select.png
   :width: 800px
.. |lab3-Terraform_VSC_Tfvars| image:: _static/lab3-Terraform_VSC_tfvars.png
   :width: 800px
.. |lab3-Terraform_VSC_Tfvars_Values| image:: _static/lab3-Terraform_VSC_tfvars_values.png
   :width: 800px
.. |lab3-Terraform_VSC_Tfvars_Save| image:: _static/lab3-Terraform_VSC_tfvars_save.png
   :width: 800px
.. |lab3-Terraform_VSC_Terminal| image:: _static/lab3-Terraform_VSC_Terminal.png
   :width: 800px
.. |lab3-Terraform_VSC_Init| image:: _static/lab3-Terraform_VSC_Init.png
   :width: 800px
.. |lab3-Terraform_VSC_Init_Success| image:: _static/lab3-Terraform_VSC_Init_Success.png
   :width: 800px
.. |lab3-Terraform_VSC_Plan| image:: _static/lab3-Terraform_VSC_Plan.png
   :width: 800px
.. |lab3-Terraform_VSC_Plan_Results| image:: _static/lab3-Terraform_VSC_Plan_Results.png
   :width: 800px
.. |lab3-Terraform_VSC_Apply| image:: _static/lab3-Terraform_VSC_Apply.png
   :width: 800px
.. |lab3-Terraform_VSC_Apply_Yes| image:: _static/lab3-Terraform_VSC_Apply_Yes.png
   :width: 800px
.. |lab3-Terraform_VSC_Apply_Results| image:: _static/lab3-Terraform_VSC_Apply_Results.png
   :width: 800px
.. |lab1-Demoshop| image:: _static/lab1-Demoshop.png
   :width: 800px
.. |lab3-Terraform_VSC_Main| image:: _static/lab3-Terraform_VSC_Main.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Create| image:: _static/lab3-Terraform_VSC_Appfw_Create.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_LB_Disable| image:: _static/lab3-Terraform_VSC_Appfw_LB_Disable.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_LB_Config| image:: _static/lab3-Terraform_VSC_Appfw_LB_Config.png
   :width: 800px
.. |lab3-Terraform_VSC_Main_Save| image:: _static/lab3-Terraform_VSC_Main_Save.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Plan| image:: _static/lab3-Terraform_VSC_Appfw_Plan.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Plan_Results| image:: _static/lab3-Terraform_VSC_Appfw_Plan_Results.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Apply| image:: _static/lab3-Terraform_VSC_Appfw_Apply.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Apply_Yes| image:: _static/lab3-Terraform_VSC_Appfw_Apply_Yes.png
   :width: 800px
.. |lab3-Terraform_VSC_Appfw_Apply_Results| image:: _static/lab3-Terraform_VSC_Appfw_Apply_Results.png
   :width: 800px
.. |lab1-XC_App_Connect| image:: _static/lab1-XC_App_Connect.png
   :width: 800px
.. |lab1-XC_LB| image:: _static/lab1-XC_LB.png
   :width: 800px
.. |lab1-XC_LB_Manage| image:: _static/lab1-XC_LB_Manage.png
   :width: 800px
.. |lab3-XC_Terraform_WAF| image:: _static/lab3-XC_Terraform_WAF.png
   :width: 800px
.. |lab3-XC_Terraform_WAF_Enable| image:: _static/lab3-XC_Terraform_WAF_Enable.png
   :width: 800px
.. |lab3-XC_Terraform_WAF_Cancel| image:: _static/lab3-XC_Terraform_WAF_Cancel.png
   :width: 800px
.. |lab3-Terraform_VSC_Destroy| image:: _static/lab3-Terraform_VSC_Destroy.png
   :width: 800px
.. |lab3-Terraform_VSC_Destroy_Yes| image:: _static/lab3-Terraform_VSC_Destroy_yes.png
   :width: 800px
.. |lab3-Terraform_VSC_Destroy_Results| image:: _static/lab3-Terraform_VSC_Destroy_Results.png
   :width: 800px
.. |labend| image:: _static/labend.png
   :width: 800px