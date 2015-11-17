# Integrating SPL operators with the Java Application API

Since the applications written with the Java Application API are capable of running on the IBM Streams platform, it's natural that the API would integrate with SPL primitive operators and toolkits. IBM Streams comes with a number of toolkits that provide functionality such as text analysis, HDFS integration, and GeoSpatial processing. Furthermore, if you've worked with IBM Streams in the past, it's possible that you've implemented your own toolkits that you'd like to utilize. 

The purpose of this guide is to demonstrate the basics of interfacing the Java Application API with such toolkits. This is important not only for backward compatibility, but also because it shows that that API can interact with C++ operators in addition to Java ones. Although it isn't assumed that the reader has an understanding of SPL and the structure of toolkits, consulting [the IBM Knowledge Center](http://www-01.ibm.com/support/knowledgecenter/SSCRJU_4.0.0/com.ibm.streams.dev.doc/doc/creating_toolkits.html?lang=en) may prove informative.

# A sample toolkit and operator
To begin, suppose that we have created a toolkit of the following structure in the home directory:
``` bash
$ cd ~/
$ tree
|--  ./myTk
|   |--  ./myTk/myPackageName
|   |   |-- ./myTk/myPackageName/myOperatorName
|   |   |   |-- ./myTk/myPackageName/myOperatorName/
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_h.pm
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_h.cgt
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_cpp.cgt
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName.xml
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_cpp.pm
|   |--  ./myTk/toolkit.xml
```
