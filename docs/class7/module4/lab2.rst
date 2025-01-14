Lab 2 - Deploy Containers on vK8s and Build Grafana Dashboard
=============================================================

**Exercise 1 - Setup Jumphost to connect to the vk8 cluster**

**Environment Setup**
To complete this lab section, we'll need to complete the following steps:

- Ensure that the NAMESPACE environment variable is set.
- Review the Load Balancer and Origin Server Configuration.
- Deploy Grafana using docker compose which will be proconfigured to match your namespace name for each of the 3 regions.

#. Returning back to the Lab Components view, click the jumphost and then click the *Access* button. From the access list, select **Web Shell**.

   .. image:: ../images/M4-L2-webshell-launch.png
      :width: 400pt

#. From Web Shell, run the following commands:

   a. to view the current kubeconfig of the jumphost.

      .. code-block:: bash

         kubectl config view
            ## output shows kubeconfig is not configured for any cluster.
         apiVersion: v1
         clusters: null
         contexts: null
         current-context: ""
         kind: Config
         preferences: {}
         users: null


   b. to verify that your file is uploaded.

      .. code-block:: bash

        ls /srv/filebrowser/
         ### Your kubeconfig file you uploaded should be listed here:
         ### ves_kind-python_asmith-vk8.yaml


   In the example above, the **NAMESPACE** is **kind-python** and the **Virtual K8s** is **asmith-vk8**.

#. From Web Shell, modify and run the following commands to set the NAMESPACE environment variable:

   .. note:: This step is key to ensure the subsequent commands are configured correctly. Environment variables will need to be set each time you launch a new Web Shell.

   .. code-block:: bash

     ### Replace <namespace> your own namespace value
     export NAMESPACE=<namespace>

#. Next, we'll configure you environment to access the vK8s cluster using the kubeconfig file we uploaded to the Jumphost.

   .. code-block:: bash

      #Assuming you only have one kubeconfig file in the /srv/filebrowser run:
      cat /srv/filebrowser/* > ~/.kube/config

      #Otherwise, modify and run:
      #export KUBECONFIG=/path/to/kubeconfig/file

      # Let's review again to confirm that we can reach the cluster:
      kubectl config view

   The output should look like this:

   .. image:: ../images/M4-L2-exp-kubeconfig.png
      :width: 400pt


**Exercise 2 - Deploy Containers on vK8s and Add Origin Pool and Load Balancer**

**Deploy Containers on vK8s**

#. Now we can deploy the containers into the vK8s cluster.
   Do this by using the kubectl command to apply the manifest files in the vk8s directory.

   .. code-block:: bash

     cd ~/caaslab
     kubectl apply -f vk8s/

#. Return to the Distributed Cloud console and in the **Distributed Apps** workspace select **Virtual K8s** under **Applications**.

   Click on your vk8 cluster to view the details.

#. Review all the tabs on your Virtual K8s; **Workloads, Deployments, ... Pods.**

   Which ones have something configured?

   Why isn't there a Workload configured for these Pods?

**Review the Load Balancer and Origin Server Configuration**

#. On the Distributed Cloud console and in the **Multi-Cloud App Connect** workspace, under **Manage**, hover over **Load Balancers**, then click **Origin Pools**.

#. Under the **Actions** menu, for the row **adjective-animal-origin** click the **...** and select **Manage Configuration**.

   .. image:: ../images/M4-L2-originpool.png
      :width: 400pt

   Note that this origin pool is referencing a K8s service called **mosquitto.adjective-animal**, and is associated with the Virtual Site **appworld2025-k8s-vsite**.

   We've also configured the Origin Pool to use the Endpoint Selection as **Local Endpoints Only**. This means that the Origin Pool will only use the local endpoints in the region where the Origin Pool is configured and will not cross regions. This is useful when you want to ensure that traffic stays local to the region.

#. Next, let's review the TCP Load Balancer to pointed to this Origin Pool.

   In the Distributed Cloud console and in the **Multi-Cloud App Connect** workspace, under **Manage**, hover over **Load Balancers**, then click **TCP Load Balancers**.

   Again Under the **Actions** menu, for the row **adjective-animal-lb** for the TCP Load Balancer, click the **...** and select **Manage Configuration**.

   The TCP Load Balancer is configured to use the Origin Pool we just reviewed.

      .. note:: For the following image, note the following:

         This LB is configured to listen on 3 different names:

            keen-duck.useast.lab-app.f5demos.com
            keen-duck.europe.lab-app.f5demos.com
            keen-duck.uswest.lab-app.f5demos.com

      - The LB is configured to listen on port 8883 and is using SNI.

   .. image:: ../images/M4-L2-tcplb-1.png
      :width: 400pt


   For the following image **Custom Advertise VIP Configuration**, note the following:
      - We're advertiseing this VIP to the Internet using the virtual site **appworld2025-k8s-vsite**. This will advertise our MQTT service on each of our regions to the Internet.

   .. image:: ../images/M4-L2-tcplb-2.png
      :width: 400pt

   .. image:: ../images/M4-L2-tcplb-3.png
      :width: 400pt


**Exercise 3 - Deploy Grafana**

In this section, we will deploy Grafana using docker compose. The Grafana dashboard will be preconfigured to match your namespace name for each of the 3 regions.

Our docker compose configuration will deploy Grafana with 3 datasources, one for each region. It will also deploy a Dashboard that will show the system stats for each region using the 3 datasources.

To bring up Grafana, run the following commands:

.. code-block:: bash

  cd ~/caaslab/docker-grafana
  docker compose up -d

Continue to the next section to access Grafana and view the dashboard.