# Objective 
Install Percona XtraDB Cluster on Kubernetes cluster and WordPress on Container environment and setup WordPress to use the Database from the Kubernets cluster, and make WordPress available from Internet using http. 

# Installation
1.  Install Percona XtraDB Cluster on Kubernetes cluster as described [here](https://www.percona.com/doc/kubernetes-operator-for-pxc/kubernetes.html#). Some changes has been made in the deployment yaml file. The requested changes are:- 

    - change cluster name to: ```wordpress-database```
    - disable automatic upgrade (```spec.upgradeOptions.apply``` -> ```Disabled```)
    - configure pxc (```spec.pxc.volumeSpec.persistentVolumeClaim.storageClassName```) storage to use ```premium-rwo``` storage class
    - configure pxc (```spec.pxc.resources.limits```) to have following limits:
    ```
        - cpu: 1
        - memory: 2G
    ```
    - expose cluster via LoadBalancer service (```spec.haproxy.serviceType``` ```ClusterIP`` -> ```LoadBalancer```)
    - disable backup scheduling by commenting out last 8 rows from the end of the file

2. Install your preferred container runtime on Linux VM
    - Docker engine is installed in the VM as follows:
    ``` 
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
3. Install container version of WordPress on Linux VM and expose it to outside VM (http)
    - Install the official Dockerv version of WordPress on Linux VM as follows. 
        ```shell
        docker run -e WORDPRESS_DB_HOST=34.88.228.21 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=root_password -e WORDPRESS_DB_NAME=wordpress -e WORDPRESS_TABLE_PREFIX=wp_ -p 8080:80 -v /root/wordpress/html:/var/www/html --name wpcontainer -d wordpress
        ```
        - Here we exposed port 80 in the container to port 8080 in the host so that wordpress is accessible from outside of the container. 
        - The evnironment variables are aslo set. For example, the ```WORDPRESS_DB_HOST``` is set to the LoadBalacer public IP address ```34.88.228.21```.
    - Exposed WordPress outside VM using http proxy. 
        - installed Apache and configured it to be used as a reverse proxy. 
          ```shell
             sudp apt-get install apache2
            # Edit /etc/apache2/sites-available/000-default.conf and put the following content.
            <VirtualHost *:80>
                ServerAdmin webmaster@localhost
                DocumentRoot /var/www/html

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                ProxyPreserveHost On
                proxypass / http://127.0.0.1:8080/
                proxypassreverse / http://127.0.0.1:8080/
            </VirtualHost>
            sudo service apache2 restart
          ```
            - Note that ```proxypass``` and ```proxypassreverse``` adress could be also set to ```http://172.17.0.2:80/```. However, this address may change when the container restarts and in such cases the proxy may not work. As such, we can use proxy via the localhost as port ```80``` of container is exposed to port ```8080``` in the host, and we can access WordPress at```http://127.0.0.1:8080/```.
4. Configure WordPress to use XtraDB as a database (you need to create database for WordPress)
    - A database named ```wordpress``` is created for in XtraDB. 
        ```shell
        kubectl run -i --rm --tty percona-client --image=percona:8.0 --restart=Never -- bash -il
        mysql -h wordpress-database-haproxy -uroot -proot_password

        # create the db
        create database wordpress;
        ```

5. Check that WordPress is available from Internet using http.
    - Yes WordPress is accessible from the Internet. The reverse proxy setup in step #3 makes WordPress available outside of the VM. The snipset from curl command. 
      ```shell
        curl -I 34.88.102.138
        HTTP/1.1 302 Found
        Date: Sat, 12 Mar 2022 21:49:18 GMT
        Server: Apache/2.4.52 (Debian)
        X-Powered-By: PHP/7.4.28
        Expires: Wed, 11 Jan 1984 05:00:00 GMT
        Cache-Control: no-cache, must-revalidate, max-age=0
        X-Redirect-By: WordPress
        Location: http://34.88.102.138/wp-admin/install.php
        Content-Type: text/html; charset=UTF-8

      ```
  

# References
1. [Apache Reverse Proxy Setup Virtual Host](https://metamug.com/article/networking/apache-reverse-proxy-setup-virtual-host.html) 
2. [Install Percona XtraDB Cluster on Kubernetes](https://www.percona.com/doc/kubernetes-operator-for-pxc/kubernetes.html#)
