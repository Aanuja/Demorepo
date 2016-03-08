
sudo apt-get update
sudo apt-get git apache2 curl openssl make wget tar gcc libssl-dev libxml2 libxml2-dev libxml-parser-perl pkg-config

wget http://www.php.net/distributions/php-5.6.8.tar.gz 
tar xvzf php-5.6.8.tar.gz 
cd php-5.6.8
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-openssl
make 
make install
export PATH=/usr/local/php/bin:$PATH
		
git clone https://github.com/zfcampus/zf-apigility-skeleton.git 
cd zf-apigility-skeleton 
git checkout 1.3.1
curl -s https://getcomposer.org/installer | php --
./composer.phar -n update
./composer.phar -n install
php public/index.php development enable
export IP=$(hostname -i)
cd /<source_root>/
php -S $IP:8080 -t public public/index.php
