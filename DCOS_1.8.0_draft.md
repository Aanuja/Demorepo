# Building DC/OS
DC/OS master branch has been successfully built on Linux on z Systems. The following instructions can be used on RHEL 7.3.

#### General Notes: ####
* When following the steps below please run as a root user.

* A directory  ```/<source_root>/ ``` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.

### Prerequisites: ###
*	**git 1.8.5** or above -- Instructions for building git can be found [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

 
*	 **Docker** -- Instructions for building Docker can be found [here](http://www.ibm.com/developerworks/linux/linux390/docker.html).

	
*	 **Python 3.4** or above -- [Instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-3.4.3).
	
	
*	 **s390x/ubuntu image** -- Create a container with the following s390x/ubuntu image installed on it.

    Install the following dependencies:
    
	```
	apt-get update
	apt-get install -y wget tar xz-utils gcc make sudo patch tcl-tls libreadline-dev libssl-dev python-lzma libncurses5-dev libncurses5 ncurses-base
	```
	
	**Python 3.4** or above -- Instructions for building Python 3.4.3 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-3.4.3).
	
	```
	ln -s /usr/local/bin/pip3.4 /usr/local/bin/pip
	pip --version
	```
	
       Save the container as s390x/ubuntu image.
	
	```
	docker commit <container-id> s390x/ubuntu
	```
	
*	 **golang 1.5.2 image** or above -- Create a golang image using the below contents.

  _**Notes:**_ 
 
        ---   this is not verified ----
	For RHEL 7.2 we need to install 'make', as well as install git 2.2.1 from source in the golang image as a dependency.
	To create an image of the golang container do the following for example:
	```
	docker commit <container-id> golang:1.6
	```
        
	--   this is not verified ----
	
	Add the below content to the dockerfile.
	```
	FROM s390x/ubuntu
	
	RUN apt-get -qq update && apt-get -y install make \
        wget \
        tar \
        git && \
        mkdir gosrc

        WORKDIR gosrc
        RUN wget https://storage.googleapis.com/golang/go1.7.5.linux-s390x.tar.gz && \
        chmod ugo+r go1.7.5.linux-s390x.tar.gz && \
        tar -C /usr/local -xzf go1.7.5.linux-s390x.tar.gz

        ENV PATH $PATH:/usr/local/go/bin
        ENV GOPATH /go
        RUN  go version
	```
	
	Build the dockerfile
	```
	docker build -t golang:1.6 .
	```

*	 **jplock/zookeeper image** -- Create a jplock/zookeeper image using the below contents.
	
	Add the below content to the dockerfile.
	```
	FROM s390x/ubuntu

	ARG MIRROR=http://apache.mirrors.pair.com
	ARG VERSION=3.4.8

	LABEL name="zookeeper" version=$VERSION

	RUN apt-get update -y
	RUN apt-get upgrade -y
	RUN apt-get install -y openjdk-8-jdk wget \
    && wget -q -O - $MIRROR/zookeeper/zookeeper-$VERSION/zookeeper-$VERSION.tar.gz | tar -xzf - -C /opt \
    && mv /opt/zookeeper-$VERSION /opt/zookeeper \
    && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg \
    && mkdir -p /tmp/zookeeper

	EXPOSE 2181 2888 3888

	WORKDIR /opt/zookeeper

	VOLUME ["/opt/zookeeper/conf", "/tmp/zookeeper"]

	ENTRYPOINT ["/opt/zookeeper/bin/zkServer.sh"]
	CMD ["start-foreground"]
	```
	
	Build the dockerfile
	```
	docker build -t jplock/zookeeper .
	```


*	 **alpine:3.4 image** -- Create a tag for the alpine image using the s390x/ubuntu image as shown below:
	```
	docker tag s390x/ubuntu alpine:3.4
	```

*	 **node:4.4.4 image** -- Create a node image using the below contents.

	Add the below content to the dockerfile.
	```
	# Base image
	FROM s390x/ubuntu

	# The author
	MAINTAINER LoZ Open Source Ecosystem (https://www.ibm.com/developerworks/community/groups/community/lozopensource)

	RUN apt-get update && apt-get install -y git make gcc g++ python clang npm

	RUN apt-get update -y && apt-get install -y flex bison gperf ruby libssl-dev libicu-dev libpng12-dev libjpeg-dev git-core
	RUN git clone git://github.com/ariya/phantomjs.git /phantomjs

	WORKDIR /phantomjs
	RUN git checkout 2.1.1 && python build.py && cp bin/phantomjs /usr/local/bin/

	RUN apt-get install -y autoconf
	RUN apt-get clean

	RUN ln -s /usr/bin/nodejs /usr/bin/node

	CMD ["node"]
	# End of Dockerfile
	```
	
	Build the dockerfile
	```
	docker build -t node:4.4.4 .
	```

### 1. Install dependencies

RHEL 7.3
```
pip install tox
```

### 2. Get the source

```
cd /<source_root>/
git clone https://github.com/linux-on-ibm-z/dcos.git
cd dcos/
git checkout 1.8_s390x
```

### 3. Edit the following files to replace ubuntu:14.04.4 with s390x/ubuntu image
```
 /<source_root>/dcos/pkgpanda/build/__init__.py
 /<source_root>/dcos/pkgpanda/build/tests/resources-nonbootstrapable/single_source_corrupt/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/resources/base/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/resources/single_source/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/resources/single_source_extra/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/resources/url_extract-tar/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/resources/url_extract-zip/buildinfo.json
 /<source_root>/dcos/pkgpanda/build/tests/test_build.py
 /<source_root>/dcos/pkgpanda/docker/dcos-builder/Dockerfile
 /<source_root>/dcos/pkgpanda/build/tests/resources/variant/ee.buildinfo.json
```

### 4. Modify the gen installer Dockerfile as per the diff contents

Edit file `/<source_root>/dcos/gen/installer/bash/Dockerfile.in`

```diff
@@ -2,8 +2,8 @@ FROM alpine:3.4
 MAINTAINER help@dcos.io

 WORKDIR /
-RUN apk add --update curl ca-certificates git openssh tar xz zlib && rm -rf /var/cache/apk/*
-RUN curl -fLsS --retry 20 -Y 100000 -y 60 -o glibc-2.23-r3.apk https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.23-r3/glibc-2.23-r3.apk && apk --allow-
+RUN apt-get update
+RUN apt-get install -y curl ca-certificates git ssh tar xz-utils glibc-source zlib1g-dev zlibc
 VOLUME ["/genconf"]
```
 
### 5. Modify the dcos-builder dockerfile as per the diff contents given below

Edit file `/<source_root>/dcos/pkgpanda/docker/dcos-builder/Dockerfile` 

```diff
@@ -1,4 +1,4 @@
-FROM ubuntu:14.04.4
+FROM s390x/ubuntu
 MAINTAINER help@dcos.io

 RUN apt-get -qq update && apt-get -y install \
@@ -32,7 +32,7 @@ RUN apt-get -qq update && apt-get -y install \
   libpopt-dev \
   libnl-3-dev \
   libnl-genl-3-dev \
-  linux-headers-3.19.0-21-generic \
+  linux-headers-4.4.0-57-generic \
   pkg-config \
```

_**Note:**_ Replace the linux-headers-3.19.* with linux-headers-4.4.0-57-generic(available for s390x/ubuntu).

### 6. Configure, test and build DC/OS

#### 6.1 Run local code quality tests ####
```
cd /<source_root>/dcos
tox
```

_**Note:**_ 
If Test-case failures related to test_version are seen, do the following:

Edit file `/<source_root>/dcos/pytest/test_installer_backend.py` as per the diff contents:

```diff
@@ -23,7 +23,7 @@ def test_version(monkeypatch):
     monkeypatch.setenv('BOOTSTRAP_VARIANT', 'some-variant')
     version_data = subprocess.check_output(['dcos_installer', '--version']).decode()
     assert json.loads(version_data) == {
-        'version': '1.8-dev',
+        'version': '1.8',
         'variant': 'some-variant'
```

#### 6.2 Build DC/OS ####
```
cd /<source_root>/dcos
./build_local.sh
```
_**Notes:**_  _If build failures are seen, refer to the below listed failures and the steps to resolve the same._  

  1. Makefile:284: recipe for target 'build_crypto' failed
     Edit file `/<source_root>/dcos/packages/openssl/build` as per the diff contents
```diff
@@ -10,7 +10,7 @@ pushd "/pkg/src/openssl"
   shared \
   zlib \
   no-krb5 \
-  linux-x86_64 \
+  linux-generic64 \
   -Wa,--noexecstack \
   -O2 -DFORTIFY_SOURCE=2
```

  2. lj_arch.h:53:2: error: #error "No support for this architecture (yet)"
     Edit file `/<source_root>/dcos/packages/adminrouter/buildinfo.json`
```diff
@@ -3,8 +3,8 @@
   "sources" :{
       "nginx": {
           "kind": "url_extract",
-          "url": "https://openresty.org/download/openresty-1.9.15.1.tar.gz",
-          "sha1": "491a84d70ed10b79abb9d1a7496ee57a9244350b"
+          "url": "http://9.12.19.112:8080/openresty-1.9.15.1.tar.gz",
+          "sha1": "7da21a8e94717fd5d3c8b3cab2ed4d69c5b0a77f"
       },
       "adminrouter": {
```
 
  3.  If you encounter a failure with the error 'cp: cannot stat '/lib/x86_64-linux-gnu/libpcre.so.3': No such file or directory' in the adminrouter package modify the file `/<source_root>/dcos/packages/adminrouter/build` as per the diff contents
```diff
@@ -34,7 +34,7 @@ popd

 # Copy in needed libraries.
 mkdir -p "$PKG_PATH/lib"
-cp /lib/x86_64-linux-gnu/libpcre.so.3 "$PKG_PATH/lib/libpcre.so.3"
+cp /lib/s390x-linux-gnu/libpcre.so.3 "$PKG_PATH/lib/libpcre.so.3"

 systemd_master="$PKG_PATH"/dcos.target.wants_master/dcos-adminrouter.service
 mkdir -p "$(dirname "$systemd_master")"
```

  4.  If you get an error in the ncurses module in file lib_gen.c as follows 'error: expected ')' before 'int''. Downgrade ncurses to version 5.7 in file `/<source_root>/dcos/packages/ncurses/buildinfo.json` as shown below
