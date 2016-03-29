MySQL can be built for Linux on z Systems running RHEL 6.6/7.1 and SLES 11/12 by following these instructions.  Version 5.7.11 has been successfully built & tested this way.
More information on MySQL is available at https://www.mysql.com and the source code can be downloaded from https://github.com/mysql/mysql-server.git
.

_**General Notes:**_

i) _**Note:** When following the steps below please use a standard permission user unless otherwise specified._

ii) _**Note:** A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building MySQL

###Obtain pre-built dependencies and create `/<source_root>/` directory.

   1. Use the following commands to obtain dependencies :

    For RHEL 6.6
    ```shell
    sudo yum install git gcc gcc-c++ make cmake bison ncurses-devel
    ```
    For RHEL 7.1
    ```shell
    sudo yum install git gcc gcc-c++ make cmake bison ncurses-devel perl-Data-Dumper
    ```

	For SLES 11 - _(Additional support packages are needed to update cmake)_
    ```shell
    sudo zypper install git gcc gcc-c++ make cmake bison ncurses-devel util-linux tar zip wget glibc-devel-32bit zlib-devel
    ```
    For SLES 12
    ```shell
    sudo zypper install git gcc gcc-c++ make cmake bison ncurses-devel wget tar
    ```

   1. Create the `/<source_root>/` directory mentioned above.

    ```shell
    mkdir /<source_root>/
    ```

###Dependency Build -  GCC 4.8.2 and cmake 3.3.0-rc2

   _**Only Required on SLES 11**  
   
   - Install GCC 4.8.2 by building from source._
   
   1. Download the GCC source code, then extract it.
      ```shell
      cd /<source_root>/
      wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
	  bunzip2  gcc-4.8.2.tar.bz2
	  tar xvf gcc-4.8.2.tar
      ```

   2. Download the prerequisites needed for GCC.
      ```shell
      cd gcc-4.8.2/
	  ./contrib/download_prerequisites
	  mkdir build
	  cd build
      ```
	  
   3. Configure the Makefile. Then Make and Install the utility.
      ```shell
      ../configure --disable-checking --enable-languages=c,c++ --enable-multiarch --enable-shared --enable-threads=posix --without-included-gettext --with-system-zlib --prefix=/opt/gcc4.8
	  make 
	  sudo make install 
      ```

   4. Set the environment variable.
      ```shell
      export PATH=/opt/gcc4.8/bin:$PATH
      export LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
      ```
   
   5. Confirm the version of `GCC`.
      ```shell
      gcc --version
      ```
	  
	  
   - Update cmake to version 3.3.0-rc2 by building from source._

   1. _[Optional]_ Check the version of any existing `cmake` executable.
    ```shell
      which cmake
      $(which cmake) --version
    ```
      _**Note:** A `cmake` at version 2.6.3 or later should be usable without upgrade._


   1. Download the cmake source code, then extract it.
      ```shell
      cd /<source_root>/
      wget http://www.cmake.org/files/v3.3/cmake-3.3.0-rc2.tar.gz
      tar xzf cmake-3.3.0-rc2.tar.gz
      ```

   1. Bootstrap to configure the Makefile. Then Make and Install the utility.
      ```shell
      cd cmake-3.3.0-rc2
      ./bootstrap --prefix=/usr
      gmake
      sudo gmake install -e LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
      ```
      _**Note:** To place `cmake` in the standard SLES location use `./bootstrap --prefix=/usr`._


   1. Confirm the location and version of the upgraded `cmake`.
      ```shell
      which cmake
      $(which cmake) --version
      ```

###Product Build - MySQL.

   1. Download the MySQL 5.7.11 source code from Github.
    ```shell
    cd /<source_root>/
    git clone https://github.com/mysql/mysql-server.git
    ```

   1. Move into the ` mysql-server` sub-directory, and checkout branch 5.6
    ```shell
    cd mysql-server
    git branch
    git checkout 5.7
    ```
    _**Note:** At the time of writing branch 5.7 returned minor version 5.7.11, - this minor version is subject to change._


   1. Configure and Build the MySQL Software.
   
    For RHEL 6.6/7.1
    ```shell
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```

	For SLES 11
	```shell
	wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
	tar xzf boost_1_59_0.tar.gz
    cmake -DCMAKE_C_COMPILER=/opt/gcc4.8/bin/gcc -DCMAKE_CXX_COMPILER=/opt/gcc4.8/bin/g++ -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```
	  
	For SLES 12
	```shell
	wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
	tar xzf boost_1_59_0.tar.gz
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```

   1. _[Optional]_ Check the make

    The testing should take only a few seconds. All 21 tests should PASS.
    ```shell
    gmake test
    ```

   1. Install the Software into the standard location.
    
	For RHEL 6.6/7.1 & SLES 12
    ```shell
    sudo gmake install
    ```
    
	For SLES 11
	```shell
    sudo gmake install -e LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
    ```

###_[Optional]_ Post installation Setup and Testing.

   This guideline demonstrates a basic free-standing MySQL database, with a script for shutdown/restart as a System Service.
   Refer to http://www.mysql.com for definitive details, and configuration for non-test environments.

   1. Add a MySQL userid and group, then reset Linux ownerships and permissions within `/usr/local/mysql`.
    ```shell
    sudo /usr/sbin/groupadd mysql
    sudo /usr/sbin/useradd -g mysql mysql
    sudo chown -R mysql.mysql /usr/local/mysql
    ```

   1. Initialize MySQL Data Directory.  (The `--user=mysql`, to match the MySQL Daemon (mysqld) userid).
    ```shell
    cd /usr/local/mysql
    sudo bin/mysqld --initialize --user=mysql
    ```

   1. _[Optional]_ Start/Stop the mysqld daemon.
    ```shell
    cd /<source_root>/
    sudo /usr/local/mysql/bin/mysqld_safe --user=mysql &
    /usr/local/mysql/bin/mysqladmin --version
    sudo /usr/local/mysql/bin/mysqladmin -u root -p shutdown
    ```
     _**Note:** Performing a version check while the daemon is running confirms MySQL is operational._
	 
	 _**Note:** ii). After starting the mysqld server, reset the root password using the mysql shell:
					For example: `/usr/local/mysql/bin/mysql -A -u root -p`. The system will prompt `Enter password:` expecting the root password (temporary password generated when mysqld is initialised) in response.
					To reset the password: `SET PASSWORD for 'root'@'localhost' = PASSWORD('newPassword');`._

   1. To start and stop server as an init.d Service

    This can be manually tested with a Start/Stop, but a system restart is needed for a full test.
    ```shell
    cd /usr/local/mysql
    sudo cp support-files/mysql.server /etc/init.d/mysql
	cd /etc/init.d
    sudo ./mysql start
    sudo /usr/local/mysql/bin/mysqlshow -p
    sudo ./mysql stop
    ```
    _**Note:** i). Operation of bin/mysql commands requires, a running mysql server, and the `mysqlshow` executable is to show the existing databases._

    _**Note:** ii). See http://www.mysql.com for full details,  ... where a Linux root password is set, the bin/mysqladmin and bin/mysql commands require -u and -p options.
            For example: `bin/mysqladmin -u root -p      version`. The system will prompt `Enter password:`  expecting the root password in response._

###_[Optional]_ Clean up.

   1. Remove the ` /<source_root>/` directory to tidy up.

     ```shell
     cd /<source_root>/
     cd ..
     sudo rm -rf /<source_root>/
     ```

###References:

https://bugs.mysql.com/bug.php?id=72752 - Explanation of the cmake upgrade for SLES 11.

http://www.mysql.com - MySQL Homepage with definitive Information and Documentation.
