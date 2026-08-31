Web transfer is the most common way most people transfer files because `HTTP`/`HTTPS` are the most common protocols allowed through firewalls. Another immense benefit is that, in many cases, the file will be encrypted in transit. There is nothing worse than being on a penetration test, and a client's network IDS picks up on a sensitive file being transferred over plaintext and having them ask why we sent a password to our cloud server without using encryption.

We have already discussed using the Python3 [uploadserver module](https://github.com/Densaugeo/uploadserver) to set up a web server with upload capabilities, but we can also use Apache or Nginx.

## Nginx - Enabling PUT

A good alternative for transferring files to `Apache` is [Nginx](https://www.nginx.com/resources/wiki/) because the configuration is less complicated, and the module system does not lead to security issues as `Apache` can.

When allowing `HTTP` uploads, it is critical to be 100% positive that users cannot upload web shells and execute them. `Apache` makes it easy to shoot ourselves in the foot with this, as the `PHP`  module loves to execute anything ending in `PHP`. Configuring `Nginx` to use PHP is nowhere near as simple.

#### Create a Directory to Handle Uploaded Files

	AstraX01@htb[/htb]$ sudo mkdir -p /var/www/uploads/SecretUploadDirectory

#### Change the Owner to www-data

	AstraX01@htb[/htb]$ sudo chown -R www-data:www-data

#### Create Nginx Configuration File

Create the Nginx configuration file by creating the file `/etc/nginx/sites-available/upload.conf` with the contents:

	server { 
	      listen 9001; 
	      
	      location /SecretUploadDirectory/ 
	{ 
	         root /var/www/uploads; 
	         dav_methods PUT; 
	    } 
	}

#### Symlink our Site to the sites-enabled Directory

	AstraX01@htb[/htb]$ sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/

#### Start Nginx

	AstraX01@htb[/htb]$ sudo systemctl restart nginx.service

If we get any error messages, check `/var/log/nginx/error.log`. If using Pwnbox, we will see port 80 is already in use.

#### Verifying Errors

	AstraX01@htb[/htb]$ tail -2 /var/log/nginx/error.log

	AstraX01@htb[/htb]$ ss -lnpt | grep 80

	AstraX01@htb[/htb]$ ps -ef | grep 2811

We see there is already a module listening on port 80. To get around this, we can remove the default Nginx configuration, which binds on port 80.

	AstraX01@htb[/htb]$ sudo rm /etc/nginx/sites-enabled/default

Now we can test uploading by using `cURL` to send a `PUT` request. In the below example, we will upload the `/etc/passwd` file to the server and call it users.txtNow we can test uploading by using `cURL` to send a `PUT` request. In the below example, we will upload the `/etc/passwd` file to the server and call it users.txt

#### Upload File Using cURL

	AstraX01@htb[/htb]$ curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt

	AstraX01@htb[/htb]$ sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt

