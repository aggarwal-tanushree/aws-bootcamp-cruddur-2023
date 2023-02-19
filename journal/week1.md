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
	
