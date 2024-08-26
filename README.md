# aws-hugo-deployment
This repo showcases how to build a Hugo Static Website that is continuously deployed via VScode. Since AWS Cloud9 has been deprecated, it is possible to launch an AWS EC2 instance in the console and using remote SSH to connect to the instance via VScode. 

To do this, follow the steps below:

## Setting Up EC2 instance on VScode
1. Ensure Remote SSH extension is installed in VScode.

2. Set up SSH keys on Github by typing `ssh-key-gen -t rsa` and click enter for all requests. cat the /.pub path and copy the key created and go to Settings > Create SSH and GPG Keys > paste the key into the new key pair.

3. Launch an AWS EC2 instance either using the AWS management console or AWS CLI in VS Code.

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
4. Create a new folder called hugo-website and set up a python virtual environment in this folder: (ensure that python and git are installed in the ec2 instance using `sudo yum install python3 -y` or `sudo yum install git -y`)
   
    a. Enter `mkdir hugo-website` in the terminal
   
    b. Install python virtual environment with `python3 -m venv ~/.hugo-website`
   
    c. Activate python virtual environment using `source ~/.hugo-website/bin/activate` (you should see `(.hugo-website)` on the left side of your command line)

5. Install extended version of hugo by going to [hugo](https://github.com/gohugoio/hugo/releases) and downloading the **extended version** of hugo into your hugo-website folder in the EC2 instance.

   e.g `wget https://github.com/gohugoio/hugo/releases/download/v0.133.0/hugo_extended_0.133.0_Linux-64bit.tar.gz`

6. Untar the `.tar.gz` file with `tar -xzvf [tar_file]` and create a bin folder `mkdir -p ~/bin` and move the hugo executable (usually highlighted in green) to `mv hugo ~/bin/`
    a. To check the hugo binary is working, type `hugo version`

## Creating a Hugo Website
7. Type `hugo new site quickstart` and you should see the following:
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
  

8. Then change into the directory and run `git init` (do not forget to install git into your EC2 instance with `sudo yum install git -y` and to setup git config)

9. Add hugo theme using git submodule via `git submodule add [github-theme-link.git] themes/theme_name`. Hugo theme options can be found [here](https://themes.gohugo.io/)

10. Then type `echo 'theme = "theme_name"' >> hugo.toml file

11. Create a post for your hugo website using `hugo new posts/my-first-post.md`

## Run Hugo Website Locally
12. Go to AWS EC2 management console, search for the security group name of your EC2-instance.

13. Go to the Security Groups tab and search for the ID containing the security gorup name found in step 11 and click on the security group ID.

14. Click on edit inbound rules and add a **custom TCP** inbound rule, enter port range as **8080** and select your source as 0.0.0.0/0

15. Type `curl ipinfo/io` into your VScode terminal and copy your IP address

16. Run hugo website locally by running `hugo serve --bind=0.0.0.0 --port=8080 --baseURL=http://[yourip]/`

17. Click on the URL where it says `Web Server is available at ....` to preview your Hugo website. 

## Setting Up An Auto Deployment System
18. Create an S3 bucket on AWS Management Console and unselect `Block all public access`
    
19. Go to the `Properties` tab of your new bucket and enable `Static website hosting` and specify your index document as `index.html`

20. Set up your bucket policy by going to `Permissions` and editting your bucket policy and add the following new statement

   ```plaintext{
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
```

21. Copy your bucket website endpoint under the `Properties` tab and Go to your `hugo.toml` file in VScode and paste the website endpoint as your baseURL.

22. Download the public folder in VScode onto your local computer and save it to your desktop. Select all files/folders in the saved folder and upload to your S3 bucket. 

23. To test that it works, go to Properties and click on the s3 bucket website link under the `Static Web Hosting` section.

## Setting Up AWS Codebuild For Continuous Deployment
24. Go to AWS Codebuild on your AWS management console and create a new project.
    a. Link the Github repository that has all the Hugo files as your source to the project.
    b. Tick "Use Git submodules" and "Rebuild everytime a code change is pushed to the repository" under Primary source webhook events
    c. Select an existing service role (Note: Create a new role under IAM using same project-name-role > Add Admininistrator Access & CodeBuildBasePolicy-aws-hugo-deployment2-eu-west-1)
    d. Tick "Use Buildspec.yml file" and specify file name from your Github repo and create build project.

26. To test if CD on AWS Codebuild works, make changes in your post in Github repo and push the changes to your repo and check status of your build projects on AWS Codebuild console.

For an in-depth tutorial (Some steps in this video may be out of date), go to [Static Hugo Website Deployment](https://www.youtube.com/watch?v=I-HTdojGdHs)
