# Java Application API environment setup guide

There are three primary ways to get started with the API. If you are trying the API for the first time, the Streams Quick Start Edition VM is likely the fastest way to start working with the tutorials on this page, so download it at the following link to get started: [Streams Quick Start Edition VM](http://www-01.ibm.com/software/data/infosphere/stream-computing/trials.html)

The Streams Quick Start Edition VM contains a ready-to-go release of IBM InfoSphere Streams. Additionally, the Quick Start VM comes bundled with IBM Streams Studio, which provides an intuitive, visual representation of your streaming application. Now that you've downloaded it, start the VM, open a console and type:
``` bash
streamtool startinstance
```
This will start the default IBM Streams instance that comes with the VM, which is disabled on startup. 

## Getting the JAA from github
Although the QuickStart VM comes with a version of the JAA out of the box, it is usually preferable to obtain the most recent version from github. To do so, simply navigate to the [releases section](https://github.com/Ibmstreams/streamsx.topology/releases) of the github site, download the latest version, and extract it to your filesystem.

Alternatively, if you want the cutting edge of the API, clone the main repository directly:
``` bash 
git clone git@github.com:IBMStreams/streamsx.topology.git
```

## Developing with StreamsStudio/eclipse
* Include correct dependencies
* Include JAA jar file. (need to build it if cloned from git)

You are now ready to go!
