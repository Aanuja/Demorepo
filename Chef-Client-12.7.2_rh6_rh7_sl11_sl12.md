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
      sudo yum -y install git gcc make wget tar
      sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar 	  
    ```
    
	For SLES 11

	```
      sudo zypper install -y git gcc make wget tar 
	  sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar 	  
            
    ```

	For SLES 12

	```
      sudo zypper install -y git ruby2.1 ruby2.1-devel ruby2.1-rubygem-bundler gcc make which tar
       
    ```

2. For SLES 11 you will need to build Openssl  

	```
      wget ftp://openssl.org/source/openssl-1.0.2g.tar.gz
	  ./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
	  make
	  sudo make install
    ```
    
	
3. For RHEL 6.6 and SLES 11 you will need to build Ruby 2.2.4 

	```
      wget http://cache.ruby-lang.org/pub/ruby/ruby-2.2.4.tar.gz
      tar zxf ruby-2.2.4.tar.gz
      cd ruby-2.2.4
	```
	
	For SLES 11
	  
	```
      ./configure LDFLAGS='-L/home/test/openssl-1.0.2f' --with-openssl-include=/home/test/openssl-1.0.2f/include --with-openssl-dir=/usr/
	```
	  
	For RHEL 6.6
	  
	```
      ./configure
	```
      
	```
      make 
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

    For SLES 12
 
	```
      export PATH=$PATH:/<source_root>/chef/bin
    ```
       
    _**Note**: Run ```gem env``` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
7. Install the required version of the bundler ruby gem

	For RHEL 6.6 & SLES 11
	```
      gem install bundler -v '1.7.3'
    ```
	
	For RHEL 7.1 & SLES 12
	```
      sudo gem install bundler -v '1.7.3'
    ```
	
8. Use bundler to install Chef Client's ruby gem dependencies
	
	For RHEL 6.6 & SLES 11
	```
      bundle install
    ```
	
	For RHEL 7.1 & SLES 12
	```
      sudo bundle install
    ```
9. Comment out the rdoc/task line in the Rakefile as below

	```
      require "chef-config/package_task"
	  #require "rdoc/task"
	  require_relative "tasks/rspec"
    ```
    
10. Build the Chef Client ruby gem packages

	```
      bundle exec rake gem
    ```

11. Install the gem you just built

	For RHEL 6.6 & SLES 11
	```
      ls pkg/*.gem | grep -v mingw32 | xargs gem install
    ```    
	
	For RHEL 7.1 & SLES 12
	```
      ls pkg/*.gem | grep -v mingw32 | xargs sudo gem install
    ``` 
12. Chef client is now built and installed (verify with chef-client or knife)


## Testing Chef Client

If you'd like to test the Chef client you've just built and installed, just follow the steps below:

1. Run the test suite
   	
   ```
      cd /<source_root>/chef/
	  bundle exec rake spec
   ```  
	 To run a Single Test File
   ```  
	  bundle exec rspec spec/PATH/TO/FILE_spec.rb
   ```  
	 To Run a Subset of Tests
   ```
	 bundle exec rspec spec/PATH/TO/DIR
   ```
   
2. Notes on Verification Test Failures (not specific to Linux on z Systems)  
  1. If test case, _chef-client when the chef repo has a cookbook with a no-op recipe should complete successfully with no other environment variables_, fails, add following  

     1. ``` vi ./spec/integration/client/client_spec.rb```  
     2.  Modify the file as follows:  

        * For SLES12: add HOME variable in ```let(:critical_env_vars)``` and it should look like following:  
        ```
          let(:critical_env_vars) { %w(PATH HOME RUBYOPT BUNDLE_GEMFILE GEM_PATH).map {|o| "#{o}=#{ENV[o]}"} .join(' ') }
        ```  

        * For RHEL6, RHEL7 and SLES11: add GEM_HOME variable in ```let(:critical_env_vars)``` and it should look like following:  
        ```
          let(:critical_env_vars) { %w(PATH GEM_HOME RUBYOPT BUNDLE_GEMFILE GEM_PATH).map {|o| "#{o}=#{ENV[o]}"} .join(' ') }
		```
		
3. Visit https://github.com/chef/chef#testing for more   

## References:

https://github.com/chef/chef.git
