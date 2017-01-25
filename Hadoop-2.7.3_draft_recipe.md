# Building Apache Hadoop

The instructions provided below specify the steps to build Apache Hadoop 2.7.3 on IBM z Systems for RHEL 6.8, SLES 11-SP4, SLES 12/12-SP2, Ubuntu 16.04.

_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

### Prerequisites 
 
Apache Hadoop requires Google Protobuf 2.5.0, please see [Building Google Protobuf 2.5.0](https://github.com/linux-on-ibm-z/docs/wiki/Building-Google-Protobuf-2.5.0) for more information.

##Step 1: Building and Installing Apache Hadoop  
####1.1) Install dependencies
  * RHEL 6.8
  
    * With IBM SDK   

	   Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  
          ```
          sudo yum install -y wget
          ```
			
  * SLES 11-SP4
	
	*  With IBM SDK   

	   Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  
           ```
           sudo zypper install -y wget
           ```
			
  * SLES 12
	
	*  With IBM SDK   

	   Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  
           ```
           sudo zypper install -y wget
           ```
		
  * SLES 12-SP2
	
	*  With IBM SDK   
	     
           ```
           sudo zypper install -y wget java-1.8.0-ibm java-1.8.0-ibm-devel
           ```
		
* Ubuntu 16.04/16.10
	
	*  With IBM SDK   

	   Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  
           ```
           sudo apt-get update
	   sudo apt-get install -y git wget maven cmake make
           ```

	*  With Open JDK
		
	   ```
	   sudo apt-get update
	   sudo apt-get install openjdk-8-jdk cmake make pkg-config maven wget
	   ```

####1.2) Install maven ( For RHEL and SLES only)

  ```
  cd /<source_root>/
  wget http://www.eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
  tar -xvzf apache-maven-3.3.9-bin.tar.gz
  export PATH=$PATH:/home/test/apache-maven-3.3.9/bin/
  ```
		
####1.3) Download source code

  ```
  cd /<source_root>/
  wget https://github.com/apache/hadoop/archive/rel/release-2.7.3.tar.gz
  tar zxvf release-2.7.3.tar.gz
  cd hadoop-rel-release-2.7.3/
  ```

####1.4) Modify the below file as per the diff contents
  Edit `line 360` of file `hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/security/UserGroupInformation.java` 

```diff
-    System.getProperty("os.arch").contains("64");
+    System.getProperty("os.arch").contains("64") || System.getProperty("os.arch").contains("s390x");
``` 

####1.5) Set Environment variables
    
  ```
  export MAVEN_OPTS="-Xmx2048m -XX:MaxPermSize=1024m"
  export JAVA_HOME=<path_to_java>
  export PATH=$JAVA_HOME/bin:$PATH 
  ```
  
####1.6) Run Maven to Create binary distribution with native code
    
  ```
  mvn clean -fn
  mvn install -DskipTests -fn
  ```
  _**Note:**  With IBM SDK, compilation errors might be seen for hadoop-yarn-registry project. Apply [this patch](https://issues.apache.org/jira/secure/attachment/12708708/HADOOP-11783-1.patch) to resolve the same._
    
####1.7) Build LevelDB JNI jar
_**Note:** Few test failure are seen as the downloaded LevelDB JNI jar is not compatible with s390x.
               The below steps gives steps to build LevelDB JNI jar to support s390x._

  * Download and configure Snappy
      
    ```
    cd /<source_root>/
    wget https://github.com/google/snappy/releases/download/1.1.3/snappy-1.1.3.tar.gz
    tar -zxvf  snappy-1.1.3.tar.gz
    export SNAPPY_HOME=`cd snappy-1.1.3; pwd`
    cd ${SNAPPY_HOME}
    ./configure --disable-shared --with-pic
    make
    ```

  * Download the source code for LevelDB & LevelDB JNI

    ```
    cd /<source_root>/
    git clone git://github.com/chirino/leveldb.git
    git clone git://github.com/fusesource/leveldbjni.git
    ```

  * Set the environment variables

    ```
    export LEVELDB_HOME=`cd leveldb; pwd`
    export LEVELDBJNI_HOME=`cd leveldbjni; pwd`
    export LIBRARY_PATH=${SNAPPY_HOME}
    export C_INCLUDE_PATH=${LIBRARY_PATH}
    export CPLUS_INCLUDE_PATH=${LIBRARY_PATH}
    ```            

  * Apply the LevelDB patch

    ```
    cd ${LEVELDB_HOME}
    git apply ../leveldbjni/leveldb.patch
    ```

*  Modify the below file as per the diff contents
   
   Edit file `./port/atomic_pointer.h`

```diff
#define ARCH_CPU_ARM_FAMILY 1
+    #elif defined(__s390x__) || defined(__s390__)
+    #define ARCH_CPU_S390_FAMILY 1
 #endif
``` 

  * Configure LevelDB

    ```
    make libleveldb.a
    ```

* Check out LevelDB JNI 1.8

   ```
   cd ${LEVELDBJNI_HOME}
   git checkout leveldbjni-1.8
   ```

* Modify the below file as per the diff contents

   Edit file `leveldbjni-all/pom.xml`

```diff
</dependency>
+    <dependency>
+      <groupId>org.fusesource.leveldbjni</groupId>
+      <artifactId>leveldbjni-linux64-s390x</artifactId>
+      <version>1.8</version>
+      <scope>provided</scope>
+    </dependency>

   </dependencies>
META-INF/native/osx/libleveldbjni.jnilib;osname=macosx;processor=x86,
               META-INF/native/osx/libleveldbjni.jnilib;osname=macosx;processor=x86-64,
               META-INF/native/linux32/libleveldbjni.so;osname=Linux;processor=x86,
-              META-INF/native/linux64/libleveldbjni.so;osname=Linux;processor=x86-64
+              META-INF/native/linux64/libleveldbjni.so;osname=Linux;processor=x86-64,
+              META-INF/native/linux64/s390x/libleveldbjni.so;osname=Linux;processor=s390x
             </Bundle-NativeCode>
          </instructions>
         </configuration>
```

  * Create the LevelDB JNI directory

    ```
    mkdir leveldbjni-linux64-s390x
    cd leveldbjni-linux64-s390x
    ```

  * Create the new pom.xml for the above . verify if s390x entries present in the below listed files.

    `../leveldbjni/src/main/native-package/configure`
    `../leveldbjni/src/main/native-package/m4/libtool.m4`

*  Modify the below file as per the diff contents
   
   Edit file `../pom.xml`

```diff
<module>leveldbjni-all</module>
+       <module>leveldbjni-linux64-s390x</module>
       </modules>


<module>leveldbjni-win64</module>
       </modules>
     </profile>
+
+    <profile>
+      <id>linux64-s390x</id>
+      <modules>
+        <module>leveldbjni-linux64-s390x</module>
+      </modules>
+    </profile>

   </profiles>
```

 * Build the jar file and place it in destination folder

   ```
   mvn clean install -P download -Plinux64-s390x –DskipTests
   mvn clean install -P download -Plinux64,all -DskipTests
cp /<source_root>/leveldbjni/leveldbjni-all/target/leveldbjni-all-1.8.jar /<source_root>/.m2/repository/org/fusesource/leveldbjni/leveldbjni-all/1.8/leveldbjni-all-1.8.jar
   ```

##Step 2: Testing (Optional)  

  ```
  mvn test -fn
  ```