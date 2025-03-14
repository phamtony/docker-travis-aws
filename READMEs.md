docker build -f Dockerfile.dev .

#wont work
docker run <image id>

#will work because of porting
docker run -p 3000:3000 <image id>

docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image id>
# no need to use $(pwd), use pwd (oh my zsh)


-v is volume
pwd is path to folder
: is indication of reference, having container reference the local folder after the : (which is /app)


-v /app/node_modules 
doesn't reference, bypasses that and has it refer to what's in the container


#Docker compose:

docker compose yml 

    ports: 
      -"3000:3000"
    volumes:
      - /app/node_modules
      - .:/app


For testing:
without compose: docker run -it <container_id> npm run test


#Docker compose testing

  web: 
    build: 
      context: .
      dockerfile: Dockerfile.dev
    ports: 
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  test:
    build: 
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]  



 Dockerfile for production, using nginx

build phase and run phase 2 diff images for one dockerfile






For github and elastic aws beatstalk

Docker Compose config Update

1. Rename the development docker-compose.yml file to docker-compose-dev.yml.

Create EC2 IAM Instance Profile

1. Go to AWS Management Console

2. Search for IAM and click the IAM Service.

3. Click Roles under Access Management in the left sidebar.

4. Click the Create role button.

5. Select AWS Service under Trusted entity type. Then select EC2 under common use cases.

6. Search for AWSElasticBeanstalk and select the AWSElasticBeanstalkWebTier, AWSElasticBeanstalkWorkerTier and AWSElasticBeanstalkMulticontainerDocker policies. Click the Next button.

7. Give the role the name of aws-elasticbeanstalk-ec2-role

8. Click the Create role button.



Create Elastic Beanstalk Environment

1. Go to AWS Management Console

2. Search for Elastic Beanstalk and click the Elastic Beanstalk service.

3. If you've never used Elastic Beanstalk before you will see a splash page. Click the Create Application button. If you have created Elastic Beanstalk environments and applications before, you will be taken directly to the Elastic Beanstalk dashboard. In this case, click the Create environment button. There is now a flow of 6 steps that you will be taken through.

5. You will need to provide an Application name, which will auto-populate an Environment Name.

6. Scroll down to find the Platform section. You will need to select the Platform of Docker. This will auto-select several default options. Change the Platform branch to Docker running on 64bit Amazon Linux 2. The new 2023 branch currently has issues with single-container deployments.

7. Scroll down to the Presets section and make sure that free tier eligible has been selected:

8. Click the Next button to move to Step #2.

9. You will be taken to a Service Access configuration form.

Select Create and use new service role and name it aws-elasticbeanstalk-service-role. You will then need to set the EC2 instance profile to the aws-elasticbeanstalk-ec2-role created earlier (this will likely be auto-populated for you).

10. Click the Skip to Review button as Steps 3-6 are not applicable.

11. Click the Submit button and wait for your new Elastic Beanstalk application and environment to be created and launch.

12. Click the link below the checkmark under Domain. This should open the application in your browser and display a Congratulations message.



Update Object Ownership of S3 Bucket

1. Go to AWS Management Console

2. Search for S3 and click the S3 service.

3. Find and click the elasticbeanstalk bucket that was automatically created with your environment.

4. Click Permissions menu tab

5. Find Object Ownership and click Edit

6. Change from ACLs disabled to ACLs enabled. Change Bucket owner Preferred to Object Writer. Check the box acknowledging the warning.

7. Click Save changes.



Add AWS configuration details to .travis.yml file's deploy script

1. Set the region. The region code can be found by clicking the region in the toolbar next to your username.

eg: 'us-east-1'

2. app should be set to the Application Name (Step #4 in the Initial Setup above)

eg: 'docker'

3. env should be set to the lower case of your Beanstalk Environment name.

eg: 'docker-env'

4. Set the bucket_name. This can be found by searching for the S3 Storage service. Click the link for the elasticbeanstalk bucket that matches your region code and copy the name.

eg: 'elasticbeanstalk-us-east-1-923445599289'

5. Set the bucket_path to 'docker'

6. Set access_key_id to $AWS_ACCESS_KEY

7. Set secret_access_key to $AWS_SECRET_KEY



Create an IAM User

1. Search for the "IAM Security, Identity & Compliance Service"

2. Click "Create Individual IAM Users" and click "Manage Users"

3. Click "Add User"

4. Enter any name you’d like in the "User Name" field.

eg: docker-react-travis-ci

5. Click "Next"

6. Click "Attach Policies Directly"

7. Search for "beanstalk"

8. Tick the box next to "AdministratorAccess-AWSElasticBeanstalk"

9. Click "Next"

10. Click "Create user"

11. Select the IAM user that was just created from the list of users

12. Click "Security Credentials"

13. Scroll down to find "Access Keys"

14. Click "Create access key"

15. Select "Command Line Interface (CLI)"

16. Scroll down and tick the "I understand..." check box and click "Next"

Copy and/or download the Access Key ID and Secret Access Key to use in the Travis Variable Setup.



Travis Variable Setup

1. Go to your Travis Dashboard and find the project repository for the application we are working on.

2. On the repository page, click "More Options" and then "Settings"

3. Create an AWS_ACCESS_KEY variable and paste your IAM access key from step #13 above.

4. Create an AWS_SECRET_KEY variable and paste your IAM secret key from step #13 above.



Deploying App

1. Make a small change to your src/App.js file in the greeting text.

2. In the project root, in your terminal run:

git add.
git commit -m “testing deployment"
git push origin main
3. Go to your Travis Dashboard and check the status of your build.

4. The status should eventually return with a green checkmark and show "build passing"

5. Go to your AWS Elastic Beanstalk application

6. It should say "Elastic Beanstalk is updating your environment"

7. It should eventually show a green checkmark under "Health". You will now be able to access your application at the external URL provided under the environment name.






Workflow from another course:


name: Deploy Frontend
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
      - run: docker build -t cygnetops/react-test -f Dockerfile.dev .
      - run: docker run -e CI=true cygnetops/react-test npm test

      - name: Generate deployment package
        run: zip -r deploy.zip . -x '*.git*'

      - name: Deploy to EB
        uses: einaregilsson/beanstalk-deploy@v18
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
          aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
          application_name: docker-gh
          environment_name: Dockergh-env
          existing_bucket_name: elasticbeanstalk-us-east-1-923445559289
          region: us-east-1
          version_label: ${{ github.sha }}
          deployment_package: deploy.zip
