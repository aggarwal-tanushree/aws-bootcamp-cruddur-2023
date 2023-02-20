# Week 1 — App Containerization
# Week 1 Journal 

## Tasks Status
1. [Watch AB's week 1 stream](https://www.youtube.com/watch?v=zJnNe5Nv4tE&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=22) :white_check_mark:
2. [Watch Grading Homework Summaries](https://www.youtube.com/watch?v=FKAScachFgk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=25) :white_check_mark:
3. [Watch Chirag's Week 1 - Spending Considerations](https://www.youtube.com/watch?v=OAMHu1NiYoI&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=24) :white_check_mark:
4. [Watch Remember to Commit Your Code](https://www.youtube.com/watch?v=b-idMgFFcpg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=23)
5. [Watch Ashish's Week 1 - Container Security Considerations](https://www.youtube.com/watch?v=OjZz4D0B-cA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=25)
6. [Watch Containerize Application (Dockerfiles, Docker Compose)](https://www.youtube.com/watch?v=zJnNe5Nv4tE&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=22)
7. [Document the Notification Endpoint for the OpenAI Document](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27)
8. [Write a Flask Backend Endpoint for Notifications](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27)
9. [Write a React Page for Notifications](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27)
10. [Run DynamoDB Local Container and ensure it works](https://www.youtube.com/watch?v=CbQNMaa6zTg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=28)
11. [Run Postgres Container and ensure it works](https://www.youtube.com/watch?v=CbQNMaa6zTg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=28)



## Personal Milestones
1. 



## Personal Notes

[Spend consideration video](https://www.youtube.com/watch?v=OAMHu1NiYoI)

## GitPod
Provides the cloud development environment
Free tier 
	- 50 hours of Standard workspace usage per month
	- Free tier standard configuration: 4 cores, 8GB RAM and 30GB storage
	- Large configuration: 8 cores, 16GB RAM and 50GB storage
	- Environemnt automatically stops after 30 minutes of in-activity
	- Upto 4 parallel workspaces are supported (should try not to spin up multiple environemtns simultaneously, as usage is aggregated)

_**Note** : Bootcamp requires average of 2 hours compute per week. Even if we go up to 11 hours compute per week, with standard configuration, we can stay within the free-tier limits_

To check pricing models:
[Gitpod pricing page](https://www.gitpod.io/pricing)

![Pricing model details](assets/week1_gitpod_pricing.png)

To check current usage:
[Gitpod billing console](https://gitpod.io/user/billing)

![My current usage](assets/week1_gitpod_my_current_usage.png)



## Github Codespaces

- With configuration: 2 cores, 4GB RAM, 15GB storage : 60 hours usage per month
 
- With configuration: 4 cores, 8GB RAM, 15GB storage : 30 hours usage per month


As configuration increases, free usage hours decreases

_**Note** maximum storage supported : **32 GB**_

To check pricing models:
[Github Codespaces pricing](https://github.com/features/codespaces)

![Pricing model details](assets/week1_github_Codespaces_pricing.png)




## AWS Cloud 9
- An independent platform acquired by AWS a few years ago
- Uses EC2 as underlying technology
- Can run AWS Cloud 9 free for the entire month, if using EC2 **t2.micro** or **t3.micro** instance type
- Avoid using Cloud 9 if t2.micro is being used for other purposes within the account, as **usage bill is aggregated**
 
 
## CloudTrail
- API logging service
- Logs trails in S3 buckets
- By default, all trails are logged for **90 days**
- To stay in free tier usgae limits, follow these steps:
	1. uncheck the option to "Log file SSE-KMS encryption" (as KMS encryption operations can be expensive)
	2. For logging "Event types", uncheck "Data events" and "Insights events" -> as these are not free tier eligible. These event logging are betetr sutited for production environments

## Week 1 live stream notes

[Watch AB's week 1 stream](https://www.youtube.com/watch?v=zJnNe5Nv4tE&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=22)

Containers
- good to use if you are working in development & testing
- a good way to ensure all OS dependencies, patch requirements etc.
 
-  linuxserver.io has lots of container images
	 
**[Dockerhub](hub.docker.com)** - a **container registry**, provided by Docker 	 to host your own container images for private purposed and/or share them used when you build your own container image and **push** to a container registry (eg: Dockerhub).
When someone wants to use this container image, they *pull** it from the registry
	
**OCI**  Open container images
- standards around how you build container images and registries etc.
- Docker hub follows OCI for registry
- As long as you use a container image following the OCI standards, it should be compatible with anything outside of Dockerhub
	
	
	 
Step 1: 
Create a **Dockerfile** under the **Backend** file structure
Paste the following code inside the _Dockerfile_

```
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

- Each line inside the Dockerfile is an instruction
- With each file we are creating a _layer_ per se 
- A docker file works on something called "union file system". Each layer of the Docker image is sandwitched together to form the final docker image

- ` FROM python:3.10-slim-buster ` : Loading a docker file "python:3.10-slim-buster"
- Here "[python](https://hub.docker.com/_/python)" is an image in Dockerhub 
- "[3.10-slim-buster](https://github.com/docker-library/python/blob/1a68bced0dc5b7deb6ecd2f7ddacc1089323409d/3.10/slim-buster/Dockerfile)" is one of the many available _tags_ in this docker image


- **[scratch](https://hub.docker.com/_/scratch)** is the name of an empty docker image. It is a minimal image reserved by Docker. It is used as a starting point for building containers.
While `scratch` appears in Docker’s repository on the hub, you can’t pull it, run it, or tag any image with the name scratch. Instead, you can refer to it in your Dockerfile. For example, to create a minimal container using scratch:

```FROM scratch
ADD rootfs.tar.xz /
CMD ["bash"]
```
- is copying "rootfs.tar.xz" to the root directory "/"
- executing the command "bash"



