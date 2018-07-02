# Log manager
The log manager is responsible for initializing the SCF logging system and loading all required log message files. Every operating system thread running in SCF will have access to a log stream which is dedicated to that particular thread. An SCF instance writes log messages to this stream and the logging system is then responsible for writing the log message on to whatever log providers (sinks) that are configured for the instance.

There are three different log providers available:
   * Console
   * File
   * Database

The configuration for the log manager is typically found in logmanager.xml in the SCF instance working directory and it looks like below:
```xml
<strs xmlns="http://schemas.streamserve.com/kernel/1.0"
      xmlns:xlink="http://www.w3.org/1999/xlink" 
      xmlns:pub="http://schemas.streamserve.com/public/1.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <include xlink:type="simple" xlink:href="file://kernelmodules.xml"/>
  <kernel>
    <managers>
      <manager type="http://schemas.streamserve.com/uid/manager/logmanager/1.0">
        <configuration>
          <logmanager xmlns="http://schemas.streamserve.com/uid/manager/logmanager/1.0">
            <general loglevel="3"/>

            <msgsources>
              <msgsource type="http://schemas.streamserve.com/uid/resource/textmsgsource/1.0" source="logmessages.txt"/>
            </msgsources>

            <providers>
              <provider enabled="yes" name="http://schema.streamserve.com/uid/resource/platform/logconsole/1.0" type="http://schema.streamserve.com/uid/component/consolelogprovider/1.0" toplevels="*" msginfo="7" asynchronous="yes"/>
              <provider enabled="yes" name="http://schema.streamserve.com/uid/resource/platform/logfile/1.0" type="http://schema.streamserve.com/uid/component/filelogprovider/1.0" fmt="yes" codepage="UTF-8" default="no" toplevels="*" msginfo="7" delete="no" source="log.txt" sizerestrictions="yes" sizelimit="10" timerestrictions="no" timelimit="0" movetime="1200" savepath="." asynchronous="yes"/>
              <provider enabled="no" name="http://schema.streamserve.com/uid/resource/platform/logdatabase/1.0" type="http://schema.streamserve.com/uid/component/databaselogprovider/1.0" fmt="no" default="no" asynchronous="yes" timelimit="0" />
            </providers>

          </logmanager>
        </configuration>
      </manager>
    </managers>
  </kernel>

</strs>
```

## Log level
The log level in the 'general' element is a number with the following meanings:

| Log level  | Meaning                                                         |
|-----------:|-----------------------------------------------------------------|
|          0 |  Severe errors only                                             |
|          1 |  All errors                                                     |
|          2 |  Errors and warnings                                            |
|          3 |  Errors, warnings and informational messages                    |
|          4 |  Errors, warnings, informational messages and debug information |
|         99 |  All messages including very verbose debug information          |

## Message sources
The 'msgsource' elements describe where to load log messages from. The type attribute is hard coded to `http://schemas.streamserve.com/uid/resource/textmsgsource/1.0`. The source attribute is the name of a file on disk that will be located by searching (in the following order)
   1. Working directory
   2. (16.2 and later only) Current solution directory (e.g. solutions/communicationsserver for Communications Server).
   3. (16.2 and later only) Global configuration file directory.
   4. Value of STRSMODULEPATH environment variable.

## Providers
The 'provider' element describes a provider or sink where to output log messages. The attributes have the following meaning:

### enabled
The enabled attribute will tell SCF if the provider is active or not. This attribute is new since 16.3 update 1 for ease of use when configuring log providers in control center. 

### name
This is a unique name (within the log manager) of the provider.

### type
The type of provider. There are three different types:
   * `http://schema.streamserve.com/uid/component/consolelogprovider/1.0`: Writes to the console window (stderr).
   * `http://schema.streamserve.com/uid/component/filelogprovider/1.0`: Writes to a file.
   * `http://schema.streamserve.com/uid/component/databaselogprovider/1.0`: Writes to a log database. Requires that the log database is configured properly.

### fmt
A value of 'yes' or 'no' where yes tells a provider to format the log message for human consumption (e.g. in a log file) while no is more intended for e.g. database logging where the actual formatting will be done by a separate tool or API later. Usually there is no reason to change the default value for this attribute.

### codepage
Encoding to use when writing formatted log messages.

### default
For internal use by SCF, please don't change.

### toplevels
It is possible to limit a log source to only log messages from a specific application or service if that service has a separate top level defined. For example, log messages for template engine are logged to top level 31 so to see only template engine log messages for a certain provider change the toplevels attribute to 31. From version 16.2 onwards there is a list of toplevels in the toplevels.txt file in the global configuration folder in the installation.

### msginfo
The msginfo attribute is a bit mask that tells the log manager what information to include in each log message. The values that are available to combine are according to the following table:

| Value | Meaning |
|------:|---------|
|     1 | Time stamp on the format 'YYMMDD hhmmss'.
|     2 | Log message ID.
|     4 | Log level.
|     8 | Application ID.
|    16 | 'Sender' user as configured in a communications builder project.
|    32 | Thread ID of the current operating system thread.
|    64 | Job ID of the current processing job (in Communications Server and Orchestration Server).
|   128 | Code location where the log message was logged. This is useful for debug logs sent to R&D.
|   256 | Top level of the log message.
|   512 | Log year using four digits instead of two.
|  1024 | External job ID.
|  2048 | 'Receiver' user as configured in a communications builder project.
|  4096 | Tenant ID.
|  8192 | Domain ID.
| 16384 | Add milliseconds to timestamp.

### delete
Only valid for the file log provider. If set to 'yes' then the log file will be deleted when an SCF instance starts up, if set to 'no' then the log file will be appended to.

### source
Only valid for the file log provider. The file to write log messages to.

### asynchronous
A value 'yes' or 'no' meaning that log messages are sent in bulks asynchronously to the provider or not. A reason for changing this to 'no' would be  if debugging a crash in an application to ensure that each log message is actually printed before the crash occurs.

### sizerestrictions and sizelimit
Only valid for the file log provider. Optionally restrict log files to a specified size, once this size is exceeded take a backup copy of the current file and truncate it before continuing to log. The sizelimit attribute is expressed in megabytes.

### timerestrictions and timelimit
Valid for the file and database log providers. For the file log provider the log file will be copied to a backup location after 'timelimit' number of days and the current log file will be truncated before logging continues. For the database log provider each log message will be kept in the database for 'timelimit' number of days.

### movetime
Only valid for the file log provider. Time expressed in hhmm, 24-hour format (e.g. 1400 for 2 PM local time) when file backups will be taken according to size or time restrictions.

### savepath
Only valid for the file log provider. Directory where backup copied log files will be placed.
