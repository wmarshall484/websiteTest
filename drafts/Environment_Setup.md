# Java Application API environment setup guide

There are three primary ways to get started with the API. If you are trying the API for the first time, the Streams Quick Start Edition VM is likely the fastest way to start working with the tutorials on this page, so download it at the following link to get started: [Streams Quick Start Edition VM](http://www-01.ibm.com/software/data/infosphere/stream-computing/trials.html)

The Streams Quick Start Edition VM contains a ready-to-go release of IBM InfoSphere Streams. Additionally, the Quick Start VM comes bundled with IBM Streams Studio, which provides an intuitive, visual representation of your streaming application. Once  you've downloaded it, start the VM, open a console and type:
``` bash
streamtool startinstance
```
This will start the default IBM Streams instance that comes with the VM, which is disabled on startup. 

If you are **not** using the IBM Streams Quick Start Edition and already have an IBM Streams installation, make sure you've followed the instructions for [setting up your domain and instance](https://github.com/wmarshall484/websiteTest/blob/master/drafts/DomainSetup.md).

## Getting the JAA from github
Although the QuickStart VM comes with a version of the JAA out of the box, it may be preferable to obtain the most recent version from github. To do so, simply navigate to the [releases section](https://github.com/Ibmstreams/streamsx.topology/releases) of the github site, download the latest version, and extract it to your filesystem.

Alternatively, if you want the cutting edge of the API, clone the main repository directly:
``` bash 
git clone git@github.com:IBMStreams/streamsx.topology.git
```
After cloning the repository, the project must be built to produce the com.ibm.streamsx.topology.jar file. Fortunately, the project provides an ant script to take care of this automatically. The script has three requirements:
* Ant version > 1.9.1
* [Optional] For tests, the JUnit jar files must be stored in ~/.ant/lib
* [Optional] For tests, the Jacoco code coverage jars must be stored in ~/.ant/lib

To build the project, navigate to the project root and simply type ```ant```. The com.ibm.streamsx.topology.jar file will be produced in the <project_root>/com.ibm.streamsx.topology/lib directory.

To run the suite of unit tests, type ```ant test```.

## Setting up with Eclipse/Streams Studio
If you've followed the previous steps and are ready to start writing code in Eclipse or StreamsStudio, take the following steps.

* Start Eclipse / Streams Studio.
* Set up classpath variable in your workspace:
  * Open Windows -> Preferences -> Java -> Build Path -> Classpath Variables
  * Click New...
  * In the New Variable Entry dialog, provide a name for the library. I put mine as "STREAMS_JAVA_FUNCTIONAL_API".
  * Click File…
  * Browse to locate the following file: <streamsx.topology install path>/com.ibm.streamsx.topology/lib/com.ibm.streamsx.topology.jar

If you haven't yet linked Eclipse/Streams Studio with IBM Streams, ensure the following jar files are part of the classpath:
* <streamsx.topology install path>/com.ibm.streamsx.topology/lib/com.ibm.streamsx.topology.jar
* $STREAMS_INSTALL/lib/com.ibm.streams.operator.samples.jar
* 
This completes the installation of the streamsx.topology project. The steps above are specific to installing the Java Application API libraries in Eclipse or Streams Studio (or any Java IDE, really).

You are now ready to go!
