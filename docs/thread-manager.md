# Thread manager
The thread manager control thread pools and dedicated worker threads in an SCF instance. It also holds functionality to attach and detach externally created threads to the SCF instance. 

The thread manager configuration is globally shared among almost all application types except for the Service Gateway which has it's own thread manager configuration. The configuration file is typically called threadmanager.xml and depending on which version you are on it looks like one of the below.

## 16.0 and later
In 16.0 the thread pools where updated to support growing and shrinking the number of worker threads for each pool depending on current SCF instance load.

```xml
<?xml version="1.0"?>
<!-- *B*U*I*L*D*V*E*R*S*I*O*N* -->
<strs xmlns="http://schemas.streamserve.com/kernel/1.0" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:pub="http://schemas.streamserve.com/public/1.0">

  <include xlink:type="simple" xlink:href="file://kernelmodules.xml"/>

  <kernel>
    <managers>
      <manager type="http://schemas.streamserve.com/uid/manager/threadmanager/1.0">
        <configuration>
          <threadmanager xmlns="http://schemas.streamserve.com/uid/manager/threadmanager/1.0">
            <dispatchqueues>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueueex/1.0" name="http://schemas.streamserve.com/uid/resource/taskdispatchqueue/1.0">
                <configuration>
                  <threadpoolex>
                    <properties>
                      <minthreads>4</minthreads>
                      <maxthreads>16</maxthreads>
                      <queuelimit>0</queuelimit>
                      <minqueuethreshold>0</minqueuethreshold>
                      <maxqueuethreshold>16</maxqueuethreshold>
                      <queuethresholdcheckinterval>1000</queuethresholdcheckinterval>
                    </properties>
                  </threadpoolex>
                </configuration>
              </dispatchqueue>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueueex/1.0" name="http://schemas.streamserve.com/uid/resource/iodispatchqueue/1.0">
                <configuration>
                  <threadpoolex>
                    <properties>
                      <minthreads>4</minthreads>
                      <maxthreads>16</maxthreads>
                      <queuelimit>0</queuelimit>
                      <minqueuethreshold>0</minqueuethreshold>
                      <maxqueuethreshold>16</maxqueuethreshold>
                      <queuethresholdcheckinterval>1000</queuethresholdcheckinterval>
                    </properties>
                  </threadpoolex>
                </configuration>
              </dispatchqueue>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueueex/1.0" name="http://schemas.streamserve.com/uid/resource/jobdispatchqueue/1.0">
                <configuration>
                  <threadpoolex>
                    <properties>
                      <minthreads>4</minthreads>
                      <maxthreads>32</maxthreads>
                      <queuelimit>2000</queuelimit>
                      <minqueuethreshold>4</minqueuethreshold>
                      <maxqueuethreshold>16</maxqueuethreshold>
                      <queuethresholdcheckinterval>1000</queuethresholdcheckinterval>
                    </properties>
                  </threadpoolex>
                </configuration>
              </dispatchqueue>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueueex/1.0" name="http://schemas.streamserve.com/uid/resource/customdispatchqueue/1.0">
                <configuration>
                  <threadpoolex>
                    <properties>
                      <minthreads>1</minthreads>
                      <maxthreads>10</maxthreads>
                      <queuelimit>10</queuelimit>
                      <minqueuethreshold>0</minqueuethreshold>
                      <maxqueuethreshold>10</maxqueuethreshold>
                      <queuethresholdcheckinterval>1000</queuethresholdcheckinterval>
                    </properties>
                  </threadpoolex>
                </configuration>
              </dispatchqueue>

            </dispatchqueues>
          </threadmanager>
        </configuration>
      </manager>
    </managers>
  </kernel>

</strs>
```

In the configuration example aboce the explanation to the different properties of a thread pool configuration is as follows:
   * minthreads: This is the minimum number of threads that the thread pool will contain. This is also the starting number of threads.
   * maxthreads: The maximum number of threads that the thread pool can grow to.
   * queuelimit: If this value is greater than 0 then this is the maximum number of work items that can be posted onto a thread at any given time. If the value is 0 then there is no limit to the maximum number of work items that can be posted onto a thread. If the work item limit is reached then the threadpool will either refuse to add the work item or block the calling thread until a free slot is available in the thread pool before an item is added. This setting is available to keep from flooding a thread pool with work items.
   * minqueuethreshold: If this value is greater than 0 and the number of work items posted onto the threadpool is less than this number then a thread will be removed from the thread pool, down to the minimum number of threads. If this value is 0 then the thread pool cannot shrink.
   * maxqueuethreshold: If the number of work items in a queue exceeds this value then an additional thread will be added to the thread pool, up to the maximum number of allowed threads.
   * queuethresholdcheckinterval: The interval in milliseconds that controls how often maxqueuethreshold and minqueuethreshold will be checked.

