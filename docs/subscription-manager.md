# Subscription manager
The subscription manager handles
   * Internal SCF instance notifications.
   * Dispatch of java notifications.
   * Distributed SCF notifications.

The subscription manager does not have any user configurable configuration.

## Internal SCF instance notifications
The internal notifications are used as an event notification system inside an SCF instance so that different services and components can keep track of events that are fired from other parts of the system. Examples of such notifications are
   * When all core services (managers) in an SCF instance has started, in order to do post-initialization tasks.
   * When all data for an input job has been created and potentially stored in the database.
   * (In 5.6.2 and earlier) When data is available for processing on a queue in Communications Server.

## Java notifications
The java notifications are actually a java extension on top of the internal SCF notifications so a java notification listener can be thought of as a hook into the event notification system. Each internal notification that is relevant to expose as a java notification has methods available to transform the notification into a java object and then the subscription manager takes care of calling the java notification handler with that notification.

For a description of Java notifications, see the Java notifications SDK.

## Distributed SCF notifications
The distributed notifications are available since 16.2 update 1 and later. In 16.2 update 1 the distributed notifications are disabled by default while in 16.3 and later they are enabled by default. The distributed notifications are currently configured in the Management Gateways parameters.xml file in the section that looks like below:

```xml
    <parameter type="http://schemas.streamserve.com/uid/resource/simpleparameter/1.0" id="strs-distributed-notifications-proxy-port" name="Distributed Notifications Proxy Listener Port">
      <configuration>
        <simple type="http://schemas.streamserve.com/uid/resource/simpleparameter/1.0">
          <values>
            <value>28802</value>
          </values>
        </simple>
      </configuration>
    </parameter>
```

There is usually no need to change the pre-configured port unless there is a port conflict for some reason. Even then it is possible to run SCF without the distributed notifications although you do get some benefits when running with them, such as better response times for posting jobs through the "communications" API (see API documentation link on the [Main page](index.md)).

If the port is changed then you need to:
   1. Stop all applications started by that Management Gateway.
   2. Restart the management gateway.
   3. In Control Center for each application domain -> right click any application -> select "Update domain information".
   4. Start the previously stopped applications.
