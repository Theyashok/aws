# AWS Practical Guide for Free Tier Services

This guide provides concise, step-by-step instructions for performing common cloud tasks on Amazon Web Services (AWS), focusing on services available within the AWS Free Tier.

## Table of Contents

1.  [Hosting Static Websites on AWS S3 and EC2](#1-hosting-static-websites-on-aws-s3-and-ec2)
2.  [EC2 Setup and MySQL Database Management](#2-ec2-setup-and-mysql-database-management)
3.  [Web Application Deployment using AWS Elastic Beanstalk](#3-web-application-deployment-using-aws-elastic-beanstalk)
4.  [Serverless Computing – S3 and Lambda Integration](#4-serverless-computing--s3-and-lambda-integration)
5.  [EC2 Auto Scaling using Launch Templates and Scaling Policies](#5-ec2-auto-scaling-using-launch-templates-and-scaling-policies)
6.  [S3 Bucket File Management and Public Access Configuration](#6-s3-bucket-file-management-and-public-access-configuration)

---

## 1. Hosting Static Websites on AWS S3 and EC2

You can host a static website using either S3 for a serverless approach or an EC2 instance for a traditional server setup.

### Method 1: Using S3 (Simple & Serverless)

1.  **Create an S3 Bucket:**
    * Navigate to the S3 console and click **Create bucket**.
    * Provide a globally unique name (e.g., `your-unique-site.com`).
    * In the **Public access settings** section, uncheck **Block all public access** and acknowledge the warning.
    * Click **Create bucket**.

2.  **Upload Your Website Files:**
    * Open your new bucket and upload your static files (`index.html`, `styles.css`, etc.).

3.  **Enable Static Website Hosting:**
    * Go to the **Properties** tab of your bucket.
    * Scroll to **Static website hosting** and click **Edit**.
    * Select **Enable**, specify your `index.html` as the **Index document**, and save changes.
    * Copy the **Bucket website endpoint** URL.

4.  **Set Bucket Policy:**
    * Go to the **Permissions** tab and click **Edit** under **Bucket policy**.
    * Paste the following JSON, replacing `YOUR-BUCKET-NAME` with your bucket's actual name.
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
            }
        ]
    }
    ```
    * Save the policy. Your site is now live at the endpoint URL.

### Method 2: Using EC2 (Virtual Server)

1.  **Launch EC2 Instance:**
    * Go to the EC2 console -> **Launch instance**.
    * Choose an Amazon Machine Image (AMI), such as **Amazon Linux 2**.
    * Select the `t2.micro` instance type (Free Tier eligible).
    * Create or select a key pair for SSH access.
    * In **Network settings**, create a security group allowing **HTTP** (Port 80) and **SSH** (Port 22) traffic.
    * Launch the instance.

2.  **Connect and Install a Web Server:**
    * Connect to your instance via SSH using its Public IP and your key pair.
    * Run the following commands to install and start the Apache web server:
    ```bash
    sudo yum update -y
    sudo yum install -y httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd
    ```

3.  **Add Your Website Files:**
    * Place your website files (e.g., `index.html`) in the Apache web root directory: `/var/www/html/`.

4.  **Access Your Site:**
    * Open a web browser and navigate to your EC2 instance's **Public IPv4 address**.

---

## 2. EC2 Setup and MySQL Database Management

Set up a MySQL database on a free-tier EC2 instance.

1.  **Launch EC2 Instance:**
    * Follow Step 1 from the "EC2 (Virtual Server)" section above.

2.  **Install MySQL Server:**
    * Connect to your instance via SSH.
    * For **Amazon Linux 2**, run:
    ```bash
    sudo yum update -y
    sudo yum install -y mysql-server
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    ```

3.  **Secure MySQL Installation:**
    * Run the security script and follow the prompts to set a root password and apply security settings.
    ```bash
    sudo mysql_secure_installation
    ```

4.  **Create Database and Tables:**
    * Log in to MySQL: `mysql -u root -p`
    * Create a database and a sample table.
    ```sql
    CREATE DATABASE my_app;
    USE my_app;
    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        username VARCHAR(50) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    ```

5.  **Create a Trigger:**
    * This trigger logs new user insertions into an `audit` table.
    ```sql
    -- First, create the audit table
    CREATE TABLE audit (
        note VARCHAR(255) NOT NULL,
        log_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );

    -- Now, create the trigger
    DELIMITER //
    CREATE TRIGGER after_user_insert
    AFTER INSERT ON users
    FOR EACH ROW
    BEGIN
        INSERT INTO audit(note) VALUES(CONCAT('New user added with ID: ', NEW.id));
    END; //
    DELIMITAER ;
    ```

6.  **Create a Stored Procedure:**
    * This procedure retrieves a user by their ID.
    ```sql
    DELIMITER //
    CREATE PROCEDURE GetUserByID(IN userId INT)
    BEGIN
        SELECT * FROM users WHERE id = userId;
    END; //
    DELIMITER ;
    ```
    * Call it with: `CALL GetUserByID(1);`

---

## 3. Web Application Deployment using AWS Elastic Beanstalk

Deploy a web application without managing the underlying infrastructure.

1.  **Create an Application:**
    * Go to the Elastic Beanstalk console and click **Create application**.
    * Enter an **Application name**.

2.  **Configure Environment:**
    * Choose a platform (e.g., **Node.js, Python, Java**).
    * For **Application code**, select **Sample application** to start quickly.

3.  **Select Free Tier Preset:**
    * Under **Presets**, choose **Single instance (free tier eligible)**. This automatically configures a `t2.micro` EC2 instance.

4.  **Launch:**
    * Click **Create application**. Elastic Beanstalk provisions all necessary resources.
    * Once the environment health is **Ok**, click the URL to view your deployed application.

5.  **Deploy Your Code:**
    * Zip your application source code.
    * On your environment's dashboard, click **Upload and Deploy** to deploy your own code.

---

## 4. Serverless Computing – S3 and Lambda Integration

Create a serverless workflow where an S3 event triggers a Lambda function.

1.  **Create an S3 Bucket:**
    * Create a new S3 bucket (e.g., `my-lambda-source-bucket`). Keep public access blocked.

2.  **Create a Lambda Function:**
    * In the Lambda console, click **Create function**.
    * Select **Author from scratch**.
    * Give it a name (e.g., `S3-Event-Processor`) and choose a runtime (e.g., **Python 3.9**).
    * Click **Create function**.

3.  **Write and Deploy Lambda Code:**
    * In the **Code source** editor, paste the following Python code. This logs the name of the file uploaded to S3.
    ```python
    import json

    def lambda_handler(event, context):
        bucket = event['Records'][0]['s3']['bucket']['name']
        key = event['Records'][0]['s3']['object']['key']
        print(f"File {key} was uploaded to bucket {bucket}.")
        return {
            'statusCode': 200,
            'body': json.dumps(f'Processed file: {key}')
        }
    ```
    * Click the **Deploy** button to save your code.

4.  **Add an S3 Trigger:**
    * In the function designer above the code, click **+ Add trigger**.
    * Select **S3** as the source.
    * Choose the bucket you created.
    * Set the **Event type** to **All object create events**.
    * Acknowledge the recursive invocation warning and click **Add**.

5.  **Test the Integration:**
    * Upload any file to your source S3 bucket.
    * Go to the **Monitor** tab in your Lambda function and **View CloudWatch logs** to see the output.

---

## 5. EC2 Auto Scaling using Launch Templates and Scaling Policies

Automatically scale your EC2 instances based on demand.

1.  **Create a Launch Template:**
    * In the EC2 console, go to **Launch Templates** -> **Create launch template**.
    * Provide a template name (e.g., `my-web-app-template`).
    * Choose an AMI and set the instance type to `t2.micro`.
    * Select a key pair and a security group that allows HTTP access.
    * Click **Create launch template**.

2.  **Create an Auto Scaling Group (ASG):**
    * Go to **Auto Scaling Groups** -> **Create Auto Scaling group**.
    * Provide a name and select the **launch template** you just created.
    * Choose your VPC and at least one subnet.
    * On the **Configure group size** page, set your capacity:
        * **Desired capacity:** 1
        * **Minimum capacity:** 1
        * **Maximum capacity:** 2 (or more)

3.  **Configure a Scaling Policy:**
    * Under **Scaling policies**, choose **Target tracking scaling policy**.
    * Set **Metric type** to **Average CPU utilization**.
    * Set **Target value** to `50` (this will trigger a scale-out event if CPU exceeds 50%).
    * Review all settings and click **Create Auto Scaling group**.

---

## 6. S3 Bucket File Management and Public Access Configuration

Learn to manage files and configure public access for an S3 bucket.

1.  **Create a Bucket:**
    * Create a new S3 bucket with a unique name. Keep **Block all public access** checked initially.

2.  **File Management:**
    * **Upload:** Open the bucket and click **Upload**.
    * **Download:** Select a file and click **Download**.
    * **Create Folder:** Click **Create folder**.
    * **Delete:** Select one or more items and click **Delete**.

3.  **Configure Public Access:**
    * **Step A: Disable Block Public Access Setting:**
        * Go to the bucket's **Permissions** tab.
        * Click **Edit** next to **Block public access (bucket settings)**.
        * Uncheck **Block all public access**, save changes, and confirm.

    * **Step B: Add a Bucket Policy:**
        * Under **Bucket policy**, click **Edit**.
        * Paste the following policy to grant public read access to all objects in the bucket. Remember to replace `YOUR-BUCKET-NAME`.
        ```json
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Sid": "AllowPublicRead",
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
            }
          ]
        }
        ```
    * Save the policy. All objects in the bucket are now publicly accessible via their Object URL.
