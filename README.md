# AWS-Codepipeline

### AWS CodeCommit

1. Created a codecommit repository .
<img width="1280" alt="create repo on code commit" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/1c725198-7b06-4f14-9df9-a13211ec2770">
2. Copied the repository URL .

<img width="1280" alt="copy the repo url" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/eaa10e31-4395-4585-b46e-aa2e87d4e193">
3. Tried to git clone using the copied URL in local editor's terminal , but a screen appeared asking for git credentials . 
<img width="1279" alt="now try to git clone at local with copied url but a git credential it will ask" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/030dda62-b244-4b39-85d4-b36445b6d8dd">
4. Created git credential for the IAM user role 
<img width="1280" alt="create git credentials" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/1ecde127-911a-45e3-8c6f-8bcc2b020acd">

5. After entering git credentials , i successfully cloned the repository .
<img width="1280" alt="git credentials" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/ad615208-a15f-44ea-81ef-60746100a75b">

6. Created a index.html file and pushed the ffile into codecommit repository . 
<img width="1247" alt="created and pushed the index html to the repo" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/9aeb67c3-448d-429c-a40a-6ca5ff856119">


### AWS CodeBuild



7. CodeBuild stage needs a buildspec.yml file . So created a buildspec.yml file and pushed it to codecommit repo .
<img width="1277" alt="add buildspec yml to repo as it is needed during code build" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/1d8f02bf-8bcf-411a-9a05-78030850f754">


```
  version : 0.2

  phases:
    install:
      commands:
        - echo Start with NGINX install
        - sudo apt-get update
        - sudo apt-get install nginx -y
    build:
      commands:
        - echo Building...
        - cp index.html /var/www/html/
  artifacts:
    files:
      - /var/www/html/index.html
```

9. Then created a build .
   

https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/d2ff18e7-bb61-484f-953b-77048ebc555c

10 . After this i have to store artifacts , so i created a s3 bucket . Then attached the artifacts to the codebuild stage .

<img width="1277" alt="Screenshot 2024-01-11 153949" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/a9bc3371-450a-43ab-af86-ce150ca218ad">

<img width="1280" alt="mention folder name" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/1ef52a09-3dae-4630-983a-8d9f44895e06">


11. Then started building .
    
<img width="1279" alt="build successful" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/9c2e6b16-e53b-4837-b7b9-a2d9442048c0">




### AWS CodeDeploy

12. Created an application on CodeDeploy
<img width="1280" alt="create application" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/b116e13f-4bda-47b3-8d6e-245b102d72b1">


13. Created an EC2 insatnce with ssh,http,https added in security group . As the ec2 insatnce needs to interact with s3 bucket , codedeploy , ec2 ,so attached the policies .

<img width="1280" alt="ec2 policy" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/1c5c5aaa-941e-4cbf-9fe6-3ad9a46d8ae8">


14. Created a IAM role for deployment group with certain policies .
    <img width="1280" alt="policy attached (2)" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/4a43d758-8082-4307-9049-b3cf377782e8">

15. Then created deployment group .

https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/64dc71ea-b708-4f32-8c11-52f97ddc42c3

16. Then connected the ec2 instance to create codedeploy agent .

```
#!/bin/bash 
# This installs the CodeDeploy agent and its prerequisites on Ubuntu 22.04.  
sudo apt-get update 
sudo apt-get install ruby-full ruby-webrick wget -y 
cd /tmp 
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb 
mkdir codedeploy-agent_1.3.2-1902_ubuntu22 
dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22 
sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control 
dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/ 
sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb 
systemctl list-units --type=service | grep codedeploy 
sudo service codedeploy-agent status

```
<img width="1280" alt="codedeploy running" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/83fedfe1-1a8f-4930-8f21-b9b7c1f0f7a9">

17. Like CodeBuild stage , CodeDeploy also needs a appspec.yml for configuration and in buildspec.yml changed the artifacts files . After making all these changes then pushed the code
to the codecoommit repository . 

appspec.yml
```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  AfterInstall:
    - location: scripts/install_nginx_service.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_nginx_service.sh
      timeout: 300
      runas: root
```

install_nginx_service.sh
```
#!/bin/bash

sudo apt-get update
sudo apt-get install -y nginx
```

start_nginx_service.sh
```
#!/bin/bash

sudo service nginx start
```
<img width="1278" alt="added appspec yml" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/f9f9bc0d-55cc-4f75-8406-b7deaa7e79c8">

builspec.yml
<img width="1247" alt="changed buildspec to solve error" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/19acadba-3cdf-400d-b229-119910c53aee">

18. Then created a deployment using revise location as s3 bucket's URL .


<img width="1280" alt="Screenshot 2024-01-11 165954" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/5854abe5-a571-41f9-970c-3dabf590a9b5">


19. After this i have checked whether all the files present in CodeCommit repo or not . Then again started the code build stage . Then started the deployment .


<img width="1280" alt="deployment suceeded" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/647cdc94-ee0b-41b9-af8c-b65b26ba81b8">

20. After successful deployment then accessed the Public IP of ec2 instance to access the webpage .
    
<img width="1279" alt="after deployment succefully accessed" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/7242e58c-1eb3-480d-bd1f-32871433845f">


### AWS CodePipeline

21. Created a pipeline
<img width="1280" alt="create pipeline" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/6c22540e-84a9-4844-9201-1d9dfdd3de70">
    
22. Added the code source and choosed "AWS CodePipeline" in "change detection option" to  

<img width="1280" alt="code source provider" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/e0707c41-fcd9-495d-b362-bb5190f7690c">


23.



<img width="1280" alt="build" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/13a22b25-0a03-4c1b-9ca7-fa73e6677a03">

24.   

    

<img width="1280" alt="deployment" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/6015e895-758c-4a11-ac34-1894d3a1e6a0">

25. 





<img width="1280" alt="pipeline succeded" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/fc83c483-1252-4f3b-a65f-6c5f5b85ef9a">




26. Then accessed webpage using public ip of ec2 instance . 

<img width="1279" alt="after deployment succefully accessed" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/349b269f-a8bb-45bf-ad0d-58416e92dde9">


27. Then made some changes in the index.html and pushed the changes into codecommit repo .


<img width="1276" alt="index html file code is updated" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/5c008eb2-f874-4fef-b73c-c59992e0f845">

28. Then pipeline is automatically started .


<img width="1280" alt="look here the message also updated" src="https://github.com/PranitRout07/AWS-Codepipeline/assets/102309095/f8a669a9-e430-4f90-a574-649aad5fad0b">

29. Then accessed the updated webpage from the public IP of ec2 instance .

    











