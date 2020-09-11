## Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

### Objectives

In this lab, you learn how to perform the following tasks:

- Create a Cloud Storage bucket and place an image into it.
- Create a Cloud SQL instance and configure it.
- Connect to the Cloud SQL instance from a web server.
- Use the image in the Cloud Storage bucket on a web page.



### Task 1: Sign in to the Google Cloud Platform (GCP) Console

**Activate Cloud Shell from the top menu bar.**

### Task 2: Deploy a web server VM instance

```bash
#setting my project
gcloud config set project qwiklabs-gcp-03-5fc0054f636b

#creating the instance
gcloud compute instances create bloghost \
--zone=us-central1-a \
--machine-type=n1-standard-1 \
--subnet=default \
--metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart \
--tags=http-server \
--image=debian-9-stretch-v20200910 \
--image-project=debian-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=bloghost

#set the firewall rule to allow http traffic
gcloud compute firewall-rules create default-allow-http \
--direction=INGRESS \
--priority=1000 \
--network=default \
--action=ALLOW \
--rules=tcp:80 \
--source-ranges=0.0.0.0/0 \
--target-tags=http-server

#copy the bloghost VM instance's internal and external IP addresses to a text editor for use #later in this lab.
#bloghost  us-central1-a  n1-standard-1  10.128.0.2   104.197.189.168

```



### Task 3: Create a Cloud Storage bucket using the gsutil command line

```bash
#set location
export LOCATION=US

gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID

#Creating gs://qwiklabs-gcp-04-36f037b027fd/

gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

```



### Task 4: Create the Cloud SQL instance

```bash

gcloud sql instances create blog-db \
--database-version=MYSQL_5_7 \
--tier=db-n1-standard-1 \
--zone=us-central1-a \
--root-password=FwB3MDfCrutD9mg1

#blog-db  MYSQL_5_7   us-central1-a  db-n1-standard-1 34.72.79.67

gcloud sql users create blogdbuser \
--instance=blog-db \
--password=36f037b

gcloud sql instances patch blog-db --authorized-networks=104.197.189.168/32 -q


```



### Task 5: Configure an application in a Compute Engine instance to use Cloud SQL

```bash
#login to bloghost instance

gcloud compute ssh bloghost

sudo bash -c 'cat > /var/www/html/index.php <<EOF
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
\$dbserver = "CLOUDSQLIP";
\$dbuser = "blogdbuser";
\$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

\$conn = new mysqli(\$dbserver, \$dbuser, \$dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>

EOF'

sudo service apache2 restart

curl http://104.197.189.168/index.php

# Database connection failed: php_network_getaddresses: getaddrinfo failed: Name or service not known

sudo sed -i 's/CLOUDSQLIP/34\.72\.79\.67/g' /var/www/html/index.php
sudo sed -i 's/DBPASSWORD/36f037b/g' /var/www/html/index.php

sudo service apache2 restart

curl http://104.197.189.168/index.php

#Database connection succeeded.</body></html>


```



### Task 6: Configure an application in a Compute Engine instance to use a Cloud Storage object

```bash

Go to Cloud Shell

gsutil ls gs://$DEVSHELL_PROJECT_ID/

#gs://qwiklabs-gcp-03-5fc0054f636b/my-excellent-blog.png

#https://storage.googleapis.com/qwiklabs-gcp-03-5fc0054f636b/my-excellent-blog.png

gcloud compute ssh bloghost

sudo bash -c 'cat > /var/www/html/index.php <<EOF
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<img src='https://storage.googleapis.com/qwiklabs-gcp-03-5fc0054f636b/my-excellent-blog.png'>
<?php
\$dbserver = "34.72.79.67";
\$dbuser = "blogdbuser";
\$dbpassword = "36f037b";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

\$conn = new mysqli(\$dbserver, \$dbuser, \$dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>

EOF'

sudo service apache2 restart


```


