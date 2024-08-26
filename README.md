# aws-hugo-deployment
This repo showcases how to build a Hugo Static Website that is continuously deployed via VScode. Since AWS Cloud9 has been deprecated, it is possible to launch an AWS EC2 instance in the console and using remote SSH to connect to the instance via VScode. 

To do this, follow the steps below:

## Setting Up EC2 instance on VScode
1. Ensure Remote SSH extension is installed in VScode.
2. Launch an AWS EC2 instance either using the AWS management console or AWS CLI in VS Code.

    a. Create a new key-pair and download the `.pem` file to your local computer.
    
    b. Move the `.pem` file to `~/.ssh/`.
    
    c. Configure your `~/.ssh/config` file and add the following:

    ```plaintext
        Host [instance_name]
        HostName [Public IPv4 DNS]
        User ec2-user  # do not change user
        IdentityFile [path_to_pem_file]
    ```

    d. Run `chmod 400 "vscode-instance.pem"` in your ~/.ssh folder via your terminal.
   
    e. Connect to your EC2 instance on VScode using Remote SSH extension by going to your command palette via `cmd + shift + P` and search for `Remote SSH: Connect to Host`

    f. If your `~/.ssh/config` file was setup correctly, you should see the instance you created in the list and click on it. (This will open a new VScode window. Click continue to allow Remote SSH to connect to your instance)

    g. Opening the terminal in VScode now will open a shell in your new EC2 instance.

## Downloading Hugo Binary and Checking Installation
3. Create a new folder called hugo-website and set up a python virtual environment in this folder: (ensure that python and git are installed in the ec2 instance using `sudo yum install python3 -y` or `sudo yum install git -y`)
   
    a. Enter `mkdir hugo-website` in the terminal
   
    b. Install python virtual environment with `python3 -m venv ~/.hugo-website`
   
    c. Activate python virtual environment using `source ~/.hugo-website/bin/activate` (you should see `(.hugo-website)` on the left side of your command line)

4. Install extended version of hugo by going to [hugo](https://github.com/gohugoio/hugo/releases) and downloading the **extended version** of hugo into your hugo-website folder in the EC2 instance.

   e.g `wget https://github.com/gohugoio/hugo/releases/download/v0.133.0/hugo_extended_0.133.0_Linux-64bit.tar.gz`

5. Untar the `.tar.gz` file with `tar -xzvf [tar_file]` and create a bin folder `mkdir -p ~/bin` and move the hugo executable (usually highlighted in green) to `mv hugo ~/bin/`
    a. To check the hugo binary is working, type `hugo version`

## Creating a Hugo Website
6. Type `hugo new site quickstart` and you should see the following:
   ```
   
   Congratulations! Your new Hugo site was created in /home/ec2-user/hugo-website/quickstart.
   Just a few more steps...
   1. Change the current directory to /home/ec2-user/hugo-website/quickstart.
   2. Create or install a theme:
     - Create a new theme with the command "hugo new theme <THEMENAME>"
     - Or, install a theme from https://themes.gohugo.io/
   3. Edit hugo.toml, setting the "theme" property to the theme name.
   4. Create new content with the command "hugo new content <SECTIONNAME>/<FILENAME>.<FORMAT>".
   5. Start the embedded web server with the command "hugo server --buildDrafts".
   See documentation at https://gohugo.io/.
  

7. Then change into the directory and run `git init` (do not forget to install git into your EC2 instance with `sudo yum install git -y` and to setup git config)

8. Add hugo theme using git submodule via `git submodule add [github-theme-link.git] themes/theme_name`. Hugo theme options can be found [here](https://themes.gohugo.io/)

9. Then type `echo 'theme = "theme_name"' >> hugo.toml file

10. Create a post for your hugo website using `hugo new posts/my-first-post.md`

## Run Hugo Website Locally
11. Go to AWS EC2 management console, search for the security group name of your EC2-instance.

12. Go to the Security Groups tab and search for the ID containing the security gorup name found in step 11 and click on the security group ID.

13. Click on edit inbound rules and add a **custom TCP** inbound rule, enter port range as **8080** and select your source as 0.0.0.0/0

14. Type `curl ipinfo/io` into your VScode terminal and copy your IP address

15. Run hugo website locally by running `hugo serve --bind=0.0.0.0 --port=8080 --baseURL=http://[yourip]/`

16. Click on the URL where it says `Web Server is available at ....` to preview your Hugo website. 

## Setting Up An Auto Deployment System
17. Create an S3 bucket on AWS Management Console and unselect `Block all public access`
    
18. Go to the `Properties` tab of your new bucket and enable `Static website hosting` and specify your index document as `index.html`

19. Set up your bucket policy by going to `Permissions` and editting your bucket policy and add the following new statement

   ```{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicReadGetObject",
			"Principal": "*",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject"
			],
			"Resource": [
			    "arn:aws:s3:::[your-bucket-name]/*"
			    ]
		}
	]
	}


20. Copy your bucket website endpoint under the `Properties` tab and Go to your `hugo.toml` file in VScode and paste the website endpoint as your baseURL.

21. Download the public folder in VScode onto your local computer and save it to your desktop. Select all files/folders in the saved folder and upload to your S3 bucket. 

22. To test that it works, go to Properties and click on the s3 bucket website link under the `Static Web Hosting` section.

## Setting Up AWS Codebuild For Continuous Deployment
23. Go to AWS Codebuild on your AWS management console and 
[Static Hugo Website Deployment](https://www.youtube.com/watch?v=I-HTdojGdHs)
