# Integrating SPL operators with the Java Application API

Since the applications written with the Java Application API are capable of running on the IBM Streams platform, it's natural that the API would integrate with SPL primitive operators and toolkits. IBM Streams comes with a number of toolkits that provide functionality such as text analysis, HDFS integration, and GeoSpatial processing. Furthermore, if you've worked with IBM Streams in the past, it's possible that you've implemented your own toolkits that you'd like to utilize. 

The purpose of this guide is to demonstrate the basics of interfacing the Java Application API with such toolkits. This is important not only for backward compatibility, but also because it shows that that API can interact with C++ operators in addition to Java ones. Although it isn't assumed that the reader has an understanding of SPL and the structure of toolkits, consulting [the IBM Knowledge Center](http://www-01.ibm.com/support/knowledgecenter/SSCRJU_4.0.0/com.ibm.streams.dev.doc/doc/creating_toolkits.html?lang=en) may prove informative.

# A small bit of background on toolkits and SPL primitive operators
Before the Java Application API was released, developing IBM Streams was a two-step process that involved defining C++ or Java *primitive operators* (which manipulated data), and then subsequently connecting them together into an application by using a language called SPL. Since, as a java developer, you may not want to learn an entirely new programming language to build your application, the Java Application API presents a useful alternative.

Yet the primitive operators are still very useful. For one thing, many primitive operators are organized into *toolkits* which come packaged with any IBM Streams release to provide tools for machine learning, statistical analysis, and pattern recognition. In addition, since primitive operators can be written in C++, a developer using Java Application API can have certain portions of the application written in C++ if so desired.

In this tutorial, we will not cover [the development of C++ or Java primitive operators](http://www-01.ibm.com/support/knowledgecenter/SSCRJU_4.0.0/com.ibm.streams.dev.doc/doc/developing_primitive_operators.html?lang=en), however the process for utilizing an already existing toolkit is outlined below.
# A sample toolkit and operator
To begin, suppose that we have created a toolkit of the following structure in the home directory:
``` bash
$ cd ~/
$ tree
|--  ./myTk
|   |--  ./myTk/myPackageName
|   |   |-- ./myTk/myPackageName/myOperatorName
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_h.pm
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_h.cgt
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_cpp.cgt
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName.xml
|   |   |   |-- ./myTk/myPackageName/myOperatorName/myOperatorName_cpp.pm
|   |--  ./myTk/toolkit.xml
```

In the *myTk* toolkit, there is one package named *myPackageName*, and one operator named *myOperatorName*. As such, when the toolkit is included into the application, the full path of the operator will be **myPackageName::myOperatorName**. The *myOperatorName* operator itself is very simple:
``` c++
#pragma SPL_NON_GENERIC_OPERATOR_HEADER_PROLOGUE

class MY_OPERATOR : public MY_BASE_OPERATOR 
{
public:
  // Constructor
  MY_OPERATOR();
    
  // Tuple processing for mutating ports 
  void process(Tuple & tuple, uint32_t port);

  // Punctuation processing
  void process(Punctuation const & punct, uint32_t port);

private:
  // Members
}; 

#pragma SPL_NON_GENERIC_OPERATOR_HEADER_EPILOGUE
```
