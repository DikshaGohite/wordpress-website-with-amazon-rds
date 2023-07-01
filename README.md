Project Description: Deploying WordPress site to run a blog with Amazon RDS

Problem Statement. Here we are making a wordpress blog post, which requires database to store its contents, while wordpress uses MYSQL as its databases, the administration tasks come along with it. With Amazon RDS these concerns go away and you can focus on your blog 

Technologies Used: Amazon RDS for MySQL, Amazon EC2

Amazon RDS for MySQL: It is a managed database offering from AWS. Amazon RDS for MySQL frees you up to focus on application development by managing time-consuming database administration tasks, including backups, upgrades, software patching, performance improvements, monitoring, scaling, and replication.

Amazon EC2: Amazon EC2 provides on-demand server provisioning. With Amazon EC2, you rent out server instances with varying sizes, each with different CPU, RAM, and network configuration. You pay by the hour for these servers, and you can use them to host websites, like your WordPress site. With an EC2 instance, your WordPress site will remain up and running and will be accessible by anyone over the internet.

Section 1: Creating RDS database

1. On the AWS management console, enter RDS in the search bar and select RDS</br>
2. Once you land on RDS page, choose create database</br>
3. First step is to select Standard create and select MYSQL as your desired engine</br>
4. Under template section select free-tier, to stay within free tier linmits and prevent any costs</br>
5. Undet Settings give meaninful name to DB instance identifier</br>
6. Specify the master username and password to protect your databse</br>
7. Now review the remaining sections keep them as default, you can specify a new VPC for your DB if you wish to</br>
8. Last change to make under additional configuration is to specify the initial database name same as you specified in step 5</br>
9. At the end, click the create database button and within few minutes database instance will be up</br>
   
Section 2: Creating an Instance </br>
1. To create an EC2 instance choose the Launch instance button to open the instance creation wizard</br>
2. Once you land on launch an instance page, enter desired instance name</br>
3. Choose the AMI, it helps you determine the base software that is installed on your new EC2 instance (choose any free tier aplicable AMI)</br>
4. Select the EC2 instance type as t2.micro if you wish to stay within free tier limit</br>
5. Scroll down to the key pair section and choose create new key pair - A prompt will appear enter key pair name and click create key pair</br>
6. Under Network settings -> security group allow SSh from your IP and HTTP traffic from the internet for your instance</br>
7. In the network settings section, choose the edit button and enter suitable name and description for your security group name</br>
8. Preview the settings and launch instance, after sometime your EC2 instance will be launched</br>
 
Section 3: Configuring your Amazon RDS database, Now its time to modify our Amazon RDS database to allow network access from EC2 instance</br>
1. Head over to RDS, from the left navigation pane Choose the MySQL database you created in Section 1</br>
2. Scroll to the Connectivity & security tab in the display and choose the security group listed under VPC security groups</br>
3. Select the Inbound rules tab, then choose the Edit inbound rules button to change the rules for your security group and allow access from the security group of EC2 instance and delete any other rules that allow unwanted 
   connections</br>
4. When you’re finished, choose the Save rules button to save your changes</br>
5. Now its time to SSH into our EC2 instance, to learn more about SSH into your instance follow this guide [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html]</br>
6. Now all thats left is to create a databse user, commands to follow are as follows:
   -> First, run the following command in your terminal to install a MySQL client to interact with the database.
     sudo yum install -y mysql
   -> In the AWS Console, head over to Amazon RDS page and find the hostname, it will available as Endpoint in the Connectivity & security section.
     export MYSQL_HOST=<your-endpoint>
   -> Run the following command to connect to your DB
      mysql --user=<user> --password=<password> wordpress
   -> create a database user for your WordPress application and give the user permission to access the wordpress database.
      CREATE USER 'wordpress' IDENTIFIED BY 'wordpress-pass';
      GRANT ALL PRIVILEGES ON wordpress.* TO wordpress;
      FLUSH PRIVILEGES;
      Exit


Step 4: Configuring Wordpress on EC2
1. We need a web server to run our Wordpress on Ec2 Instance therefore installing a apache server</br>
   sudo yum install -y httpd</br>
2. Now to start the Apache server, we need the following command</br>
   sudo service httpd start</br>
3. We can now download and configure wordpress software by running the following commands in your ssh terminal</br>
   -> download and incompress the software</br>
      wget https://wordpress.org/latest.tar.gz</br>
      tar -xzf latest.tar.gz</br>
   -> run ls to view the contents of your directory, you will see a tar file and a directory called wordpress with the uncompressed contents</br>
   -> Change the directory to the wordpress directory and create a copy of the default config file using the following commands</br>
      cd wordpress</br>
      cp wp-config-sample.php wp-config.php</br>
   -> open the wp-config.php file using nano editor using the following command</br>
      nano wp-config.php</br>
   -> A file opens up , here you need to edit 2 areas</br>
      DB_NAME, DB_USER, D_PASSWORD, DB_HOST (OR endpoint name for RDS) refer to previous values used when setting up RDS</br>
      Authentication Unique Keys and Salts, you can copy the contents from this file and replace with these: (https://api.wordpress.org/secret-key/1.1/salt/)</br>
4. Now we are all set, just save and exit from nano by entering ctrl+o and ctrl+x</br>
5. Here we will make apache web server handle requests for wordpress, commands are as follows:</br>
   -> sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2</br>
   -> cd /home/ec2-user</br>
   -> sudo cp -r wordpress/* /var/www/html/</br>
   -> sudo service httpd restart</br>

That’s it. You have a live, publicly accessible WordPress installation using a fully managed MySQL database on Amazon RDS</br>

Important:
Remember to terminate all your resources at the end to avoid exhausting the aws free tier limit 
