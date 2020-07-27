# Cloud?
“If someone asks me what cloud computing is,
 I try not to get bogged down with definitions.
 I tell them that, simply put,
 cloud computing is a better way to run your business.
              **” ~ Marc Benioff, Founder, CEO and Chairman of Salesforce**
              
And he’s right. In a cost-benefit analysis, the cloud offers better tech for lower prices. And that tech makes your business more secure and flexible than ever before. If you can think of it, there’s an app to do it. And if there isn’t an app, there’s a business opportunity to write one.

_**A Terraform code is similar for all clouds and it also helps in maintaining records of what all has been done.**_

# The problem we have- 
Different clouds have different CLI commands. Hence, it poses a problem for Cloud Engineers.

# The Solution we have- 
We have the solution that using a single method that can be used for all the clouds. One such tool is Terraform. A Terraform code is similar for all clouds and it also helps in maintaining records of what all has been done.

**Steps that are to be taken in making this project:-
Create Security group which allow the port 80.

 👉 Launch EC2 instance.

 👉 In this Ec2 instance use the existing key or provided key and security group which we have created in step 1.

 👉 Launch one Volume using the EFS service and attach it in your vpc, then mount that volume into /var/www/html

 👉 Developer have uploded the code into github repo also the repo has some images.

 👉 Copy the github repo code into /var/www/html.

 👉 Create S3 bucket, and copy/deploy the images from github repo into the s3 bucket and change the permission to public readable.

 👉 Create a Cloudfront using s3 bucket(which contains images) and use the Cloudfront URL to update in code in /var/www/html

 👉 Step - 1 First of all, configure your AWS profile in your local system using cmd. Fill your details & press Enter.
 
 **Step - 1** First of all, configure your AWS profile in your local system using cmd. Fill up your credentials & press Enter.
 
 
                                        aws configure --profile Vishnu
                                        AWS Access Key ID [****************WO3Z]:
                                        AWS Secret Access Key [****************b/hJ]:
                                        Default region name [ap-south-1]:
                                        Default output format [None]:
 
 **Step - 2** Next, we create a VPC which will be used later.
 
                                       resource "aws_vpc" "vishnu_vpc" {
                                        cidr_block = "192.168.0.0/16"
                                        instance_tenancy = "default"
                                        enable_dns_hostnames = true
                                        tags = {
                                          Name = "vishnu_vpc"
                                        }
                                      }
                                      
  **Step - 3** Now. a subnet needs to be created which will be used to launch the instance later.                               
  
                                        resource "aws_subnet" "vishnu_subnet" {
                                        vpc_id = "${aws_vpc.vishnu_vpc.id}"
                                        cidr_block = "192.168.0.0/24"
                                        availability_zone = "ap-south-1a"
                                        map_public_ip_on_launch = "true"
                                        tags = {
                                          Name = "vishnu_subnet"
                                        }
                                      }
**Step - 4** I have also created a custom security group with all the reqd. permissions. I'll be using this same security group to launch my instance.

                                      resource "aws_security_group" "vishnu_sg" {

                                          name        = "vishnu_sg"
                                          vpc_id      = "${aws_vpc.vishnu_vpc.id}"


                                          ingress {

                                            from_port   = 80
                                            to_port     = 80
                                            protocol    = "tcp"
                                            cidr_blocks = [ "0.0.0.0/0"]

                                          }


                                          ingress {

                                            from_port   = 2049
                                            to_port     = 2049
                                            protocol    = "tcp"
                                            cidr_blocks = [ "0.0.0.0/0"]

                                          }



                                          ingress {

                                            from_port   = 22
                                            to_port     = 22
                                            protocol    = "tcp"
                                            cidr_blocks = [ "0.0.0.0/0"]

                                          }




                                          egress {

                                            from_port   = 0
                                            to_port     = 0
                                            protocol    = "-1"
                                            cidr_blocks = ["0.0.0.0/0"]
                                          }


                                          tags = {

                                            Name = "vishnu_sg"
                                          }
                                        }

**Step - 5** Now, let's create an EFS Account & configure it.


                                      resource "aws_efs_file_system" "vishnu_efs" {
                                        creation_token = "vishnu_efs"
                                        tags = {
                                          Name = "vishnu_efs"
                                        }
                                      }

                                      resource "aws_efs_mount_target" "vishnu_efs_mount" {
                                        file_system_id = "${aws_efs_file_system.vishnu_efs.id}"
                                        subnet_id = "${aws_subnet.vishnu_subnet.id}"
                                        security_groups = [aws_security_group.vishnu_sg.id]
                                      }
                                   