```diff
@@ -2,6 +2,6 @@
   "single_source" : {
     "kind": "url_extract",
     "url": "http://ftp.gnu.org/gnu/ncurses/ncurses-5.7.tar.gz",
-    "sha1": "acd606135a5124905da770803c05f1f20dd3b21c"
+    "sha1": "8233ee56ed84ae05421e4e6d6db6c1fe72ee6797"
   }
 }
```

 5. ERROR Building packages: Validation error when fetching sources for package:Provided sha1 didn't match sha1 of downloaded file, corrupt download for `/<dcos_path>/logrotate-3.9.1.tar.gz.corrupt`.
 Edit file `/<source_root>/dcos/packages/logrotate/buildinfo.json` as per the diff contents
```diff
@@ -1,7 +1,7 @@
 {
   "single_source": {
     "kind": "url_extract",
-    "url": "https://fedorahosted.org/releases/l/o/logrotate/logrotate-3.9.1.tar.gz",
-    "sha1": "3ed28b8b4b9f3a9175a5510b53217a60db8fdf32"
+    "url": "http://pkgs.fedoraproject.org/repo/pkgs/logrotate/logrotate-3.9.1.tar.gz/4492b145b6d542e4a2f41e77fa199ab0/logrotate-3.9.1.tar.gz",
+    "sha1": "7ba734cd1ffa7198b66edc4bca17a28ea8999386"
   }
 }
```

