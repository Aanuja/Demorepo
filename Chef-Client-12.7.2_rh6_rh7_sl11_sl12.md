## Building Chef Client 12.7.2

The Chef Client 12.7.2 code can be built for a Linux on z System running RHEL 7.1/6.6 or SLES 12/11 by following these instructions (Chef is available at https://www.chef.io/ and the github repository for the client can be found at https://github.com/chef/chef):

_**General Notes:**_   
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


## Building Chef Client on RHEL 7.1/6.6, SLES 12/11

1. Install the build time dependencies

    For RHEL 7.1 
    ```
    sudo yum install -y git ruby ruby-devel rubygems rubygem-bundler gcc make wget which tar
    ```
	
    For RHEL 6.6 
    ```
    sudo yum -y install git gcc make wget tar bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel which	  
    ```
    
    For SLES 11
    ```
    sudo zypper install -y git gcc make wget tar bison flex libopenssl-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel	      
    ```

    For SLES 12
    ```
    sudo zypper install -y git ruby2.1 ruby2.1-devel ruby2.1-rubygem-bundler gcc make which tar    
    ```

2. For SLES 11 you will need to build Openssl  

    ```
    cd /<source_root>/
    wget ftp://openssl.org/source/openssl-1.0.2g.tar.gz
    tar zxf openssl-1.0.2g.tar.gz
    cd openssl-1.0.2g
    ./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
    make
    sudo make install
    ```
    
3. For RHEL 6.6 and SLES 11 you will need Ruby 2.2.4
   
   3.1. Download the source code
   ```
   cd /<source_root>/
   wget http://cache.ruby-lang.org/pub/ruby/ruby-2.2.4.tar.gz
   tar zxf ruby-2.2.4.tar.gz
   cd ruby-2.2.4
   ```
	
  For SLES 11
  ```
  ./configure LDFLAGS='-L/<source_root>/openssl-1.0.2g' --with-openssl-include=/<source_root>/openssl-1.0.2g/include --with-openssl-dir=/usr/
  ```
	  
  For RHEL 6.6
  ```
  ./configure
  ```

   3.2. Build the source code
   ```
   make
   make test	  
   sudo make install
   ```
	
4. Move to the location you wish to store the Chef source in

    ```
    cd /<source_root>/
    ```

5. Clone the github Chef client repository checkout the correct version

    ```
    git clone https://github.com/chef/chef.git
    cd chef
    git checkout 12.7.2
    ```

6. Skip this step if you are on RHEL 7.1, on all other OS correct the gem environment for a standard user

    ```
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ``` 

    _where `<USER>` is the standard user you are installing under._
       
   _**Note**: Run ```gem env``` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
	
7. Install the required version of the bundler ruby gem

   ```
   gem install bundler -v '1.7.3'
   ```
	
8. Use bundler to install Chef Client's ruby gem dependencies

   ```
   bundle install
   ```
    
9. Build the Chef Client ruby gem packages

   ```
   bundle exec rake gem
   ```

   _**NOTE:** For RHEL7, if ```bundler: command not found: rake``` error occurs, set ```rake``` binary path to PATH environment variable and rerun above command._  

    ```
      export PATH=$PATH:$HOME/bin
    ```

10. Install the gem you just built

   ```
   ls pkg/*.gem | grep -v mingw32 | xargs gem install
   ```    
   
11. Chef client is now built and installed (verify with chef-client or knife)


## Testing Chef Client

If you'd like to test the Chef client you've just built and installed, just follow the steps below:

1. Run the test suite
   	
   ```
   cd /<source_root>/chef/
   bundle exec rake spec
   ```  
   To run a single test file
   ```  
   bundle exec rspec spec/PATH/TO/FILE_spec.rb
   ```  
   To run a subset of tests
   ```
   bundle exec rspec spec/PATH/TO/DIR
   ```
   
   _**NOTE:**Note: This may fail as there can be a dependency on rdoc/task in the Rakefile, if that happens just comment out the rdoc/task line as below._

   ```
   require "chef-config/package_task"
   require 'rdoc/task'
   require_relative 'tasks/rspec'
   ```
   can be changed to be

   ```
   require "chef-config/package_task"
   #require 'rdoc/task'
   require_relative 'tasks/rspec'
   ```

2. Notes on Verification Test Failures (not specific to Linux on z Systems)  
   1. If test case "chef-client when the chef repo has a cookbook with a no-op recipe should complete successfully with no other environment variables" fails, modify the following file  

   		``` vi ./spec/integration/client/client_spec.rb```  
     
     		For RHEL6/7 & SLES11, The following line of code:
		
		``` let(:critical_env_vars) { %w(PATH RUBYOPT BUNDLE_GEMFILE GEM_PATH).map {|o| "#{o}=#{ENV[o]}"} .join(' ') }```
	
		can be changed to 
		
		``` 
		let(:critical_env_vars) { %w(PATH RUBYOPT BUNDLE_GEMFILE GEM_PATH GEM_HOME).map {|o| "#{o}=#{ENV[o]}"} .join(' ') }
		```
        	
3. Visit https://github.com/chef/chef#testing for more   

## References:

https://github.com/chef/chef.git
