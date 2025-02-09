# Apache Zeppelin<a name="emr-zeppelin"></a>

Use Apache Zeppelin as a notebook for interactive data exploration\. For more information about Zeppelin, see [https://zeppelin\.apache\.org/](https://zeppelin.apache.org/)\. Zeppelin is included in Amazon EMR release versions 5\.0\.0 and later\. Earlier release versions include Zeppelin as a sandbox application\. For more information, see [Amazon EMR 4\.x release versions](emr-release-4x.md)\.

To access the Zeppelin web interface, set up an SSH tunnel to the master node and a proxy connection\. For more information, see [View web interfaces hosted on EMR clusters](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html)\.

The following table lists the version of Zeppelin included in the latest release of the Amazon EMR 6\.x series, along with the components that Amazon EMR installs with Zeppelin\.

For the version of components installed with Zeppelin in this release, see [Release 6\.5\.0 Component Versions](emr-650-release.md)\.


**Zeppelin version information for emr\-6\.5\.0**  

| Amazon EMR Release Label | Zeppelin Version | Components Installed With Zeppelin | 
| --- | --- | --- | 
| emr\-6\.5\.0 | Zeppelin 0\.10\.0 | aws\-sagemaker\-spark\-sdk, emrfs, emr\-goodies, hadoop\-client, hadoop\-hdfs\-datanode, hadoop\-hdfs\-library, hadoop\-hdfs\-namenode, hadoop\-httpfs\-server, hadoop\-kms\-server, hadoop\-yarn\-nodemanager, hadoop\-yarn\-resourcemanager, hadoop\-yarn\-timeline\-server, livy\-server, r, spark\-client, spark\-history\-server, spark\-on\-yarn, spark\-yarn\-slave, zeppelin\-server | 

The following table lists the version of Zeppelin included in the latest release of the Amazon EMR 5\.x series, along with the components that Amazon EMR installs with Zeppelin\.

For the version of components installed with Zeppelin in this release, see [Release 5\.34\.0 Component Versions](emr-5340-release.md)\.


**Zeppelin version information for emr\-5\.34\.0**  

| Amazon EMR Release Label | Zeppelin Version | Components Installed With Zeppelin | 
| --- | --- | --- | 
| emr\-5\.34\.0 | Zeppelin 0\.10\.0 | aws\-sagemaker\-spark\-sdk, emrfs, emr\-goodies, hadoop\-client, hadoop\-hdfs\-datanode, hadoop\-hdfs\-library, hadoop\-hdfs\-namenode, hadoop\-httpfs\-server, hadoop\-kms\-server, hadoop\-yarn\-nodemanager, hadoop\-yarn\-resourcemanager, hadoop\-yarn\-timeline\-server, livy\-server, r, spark\-client, spark\-history\-server, spark\-on\-yarn, spark\-yarn\-slave, zeppelin\-server | 

**Topics**
+ [Considerations when using Zeppelin on Amazon EMR](zeppelin-considerations.md)
+ [Zeppelin release history](Zeppelin-release-history.md)