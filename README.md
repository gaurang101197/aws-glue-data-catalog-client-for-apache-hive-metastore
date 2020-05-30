## AWS Glue Data Catalog Client for Apache Hive Metastore
The AWS Glue Data Catalog is a fully managed, Apache Hive Metastore compatible, metadata repository. Customers can use the Data Catalog as a central repository to store structural and operational metadata for their data.

AWS Glue provides out-of-box integration with Amazon EMR that enables customers to use the AWS Glue Data Catalog as an external Hive Metastore. To learn more, visit our [documentation](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hive-metastore-glue.html).

This is an open-source implementation of the Apache Hive Metastore client on Amazon EMR clusters that uses the AWS Glue Data Catalog as an external Hive Metastore. It serves as a reference implementation for building a Hive Metastore-compatible client that connects to the AWS Glue Data Catalog. It may be ported to other Hive Metastore-compatible platforms such as other Hadoop and Apache Spark distributions.

**Note**: in order for this client implementation to be used with Apache Hive, a patch included in this [JIRA](https://issues.apache.org/jira/browse/HIVE-12679) must be applied to it. All versions of Apache Hive running on Amazon EMR that support the AWS Glue Data Catalog as the metastore already include this patch.

## Patching Apache Hive and Installing It Locally

Obtain a copy of Hive from GitHub at https://github.com/apache/hive.  

	git clone https://github.com/apache/hive.git

To build the Hive client, you need to first apply this [patch](https://issues.apache.org/jira/secure/attachment/12958418/HIVE-12679.branch-2.3.patch).  Download this patch and move it to your local Hive git repository you created above. Apply the patch and build Hive.

	git checkout tags/rel/release-2.3.5 -b feature/hive-with-glue-2.3.5
	wget https://issues.apache.org/jira/secure/attachment/12958418/HIVE-12679.branch-2.3.patch
	patch -p0 <HIVE-12679.branch-2.3.patch
	sudo apt install maven #(if maven is not installed, assuming ste-up is done on ubuntu)
	mvn clean install -Pdist -DskipTests

If you are using the default Maven settings, this will install a new version of patched Hive in ~/.m2/repositories/, i.e. ~/.m2/repository/org/apache/hive/hive/2.3.5/. And tar ball will be generated at $hive_directory/packaging/target/apache-hive-2.3.5-bin.tar.gz

## Building the Hive Client

Once you have successfully patched and installed Hive locally, move into the AWS Glue Data Catalog Client repository and update the following property in pom.xml.

	<hive2.version>2.3.5</hive2.version>
	<spark-hive.version>1.2.1.spark2</spark-hive.version>

You are now ready to build the Hive client. Refer these links if you find any issue in building client jar, follow these links [link1](https://github.com/awslabs/aws-glue-data-catalog-client-for-apache-hive-metastore/issues/21) [link2](https://github.com/awslabs/aws-glue-data-catalog-client-for-apache-hive-metastore/pull/14) [link3](https://github.com/mitochon/aws-glue-data-catalog-client-for-apache-hive-metastore/tree/update-readme).

	cd aws-glue-datacatalog-hive2-client
	mvn clean package -DskipTests

([ERROR] Failed to execute goal on project aws-glue-datacatalog-hive2-client: Could not resolve dependencies for project com.amazonaws.glue:aws-glue-datacatalog-hive2-client:jar:1.10.0-SNAPSHOT: The following artifacts could not be resolved: com.amazonaws.glue:aws-glue-datacatalog-client-common:jar:1.10.0-SNAPSHOT, com.amazonaws.glue:aws-glue-datacatalog-client-common:jar:tests:1.10.0-SNAPSHOT: Could not find artifact com.amazonaws.glue:aws-glue-datacatalog-client-common:jar:1.10.0-SNAPSHOT -> [Help 1])

Resolution
	
	mvn clean install -DskipTests (in shims/common)
	mvn clean install -DskipTests (in shims)
	mvn clean install -DskipTests (in aws-glue-datacatalog-client-common)
	mvn clean install -DskipTests (in aws-glue-data-catalog-client-for-apache-hive-metastore) (may be this step may fail due to compilation error)
	mvn clean install -DskipTests (in aws-glue-datacatalog-hive2-client)

## Building the Spark Client

As Spark uses a fork of Hive based off the 1.2.1 branch, in order to build the Spark client, you need Hive 1.2 built with this [patch](https://issues.apache.org/jira/secure/attachment/12958417/HIVE-12679.branch-1.2.patch).  Unlike Hive 2.x, Hive 1.x must be built with a Maven profile set to either "hadoop-1" or "hadoop-2".

	cd <your local Hive repo>
	git checkout branch-1.2
	patch -p0 <HIVE-12679.branch-1.2.patch
	mvn clean install -DskipTests -Phadoop-2

Go back to the AWS Glue Data Catalog Client repository and update the following property in pom.xml to match the version of Hive you just patched and installed locally.  Presently, the latest version in the 1.2 branch (branch-1.2) is "1.2.3-SNAPSHOT".

	<spark-hive.version>1.2.3-SNAPSHOT</spark-hive.version>

You are now ready to build the Spark client.

	cd aws-glue-datacatalog-spark-client
	mvn clean package -DskipTests

If you have both versions of Hive patched and installed locally, you can build both of these clients from the root directory of the AWS Glue Data Catalog Client repository.

## Configuring Hive to Use the Hive Client

	cp $hive_directory/packaging/target/apache-hive-2.3.5-bin.tar.gz ~/
	cd ~/
	tar -xzvf apache-hive-2.3.5-bin.tar.gz
	cd apache-hive-2.3.5-bin

open ~/.bashrc and add these lines at end 	

	export HIVE_HOME=/path/to/apache-hive-2.3.5-bin
	export PATH=$PATH:$HIVE_HOME/bin
	
reload bash by running

	`source ~/.bashrc`
	
add client jar and other aws dependencies to hive auxlib directory, if auxlib directory not present then create one by running `mkdir $HIVE_HOME/auxlib`
	
	cd $HIVE_HOME/auxlib/
	cp /path/to/aws-glue-datacatalog-hive2-client-1.10.0-SNAPSHOT.jar .
	wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-s3/1.11.566/aws-java-sdk-s3-1.11.566.jar (optional)
	wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-glue/1.11.566/aws-java-sdk-glue-1.11.566.jar
	wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-core/1.11.566/aws-java-sdk-core-1.11.566.jar
	wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.566/aws-java-sdk-1.11.566.jar
	wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.8.5/hadoop-aws-2.8.5.jar
	
Add client and other property to `$HIVE_HOME/conf/hive-site.xml` (if hive-site.xml is not present then create one.)
	
	<configuration>
	  <property>
	    <name>fs.s3.awsAccessKeyId</name>
	    <value>awsAccessKeyId</value>
	  </property>
	  <property>
	    <name>fs.s3.awsSecretAccessKey</name>
	    <value>awsSecretAccessKey</value>
	  </property>
	  <property>
	    <name>fs.defaultFS</name>
	    <value>s3://defaultFS</value>
	    <final>true</final>
	  </property>
	  <property>
	    <name>hive.server2.thrift.port</name>
	    <value>10000</value>
	  </property>
	  <property>
	    <name>datanucleus.fixedDatastore</name>
	    <value>true</value>
	  </property>
	  <property>
	    <name>hive.metastore.connect.retries</name>
	    <value>15</value>
	  </property>
	  <property>
	    <name>hive.server2.thrift.http.port</name>
	    <value>10001</value>
	  </property>
	  <property>
	    <name>hive.imetastoreclient.factory.class</name>
	    <value>com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory</value>
	  </property>
	</configuration>

S3 integration in hive is optional. Without adding s3 properties in hive-site.xml, hive can still able to connect to glue metastore. Run below commands to start hive server.
To start hive server, HADOOP_HOME must be set, Follow [this](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/images/emr-releases-5x.png) to figure out appropriate aws-sdk and hadoop versions.

HADOOP set-up
	
	cd ~/
	wget https://archive.apache.org/dist/hadoop/core/hadoop-2.8.5/hadoop-2.8.5.tar.gz
	tar -xzvf hadoop-2.8.5.tar.gz
	
Open ~/.bashrc and add following lines at the end to set up hadoop home.

	export HADOOP_HOME=/home/ubuntu/hive-with-glue/hadoop-2.8.5
	export PATH=$PATH:HADOOP_HOME/bin

Reload bashrc file by running `source ~/.bashrc`, to make HADOOP_HOME variable available.
Now as we have added properties and required dependencies, we can start hive server.
	
	$HIVE_HOME/bin/schematool -dbType derby -initSchema
	hive --service metastore -p 9083 --hiveconf hive.root.logger=INFO,console
	hiveserver2 --hiveconf hive.metastore.uris=thrift://localhost:9083 --hiveconf hive.root.logger=INFO,console
	$HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000
	
Try querying `show databases`, or `show tables` in beeline to verify.

## Configuring Spark to Use the Spark Client

Similarly, for Spark, you need to install the client jar in Spark's CLASSPATH and create or update Spark's own hive-site.xml to add the above property.  On Amazon EMR, this is set in /usr/lib/spark/conf/hive-site.xml.  You can also find the location of the Spark client jar in /usr/lib/spark/conf/spark-defaults.conf.

## Enabling client side caching for catalog

Currently, we provide support for caching:

a) Table metadata - Response from Glue's GetTable operation (https://docs.aws.amazon.com/glue/latest/webapi/API_GetTable.html#API_GetTable_ResponseSyntax)

b) Database metadata - Response from Glue's GetDatabase operation (https://docs.aws.amazon.com/glue/latest/webapi/API_GetDatabase.html#API_GetDatabase_ResponseSyntax)

Both these entities have dedicated caches for themselves and can be enabled/tuned individually.

To enable/tune Table cache, use the following properties in your hive/spark configuration file:

	<property>
 		<name>aws.glue.cache.table.enable</name>
 		<value>true</value>
	</property>
	<property>
 		<name>aws.glue.cache.table.size</name>
 		<value>1000</value>
	</property>
	<property>
 		<name>aws.glue.cache.table.ttl-mins</name>
 		<value>30</value>
	</property>

To enable/tune Database cache:

	<property>
 		<name>aws.glue.cache.db.enable</name>
 		<value>true</value>
	</property>
	<property>
 		<name>aws.glue.cache.db.size</name>
 		<value>1000</value>
	</property>
	<property>
 		<name>aws.glue.cache.db.ttl-mins</name>
 		<value>30</value>
	</property>

NOTE: The caching logic is disabled by default. Also, there is no one-size-fits-all value for the cache size and ttl; feel free to tune these values as per your use case.

## License

This library is licensed under the Apache 2.0 License. 
