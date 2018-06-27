# IO manager
The IO manager handles input connectors and temporary data streams.

The IO manager configuration is typically called iomanager.xml and can be located in the working directory of an application, in the application types solution directory in the installation or in the installations global configuration directory. An overview of the configuration is presented here.

```xml
<?xml version="1.0"?>
<strs xmlns="http://schemas.streamserve.com/kernel/1.0"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      xmlns:pub="http://schemas.streamserve.com/public/1.0">

  <include xlink:type="simple" xlink:href="file://kernelmodules.xml"/>

  <kernel>
    <managers>
      <manager type="http://schemas.streamserve.com/uid/manager/iomanager/1.0">
        <configuration>
          <iomanager xmlns="http://schemas.streamserve.com/uid/manager/iomanager/1.0">

            <!-- Optional connectors configuration here -->

            <temporarystream bufferKBsize="150">
              <objectpool type="http://schemas.streamserve.com/uid/component/temporarystreamobjectpool/1.0" name="http://schemas.streamserve.com/uid/resource/objectpool/temporarystream">
                <configuration>
                  <objectpool xmlns="http://schemas.streamserve.com/uid/component/objectpool/1.0">
                    <general min="1" max="1000" allowtemporaryobjects="true" />
                  </objectpool>
                </configuration>
              </objectpool>
            </temporarystream>

          </iomanager>
        </configuration>
      </manager>
    </managers>
  </kernel>

</strs>
```

## Temporary data streams
In the configuration example above the temporarystream element configures how the SCF instance deals with temporary data. The example configuration describes the default configuration for an SCF instance and the various attributes and elements are interpreted as follows:

bufferKBsize is the maximum size of a temporary data stream that is stored in memory. As long as the temporary data is at most this size it will not be serialized to a temporary file. This limit can and should be changed if there are a lot of temporary files that are created by the SCF instance and if the size of those temporary files are near the value of bufferKBsize. 

The general element has a min and max value, these values determine the number of temporary memory data streams that are cached to avoid having to reallocate them. Caching the temporary memory data stream can in some cases be a huge performance benefit since the SCF instance will be able to reuse already allocated memory instead of having to reallocate it every time a new temporary data stream is created. 

The general element also has a allowtemporaryobjects attribute which by default is set to true. This attribute should not be change unless there are exceptional circumstances that require it. A value of true means that if the max number of cached temporary streams is allocated by the SCF instance and yet another temporary stream is required then a temporary stream will be allocated for this and when that stream is no longer needed it will be deallocated instead of being cached. If the value is changed to false then the IO manager will not allocate temporary streams, instead the thread that requested the temporary data stream will be halted until a cached stream is released by some other thread. Changing the value to false will lead to slightly more predictable memory consumption but it also carries a risk of deadlocking the SCF instance.

## Input connectors
Input connectors are configured within the IO manager at startup or they can be programmatically added at any point in an SCF instances lifetime.
A connector configuration example can be seen below.

```xml
      <connector name="connector name">
        <configuration>
          <protocoldrivers synchronize="true">

            <protocoldriver type="protocol driver type">
              <configuration>
                <!-- Protocol driver configuration -->
              </configuration>
            </protocoldriver>

            <protocoldriver type="protocol driver type">
              <configuration>
                <!-- Protocol driver configuration -->
              </configuration>
            </protocoldriver>

            <!-- Additional protocol drivers -->

          </protocoldrivers>
          <connectordriver type="connector driver type">
            <configuration>
              <!-- Connector driver configuration -->
            </configuration>
          </connectordriver>
        </configuration>
      </connector>

```

The connector name is as you might guess the name of the connector, for example the name you give the connector in Communications Builder.

The synchronize attribute is a boolean that if true will cause the IO manager to send whatever data that the connector receives for processing in the same thread that is running the connector when it found the data. If the synchronize attribute is false then the IO manager will offload processing of the data to a thread on the IO thread pool. For a description of the thread pools, please see the [Thread manager](thread-manager.md) documentation.

The protocol drivers is a chain of objects that each will do some transformation of the data or otherwise apply logic to the interpretation of the data. The last protocol driver of the chain will typically be an object that knows how to send the data further on to an application for processing. For example the Communications Server has a protocol driver specific to it that knows how to send the data to an input queue (in memory or in a database). Some examples of protocol drivers:
   * HTTP protocol driver: Knows how to interpret an HTTP header and how to extract the data from an incoming HTTP request.
   * File protocol driver: Stores incoming data as a temporary file for easy backup purposes.

The connector driver is the object that actually knows how to find and / or receive data from a certain source. A connector driver can be either a scanner or a listener. Scanners poll some source of data at regular intervals to find new data for processing while listeners wait for data to be sent to them. Some examples of scanners are directory scanner or FTP scanner. An example of a listener is a TCP listener.
