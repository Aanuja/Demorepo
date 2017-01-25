# Building Apache Hadoop

The instructions provided below specify the steps to build Apache Hadoop 2.7.3 on IBM z Systems for following distributions:
* RHEL (6.8)
* SLES 11 SP4
* SLES (12, 12 SP2)
* Ubuntu (16.04)

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
			
  * SLES 11 SP4
	
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
		
  * SLES 12 SP2
	
	*  With IBM SDK   
	     
           ```
           sudo zypper install -y wget java-1.8.0-ibm java-1.8.0-ibm-devel
           ```
		
* Ubuntu 16.04
	
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
               Build LevelDB JNI jar to support s390x using below instructions._

  * Download and configure Snappy
      
    ```
    cd /<source_root>/
    wget https://github.com/google/snappy/releases/download/1.1.3/snappy-1.1.3.tar.gz
    tar -zxvf  snappy-1.1.3.tar.gz
    export SNAPPY_HOME=`pwd`/snappy-1.1.3
    cd ${SNAPPY_HOME}
    ./configure --disable-shared --with-pic
    make
    ```

  * Download the source code for LevelDB & LevelDB JNI

    ```
    cd /<source_root>/
    git clone https://github.com/google/leveldb.git
    git clone https://github.com/fusesource/leveldbjni.git
    ```

  * Set the environment variables

    ```
    export LEVELDB_HOME=`pwd`/leveldb
    export LEVELDBJNI_HOME=`pwd`/leveldbjni
    export LIBRARY_PATH=${SNAPPY_HOME}
    export C_INCLUDE_PATH=${LIBRARY_PATH}
    export CPLUS_INCLUDE_PATH=${LIBRARY_PATH}
    ```            

  * Apply the LevelDB patch

    ```
    cd ${LEVELDB_HOME}
    git apply ${LEVELDBJNI_HOME}/leveldb.patch
    ```

*  Modify the below file as per the diff contents
   
   Edit file `${LEVELDB_HOME}/port/atomic_pointer.h`

```diff
@@ -36,6 +36,8 @@
 #define ARCH_CPU_X86_FAMILY 1
 #elif defined(__ARMEL__)
 #define ARCH_CPU_ARM_FAMILY 1
+#elif defined(__s390x__) || defined(__s390__)
+#define ARCH_CPU_S390_FAMILY 1
 #elif defined(__ppc__) || defined(__powerpc__) || defined(__powerpc64__)
 #define ARCH_CPU_PPC_FAMILY 1
 #endif
@@ -50,6 +52,14 @@ namespace port {
 // http://msdn.microsoft.com/en-us/library/ms684208(v=vs.85).aspx
 #define LEVELDB_HAVE_MEMORY_BARRIER

+// S390
+#elif defined(ARCH_CPU_S390_FAMILY)
+
+inline void MemoryBarrier() {
+  asm volatile("bcr 15,0" : : : "memory");
+}
+#define LEVELDB_HAVE_MEMORY_BARRIER
+
 // Gcc on x86
 #elif defined(ARCH_CPU_X86_FAMILY) && defined(__GNUC__)
 inline void MemoryBarrier() {
@@ -217,6 +227,7 @@ class AtomicPointer {
 #undef ARCH_CPU_X86_FAMILY
 #undef ARCH_CPU_ARM_FAMILY
 #undef ARCH_CPU_PPC_FAMILY
+#undef ARCH_CPU_S390_FAMILY

 }  // namespace port
 }  // namespace leveldb
``` 

  * Configure LevelDB and LevelDB JNI 1.8

    ```
    make libleveldb.a
    cd ${LEVELDBJNI_HOME}
    git checkout leveldbjni-1.8
    mkdir leveldbjni-linux64-s390x
    cd leveldbjni-linux64-s390x
    ```

* Modify the below file as per the diff contents

   Edit file `${LEVELDBJNI_HOME}/leveldbjni-all/pom.xml`

