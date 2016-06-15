## Building MySQL
More information on MySQL is available at https://www.mysql.com and the source code can be downloaded from https://github.com/mysql/mysql-server.git.

Below versions of MySQL are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `5.7.12-0ubuntu1`

The instructions provided below specify the steps to build MySQL version 5.7 on Linux on the IBM z Systems for RHEL 7.1 and SLES12.

_**General Notes:**_

i) _**Note:** When following the steps below please use a standard permission user unless otherwise specified._

ii) _**Note:** A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building MySQL

###Obtain pre-built dependencies and create `/<source_root>/` directory.

   1. Use the following commands to obtain dependencies :

    For RHEL 7.1
    ```shell
    sudo yum install git gcc gcc-c++ make cmake bison ncurses-devel perl-Data-Dumper
    ```
    For SLES 12
    ```shell
    sudo zypper install git gcc gcc-c++ make cmake bison ncurses-devel wget tar
    ```

	For UBUNTU 16.04
    ```shell
	sudo apt-get update
    sudo apt-get install git make cmake gcc g++ libncurses5-dev bison
    ```
	
   1. Create the `/<source_root>/` directory mentioned above.

    ```shell
    mkdir /<source_root>/
    ```
   
###Product Build - MySQL.

   1. Download the MySQL 5.7.13 source code from Github.
    ```shell
    cd /<source_root>/
    git clone https://github.com/mysql/mysql-server.git
    ```

   1. Move into the ` mysql-server` sub-directory, and checkout branch 5.7
    ```shell
    cd mysql-server
    git branch
    git checkout 5.7
    ```
    _**Note:** At the time of writing branch 5.7 returned minor version 5.7.13, - this minor version is subject to change._


   1. Configure and Build the MySQL Software.
   
	For RHEL 7.1
	```shell
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```
	
	For UBUNTU 16.04
	```shell
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    make
    ```
	
	For SLES12
	```shell
	wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz 
	tar xzf boost_1_59_0.tar.gz 
	cmake .
	gmake
    ```

   1. _[Optional]_ Check the make

    The testing should take only a few seconds. All 21 tests should PASS.
    ```shell
    gmake test
    ```

   1. Install the Software into the standard location.
    ```shell
    sudo gmake install
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
    bin/mysqld --initialize --user=mysql
    ```

   1. _[Optional]_ Start/Stop the mysqld daemon.
    ```shell
    cd /<source_root>/
    sudo /usr/local/mysql/bin/mysqld_safe --user=mysql &
    /usr/local/mysql/bin/mysqladmin version
    sudo /usr/local/mysql/bin/mysqladmin -u root shutdown
    ```
     _**Note:** Performing a version check while the daemon is running confirms MySQL is operational._

   1. To start and stop server as an init.d Service

    This can be manually tested with a Start/Stop, but a system restart is needed for a full test.
    ```shell
    cd /usr/local/mysql
    sudo  cp support-files/mysql.server /etc/init.d/mysql
    sudo /etc/init.d/mysql start
    /usr/local/mysql/bin/mysqlshow
    sudo /etc/init.d/mysql stop
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
