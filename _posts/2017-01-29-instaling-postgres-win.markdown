---
layout: post
title:  "Installing PostgreSQL/PostGIS Windows"
date:   2017-01-29 15:07:38 -0700
categories: install windows
---
This guide will cover the necessary steps to install both PostgreSQL and PostGIS on a Windows machine.

### Downloading, Installing and Setting-up.

The first step is to download the PostgreSQL base software.
The official website is: [http://www.postgresql.org/](http://www.postgresql.org/) there you can find the official documentation about the system.

There are also different ways to download and install PostgreSQL, there are third party companies who offer support and pre-compiled packages for different systems, [EnterpriseDB](http://www
.enterprisedb.com/) is one of these companies. At this [link](http://www.enterprisedb.com/products-services-training/pgdownload) you will find the necessary files to install PostgreSQL in your 
machine, choose the link that refersto your machine’s operating system. The following section will cover step-by-step how to set-up PostgreSQL in a Windows based machine.

### Installing

Double click the downloaded file and you shall see this welcome screen, press next until you’re requested to provide a password.

![Step 1]({{ site.url }}/images/picture1.png)

As in any database system PostgreSQL has the capabilities of having multiple stored users. The main user, also referred as superuser, has all the required permissions to manipulate any aspect of database, from creating new databases to manage users. This superuser is created at the time the system is installed and the name is set by default to be postgres. On this book we will not create other users, we will be manipulating the database using the postgres credentials. Although you cannot change the name of this ‘postgres’ user you can however choose whatever password you desire. Press next.

![Step 2]({{ site.url }}/images/picture2.png)

This screen show the port number PostgreSQL will use to access the database in your computer, leave it with the default value and press next.

![Step 3]({{ site.url }}/images/picture3.png)

Also leave it at default and press next.

![Step 4]({{ site.url }}/images/picture4.png)

Now the database system will be installed is installing, this may take a few minutes.

![Step 5]({{ site.url }}/images/picture5.png)

Once the installation is done, you will see this screen asking if you wish to launch the Stack Builder. Stack Builder is add-on from PostgreSQL that allow us to download and install additional plug-ins for PostgreSQL. Since PostGIS is a plug-in we can get the latest version through the Stack Builder. Check the box and press finish.

![Step 6]({{ site.url }}/images/picture6.png)

This is the Stack Builder dialog box, this is where we will download PostGIS. From the drop menu select PostgreSQL and press next.

![Step 7]({{ site.url }}/images/picture7.png)

From the Spatial Extension category select PostGIS based on the spec of your machine, then press next.

![Step 8]({{ site.url }}/images/picture8.png)

After the download is done press next and we will see this scree, this is the PostGIS installation screen, agree with the terms.

![Step 9]({{ site.url }}/images/picture9.png)

Check only the PostGIS box since we will be creating a new database later in the program.

![Step 10]({{ site.url }}/images/picture10.png)

Now we have to provide the password that we created previously during the PostgreSQL installation. Press next and the installation process will begin.

![Step 11]({{ site.url }}/images/picture11.png)

Done! Now you have PostgreSQL and PostGIS installed on your machine.