* For 'auto_ptr deprecated error for boost in mesos-modules do the below:
```
$ vi /<source_root>/dcos/packages/mesos/build
```
Add to the configure step to build using --with-boost option as shown below:
```
LIBS="-lssl -lcrypto" LDFLAGS="-L/opt/mesosphere/active/openssl/lib" "/pkg/src/mesos/configure" \
  --prefix="$PKG_PATH" --enable-optimize --disable-python \
  --enable-libevent --enable-ssl \
  --enable-install-module-dependencies \
  --with-ssl=/opt/mesosphere/active/openssl \
  --with-libevent=/opt/mesosphere/active/libevent \
  --with-curl=/opt/mesosphere/active/curl \
  --sbindir="$PKG_PATH/bin" \
  --with-boost=/usr/include/boost
```
Also modify the dcos-builder dockerfile to install libboost-dev.
```
$ vi /<source_root>/dcos/pkgpanda/docker/dcos-builder/Dockerfile
```
Add to the dependency list libboost-dev as shown below:
```
RUN apt-get -qq update && apt-get -y install \
  autoconf \
  .
  .
  gettext-base \
  unzip \
  libboost-dev
```


### 7. Installing DC/OS
Instructions for installing Dc/OS can be found [here](https://dcos.io/docs/1.8/administration/installing/custom/gui).

## References:
https://github.com/dcos/dcos/tree/master/packages

https://dcos.io/docs/1.8/overview/
