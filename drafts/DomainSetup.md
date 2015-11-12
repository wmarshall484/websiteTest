# Streams Domain and Instance
To run your application in distributed mode, you need a Streams Domain and a Streams Instance. A domain is a logical grouping of resources (or containers) in a network for common management and administration. It can contain one or more instances that share a security model, and a set of domain services. An instance is the Streams distributed runtime environment. It is composed of a set of interacting services running across one or multiple resources. The Streams Instance is responsible for running Streams applications. When an application is submitted onto a Streams instance, it distributes the application code onto each of the resources. It coordinates with the instance services to execute the processing elements.
Setting up a Development Domain and Instance
To set up a development domain and instance, follow these steps. A development domain and instance runs on a single host. You can dynamically add additional host to the domain later. First, if you have not already done so, set up the necessary environment variables by running the streamsprofile.sh.
``` bash
  $  source  <Streams_Install>/bin/streamsprofile.sh
```	

Next, start streamstool by typing the following command:

    streamtool
	

When prompted to provide a ZooKeeper ensemble, enter the ZooKeeper ensemble string if you have a ZooKeeper server set up. Otherwise, press enter to use the embedded ZooKeeper. streamtool is an interactive tool. To get content assist and auto-complete, press <Tab>.
Domain
To make a new domain, enter this command in the streamtool interactive command session:

    mkdomain -d <domainName>
	

Generate public and private key for Streams, so you do not have to keep logging in:

    genkey
	

Start the domain:

    startdomain
	

Tip: If the domain fails to start because a port is in use, you may change the port number by using setdomainproperty. For example, if JMX and SWS ports are in use:

    setdomainproperty jmx.port=<jmxPort> sws.port=<sws.Port>
	

Instance
To make a new instance, enter this command:

    mkinstance -i <instance-id>
	

Start the instance:

    startinstance
	

Running Streams Applications in Distributed Mode
Now that you have a domain and instance started, you can run your application in distributed mode. To submit a job, find the application bundle file (*.sab), and run the following command:

    streamtool submitjob appBundleName.sab
	

To submit our sample application, we will change into the output directory of the application and submit the application bundle:

    cd output/application.TradesAppMain/
    streamtool submitjob application.TradesAppMain.sab
	

Querying for Job Status
You may query job status from your Streams Instance using streamtool commands. If using embedded ZooKeeper:

    streamtool lsjob -d <streamsDomainName> -i <instanceName> --embeddedzk
	

If using external ZooKeeper ensemble:

    streamtool lsjob -d <streamsDomainName> -i <instanceName> --zkconnect <zooKeeperHost>:<zooKeeperPort>
	

You will see job status similar to this:

    [streamsadmin@streamsqse Distributed]$ streamtool lsjobs -d StreamsDomain -i StreamsInstance --embeddedzk
    Instance: StreamsInstance
    Id State Healthy User Date Name Group
    6 Running yes streamsadmin 2015-04-30T18:32:48-0400 application::TradesAppMain_6 default
	
