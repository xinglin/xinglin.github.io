# Install hotCRP at a Ubuntu 14 Machine


0. Update packages  
	
		sudo apt-get update
	
1. Install php7. The default php coming with Ubuntu 14 is 5.5.9 while hotCRP prefers 5.6 and later. 
	1. Install the `software-properties-common` package to get the `add-apt-repository` command.  
		
			sudo apt-get install software-properties-common

	2. Add php7 ppa and do an update  
		
			sudo add-apt-repository ppa:ondrej/php	
			sudo apt-get update

	3. Install php7 

			sudo apt-get install php7.2 php7.2-mysql

2. Install other required packages  
        
		sudo apt-get install git apache2 mysql-server zip poppler-utils sendmail

3. Get hotCRP source code  

		git clone https://github.com/kohler/hotcrp.git 

4. Create the database for hotCRP  

		lib/createdb.sh --user root --password  

5. Add the read permission for conf/options.conf  
	
		chmod +r conf/options.conf
  	
6. Add the directory to /etc/apache2/apache2.conf    
		  
			<Directory "/w/hotcrp">    
				Options Indexes Includes FollowSymLinks  
				AllowOverride all  
				Require all granted  
			</Directory>  
			Alias /hotcrp /w/hotcrp  
		
9. Set `upload_max_filesize`, `post_max_size`, and `max_input_vars` in hotcrp/.htaccess file. Use default values are good enough.  
10. /etc/php/7.2/apache2/php.in  
	* Increase the session gc life time  
	
			session.gc_maxlifetime = 86400

	* Enable mysqli extension, by uncommenting the following line  
	
			;extension=mysqli

7. Restart the Apache web server  
	
		sudo service apache2 restart

8. The website is up and running. Visit `IP_address/hotcrp` to check it out.


