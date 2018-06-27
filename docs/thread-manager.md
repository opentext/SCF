# Thread manager
The thread manager control thread pools and dedicated worker threads in an SCF instance. It also holds functionality to attach and detach externally created threads to the SCF instance. 

The thread manager configuration is globally shared among almost all application types except for the Service Gateway which has it's own thread manager configuration. The configuration file is typically called threadmanager.xml.
