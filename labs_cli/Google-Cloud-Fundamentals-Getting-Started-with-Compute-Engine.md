# Google Cloud Fundamentals: Getting Started with Compute Engine

## Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.



```bash
#setting my project
gcloud config set project qwiklabs-gcp-03-b7387e5db590

#creating the instance
gcloud compute instances create my-vm-1 \
--zone=us-central1-a \
--machine-type=n1-standard-1 \
--subnet=default \
--tags=http-server \
--image=debian-9-stretch-v20200910 \
--image-project=debian-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=my-vm-1

#set the firewall rule to allow http traffic
gcloud compute firewall-rules create default-allow-http \
--direction=INGRESS \
--priority=1000 \
--network=default \
--action=ALLOW \
--rules=tcp:80 \
--source-ranges=0.0.0.0/0 \
--target-tags=http-server


```



## Create a Compute Engine virtual machine using the gcloud command-line interface.



```bash
#find the zone assigned
gcloud compute zones list | grep us-central1

#set a different zone
gcloud config set compute/zone us-central1-f

#creating the instance
gcloud compute instances create my-vm-2 \
--machine-type=n1-standard-1 \
--subnet=default \
--tags=http-server \
--image=debian-9-stretch-v20200910 \
--image-project=debian-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=my-vm-2

```



## Connect between the two instances.



**Activate Cloud Shell from the top menu bar.**

```bash
#login to the my-vm-2 instance

gcloud compute ssh my-vm-2

#ping first vm with 4 packets
ping -c4 my-vm-1

#logout of my-vm-2
exit

#login to my-vm-1 instance, need to specify the zone because we changed the default zone earlier

gcloud compute ssh my-vm-1 --zone=us-central1-a

sudo apt-get install nginx-light -y

sudo sh -c 'echo "Hi from Mohamed Kassem" > /var/www/html/index.nginx-debian.html'

curl http://localhost/

exit

gcloud compute ssh my-vm-2

curl http://my-vm-1/




```