```diff
@@ -84,7 +84,13 @@
       <version>1.8</version>
       <scope>provided</scope>
     </dependency>
-
+   <dependency>
+      <groupId>org.fusesource.leveldbjni</groupId>
+      <artifactId>leveldbjni-linux64-s390x</artifactId>
+      <version>1.8</version>
+      <scope>provided</scope>
+    </dependency>
+
   </dependencies>

   <build>
@@ -119,7 +125,8 @@
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

  * Create file `${LEVELDBJNI_HOME}/leveldbjni-linux64-s390x/pom.xml` with the below contents 
  
```diff
@@ -0,0 +1,113 @@
+<?xml version="1.0" encoding="UTF-8"?>
+<!--
+       Copyright (C) 2011, FuseSource Corp.  All rights reserved.
+      http://fusesource.com
+  Redistribution and use in source and binary forms, with or without
+  modification, are permitted provided that the following conditions are
+  met:
+     * Redistributions of source code must retain the above copyright
+  notice, this list of conditions and the following disclaimer.
+     * Redistributions in binary form must reproduce the above
+  copyright notice, this list of conditions and the following disclaimer
+  in the documentation and/or other materials provided with the
+  distribution.
+     * Neither the name of FuseSource Corp. nor the names of its
+  contributors may be used to endorse or promote products derived from
+  this software without specific prior written permission.
+  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
+  "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
+  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
+  A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
+  OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
+  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
+  LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
+  DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
+  THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
+  (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
+  OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
+-->
+<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
+
+  <modelVersion>4.0.0</modelVersion>
+  <parent>
+    <groupId>org.fusesource.leveldbjni</groupId>
+    <artifactId>leveldbjni-project</artifactId>
+    <version>1.8</version>
+  </parent>
+
+  <groupId>org.fusesource.leveldbjni</groupId>
+  <artifactId>leveldbjni-linux64-s390x</artifactId>
+  <version>1.8</version>
+
+  <name>${project.artifactId}</name>
+  <description>The leveldbjni linux 64 native libraries on s390x machines</description>
+
+  <dependencies>
+    <dependency>
+      <groupId>org.fusesource.leveldbjni</groupId>
+      <artifactId>leveldbjni</artifactId>
+      <version>1.8</version>
+    </dependency>
+  </dependencies>
+
+  <build>
+    <testSourceDirectory>${basedir}/../leveldbjni/src/test/java</testSourceDirectory>
+
+    <plugins>
+      <plugin>
+        <groupId>org.apache.maven.plugins</groupId>
+        <artifactId>maven-jar-plugin</artifactId>
+        <version>2.3.1</version>
+        <configuration>
+          <classesDirectory>${basedir}/target/generated-sources/hawtjni/lib</classesDirectory>
+        </configuration>
+      </plugin>
+      <plugin>
+        <groupId>org.fusesource.hawtjni</groupId>
+        <artifactId>maven-hawtjni-plugin</artifactId>
+        <version>${hawtjni-version}</version>
+        <executions>
+          <execution>
+            <goals>
+              <goal>build</goal>
+            </goals>
+          </execution>
+        </executions>
+        <configuration>
+          <platform>linux64/s390x</platform>
+          <name>leveldbjni</name>
+          <classified>false</classified>
+          <nativeSrcDependency>
+            <groupId>org.fusesource.leveldbjni</groupId>
+           <artifactId>leveldbjni</artifactId>
+            <version>${project.version}</version>
+            <classifier>native-src</classifier>
+            <type>zip</type>
+          </nativeSrcDependency>
+          <configureArgs>
+            <arg>--with-leveldb=${env.LEVELDB_HOME}</arg>
+            <arg>--with-snappy=${env.SNAPPY_HOME}</arg>
+            <arg>CFLAGS=-m64</arg>
+            <arg>CXXFLAGS=-m64</arg>
+          </configureArgs>
+        </configuration>
+      </plugin>
+      <plugin>
+        <groupId>org.apache.maven.plugins</groupId>
+        <artifactId>maven-surefire-plugin</artifactId>
+        <version>2.4.3</version>
+        <configuration>
+          <redirectTestOutputToFile>true</redirectTestOutputToFile>
+          <forkMode>once</forkMode>
+          <argLine>-d64 -ea</argLine>
+          <failIfNoTests>false</failIfNoTests>
+          <workingDirectory>${project.build.directory}</workingDirectory>
+          <includes>
+            <include>**/*Test.java</include>
+          </includes>
+        </configuration>
+      </plugin>
+    </plugins>
+  </build>
+
+</project>
```

*  Modify the below file as per the diff contents
   
   Edit file `${LEVELDBJNI_HOME}/pom.xml`

```diff
@@ -249,6 +249,7 @@
         <module>leveldbjni-win32</module>
         <module>leveldbjni-win64</module>
         <module>leveldbjni-all</module>
+        <module>leveldbjni-linux64-s390x</module>
       </modules>
     </profile>

@@ -298,6 +299,13 @@
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
 </project>
```

 * Build the jar file and place it in destination folder

   ```
   mvn clean install -P download -Plinux64-s390x –DskipTests
   mvn clean install -P download -Plinux64,all -DskipTests
cp ${LEVELDBJNI_HOME}/leveldbjni/leveldbjni-all/target/leveldbjni-all-1.8.jar /<source_root>/.m2/repository/org/fusesource/leveldbjni/leveldbjni-all/1.8/leveldbjni-all-1.8.jar
   ```

##Step 2: Testing (Optional)  

  ```
  mvn test -fn
  ```
