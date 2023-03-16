# Week 4 — Postgres and RDS

# Week 4 Journal 

## Task status
1. Watch Week 4 Live-Stream [Video](https://www.youtube.com/watch?v=EtD7Kv5YCUs&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=49) ✅ 
2. Create RDS Postgres Instance ✅
3. Bash scripting for common database actions ✅
4. Install Postgres Driver in Backend Application ✅
5. Connect Gitpod to RDS Instance ✅
6. Create Congito Trigger to insert user into database ✅
7. Create new activities with a database insert :x:
8. Watch Chirag Week 4 - Spending Considerations :x: (not uploaded)
9. Watch Ashish's Week 4 -Securing Your Amazon RDS Postgres Database [video](https://www.youtube.com/watch?v=UourWxz7iQg&t=10s) ✅
10. Submit Security quiz ✅
11. Submit Spend considerations quiz :x: (not uploaded)

====================================================================================

## Week 4 Assignment Proof

| 	| Topic	| Assignment Proof 	|
|---	|--------------------------------	|--------------------------------------------	|
| 1 	| Setup DB scripts 	| [journal](#setup-database-bash-scripts) 	|
| 2	| Create RDS Postgres Instance | [journal]() |
| 3	| Bash scripting for common database actions | [journal]() |
| 4	| Install Postgres Driver in Backend Application | [journal](#install-postgres-driver-in-backend-application) |
| 5	| Connect Gitpod to RDS Instance | [journal]() |
| 6	| Create Congito Trigger to insert user into database | [journal](#implement-custom-authorizer-for-cognito) |
| 7	| Security Considerations | [journal](#week-4-security-considerations) |

====================================================================================

## Stretch Assignments

none so far

====================================================================================

## Personal Milestones  👯

====================================================================================

## Issues faced 😰 :x: :warning: :no_entry:


====================================================================================


## Detailed documentation

### Setup Database bash scripts


Provision the RDS PostgreSQL database
At the terminal, execute the below command"

```sh
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username cruddurroot \
  --master-user-password xxxxxxxxxxxx \
  --allocated-storage 20 \
  --availability-zone eu-central-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
```

![create_db](assets/week4_create_db.png)

![db_verify](assets/week4_create_db_verify.png)


2. Comment out the `DynamoDB` container in your `docker-compose.yml`. As we are not using dynamoDB until week-5, we can save some gitPod credits by not spinning up this container.

3. `compose up` your `docker-compose.yml` and check if the `PostgreSQL container` is running. Open the shell attached to this container.

4. Stop the newly created `cruddur-db-instance` from the AWS Management Console (so we do not incur costs)


![stop_db](assets/week4_stop_db_temporarily.png)


5. Try connecting to your PostgreSQL client
_Note: To connect to psql via the psql client cli tool remember to use the host flag to specific localhost._

	`psql -Upostgres --host localhost`

Enter the password when prompted.

6. Play around a bit in the Postgres client

Common PSQL commands:
	```sh
	\x on -- expanded display when looking at data
	\q -- Quit PSQL
	\l -- List all databases
	\c database_name -- Connect to a specific database
	\dt -- List all tables in the current database
	\d table_name -- Describe a specific table
	\du -- List all users and their roles
	\dn -- List all schemas in the current database
	```

7. While still in the PSQL client, let's create a local `cruddur` database
	`create database cruddur;`
use `\q` to exit the PSQL client.

![create_local](assets/week4_create_local_db.png)


8. Inside `backend-flask` create a new folder and name it `db`

9. Create a file inside `backend-flask/db` and name it `schema.sql`

- We are going to have Postgres generate out UUIDs. We'll need to use an extension called `uuid-ossp`

Add the below to  `schema.sql`
	```sql
	CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
	```

10. Import the contents of `schema.sql` to your local `cruddur` database
At the postgres shell, execute 

	```sql
	cd backend-flask
	psql cruddur < db/schema.sql -h localhost -U postgres
	cd ..
	```

11. Create a connection string for our local database

`export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`

Before setting it in our environemnt, let's make sure it is correct.
To test it, at the db container prompt, type 
	`psql postgresql://postgres:password@localhost:5432/cruddur`
It it works, you will be connected to the `cruddur` database.
type `\q` to exit the PSQL.

Now that we have verified the connection url, we can set it

`export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`
`psql $CONNECTION_URL`
`\q`

While we are at it, let's also set the env var in Gitpod

`gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`


12. Similar to step-11, we can also set the connection string for our production database 

`export PROD_CONNECTION_URL="postgresql://cruddurroot:xxxxxxxx@cruddur-db-instance.c6daq2nm03pr.eu-central-1.rds.amazonaws.com:5432/cruddur"`
`gp env PROD_CONNECTION_URL="postgresql://cruddurroot:xxxxxxxx@cruddur-db-instance.c6daq2nm03pr.eu-central-1.rds.amazonaws.com:5432/cruddur"`


![export_vars](assets/week4_export_db_vars.png)

13. Inside `backend-flask` folder, create a new folder and name it `bin`

14. Inside `backend-flask/bin` create 3 new files `db-create` , `db-drop`, `db-connect`, `db-seed` and `db-schema-load`

15. Grant executable permissions to these files  (execution permissions are not granted by default)
`chmod u+x bin/db-create bin/db-drop bin/db-schema-load bin/db-connect bin/db-connect bin/db-seed`
`ls -l bin/db-create bin/db-drop bin/db-schema-load bin/db-connect bin/db-connect bin/db-seed`

16. Working with our db-drop file 

16.1 Add the below to [backend-flask/bin/db-drop](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-drop)

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-drop"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "drop database cruddur;"
```



16.2 Execute [backend-flask/bin/db-drop](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-drop) script

at the terminal execute:

`./bin/db-drop`

This should drop our local database `cruddur`


17. Working with our db-create file
17.1 Add the below to [backend-flask/bin/db-create](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-create)

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-create"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "create database cruddur;"
```



17.2 Execute [backend-flask/bin/db-create](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-create) script

at the terminal execute:

`./bin/db-create`

This should create a new local database `cruddur`



18. Working with our db-schema-load file
18.1 Add the below to [backend-flask/bin/db-schema-load](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-schema-load)

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

schema_path="$(realpath .)/db/schema.sql"
echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```


18.2 Execute [backend-flask/bin/db-schema-load](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-schema-load) script

at the terminal execute:

`./bin/db-schema-load prod`

or

`./bin/db-schema-load test`

This should load the database `cruddur` from the environment we pass to the script


19. Working with our db-connect file
19.1 Add the below to [backend-flask/bin/db-connect](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-connect)

```sh
#! /usr/bin/bash
if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL
```


19.2 Execute [backend-flask/bin/db-connect](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-connect) script

at the terminal execute:

`./bin/db-connect prod`

or

`./bin/db-connect test`

This should connect to the database `cruddur` from the environment we pass to the script

20. Working with our db-seed file
20.1 Add the below to [backend-flask/bin/db-seed](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-seed)

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-seed"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

seed_path="$(realpath .)/db/seed.sql"
echo $seed_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $seed_path
```


![db_scripts](assets/week4_db_scripts.png)

20.2 Execute [backend-flask/bin/db-seed](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/bin/db-seed) script

at the terminal execute:

`./bin/db-seed prod`

or

`./bin/db-seed test`

This should connect to the database `cruddur` from the environment we pass to the script



21. Update [db/schema.sql](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/db/schema.sql) to create tables

```sql
DROP TABLE IF EXISTS public.users;
DROP TABLE IF EXISTS public.activities;


CREATE TABLE public.users (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  display_name text NOT NULL,
  handle text NOT NULL,
  email text NOT NULL,
  cognito_user_id text NOT NULL,
  created_at TIMESTAMP default current_timestamp NOT NULL
);

CREATE TABLE public.activities (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  user_uuid UUID NOT NULL,
  message text NOT NULL,
  replies_count integer DEFAULT 0,
  reposts_count integer DEFAULT 0,
  likes_count integer DEFAULT 0,
  reply_to_activity_uuid integer,
  expires_at TIMESTAMP,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```

22. Grant executable permission to `seed.sql`

`chmod u+x db/schema.sql`


![create_tables](assets/week4_create_tables.png)



23. Create a file inside `backend-flask/db` and name it `seed.sql`

24. Update [db/seed.sql](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/db/seed.sql) with some mock data

```sql
-- this file was manually created
INSERT INTO public.users (display_name, handle, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown' ,'MOCK'),
  ('Andrew Bayko', 'bayko' ,'MOCK');

INSERT INTO public.activities (user_uuid, message, expires_at)
VALUES
  (
    (SELECT uuid from public.users WHERE users.handle = 'andrewbrown' LIMIT 1),
    'This was imported as seed data!',
    current_timestamp + interval '10 day'
  )
```

25. Execute the seed file
`./bin/db-seed`


26. To idenitify active db sessions i.e. all active sessions connected to our db `cruddur`

26.1 Create a new script called inside `backend-flask\bin] and name it `db-sessions`

```sh
#! /usr/bin/bash
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-sessions"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

NO_DB_URL=$(sed 's/\/cruddur//g' <<<"$URL")
psql $NO_DB_URL -c "select pid as process_id, \
       usename as user,  \
       datname as db, \
       client_addr, \
       application_name as app,\
       state \
from pg_stat_activity;"
```


26.2 grant this script executable permissions
	`chmod u+x bin/db-sessions`

26.3 Execute the script to view the db sessions currently open
	`./bin/db-sessions`



27.1 Create another script inside `backend-flask\bin` called `db-setup` to run multiple bash scripts we create above, together

```sh
#! /usr/bin/bash
-e # stop if it fails at any point

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-setup"
printf "${CYAN}==== ${LABEL}${NO_COLOR}\n"

bin_path="$(realpath .)/bin"

source "$bin_path/db-drop"
source "$bin_path/db-create"
source "$bin_path/db-schema-load"
source "$bin_path/db-seed"
```

27.2 grant this script executable permissions
	`chmod u+x bin/db-setup`

27.3 Execute the script 
	`./bin/db-setup`


28. Commit and sync changes to your github repo

====================================================================================

### Install Postgres Driver in Backend Application

1. Add requirements to [backend-flask/requirements.txt](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/requirements.txt) to create our db connection pool

```sh
psycopg[binary]
psycopg[pool]
```

2. At the terminal, install the requirements

```sh
cd backend-flask
pip install -r requirements.txt
```

3. Inside `backend-flask/lib` create a new file `db.py`

```py
from psycopg_pool import ConnectionPool
import os
import re
import sys
from flask import current_app as app

class Db:
  def __init__(self):
    self.init_pool()

  def template(self,*args):
    pathing = list((app.root_path,'db','sql',) + args)
    pathing[-1] = pathing[-1] + ".sql"

    template_path = os.path.join(*pathing)

    green = '\033[92m'
    no_color = '\033[0m'
    print("\n")
    print(f'{green} Load SQL Template: {template_path} {no_color}')

    with open(template_path, 'r') as f:
      template_content = f.read()
    return template_content

  def init_pool(self):
    connection_url = os.getenv("CONNECTION_URL")
    self.pool = ConnectionPool(connection_url)
  # we want to commit data such as an insert
  # be sure to check for RETURNING in all uppercases
  def print_params(self,params):
    blue = '\033[94m'
    no_color = '\033[0m'
    print(f'{blue} SQL Params:{no_color}')
    for key, value in params.items():
      print(key, ":", value)

  def print_sql(self,title,sql):
    cyan = '\033[96m'
    no_color = '\033[0m'
    print(f'{cyan} SQL STATEMENT-[{title}]------{no_color}')
    print(sql)
  def query_commit(self,sql,params={}):
    self.print_sql('commit with returning',sql)

    pattern = r"\bRETURNING\b"
    is_returning_id = re.search(pattern, sql)

    try:
      with self.pool.connection() as conn:
        cur =  conn.cursor()
        cur.execute(sql,params)
        if is_returning_id:
          returning_id = cur.fetchone()[0]
        conn.commit() 
        if is_returning_id:
          return returning_id
    except Exception as err:
      self.print_sql_err(err)
  # when we want to return a json object
  def query_array_json(self,sql,params={}):
    self.print_sql('array',sql)

    wrapped_sql = self.query_wrap_array(sql)
    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql,params)
        json = cur.fetchone()
        return json[0]
  # When we want to return an array of json objects
  def query_object_json(self,sql,params={}):

    self.print_sql('json',sql)
    self.print_params(params)
    wrapped_sql = self.query_wrap_object(sql)

    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql,params)
        json = cur.fetchone()
        if json == None:
          "{}"
        else:
          return json[0]
  def query_wrap_object(self,template):
    sql = f"""
    (SELECT COALESCE(row_to_json(object_row),'{{}}'::json) FROM (
    {template}
    ) object_row);
    """
    return sql
  def query_wrap_array(self,template):
    sql = f"""
    (SELECT COALESCE(array_to_json(array_agg(row_to_json(array_row))),'[]'::json) FROM (
    {template}
    ) array_row);
    """
    return sql
  def print_sql_err(self,err):
    # get details about the exception
    err_type, err_obj, traceback = sys.exc_info()

    # get the line number when exception occured
    line_num = traceback.tb_lineno

    # print the connect() error
    print ("\npsycopg ERROR:", err, "on line number:", line_num)
    print ("psycopg traceback:", traceback, "-- type:", err_type)

    # print the pgcode and pgerror exceptions
    print ("pgerror:", err.pgerror)
    print ("pgcode:", err.pgcode, "\n")

db = Db()
```

4. Reference this connection url to our `docker-compose.yml`
In the `environment` section of `backend-flask` add:
```yml
CONNECTION_URL: "${PROD_CONNECTION_URL}"
```

5. Update [backend-flask/services/home_activities.py](https://github.com/aggarwal-tanushree/aws-bootcamp-cruddur-2023/blob/a8ee9737df34f8b4f18f8cf3520a60e6081c093f/backend-flask/services/home_activities.py) 
Replace the contents of the file with the below code:

```py
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

from lib.db import db

#tracer = trace.get_tracer("home.activities")

class HomeActivities:
  def run(cognito_user_id=None):
    #logger.info("HomeActivities")
    #with tracer.start_as_current_span("home-activites-mock-data"):
    #  span = trace.get_current_span()
    #  now = datetime.now(timezone.utc).astimezone()
    #  span.set_attribute("app.now", now.isoformat())
    sql = db.template('activities','home')
    results = db.query_array_json(sql)
    return results
```


### Connect RDS from GitPod

1. Check your Gitpod environment's i.p address.
Execute the below command at the terminal

`GITPOD_IP=$(curl ifconfig.me)`

2. Update the inbound rule in the security group attached to your RDS instance via the AWS Management Console.
- Open TCP port 5432 for the gitpod i.p address returned in the above step

3. Try establishing connection from your Gitpod environment to your production `Cruddur` database
`psql $PROD_CONNECTION_URL`

Check if the conneciton is established.

4. Note that we have opened the Postgres port 5432 for our current Gitpod env i.p address. This i.p address is not static and will change every time we launch our Gitpod env.
That would mean we need to update the SG with the ne wi.p address every single time the i.p address changes.
To save ourselves this effort, let's create a script to automate these actions.

4.1 Copy the `SG Rule ID` and the `SG ID` from the AWS Management console.
4.2 Execute the below commands with the relevant values
```sh
export DB_SG_ID="sg-xxxxxxx"
gp env DB_SG_ID="sg-xxxxxxx"

export DB_SG_RULE_ID="sgr-xxxxxxxxx"
gp env DB_SG_RULE_ID="sgr-xxxxxxxxx"
```

![db_sg](assets/week4_db_sg.png)

4.3 Create a new file inside `backend-flask/bin` and name it `rds-update-sg-rule`

```
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="rds-update-sg-rule"
printf "${CYAN}==== ${LABEL}${NO_COLOR}\n"

aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```

4.4 Grant executable permissions to `rds-update-sg-rule`
`chmod u+x bin/rds-update-sg-rule`

4.5 Execute the script 
	`./bin/rds-update-sg-rule`
	
====================================================================================

## Implement custom authorizer for Cognito

1 From the hamburger on the AWS Management Console home page, navigate to `Lamdba`
2 Create Lambda - > Author From Scratch 
3 Enter `Function name` as `cruddur-post-confirmation`
4 From the `Runtime	 drop down select `Python 3.8`
5 `Architecture` : `x86_64`

![lambda](assets/week4_lambda1.png)

6 `change default execution role` : `Create a new role with basic Lambda permissions` and click `Create`

![lambda](assets/week4_lambda.png)


7 In your Github repo under `aws` create a new folder `lambdas`
8 Inside `aws/lambdas` create a file and name it `cruddur-post-confirrmation.py`
9 Add the below code in [/aws/lambdas/cruddur-post-confirrmation.py]()

```py
import json
import psycopg2
import os

def lambda_handler(event, context):
    user = event['request']['userAttributes']
    print('userAttributes')
    print(user)

    user_display_name  = user['name']
    user_email         = user['email']
    user_handle        = user['preferred_username']
    user_cognito_id    = user['sub']
    try:
      print('entered-try')
      sql = f"""
         INSERT INTO public.users (
          display_name, 
          email,
          handle, 
          cognito_user_id
          ) 
        VALUES(%s,%s,%s,%s)
      """
      print('SQL Statement ----')
      print(sql)
      conn = psycopg2.connect(os.getenv('CONNECTION_URL'))
      cur = conn.cursor()
      params = [
        user_display_name,
        user_email,
        user_handle,
        user_cognito_id
      ]
      cur.execute(sql,*params)
      conn.commit() 

    except (Exception, psycopg2.DatabaseError) as error:
      print(error)
    finally:
      if conn is not None:
          cur.close()
          conn.close()
          print('Database connection closed.')
    return event
```	

10. Back in the AWS Management console, paste the code from [/aws/lambdas/cruddur-post-confirrmation.py]() into the `code` section of your `lambda function`
and click on `Deploy`

![lambda](assets/week4_lambda2.png)

11. Under the `Configuration` tab, set the required environment variable.
Click `Edit` 
`Key`         : `Value`
CONNECTION_URL : <enter your production DB connection string here>
and click save

![lambda](assets/week4_lambda3.png)


12. In the `Code` tab, click on `Add a Layer`
`Choose a layer` : `Specify an ARN`
and add the value `arn:aws:lambda:eu-central-1:898466741470:layer:psycopg2-py38:1`

![lambda](assets/week4_lambda_layer.png)

13. Navigate to `IAM` service and create a new `Policy`
Click `Create Policy` and choose `JSON`
Enter the following policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateNetworkInterface",
                "ec2:DeleteNetworkInterface",
                "ec2:DescribeNetworkInterfaces"
                ],
            "Resource": "*"
        }
    ]
}
```

`Name` : `AWSLambdaVPCAccessExecutionRole`

![iampolicy](assets/week4_iam_policy.png)


13. In Lambda, under the `Configuration` tab, click `Permissions` on the left pane
13.1 This will open an new tab for `IAM`
Under `Permission Policies` click `Add permissions` -> `Attach Policy`

![iampolicy](assets/week4_iam_permissions.png)

14. Under the Configuration` tab, click `VPC` and edit it
- Choose your VPC from the dropdown list
- From the `Subnets` dropdown list, select `eu-central-1a` and a few other available subnets for HA
- Select the same `Subnet` as your pervious configuration
- Click `save`

![VPC](assets/week4_vpc.png)

15. Navigate to `Cognito` in your AWS management console and go to the `User pool properties` of your exisitng `crudder-user-pool`
15.2 Click on `Add Lambda Trigger`
15.3. `Trigger type` : `Sign up`
and select `post confirmation trigger`

15.4 Select your lambda function from the dropdown list
15.5 Click `Add Lambda Trigger`

![cognito_trigger](assets/week4_cognito_trigger.png)

![cognito_trigger](assets/week4_lambda_trigger.png)



16. `compose up` your app and try signing up a user to your `cruddur` app.
- Verify the Cloudwatch logs for errors.

![lambda_logs](assets/week4_verify_cloudwatchlogs_for_errors.png)



- Connect to your database and check presence of new user




====================================================================================

## Week 4 Security considerations
[Ashish's Decenteralized Authentication security considerations](https://www.youtube.com/watch?v=UourWxz7iQg&t=10s)

====================================================================================

### Week 4 Spend Considerations

====================================================================================
