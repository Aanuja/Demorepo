
# Building Apigility

Apigility can be built for Ubuntu, by following these instructions. Version 1.3.1 has been successfully built & tested this way.

##### General Notes:
      
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.


#### Section 1: Install the following dependencies
```
	sudo apt-get update git apache2 curl openssl make wget tar gcc libssl-dev libxml2 libxml2-dev libxml-parser-perl pkg-config
```

#### Section 2: Build and install
1. Download, configure and install PHP with openssl
 
	```
	cd /<source_root>/
	wget http://www.php.net/distributions/php-5.6.8.tar.gz 
	tar xvzf php-5.6.8.tar.gz
	cd php-5.6.8
	./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-openssl
	make
	sudo make install
	```
	
2. Set environment path for the PHP

	```
	export PATH=/usr/local/php/bin:$PATH
	```

3. Get the source for Apigility
        
  ```
		cd /<source_root>/
        git clone https://github.com/zfcampus/zf-apigility-skeleton.git 
        cd zf-apigility-skeleton 
        git checkout 1.3.1
  ```

4. Install composer

 ```
        curl -s https://getcomposer.org/installer | php --
        ./composer.phar -n update
        ./composer.phar -n install
        
 ```

5. Put the skeleton/app in development mode

 ```
        php public/index.php development enable
 ```
        
6. Start the Apigility application

 ```
        export IP=$(hostname -i)
        cd  /<source_root>/zf-apigility-skeleton
        php -S $IP:8080 -t public public/index.php
 ```

## References:

https://github.com/zfcampus/zf-apigility-skeleton