**Step - 6** Next, we create a Gateway & a Routing Table.


                                      resource "aws_internet_gateway" "vishnu_gw" {
                                        vpc_id = "${aws_vpc.vishnu_vpc.id}"
                                        tags = {
                                          Name = "vishnu_gw"
                                        }
                                      }


                                      resource "aws_route_table" "vishnu_rt" {
                                        vpc_id = "${aws_vpc.vishnu_vpc.id}"

                                        route {
                                          cidr_block = "0.0.0.0/0"
                                          gateway_id = "${aws_internet_gateway.vishnu_gw.id}"
                                        }

                                        tags = {
                                          Name = "vishnu_rt"
                                        }
                                      }
                                      resource "aws_route_table_association" "vishnu_rta" {
                                      subnet_id = "${aws_subnet.vishnu_subnet.id}"
                                      route_table_id = "${aws_route_table.vishnu_rt.id}"
                                    }
   
   **Step - 7** Now, it's time to launch our instance. I've also written a provisioner code to downlaod & setup the Apache Web Server inside this instance.

                                                   resource "aws_instance" "test_ins" {
                                                  ami             =  "ami-052c08d70def0ac62"
                                                  instance_type   =  "t2.micro"
                                                  key_name        =  "newkey"
                                                  subnet_id     = "${aws_subnet.vishnu_subnet.id}"
                                                  security_groups = ["${aws_security_group.vishnu_sg.id}"]


                                                 connection {
                                                    type     = "ssh"
                                                    user     = "ec2-user"
                                                    private_key = file("C:/Users/This PC/Downloads/newkey.pem")
                                                    host     = aws_instance.test_ins.public_ip
                                                  }

                                                  provisioner "remote-exec" {
                                                    inline = [
                                                      "sudo yum install amazon-efs-utils -y",
                                                      "sudo yum install httpd  php git -y",
                                                      "sudo systemctl restart httpd",
                                                      "sudo systemctl enable httpd",
                                                      "sudo setenforce 0",
                                                      "sudo yum -y install nfs-utils"
                                                    ]
                                                  }

                                                  tags = {
                                                    Name = "my_os"
                                                  }
                                                }
                                                
   **Step - 8** Now since our instance is launched, we mount the EFS Volume to /var/www/html folder where all the code is stored. This will ensure no data loss in case the instance crashes or is accidently deleted.     
   
   
   
                                                    resource "null_resource" "mount"  {
                                                      depends_on = [aws_efs_mount_target.vishnu_efs_mount]
                                                      connection {
                                                        type     = "ssh"
                                                        user     = "ec2-user"
                                                        private_key = file("C:/Users/This PC/Downloads/newkey.pem")
                                                        host     = aws_instance.test_ins.public_ip
                                                      }
                                                    provisioner "remote-exec" {
                                                        inline = [
                                                          "sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport                                                                                 ${aws_efs_file_system.vishnu_efs.id}.efs.ap-south-1.amazonaws.com:/ /var/www/html",
                                                          "sudo rm -rf /var/www/html/*",
                                                          "sudo git clone https://github.com/vishnu7455/Task-1-.git /var/www/html/",
                                                          "sudo sed -i 's/url/${aws_cloudfront_distribution.my_front.domain_name}/g' /var/www/html/index.html"
                                                        ]
                                                      }
                                                    }
**Step - 9** Now, we create an S3 bucket on AWS. The code snippet for doing the same is as follows -


                                                      resource "aws_s3_bucket" "vishnu_bucket" {
                                                      bucket = "vishnu23"
                                                      acl    = "private"
                                                      force_destroy = true

                                                      tags = {
                                                        Name        = "vishnu2301"
                                                      }
                                                    }

                                                    locals {
                                                      s3_origin_id = "myS3Origin"
                                                    }

**Step - 10** Now that the S3 bucket has been created, we will upload the images that we had downloaded from Github in our local system in the above step. Here, I have uploaded just one pic. You can upload more if you wish.

                                                    resource "aws_s3_bucket_object" "object" {
                                                      bucket = "${aws_s3_bucket.vishnu_bucket.id}"
                                                      key    = "test_pic"
                                                      source = "C:/Users/This PC/Pictures/sir.png"
                                                      acl    = "public-read"
                                                      content_type = "image/png"
                                                    }

**Step - 11** Now, we create a CloudFront & connect it to our S3 bucket. The CloudFront ensures speedy delievery of content using the edge locations from AWS across the world.

                                                resource "aws_cloudfront_distribution" "my_front" {
                                                   origin {
                                                         domain_name = "${aws_s3_bucket.vishnu_bucket.bucket_regional_domain_name}"
                                                         origin_id   = "${local.s3_origin_id}"

                                                 custom_origin_config {

                                                         http_port = 80
                                                         https_port = 80
                                                         origin_protocol_policy = "match-viewer"
                                                         origin_ssl_protocols = ["TLSv1", "TLSv1.1", "TLSv1.2"] 
                                                        }
                                                      }
                                                         enabled = true

                                                 default_cache_behavior {

                                                         allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
                                                         cached_methods   = ["GET", "HEAD"]
                                                         target_origin_id = "${local.s3_origin_id}"

                                                 forwarded_values {

                                                       query_string = false

                                                 cookies {
                                                          forward = "none"
                                                         }
                                                    }

                                                          viewer_protocol_policy = "allow-all"
                                                          min_ttl                = 0
                                                          default_ttl            = 3600
                                                          max_ttl                = 86400

                                                }
                                                  restrictions {
                                                         geo_restriction {
                                                           restriction_type = "none"
                                                          }
                                                     }
                                                 viewer_certificate {
                                                       cloudfront_default_certificate = true
                                                       }
                                                }
                                                
**Now, we go to /var/www/html & update the link of the images with the link from CloudFront.**

**Step - 12** Now, we write a terraform code snippet to automatically retrieve the public ip of our instance and open it in chrome. This will land us on the home page of our website that is present in /var/www/html.

                                                resource "null_resource" "local_exec"  {
                                                depends_on = [
                                                    null_resource.mount,
                                                  ]
                                                  provisioner "local-exec" {
                                                      command = "start chrome  ${aws_instance.test_ins.public_ip}"
                                                         }
                                                }
**Step - 13** Now, we go to our Command Prompt & write the following command :

                                                terraform init
                                               
**After the plugins have been downloaded, we do terraform apply.**                                                
**We give our approval when the permission is asked.**


# Finally, we see our webserver and its content


![](/Output.png)
