
# Creating the Elastic Beanstalk Environment

Please use the following instructions to create an environment with the Amazon Linux 2023 Platform. Since the AWS UI and options change frequently, these are provided as a lecture note instead of a video lecture.

1. Go to AWS Management Console
2. Search for **Elastic Beanstalk** and click the Elastic Beanstalk service.
![1000](https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-01-03_18-38-11-e6468b90cfb1a1840d3d866f11f01bf7.png)

3. Click the **Create environment** button.
![1000](https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-01-03_18-38-11-24fb159e322909802b3165c741aa2267.png)

4. You will need to provide an **Application name**, which will auto-populate an **Environment name**.
![800](https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-01-03_18-38-12-679a3e16830c8b5f6adbe5170c6683d3.png)

5. Scroll down to find the **Platform** section. You will need to select the Platform of **Docker**. This will auto-select several default options.

![800](https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-01-03_18-38-12-aa6f89117ffff77bc62c7af80c403ca3.png)

6.  Scroll down to the **Presets** section and make sure that **free tier eligible** has been selected
![800](https://img-c.udemycdn.com/redactor/raw/article_lecture/2023-04-28_21-06-41-627f721951c492d8085a9e8953c806d8.png)

7. Click the **Next** button to move to Step #2.
8. You will be taken to a Service Access configuration form.
Ensure that **Use an existing service role** is selected and that the service role created in Section 7 is listed. This should also auto-populate the EC2 instance profile with the ec2-role that was previously created.
![800](https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-01-03_19-29-27-f777ec8f77a626d458ef7c2210ff962b.png)

9. Click the **Skip to Review** button.
10. Click the **Submit** button and wait for your new Elastic Beanstalk application and environment to be created and launch.

**Important** - Attached to this lecture is a cheatsheet complete with all the steps to configure the multi-container project in AWS.

## Required Updates for Docker Compose

1. Rename the current docker-compose file
	- Rename the **docker-compose.yml** file to **docker-compose-dev.yml**. Going forward you will need to pass a flag to specify which compose file you want to build and run from:  
`docker-compose -f docker-compose-dev.yml up`  
`docker-compose -f docker-compose-dev.yml up --build`  
`docker-compose -f docker-compose-dev.yml down`

2. Create a production-only docker-compose.yml file
	- The production compose file will follow closely what was written in the Dockerrun.aws.json. There are two major differences:

**No Container Links**: In the ["Forming Container Links"](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437364#questions) lecture we add the client and server services to the links array of the nginx service. Docker Compose will handle this container communication automatically for us.

**Environment Variables:** When using a compose file we will need to explicitly specify the environment variables each service will need access to. The value for each variable must match the corresponding variable names you have specified in the Elastic Beanstalk environment. The AWS variables are created in the ["Setting Environment Variables"](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437388#questions) lecture.

**Note - You must NOT have a Dockerrun.aws.json file in your project directory.** If AWS EBS sees this file the deployment will fail. If you have previously followed this course and deployed to the old Multi-container platform **you will need to delete this file before using the new Amazon Linux 2023 platform!!!**

Complete docker-compose.yml file:

1. version: "3"
2. services:
3.   client:
4.     image: "rallycoding/multi-client"
5.     mem_limit: 128m
6.     hostname: client
7.   server:
8.     image: "rallycoding/multi-server"
9.     mem_limit: 128m
10.     hostname: api
11.     environment:
12.       - REDIS_HOST=$REDIS_HOST
13.       - REDIS_PORT=$REDIS_PORT
14.       - PGUSER=$PGUSER
15.       - PGHOST=$PGHOST
16.       - PGDATABASE=$PGDATABASE
17.       - PGPASSWORD=$PGPASSWORD
18.       - PGPORT=$PGPORT
19.   worker:
20.     image: "rallycoding/multi-worker"
21.     mem_limit: 128m
22.     hostname: worker
23.     environment:
24.       - REDIS_HOST=$REDIS_HOST
25.       - REDIS_PORT=$REDIS_PORT
26.   nginx:
27.     image: "rallycoding/multi-nginx"
28.     mem_limit: 128m
29.     hostname: nginx
30.     ports:
31.       - "80:80"