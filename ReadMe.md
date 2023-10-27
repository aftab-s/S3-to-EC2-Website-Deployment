# Overview 
<p>Storing media files, such as images, videos, or other large binary files, directly in a GitHub repository may not be the best practice for several reasons like the ones listed below.

1. #### Repository Size 
    Including large media files can significantly increase the size of your repository. GitHub has storage limitations and restrictions on file sizes. Large repositories can be cumbersome to clone, fork, and contribute to.

2. #### Performance
    GitHub repositories are designed for source code, and large binary files can impact the performance of certain Git operations, such as cloning and fetching.

3. #### Version Control Overhead 
    Git is not optimized for versioning large binary files. Every change to a binary file is stored as a new version, which can lead to unnecessary repository bloat.

4. #### Collaboration
    Collaboration with multiple developers can become challenging when dealing with large repositories. Pull requests and merges may become slow and prone to conflicts.

5. #### Distribution
    GitHub is not designed as a content delivery network (CDN). Serving media files directly from a repository may result in slower load times for your website compared to using a dedicated CDN.
    
In other words, separating code and media files not only helps with performance and collaboration but also allows you to manage these assets more effectively using tools specifically designed for their use case.</p>

<p> One way to avoid the use of github for media files is to use Cloud storage services like Amazon S3, Google Cloud Storage, and Azure Blob Storage which provide scalable and efficient solutions for storing large binary files, including media files. Here, we'll be concentrating on using Amazon S3 Bucket for a typical website deployment.
</p>

<h4>Basic diagram depicting the structure of the project.</h4>
<img alt="Architecture" width="100%" src="Github + S3 to EC2.png">

## Steps involved

### Phase 1 : Creation of S3 Bucket

1. Create an S3 bucket with default configurations (make sure to note down the region as both the bucket and EC2 instance should be running in the same region). Give a proper name for the bucket as it is also necessary in the upcoming steps. 

2. Drag and drop the files from your local machine to the S3 bucket. It will take some time to upload based on the file size. (If you like to access the files now, you can click on the "Open" button on the top right)

### Phase 2 : IAM Roles and Policies

1. Go to the IAM console and create a policy from the "Policies" (give a proper name) option which is available under the "Access Management" section on the left. Choose S3 and then specify the further permissions. For simplicity you can choose "All S3 Actions (s3.*)", but it's recommended to select only the specific actions based on the Principle of Least Privileges.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::abilytics-media",
                "arn:aws:s3:::abilytics-media/*"
            ]
        }
    ]
}
```

2. Go to the IAM console and create a role from the "Roles" (give a proper name) option which is available under the "Access Management" section on the left. Under the use case, select EC2 and then choose EC2 among the given options that says "Allows EC2 instances to call AWS services on your behalf." Now you'll be presented with an "Add permissions" tab where you'll be able to select the previously created policy. Leave the rest of the options in default settings.

### Phase 3 : Creating and configuring EC2 Instance 

1. Create an EC2 instance through the Management Console and give it a proper name and choose the required instance type and launch it. Once it's up and running, select that EC2 instance and under Actions/Security select "Modify IAM role". Choose the IAM role created in Phase-2. 

2. Find your Instance In the EC2 Dashboard, select your instance.
Look for the public IP address or DNS name.
Connect via SSH: Open a terminal on your local machine and use the following command to connect to your instance (replace your-key.pem and your-ec2-public-ip with your key pair and EC2 instance's public IP):

```
ssh -i "your-key.pem" ec2-user@your-ec2-public-ip
```


Once inside the instance, run the following commands.

Update Packages
```
sudo apt-get update
sudo apt-get upgrade -y
```

Install AWSCLI
```
sudo apt install awscli
```

Install Apache2 Web server
```
sudo apt install apache2
```

3. Create a deployment file (.sh file) and make it executable. If the name of the file is "deploy.sh", then run the following command.

To create the file
```
sudo nano deploy.sh
```
Then paste the following code:
```
#!/bin/bash

# Clone your Git repository
cd /var/www
sudo git clone https://ghp_personalaccesstoken@github.com/user-name/Repository-Name.git

# Remove existing html directory
sudo rm -r html

# Move the cloned repository to the destination directory
sudo mv Repository-Name html
sudo chmod 777 html
cd html

# Make required directories and add permissions
sudo mkdir public
sudo chmod 777 public

# Specify your AWS credentials and region
#export AWS_ACCESS_KEY_ID=" "
#export AWS_SECRET_ACCESS_KEY=" "
#export AWS_DEFAULT_REGION=" "

# Set the S3 bucket name and folder
S3_BUCKET_NAME="bucket-name"
S3_FOLDER_NAME="folder"

# Set the destination directory on your EC2 instance
DESTINATION_DIR="/var/www/html/public"

# Copy files from S3 to EC2
aws s3 cp s3://$S3_BUCKET_NAME/$S3_FOLDER_NAME $DESTINATION_DIR --recursive

# Restart Apache
sudo systemctl restart apache2
```