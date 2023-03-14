# Week 4 — Postgres and RDS

# Week 4 Journal 

## Task status
1. Watch Week 4 Live-Stream [Video](https://www.youtube.com/watch?v=EtD7Kv5YCUs&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=49) ✅ 
2. Watch Chirag Week 4 - Spending Considerations :x: (not uploaded)
3. Watch Ashish's Week 4 -Securing Your Amazon RDS Postgres Database [video](https://www.youtube.com/watch?v=UourWxz7iQg&t=10s) ✅
4. Submit Security quiz ✅
5. Submit Spend considerations quiz :x: (not uploaded)

====================================================================================

## Week 4 Assignment Proof

| 	| Topic	| Assignment Proof 	|
|---	|--------------------------------	|--------------------------------------------	|
| 1 	| Setup DB scripts 	| [journal](#setup-database-bash-scripts) 	|
| 2	| Security Considerations | [journal](#week-4-security-considerations) |


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

week4_create_db

week4_create_db_verify.png


2. Comment out the `DynamoDB` container in your `docker-compose.yml`. As we are not using dynamoDB until week-5, we can save some gitPod credits by not spinning up this container.

3. `compose up` your `docker-compose.yml` and check if the `PostgreSQL container` is running. Open the shell attached to this container.

4. Stop the newly created `cruddur-db-instance` from the AWS Management Console (so we do not incur costs)


week4_stop_db_temporarily.png


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

week4_create_local_db


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


week4_export_db_vars.png

13. Inside `backend-flask` folder, create a new folder and name it `bin`

14. Inside `backend-flask/bin` create 3 new files `db-create` , `db-drop`, `db-connect`, `db-seed` and `db-schema-load`

15. Grant executable permissions to these files  (execution permissions are not granted by default)
`chmod u+x bin/db-create bin/db-drop bin/db-schema-load bin/db-connect bin/db-connect bin/db-seed`
`ls -l bin/db-create bin/db-drop bin/db-schema-load bin/db-connect bin/db-connect bin/db-seed`

16. Working with our db-drop file 

16.1 Add the below to [backend-flask/bin/db-drop]()

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-drop"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "drop database cruddur;"
```



16.2 Execute [backend-flask/bin/db-drop]() script

at the terminal execute:

`./bin/db-drop`

This should drop our local database `cruddur`


17. Working with our db-create file
17.1 Add the below to [backend-flask/bin/db-create]()

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-create"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "create database cruddur;"
```



17.2 Execute [backend-flask/bin/db-create]() script

at the terminal execute:

`./bin/db-create`

This should create a new local database `cruddur`



18. Working with our db-schema-load file
18.1 Add the below to [backend-flask/bin/db-schema-load]()

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


18.2 Execute [backend-flask/bin/db-schema-load]() script

at the terminal execute:

`./bin/db-schema-load prod`

or

`./bin/db-schema-load test`

This should load the database `cruddur` from the environment we pass to the script


19. Working with our db-connect file
19.1 Add the below to [backend-flask/bin/db-connect]()

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


19.2 Execute [backend-flask/bin/db-connect]() script

at the terminal execute:

`./bin/db-connect prod`

or

`./bin/db-connect test`

This should connect to the database `cruddur` from the environment we pass to the script

20. Working with our db-seed file
20.1 Add the below to [backend-flask/bin/db-seed]()

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

20.2 Execute [backend-flask/bin/db-seed]() script

at the terminal execute:

`./bin/db-seed prod`

or

`./bin/db-seed test`

This should connect to the database `cruddur` from the environment we pass to the script



21. Update [db/schema.sql]() to create tables

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


week4_create_tables.png



23. Create a file inside `backend-flask/db` and name it `seed.sql`

24. Update [db/seed.sql]() with some mock data

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

26. Commit and sync changes to your github repo





====================================================================================

## Week 4 Security considerations
[Ashish's Decenteralized Authentication security considerations](https://www.youtube.com/watch?v=UourWxz7iQg&t=10s)

====================================================================================

### Week 4 Spend Considerations

====================================================================================
