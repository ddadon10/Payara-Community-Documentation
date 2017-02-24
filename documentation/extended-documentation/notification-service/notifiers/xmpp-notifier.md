# XMPP Notifier

The **Ex**tensible **M**essaging and **P**resence **P**rotocol is a communication protocol based on XML notation, focused through the exchange of structured messages across services and devices on multiple network arrangements and targeted towards Message Oriented Middleware. Formerly known as **Jabber**, XMPP is an open standard, which means that anyone can implement  toolsets used to send and receive messages across its implementations.

## XMPP Server Configuration

XMPP works using a _client-server_ architecture, akin to other messaging solutions like Hipchat, Slack or IRC for example. Unlike those, XMPP is **decentralized**, which means that any person or organization can run its own server using a propietary domain \(which can be public or private depending of the user needs\). There are multiple server implementations in the market, with both open source and commercial licensing models like the following:

* [Apache Vysper](https://mina.apache.org/vysper-project) \(Open Source\)
* [Oracle Communications Instant Messaging Server](https://www.oracle.com/industries/communications/enterprise/products/instant-messaging/index.html) \(Commercial\)
* [OpenFire ](http://igniterealtime.org/projects/openfire/index.jsp)\(Open Source\)
* [CommuniGate Pro](https://www.communigate.com/default.html) \(Commercial\)
* [Jabberd 2](http://jabberd2.org/) \(Open Source\)

No matter what XMPP server you or your organization is using, the Payara XMPP Notifier Service will be able to communicate with it and push notifications to a specific chat room hosted on the server.

The following are the requirements needed to be fullfilled by the XMPP server:

* Configure the XMPP server with a valid domain name/FQDN
* If considering secure communication, the XMPP server must have a valid SSL certificate configured for its FQDN
* Configure the service for text-based conferencing \(this service is usually named **conference** and is a subdomain of the server\)
* Create room that will be used to receive the notifications sent by Payara Server.
* Create a user that is allowed to access the room created and push notification messages.

We will use OpenFire to illustrate how to correctly configure these requirements. If you are using a different server, check its documentation and follow the instructions on how to configure them correctly.

### OpenFire Configuration

Proceed to download and install the OpenFire server by following the instructions detailed [here](http://download.igniterealtime.org/openfire/docs/latest/documentation/install-guide.html). Once the server is installed, start the OpenFire service and head to the server's admin console located at `http://{FQDN}:9090/`

Since the server was recently installed it will prepare a bootstrapping wizard. First, select the language that will be used by the administration interface:

![OpenFire Language Select Screen](/images/xmpp-notifier-openfire-install-1.png)

Now, proceed to configure the domain name and _FQDN_ of your server. You can also alter the ports for the admin console and configure secure access using property encryption if necessary:

![OpenFire Server Settings Screen](/images/xmpp-notifier-openfire-install-2.png)

To use OpenFire on production environments, its recommended to install an external database used to store its configuration and data. For the sake of simplicity, we will use the embedded option instead:

![OpenFire Database Select Screen](/images/xmpp-notifier-openfire-install-3.png)

Next, we have to configure the user data store that will back the server's authentication. Generally speaking, is not uncommon that an organization uses a LDAP directory to store profiles so that option is recommended on production environments. As with the previous configuration we will use the simpler option:

![OpenFire Profile Screen](/images/xmpp-notifier-openfire-install-4.png)

Finally, configure the credentials for the **admin** user:

![OpenFire Admin User Screen](/images/xmpp-notifier-openfire-install-5.png)

With the server installation complete, login to the admin console with the credentials of the administrator user:

![OpenFire Login](/images/xmpp-notifier-openfire-login.png)

Now we need to create the user that will be used to push notifications on the service's room. Select the **User/Groups** option in the top menu:

![OpenFire User Summary](/images/xmpp-notifier-openfire-users-1.png)

Click on the **Create New User** option in the sidebar. Input the information for this new user \(**Username**, **Name** and **Password** are required\):

![OpenFire New User](/images/xmpp-notifier-openfire-users-2.png)

With the user created, we will proceed to create the room used to display notifications. Select the **Group Chat** option in the top menu:

![OpenFire Room Summary](/images/xmpp-notifier-openfire-create-room-1.png)

Now, click on the **Create New Room** option in the sidebar. Be sure to input the room's **ID**, **Name** and **Description** as requested:

![OpenFire New Room](/images/xmpp-notifier-openfire-create-room-2.png)

Check that the room was created successfully. Click on the room's link to enter its details. Take special note of the **Service Name**, which will be used to configure the notifier later:

![OpenFire Room Details](/images/xmpp-notifier-openfire-room-details.png)

Finally, select the **Permissions** option in the sidebar and add the user we created earlier in the **Room Occupants** section. You can do this by searching using its username in the search box:

![OpenFire Room Permissions](/images/xmpp-notifier-openfire-room-permissions.png)

With this, the XMPP server configuration is completed.

## Payara Server Configuration

With the XMPP server properly configured, now it's time to setup the _Notification Service_ in the domain's configuration. As usual you can do this using the administration web console, from the command line or editing the _domain.xml_ configuration file directly.

The configuration settings required by the service are the following:

* _Server's Location_: _Hostname_ and _Port_ where the XMPP is listening for requests. The hostname is required, the port defaults to _**5222**_ if not provided.
* _Service name_: Used by the XMPP server to manage group chat sessions, always required.
* _Room ID_: The ID of the room that will be used to host the notification events, always required.  
* _Credentials_: The _Username_ and _Password_ of the user that will post notification events in the room.

You can also configure an option whether or not to disable security transport \(SSL\) when establishing communication to the server. The default value for this setting is `false`. It's not recommended to disable secure access on production environments, so use it with discretion.

### Using the Administration Web Console

To configure the Notification Service in the Administration Console, go to _Configuration -&gt; \[instance-configuration \(like server-config\)\] -&gt; Notification Service_ and click on the **XMPP** tab:

![XMPP Notifier in Admin Console](/images/xmpp-notifier-admin-console.png)

Check the **Enabled** box \(and the **Dynamic** box too if you don't want to restart the domain\) and input the required information.

**NOTE**: On release _171_, the room's ID is incorrectly labeled as _Room Name_, so be sure to always input the room's ID. This will be fixed on future releases.

Hit the **Save** button to preserve the changes.

### From the Command Line

To configure the Notification Service from the command line, use the `notification-xmpp-configure` asadmin command, specifying the configuration options like this:

```
asadmin > notification-xmpp-configure --enabled=true --dynamic=true --hostname="172.28.128.3" --port=5222 --username="payara_notifier" --password="******" --securityDisabled=false --roomname=server
```

You can use the `--enabled` and `--dynamic` options to enable/disable the XMPP notifier on demand.

Also, you can retrieve the current configuration for the XMPP notifier using the `get-xmpp-notifier-configuration` asadmin command like this:

```
asadmin > get-xmpp-notifier-configuration

Enabled  Host          Port  Service Name            Username         Password  Security Disabled  Room Name
true     172.28.128.3  5222  conference.payara.fish  payara_notifier  payara    true               server
```

### On the _domain.xml_ configuration file

Modifying the domain.xml configuration is not a supported configuration method, so be careful when considering this option. To configure the Notification Service in the _Domain.xml_ configuration file, locate the `notification-service-configuration element` in the tree and insert the `xmpp-notifier-configuration` with the respective configuration attributes like this:

```
<notification-service-configuration enabled="true">
    <xmpp-notifier-configuration room-name="server" service-name="conference.payara.fish" password="******" security-disabled="true" host="172.28.128.3" username="payara_notifier"></xmpp-notifier-configuration>
</notification-service-configuration>
```

## Troubleshooting

When you have correctly configured the XMPP notifier, it can be used to push notifications to your configured server. You can visualize the messages in a XMPP client of your choice. If you do not see any notification event messages in the client, check the following:

* Is the XMPP notifier enabled?
* Is the Notification Service itself enabled?
* Is there a service configured to use the notifier? \(e.g. the HealthCheck service\)
* Is the service configured to send notifications frequently enough to observe?
* Have you enabled the service after configuring it?
* Is the XMPP server correctly configured?
* Is there a firewall between both servers that is correctly configured to allow sending messages in the respective port?
* Are the room permissions configured correctly?
* If using secure transport, is the server configured with a valid SSL certificate for its _FQDN_?

Here's a sample of how the notifications are visualized on a chat room using the [Spark](https://www.igniterealtime.org/projects/spark/) XMPP client:

![Spark Chat Room](/images/xmpp-notifier-spark-chat.png)
