Lab 3 - Review Results
======================

**Access Grafana**

Within the lab components screen, select Access on the Jumphost, and choose Grafana from the dropdown.

Launch Grafana and logon with the username *admin*, and the password *grafana*

Once logged in, navigate to **Dashboards** --> **Services** --> **Distributed Host Stats via MQTT**

On the Dashboard, you should see one chart for each Customer Edge region we deployed to. We haven't sent any data yet, so the charts will be empty.

Note: The Green icons in the image will be an indicator that the connection to the MQTT broker was successful.

.. image:: ../images/grafana-dashboard-empty.png
   :width: 650pt

**Let's Publish Some Data**

Now that we have the Grafana dashboard setup, we can start publishing data to the MQTT brokers.

From the Web Shell, run the following command to start the data publisher:

.. code-block:: bash

  cd ~/caaslab
  ./systemstats2mqtt.sh

The script will start publishing data to the MQTT brokers in each region. After a few minutes, you should start to see data on the Grafana dashboard.

In a couple minutes your Web Shell will look like this:

.. image:: ../images/systemstats2mqtt.png
   :width: 650pt

**Review the Data**

Navigate back to the Grafana dashboard and you should see data populating the charts. If you wait 5 minutes, your dashboard will look something like this:

.. image:: ../images/grafana-dashboard-populated.png
   :width: 650pt

**Conclusion**

**Access Grafana**

Within the lab

**this screenshot needs adjusted to match the classroom interface**

.. image:: ../images/launchgrafana.png
  :width: 400pt

