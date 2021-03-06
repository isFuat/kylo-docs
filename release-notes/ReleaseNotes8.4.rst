Release 0.8.4 (November 28, 2017)
=================================

Highlights
----------
- NiFi-1.4.0 support
- Enhanced Operations Manager dashboard with more user interaction and better performance
- A number of SLA improvements including the ability to configure customizable SLA email templates
- Enhanced Operations streaming statistics which now supports many more viewing options
- Ability to clone an existing Feed
- Visual query enhancements. The Transform Data step has been improved with UI enhancements including a context menu when clicking on a row or highlighting text.
- Preview validation errors. Apply domain types in a Data Transformation feed and preview which rows are invalid.
- Secure installation. Default usernames and passwords can be customized during installation to ensure a secure environment.
- Global search enhancements. Deleting a feed will remove its data from search results. Re-processing same data via a feed will not duplicate search results.
- 80 Bugs fixed


Download Links
--------------
- Visit the :doc:`Downloads <../about/Downloads>` page for links.


Upgrade Instructions from v0.8.3
--------------------------------
- **Order of execution of instructions to be updated**

1. Stop NiFi:

 .. code-block:: shell

   service nifi stop

 ..

2. Backup any Kylo plugins

  When Kylo is uninstalled it will backup configuration files, but not the `/plugin` jar files.
  If you have any custom plugins in either `kylo-services/plugin`  or `kylo-ui/plugin` then you will want to manually back them up to a different location.

3. Uninstall Kylo:

 .. code-block:: shell

   /opt/kylo/remove-kylo.sh

 ..

3. Install the new RPM:

 .. code-block:: shell

     rpm –ivh <RPM_FILE>

 ..


4. Global search configuration (only applicable if using Elasticsearch):

    4.1. This step to create kylo indexes may already have been performed as part of v0.8.3 installation. If indexes already exist, Elasticsearch will report an ``index_already_exists_exception``. It is safe to ignore this and continue.

    Change the host and port if necessary. The last two parameters define *num-shards* and *num-replicas*, and can be kept as 1 for development environment.

    .. code-block:: shell

        /opt/kylo/bin/create-kylo-indexes-es.sh localhost 9200 1 1
    ..

    4.2. If using Elasticsearch v5, update the **Index Text Service** feed. This step should be done once Kylo services are started and Kylo is up and running. [Note: This requires NiFi 1.3 or later]

    Import the feed ``index_text_service_v2.feed.zip`` file available at ``/opt/kylo/setup/data/feeds/nifi-1.3``. Click 'Yes' for these options during feed import (a) Overwrite Feed (b) Replace Feed Template (c) Replace Reusable Template.

    4.3. If using Elasticsearch v2, install an additional plugin to support deletes. If required, change the location to where Elasticsearch is installed.

    .. code-block:: shell

         sudo /usr/share/elasticsearch/bin/plugin install delete-by-query
         service elasticsearch restart

    ..


5. Restore previous application.properties files. If you have customized the the application.properties, copy the backup from the 0.8.3 install.

     5.1 Find the /bkup-config/TIMESTAMP/kylo-services/application.properties file

        - Kylo will backup the application.properties file to the following location, */opt/kylo/bkup-config/YYYY_MM_DD_HH_MM_millis/kylo-services/application.properties*, replacing the "YYYY_MM_DD_HH_MM_millis" with a valid time:

     5.2 Copy the backup file over to the /opt/kylo/kylo-services/conf folder

        .. code-block:: shell

          ### move the application.properties shipped with the .rpm to a backup file
          mv /opt/kylo/kylo-services/conf/application.properties /opt/kylo/kylo-services/conf/application.properties.0_8_3_template
          ### copy the backup properties  (Replace the YYYY_MM_DD_HH_MM_millis  with the valid timestamp)
          cp /opt/kylo/bkup-config/YYYY_MM_DD_HH_MM_millis/kylo-services/application.properties /opt/kylo/kylo-services/conf

        ..

     5.3 Copy the /bkup-config/TIMESTAMP/kylo-ui/application.properties file to `/opt/kylo/kylo-ui/conf`

     5.4 Ensure the property ``security.jwt.key`` in both kylo-services and kylo-ui application.properties file match.  They property below needs to match in both of these files:

        - */opt/kylo/kylo-ui/conf/application.properties*
        - */opt/kylo/kylo-services/conf/application.properties*

          .. code-block:: properties

            security.jwt.key=

          ..


6. JMS configuration:

It was previously possible to provide ActiveMQ and AmazonSQS configuration in their respective configuration files called ``activemq.properties`` and ``amazon-sqs.properties``.
It is no longer possible and these properties should be moved over to standard Kylo configuration file found in ``<KYLO_HOME>/kylo-services/conf/application.properties``.


7.  **NOTE:** Kylo no longer ships with the default **dladmin** user. You will need to re-add this user only if you're using the default authentication configuration:

   - Uncomment the following line in :code:`/opt/kylo/kylo-services/conf/application.properties` and :code:`/opt/kylo/kylo-ui/conf/application.properties` :

    .. code-block:: properties

        security.auth.file.users=file:///opt/kylo/users.properties
        security.auth.file.groups=file:///opt/kylo/groups.properties

    ..

   - Create a file called :code:`users.properties` file that is owned by kylo and replace **dladmin** with a new username and **thinkbig** with a new password:

    .. code-block:: shell

        echo "dladmin=thinkbig" > /opt/kylo/users.properties
        chown kylo:users /opt/kylo/users.properties
        chmod 600 /opt/kylo/users.properties

    ..

   - Create a file called :code:`groups.properties` file that is owned by kylo and set the default groups:

    .. code-block:: shell

        vi /opt/kylo/groups.properties


    .. code-block:: properties

        dladmin=admin,user
        analyst=analyst,user
        designer=designer,user
        operator=operations,user

    .. code-block:: shell

        chown kylo:users /opt/kylo/groups.properties
        chmod 600 /opt/kylo/groups.properties

7. Update the NiFi nars.  Run the following shell script to copy over the new NiFi nars/jars to get new changes to NiFi processors and services.

   .. code-block:: shell

      /opt/kylo/setup/nifi/update-nars-jars.sh <NIFI_HOME> <KYLO_SETUP_FOLDER> <NIFI_LINUX_USER> <NIFI_LINUX_GROUP>

      Example:  /opt/kylo/setup/nifi/update-nars-jars.sh /opt/nifi /opt/kylo/setup nifi users

8. Start NiFi and Kylo

 .. code-block:: shell

   service nifi start

   /opt/kylo/start-kylo-apps.sh

 ..

   8.1 Once Kylo is up and running, refer back to step 4.2 to update the **Index Text Service** feed if using Elasticsearch v5.