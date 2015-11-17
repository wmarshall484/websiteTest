# Java Application API environment setup guide

There are three primary ways to get started with the API. If you are trying the API for the first time, the Streams Quick Start Edition VM is likely the fastest way to start working with the tutorials on this page. Download it at the following link to get started: [Streams Quick Start Edition VM](http://www-01.ibm.com/software/data/infosphere/stream-computing/trials.html)

The Streams Quick Start Edition VM contains a ready-to-go release of IBM  Streams. Additionally, the Quick Start VM comes bundled with IBM Streams Studio, which provides an intuitive, visual representation of your streaming application. After you've downloaded it, start the VM, open a console and type:
``` bash
streamtool startinstance
```
This starts the default IBM Streams instance that comes with the VM. This instance is disabled on startup. 

If you are **not** using the IBM Streams Quick Start Edition and already have an IBM Streams installation, make sure you've followed the instructions for [setting up your domain and instance](https://github.com/wmarshall484/websiteTest/blob/master/drafts/DomainSetup.md).

## Getting the Java Application API from GitHub
Although the QuickStart VM comes with a version of the Java Application API out of the box, you might want to obtain the most recent version from GitHub. To do so, simply navigate to the [releases section](https://github.com/Ibmstreams/streamsx.topology/releases) of the GitHub site, download the latest version, and extract it to your file system.

Alternatively, if you want the cutting edge of the API, clone the main repository directly:
``` bash 
git clone git@github.com:IBMStreams/streamsx.topology.git
```
After cloning the repository, you must build the project to produce the `com.ibm.streamsx.topology.jar`file. Fortunately, the project provides an Ant script to take care of this automatically. The script has three requirements:
* Ant version later than 1.9.1
* [Optional] For tests, the JUnit JAR files must be stored in `~/.ant/lib`
* [Optional] For tests, the Jacoco code coverage JAR files must be stored in `~/.ant/lib`

To build the project, navigate to the project root and simply type ```ant```. The `com.ibm.streamsx.topology.jar`file is produced in the `<project_root>/com.ibm.streamsx.topology/lib directory`.

To run the suite of unit tests, type ```ant test```.

## Setting up with Eclipse/Streams Studio
If you've followed the previous instructions and are ready to start writing code in Eclipse or StreamsStudio, take the following steps.

1. Start Eclipse / Streams Studio.
* Set the class path variable in your workspace:
  1. Open __Window > Preferences > Java > Build Path > Classpath Variables__.
  * Click __New__.
  * In the __New Variable Entry__ dialog, provide a name for the library, for example, STREAMS_JAVA_FUNCTIONAL_API.
  * Click __File__.
  * Browse to locate the following file: `<streamsx.topology install path>/com.ibm.streamsx.topology/lib/com.ibm.streamsx.topology.jar`
  * Save this setting.

If you haven't yet linked Eclipse/Streams Studio with IBM Streams, ensure the following JAR files are part of the class path:
* `<streamsx.topology install path>/com.ibm.streamsx.topology/lib/com.ibm.streamsx.topology.jar`
* `$STREAMS_INSTALL/lib/com.ibm.streams.operator.samples.jar`

This completes the installation of the streamsx.topology project. Note, however, that the provided instructions are specific to installing the Java Application API libraries in Eclipse or Streams Studio (or any Java IDE, really).

You are now ready to go!