In version 16 and onwards there are three default thread pools available, the IO thread pool, task thread pool and job thread pool. Threre is also a thread pool template for custom thread pools that an application may wish to add. See further down on this page for a description of those.

## 5.6.2 and earlier
In 5.6.2 and earlier there are two common thread pools available. These thread pools have a fixed number of threads and the configuration for them looks like below.

```xml
<?xml version="1.0"?>
<!-- 5.6.2_GA_201 -->
<strs	xmlns="http://schemas.streamserve.com/kernel/1.0" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:pub="http://schemas.streamserve.com/public/1.0">

  <include xlink:type="simple" xlink:href="file://kernelmodules.xml"/>

  <kernel>
    <managers>
      <manager type="http://schemas.streamserve.com/uid/manager/threadmanager/1.0">
        <configuration>
          <threadmanager xmlns="http://schemas.streamserve.com/uid/manager/threadmanager/1.0">
            <dispatchqueues>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueue/1.0" name="http://schemas.streamserve.com/uid/resource/taskdispatchqueue/1.0">
                <configuration>
                  <threadpool min="16" max="16"/>
                </configuration>
              </dispatchqueue>

              <dispatchqueue type="http://schemas.streamserve.com/uid/component/dispatchqueue/1.0" name="http://schemas.streamserve.com/uid/resource/iodispatchqueue/1.0">
                <configuration>
                  <threadpool min="20" max="20"/>
                </configuration>
              </dispatchqueue>
            </dispatchqueues>

          </threadmanager>
        </configuration>
      </manager>
    </managers>
  </kernel>

</strs>
```

In the configuration above the properties are interpreted as follows:
   * min: The minimum number of threads in a thread pool.
   * max: the maximum number of threads in a thread pool.

For these thread pools the minimum and maximum values must be equal, there is no functionality in 5.6.2 and earlier for resizing the thread pools, despite what the configuration implies. This functionality was implemented in an enhanced way in 16.0 instead.

In 5.6.2 and earlier there are 2 different thread pools, the task thread pool and IO thread pool. Read more on those below.

## Task thread pool
The task thread pool is used to execute background tasks such as delivering notifications for the [subscription manager](subscription-manager.md) or handling work items scheduled by the [schedule manager](schedule-manager.md). These tasks will always be small and quick to execute which is wny the thread pool does not have a queue limit.

## IO thread pool
The IO thread pool deals with background IO, such as storing documents (queue items, messages, ...) to the database or reading and writing large files. The IO thread pool also handles asynchronous input connector requests, see [IO manager](io-manager.md). Additionally the IO thread pool is used by the job submit web service and to some extent by the communications REST API in the service gateway application, so an option for increasing throughput for job submit / communications is to increase the maximum and/or minimum number of threads for this thread pool in the service gateway applications working directory.

## Job thread pool (16.0.0 and above)
This thread pools replaces the 5.6.2 queue thread pools. In 16.0 onwards all jobs in any application are executed on the job thread pool instead of allowing each queue to have their own thread pools. The benefit to this is that the communications server is able to utilize it's resources more efficiently, it can do more efficient caching and there is a central place to configure job parallelism for all job processing applications.

### Limit the threads to execute on a communications server queue
In version 16 and later it is possible to limit the number of parallel requests that are allowed on a specific queue in communications server by specifying a custom parameter on the queue. You may want to do this if you for example send documents to a printer that cannot handle a large number of parallel prints. The syntax for this is `maxthreads N` where N is the number of maximum parallel requests, for example `maxthreads 1` to make the queue single threaded.

## Custom thread pool template (16.0.0 and above)
The custom thread pool is not instantiated with the rest of the thread pools, instead it is used as a template if an application requires a thread pool for a specific type of jobs. The configuration is included for future use, currently there are no applications that make use of this feature.
