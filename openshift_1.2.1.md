<!---PACKAGE:OpenShift Origin--->
<!---DISTRO:RHEL 7.1:1.1.3--->

# Building OpenShift Origin

[OpenShift Origin V3](https://github.com/openshift/origin) is a distribution of Kubernetes optimized for continuous application development and multi-tenant deployment. The instructions provided below specify the steps to build OpenShift Origin 1.2.1 on Linux on the IBM z Systems for RHEL 7.1 and RHEL 7.2.

### _**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building and Installing OpenShift Origin

### Step 1: Install the Dependencies

    sudo yum install git tar

### Step 2: Install Docker 1.8.2 or higher
See instructions [here](http://www.ibm.com/developerworks/linux/linux390/docker.html)

### Step 3: Install Go 1.6

See instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).  

**Special instructions when following the Go recipe**
* After performing the git clone on the Go repository, follow up with this step to get the 1.6 branch

    ```
    git checkout release-branch.go1.6
    ```


### Step 4: Build OpenShift Origin by following steps 1 through 5
See instructions [here](https://github.com/openshift/origin/blob/master/CONTRIBUTING.adoc#develop-locally-on-your-host) to develop locally on your host

####In summary:
     export GOPATH=/<source_root>/go
     export PATH=$PATH:$GOPATH/bin
     export OS_OUTPUT_GOPATH=1
     mkdir -p $GOPATH/src/github.com/openshift
     cd $GOPATH/src/github.com/openshift
     git clone https://github.com/openshift/origin.git
     cd origin
     git checkout tags/v1.3.0 -b v1.3.0
     ##cp $GOPATH/src/github.com/openshift/origin/Godeps/_workspace/src/github.com/boltdb/bolt/bolt_amd64.go $GOPATH/src/github.com/openshift/origin/Godeps/_workspace/src/github.com/boltdb/bolt/bolt_s390x.go
     make clean build

Above steps will generate OpenShift Origin executable in the directory specified in the instructions.  You are now ready to use OpenShift Origin.

__Note:__ In order to use OpenShift Origin you must generate core OpenShift Docker images. Here we provide guidance on how to do that on RHEL 7.1. Feel free to adapt it to your environment.

### Optional: Build required Docker images

1. Build base RHEL image - see [details](http://containerz.blogspot.ca/2015/03/creating-base-images.html#more)

2. Modify dockerfiles and scripts, assuming your current working directory is `$GOPATH/src/github.com/openshift/origin/`
  1. The scripts need to be modified: 
    * `hack/build-images.sh`, add s390x support.
        ```diff
	@@ -12,7 +12,7 @@ source "${OS_ROOT}/contrib/node/install-sdn.sh"

            if [[ "${OS_RELEASE:-}" == "n" ]]; then
            # Use local binaries
       -  imagedir="${OS_OUTPUT_BINPATH}/linux/amd64"
       +  imagedir="${OS_OUTPUT_BINPATH}/linux/s390x"
            # identical to build-cross.sh
            os::build::os_version_vars
            OS_RELEASE_COMMIT="${OS_GIT_SHORT_VERSION}"
       @@ -31,7 +31,7 @@ else
           fi

           # Extract the release achives to a staging area.
       -  os::build::detect_local_release_tars "linux-64bit"
       +  os::build::detect_local_release_tars "linux-s390x"

           echo "Building images from release tars for commit ${OS_RELEASE_COMMIT}:"
           echo " primary: $(basename ${OS_PRIMARY_RELEASE_TAR})"
        ```
    * `hack/common.sh`, change some variables and add s390x support.
        ```diff
        @@ -9,13 +9,14 @@ readonly OS_BUILD_ENV_IMAGE="${OS_BUILD_ENV_IMAGE:-openshift/origin-release:gola
 		readonly OS_OUTPUT_SUBPATH="${OS_OUTPUT_SUBPATH:-_output/local}"
 		readonly OS_OUTPUT="${OS_ROOT}/${OS_OUTPUT_SUBPATH}"
 		readonly OS_LOCAL_RELEASEPATH="${OS_OUTPUT}/releases"
	-readonly OS_OUTPUT_BINPATH="${OS_OUTPUT}/bin"
	+#readonly OS_OUTPUT_BINPATH="${OS_OUTPUT}/bin"
	+readonly OS_OUTPUT_BINPATH="/go/bin"
 		readonly OS_OUTPUT_PKGDIR="${OS_OUTPUT}/pkgdir"

 		readonly OS_GO_PACKAGE=github.com/openshift/origin

 		readonly OS_IMAGE_COMPILE_PLATFORMS=(
	-  linux/amd64
	+  linux/s390x
 		)
 		readonly OS_IMAGE_COMPILE_TARGETS=(
   		images/pod
       @@ -34,6 +35,7 @@ readonly OS_CROSS_COMPILE_PLATFORMS=(
   		darwin/amd64
   		windows/amd64
   		linux/386
	+  linux/s390x
 		)
 		readonly OS_CROSS_COMPILE_TARGETS=(
   		cmd/openshift
       @@ -128,9 +130,11 @@ function os::build::host_platform_friendly() {
   		elif [[ $platform == "darwin/amd64" ]]; then
     			echo "mac"
   		elif [[ $platform == "linux/386" ]]; then
	-    echo "linux-32bit"
	+    echo "linux-386"
   		elif [[ $platform == "linux/amd64" ]]; then
	-    echo "linux-64bit"
	+    echo "linux-amd64"
	+  elif [[ $platform == "linux/s390x" ]]; then
	+    echo "linux-s390x"
   		else
			echo "$(go env GOHOSTOS)-$(go env GOHOSTARCH)"
   		fi
       @@ -319,6 +323,7 @@ os::build::internal::build_binaries() {
       		fi

       		for test in "${tests[@]:+${tests[@]}}"; do
	+        mkdir -p "${OS_OUTPUT_BINPATH}/${platform}"
         	 local outfile="${OS_OUTPUT_BINPATH}/${platform}/$(basename ${test})"
         	 GOOS=${platform%/*} GOARCH=${platform##*/} go test \
           	 -c -o "${outfile}" \
	@@ -442,13 +447,15 @@ function os::build::place_bins() {
         	elif [[ $platform == "linux/386" ]]; then
           		platform="linux/32bit" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
         	elif [[ $platform == "linux/amd64" ]]; then
	-          platform="linux/64bit" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
	-          platform="linux/64bit" OS_RELEASE_ARCHIVE="openshift-origin-server" os::build::archive_tar "${OS_BINARY_RELEASE_SERVER_LINUX[@]}"
	+          platform="linux/amd64" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
	+        elif [[ $platform == "linux/s390x" ]]; then
	+          platform="linux/s390x" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar         "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
	+          platform="linux/s390x" OS_RELEASE_ARCHIVE="openshift-origin-server" os::build::archive_tar         "${OS_BINARY_RELEASE_SERVER_LINUX[@]}"
         	else
           		echo "++ ERROR: No release type defined for $platform"
         	fi
       		else
	-        if [[ $platform == "linux/amd64" ]]; then
	+        if [[ $platform == "linux/s390x" ]]; then
        	 	platform="linux/64bit" os::build::archive_tar "./*"
        	 else
           		echo "++ ERROR: No release type defined for $platform"
        ```
    * `images/router/haproxy/reload-haproxy`, switch to `/usr/local/sbin/haproxy` for the customized haproxy installation.
        ```diff
        @@ -81,7 +81,8 @@ if [ -n "$old_pids" ]; then
     		fi
   		fi

	-  /usr/sbin/haproxy -f $config_file -p $pid_file -sf $old_pids
	+#  /usr/sbin/haproxy -f $config_file -p $pid_file -sf $old_pids
	+    /usr/local/sbin/haproxy -f $config_file -p $pid_file -sf $old_pid
   		reload_status=$?

   		if [[ "$installed_iptables" == 1 ]]; then
	@@ -108,7 +109,8 @@ if [ -n "$old_pids" ]; then
		 fi
   		fi
 		else
	-  /usr/sbin/haproxy -f $config_file -p $pid_file
	+#  /usr/sbin/haproxy -f $config_file -p $pid_file
	+   /usr/local/sbin/haproxy -f $config_file -p $pid_file
   		reload_status=$?
 		fi
        ```
    * `images/router/haproxy/conf/haproxy-config.template`.
        ```diff
        @@ -39,9 +39,11 @@ defaults
              timeout tunnel 1h
 
              {{ if (gt .StatsPort 0) }}
        -listen stats :{{.StatsPort}}
        +listen stats
        +    bind :{{.StatsPort}}
              {{ else }}
        -listen stats :1936
        +listen stats
        +    bind :1936
              {{ end }}
              mode http
              # Health check monitoring uri.
        ```
  2. The docker files need to be changed: 
    * `images/base/Dockerfile`, all images should be based on our rhel7 base, so package installation should follow the rhel flavor, e.g., `yum install`.
        ```diff
        @@ -4,10 +4,10 @@
	 	#
 		# The standard name for this image is openshift/origin-base
 		#
	-FROM centos:centos7
	+FROM ecos0003:5000/rhel:7.2
 
	-RUN INSTALL_PKGS="which git tar wget hostname sysvinit-tools util-linux bsdtar epel-release \
	-      socat ethtool device-mapper iptables tree findutils nmap-ncat e2fsprogs xfsprogs lsof" && \
	+RUN INSTALL_PKGS="which git tar wget hostname sysvinit-tools util-linux bsdtar \
	+      socat ethtool tree findutils nmap-ncat e2fsprogs xfsprogs lsof" && \
     		yum install -y $INSTALL_PKGS && \
     		rpm -V $INSTALL_PKGS && \
     		yum clean all && \
        ```
    * `images/builder/docker/custom-docker-builder/Dockerfile`, add scripts for docker installation on s390x.
        ```diff
        @@ -16,10 +16,13 @@
              #
              FROM openshift/origin-base
 
        -RUN INSTALL_PKGS="gettext automake make docker" && \
        -    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        -    yum clean all
        +RUN yum install -y gettext automake make
        +
        +# Install docker
        +RUN mkdir /docker && cd /docker
        +RUN wget ftp://ftp.unicamp.br/pub/linuxpatch/s390x/redhat/rhel7.1/docker/docker-1.9.1-rhel7.1-20151205.tar.gz
        +RUN tar -xvzf docker-1.9.1-rhel7.1-20151205.tar.gz && cp docker-1.9.1-rhel7.1-20151205/docker /usr/bin
        +
             ENV HOME /root
             ADD ./build.sh /tmp/build.sh
             CMD ["/tmp/build.sh"]
        ```
    * `images/node/Dockerfile`, add scripts for openvswitch installation on s390x.
        ```diff
        @@ -13,8 +13,15 @@ MAINTAINER Devan Goodwin <dgoodwin@redhat.com>
             # We need to install openvswitch for the client portions, the daemons are expected
             # to run in another container
 
        -ADD https://copr.fedoraproject.org/coprs/maxamillion/origin-next/repo/epel-7/maxamillion-origin-next-epel-7.repo /etc/yum.repos.d/
        -RUN INSTALL_PKGS="libmnl libnetfilter_conntrack openvswitch \
        +RUN yum install -y wget tar make gcc openssl python perl && yum clean all
        + 
        +# Install openvswitch from the source
        +RUN mkdir openvswitch && cd ./openvswitch
        +RUN wget http://openvswitch.org/releases/openvswitch-2.4.0.tar.gz
        +RUN tar -xvf openvswitch-2.4.0.tar.gz
        +RUN cd openvswitch-2.4.0/ && ./configure && make && make install
        + 
        +RUN yum install -y libmnl libnetfilter_conntrack \
              libnfnetlink iptables iproute bridge-utils procps-ng ethtool socat openssl \
              binutils xz kmod-libs kmod sysvinit-tools device-mapper-libs dbus \
              ceph-common iscsi-initiator-utils" && \
        ```
    * `images/openvswitch/Dockerfile`, add scripts for openvswitch installation on s390x.
        ```diff
        @@ -4,20 +4,15 @@
              # The standard name for this image is openshift/openvswitch
              #
 
        -FROM centos:centos7
        +FROM ecos0003:5000/rhel:7.2
 
        -MAINTAINER Scott Dodson <sdodson@redhat.com>
        +RUN yum install -y wget tar make gcc openssl python perl && yum clean all
 
        -ADD https://copr.fedoraproject.org/coprs/maxamillion/origin-next/repo/epel-7/maxamillion-origin-next-epel-7.repo /etc/yum.repos.d/
        -
        -# TODO: systemd update from centos 7.1 -> 7.2 is broken, remove this once 7.2
        -# base images land
        -RUN yum swap -y -- remove systemd-container\* -- install systemd systemd-libs
        -
        -RUN INSTALL_PKGS="openvswitch" && \
        -    yum install -y $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        -    yum clean all
        +# Install openvswitch from the source
        +RUN mkdir openvswitch && cd ./openvswitch
        +RUN wget http://openvswitch.org/releases/openvswitch-2.4.0.tar.gz
        +RUN tar -xvf openvswitch-2.4.0.tar.gz
        +RUN cd openvswitch-2.4.0/ && ./configure && make && make install
 
              ADD  scripts/* /usr/local/bin/
              RUN chmod +x /usr/local/bin/*
        ```
    * `images/release/Dockerfile`, add scripts for go installation on s390x. 
        
        __Note:__ We copy the compiled go directory to the image directly. 

        ```diff
          @@ -12,16 +12,12 @@ ENV VERSION=1.6 \
               GOROOT=/usr/local/go \
               OS_VERSION_FILE=/go/src/github.com/openshift/origin/os-version-defs \
               TMPDIR=/openshifttmp
        -ENV PATH=$PATH:$GOROOT/bin:$GOPATH/bin
        -RUN mkdir $TMPDIR && \
        -    INSTALL_PKGS="gcc zip mercurial" && \
        -    yum install -y $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        -    yum clean all && \
        -    curl https://storage.googleapis.com/golang/go$VERSION.linux-amd64.tar.gz | tar -C /usr/local -xzf - && \
        -    go get golang.org/x/tools/cmd/cover github.com/tools/godep github.com/golang/lint/golint && \
        -    touch /os-build-image
        +RUN yum install --nogpgcheck -y gcc-c++ gcc glibc-devel make curl zip hg && yum clean all
        +
        +ADD go/ /usr/local/go/
        +ENV GOROOT /usr/local/go
        +ENV PATH $PATH:/usr/local/go/bin/
                WORKDIR /go/src/github.com/openshift/origin
        ```
    * `images/router/haproxy-base/Dockerfile`, add scripts for haproxy installation on s390x.
       
        ```diff
         @@ -11,12 +11,28 @@ FROM openshift/origin-base
                #       this is temporary and should be removed when the container is switch to an empty-dir
                #       with gid support.
                #
        -RUN INSTALL_PKGS="haproxy iptables lsof" && \
        -    yum install -y $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        +RUN yum install --nogpgcheck -y \
        +        git \
        +        java-1.8.0-openjdk \
        +        gcc-c++ \
        +        tar \
        +        openssl \
        +        openssl-devel \
        +        pcre \
        +        pcre-devel \
        +        make
        +RUN yum clean all
        +
        +# Clone the source code of HAProxy from github
        +RUN git clone http://git.haproxy.org/git/haproxy-1.6.git
        +
        +# Build and install Node.js
        +RUN cd haproxy-1.6/ && make TARGET=linux26 USE_OPENSSL=1 && make install
        +
        +RUN \
                mkdir -p /var/lib/containers/router/{certs,cacerts} && \
                mkdir -p /var/lib/haproxy/{conf,run,bin,log} && \
        -    touch /var/lib/haproxy/conf/{{os_http_be,os_edge_http_be,os_tcp_be,os_sni_passthrough,os_reencrypt,os_edge_http_expose,os_edge_http_redirect}.map,haproxy.config} && \
        +# touch /var/lib/haproxy/conf/{{os_http_be,os_edge_http_be,os_tcp_be,os_sni_passthrough,os_reencrypt,os_edge_http_expose,os_edge_http_redirect}.map,haproxy.config} && \
                chmod -R 777 /var && \
                yum clean all
        ```
    * `images/router/haproxy/Dockerfile`.
        
        ```diff
         @@ -17,7 +17,7 @@ ADD bin/openshift /usr/bin/openshift
                #
                RUN ln -s /usr/bin/openshift /usr/bin/openshift-router && \
                chmod -R 777 /var && \
        -    setcap 'cap_net_bind_service=ep' /usr/sbin/haproxy
        +    setcap 'cap_net_bind_service=ep' /usr/local/sbin/haproxy
                WORKDIR /var/lib/haproxy/conf
 
                EXPOSE 80
        ```

3. Copy your pre-built go folder in to `$GOPATH/src/github.com/openshift/origin/images/release`.

4. Build images and release. 
 
    __Note:__ If you want to build your own release, you have to git commit all changes back to your git server so that no dirty version remains.
    ```
    cd $GOPATH/src/github.com/openshift/origin/
    hack/build-base-images.sh
    make release
    ```

5. If you intend to use Auto-scaling feature of OpenShift Origin then OpenShift [origin-metrics](https://github.com/openshift/origin-metrics) images are required to be built:
  1. Git clone origin-metrics in a directory of your choosing: 
    ```
      cd /<source_root>/
      git clone https://github.com/openshift/origin-metrics.git
      cd /<source_root>/origin-metrics
      git checkout v1.2.0
    ```

  2. Create [wildfly](https://hub.docker.com/r/jboss/wildfly/) images required by origin-metrics. You will also need to build dependent Docker images for wildfly. Dockerfile of hawkular-metrics uses this docker image as base image.

  3. Modify Cassandra Dockerfile to include Linux on Z system changes. You will need to integrate Cassandra changes as per [Apache Cassandra 2.2.5](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Cassandra-2.2.5)

  4. Copy your pre-built go folder in to /\<source_root>/origin-metrics/heapster-base. Add following line before 'yum install -y -q git wget make &&' and modify Heapster Base dockerfile to remove Go installation steps:
    ```
    COPY go /tmp/go
    ```  

  5. You will need to build dependent Docker images for Heapster Base. Heapster dockerfile uses this docker image as base image.

  6. Modify Heapster Dockerfile to include Linux on Z system changes. Add the following line after 'git checkout $HEAPSTER_COMMIT &&':
    ```
    cp ./Godeps/_workspace/src/github.com/boltdb/bolt/bolt_amd64.go ./Godeps/_workspace/src/github.com/boltdb/bolt/bolt_s390x.go
    ```

  7. Run `hack/build-images.sh` to create origin-metrics images

  
### Optional: Run Unit tests

    TEST_KUBE=1 hack/test-go.sh

__Note:__ There is one failing test case regarding the oidc token, but this is not specific to s390x. Also there could be few errors related to router services, HTTPProbeChecker, TCPHealthChecker, etc. which we can skip for this version.

### Optional: Run Integration tests

    hack/test-integration.sh

__Note:__ There could be some errors happening regarding to token and router services. The token issue is because of the token generation latency which can be avoided by manually inserting sleeping function. The router issues is believed to be fixed in the newer version of OpenShift (v1.1.4).
    
## Reference
https://github.com/openshift/origin
