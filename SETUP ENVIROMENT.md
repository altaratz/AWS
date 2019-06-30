# Setup an Environment
## Based on KIN best practices

1. With a KIN github account access [this page](https://github.com/kinecosystem/aws-best-practice-CF) 
    1. Download the templates folder.
1. Go to Cloud Formation in AWS management console and deploy using the **template**<br>**NOTE**: If you already have stacks defined, concider deleting them to avoid confusion
    1. Clone and pull latest version of GIT [best practices repo](https://github.com/kinecosystem/aws-best-practice-CF)
    1. Create an S3 bucket
    1. Create a folder named templates inside the bucket
    1. Upload all the files in the GIT repo in the templates folder to the templates folder in the bucket
    1. Open AWS Console and go to EC2
        1. Generate two key pairs. One for **bastion** and one for the **other instances**
        1. KEEP THESE KEYS SOMEWHERE secure so you can find them
    1. On the AWS Console go to CloudFormation
    1. Click on Create **Stack**
        1. Select the option to upload a file
        1. Select the file main.template.json in the GIT repo, in the templates folder
        1. Next
        1. Give it a name
        1. Set an email - yours probably
        1. Select the bastion key you created
        1. Select the instances key you created
        1. Set availability zones
        1. In the bucket name, write the name of the bucket into which you uploaded all the templates (the one mentioned above)
        1. Next
        1. Set tags if you want … Next
        1. Kick stack creation - will take a few minutes
    1. Create a **bastion** server (**NOT NEEDED** - The template creates a bastion)
        1. In the Management Zone
        1. Give it a public IP
        1. Security Groups: Open port 22 from anywhere (if you have a predefined SG, better use it)
        1. Network ACL: when the bastion selected, click on the link of the subnet (at the bottom)
        1. With the subnet selected, click on Network ACL at the bottom
        1. Make sure 22 is open from anywhere
    1. Create a **web server** visible from **the world**
        1. In the Production DMZ Subnet A
        1. Give it a public IP
        1. In the Security Groups
            1. Port 22 from 10.0.0.0/16
            1. Other ports you may need (80, 8080, 3000, 5000, 443, etc) from 0.0.0.0/0, ::/0	(anywhere)
            1. Outbound open to anywhere
        1. **Network ACL**:
            1. While instance selected click on link to the subnet. Then open the subnet ACL
            1. Give the rules numbers 100, 101 (not sure why, yet)
            1. Open port 22 from 10.0.0.0/16
            1. Open any port needed from the world 0.0.0./0, ::/0
            1. Allow all traffic outbound
    1. Create a **web server** available **only** through a **load balancer**
        1. In Production App Subnet A
        1. Security Gropups
            1. Port 22 from 10.0.0.0/16 (SSH only from bastion)
            1. Other ports you may need from the LB like 3000 for Node or 5000 for flask default ports (80 in linux is a problem): from 10.100.0.0/16
            1. Outbound anywhere
        1. **Network ACL**:
            1. Give the rules numbers 100, 101 (not sure why, yet)
            1. Open port 22 from 10.0.0.0/16
            1. Open any port needed from the LB: from 10.100.0.0/16
            1. Allow all traffic outbound
    1. Create a **Load Balancer**
    for making the web server accessible from the web
        1. Setup a domain (not thoroughly covered here)
            1. AWS console ⇒ Got to Route 53
            1. Register a domain name
        1. Create a certificate (Also not covered - sorry)
            1. Go to AWS console ⇒ Certificates Manager
            1. Request a public certificate for the domain you got - it is recommended you create a wild card certificate, for enabling some subdomains.
        1. In EC2 management console open **Load Balancers**
        1. Create a Load Balancer
            1. **TCP**: Network LB a basic forwarding for very high performance and 
            1. **HTTP / HTTPS**: The application LB is useful for logical routing such as port translation, logical forwarding, etc … prolly this is the one for you
        1. Name: Whatever makes sense
        1. Internet-facing: For public availability
        1. Add a listener for HTTPS
            1. Remove HTTP if you don’t need to listen to HTTP
            1. You can listen to any custom port you like over HTTP or HTTPS
        1. In the VPC make sure you chose the same VPC as your web server is found, Production VPC (if you’ve been a good boy)
        1. Chose the subnet that has internet access, Production DMZ Subnet A and if you have an additional region chose also B
        1. NEXT
        1. Chose the certificate you’ve previously created
        1. Keep the selected security policy that is selected
        1. NEXT
        1. Chose from the existing security groups the one that let’s you get HTTPS or HTTP and the ports you defined for your load balancer
        1. NEXT
        1. Define a target group
            1. Select type **instance**, so if your instance gets a different IP the LB keeps pointing at it
            1. Select HTTP/HTTPS and the port as you defined you LB
            1. Define health checks
            This is a path to some basic test call to your web server so the LB knows its there. This is a very basic test to make sure the LB sees your web browser. Though not mandatory, it is recommended
        1. NEXT
        1. Select to what instances you want the LB to fwd traffic to
        1. CREATE
        1. Once created, click on the LB VPC
        1. In the VPC page, check the Network ACLs and make sure that the ports you decided to enable to reach the LB are also allowed by the Network ACL
    1. Forward the domain you created to the load balancer, to any subdomain you chose using the certificate you created.

