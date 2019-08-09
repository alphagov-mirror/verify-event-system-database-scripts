# verify-event-system-database-scripts

SQL scripts and schemas for the Event System database.

## Migrations

The migrations directory holds the SQL scripts which were used to create the current database. They are used by the
[verify-event-recorder-service](https://github.com/alphagov/verify-event-recorder-service) and the
[verify-billing-reporter](https://github.com/alphagov/verify-billing-reporter) tests.

## Permissions

The permissions directory contains the SQL scripts that were used to create and then later revoke the reader and writer
users. These were replaced by IAM permissions. Only here as a historical reference.


## Migrations Docker Image

The migrations are built into a Docker image based on the Flyway database migrations tool.

The image uses a custom entry point script (`docker-entrypoint.sh`) to generate a AWS IAM token if required.

The Docker image uses the following environment variables to establish a Postgres database connection:

* `PGHOST` - The Postgres host to connect to
* `PGUSER` - The username to use while connecting to Postgres
* `PGDATABASE` - The database to connect to
* `USE_IAM_AUTH` - If set to `1` then an AWS IAM token is generated to use as the password for the connection.
* `PGPASSWORD` - If `USE_IAM_AUTH` is not set `1` then this variable should be set to the password to use for the connection.

## Building the Image

The image is automatically built and pushed to the repository by the event system build pipeline 
for deployment to the AWS environments.

For local testing, the image can be built using the `./build.sh` script.

## Testing

### The Automated Way

The `run-test.sh` script will build the Migrations Docker image, start a new Postgres container
and then run the migration scripts against that database. This script should be used to validate that your migration
scripts work before applying them to any AWS environment.

### Debugging/Manual Testing

First of all you should start the postgres event-store in the background:

```bash
docker-compose up -d event-store
```

Next, you can run the database migrations with the following command:

```bash
docker-compose run flyway migrate
```

N.B. If you make changes to the migration scripts and want to (re)-test them, you must build the image
before running the migrations.

Also note, that you could run any Flyway command here, not just `migrate` ([see Flyway docs](https://flywaydb.org/documentation)), 
for example, `docker-compose run flyway clean`.

To inspect the results of the migration you can use `psql` shell to connect to the PostgreSQL instance
and issue SQL statements to the database:

```bash
docker-compose run psql
```

N.B. You can also use the `psql` shell to insert test data either by issuing SQL statements or
by running a SQL script (such as those generated by `pg_dump`):

```bash
docker-compose run psql < some-data-insertion-script.sql
```

When you're finished (or if you want to destroy database and restart tests from scratch), you should 
stop the Postgres instance:

```bash
docker-compose down
```

## Connecting

It is sometimes necessary to connect to a PostgreSQL database directly using the `psql` shell.
To expedite connections to AWS-hosted instances, the `postgres.env.sh.template` has been provided.
Once configured, it exports appropriate variables for connecting with the `psql` shell without arguments.

A minimal configuration should make a copy of the template.
`postgres.env.sh` has been ignored, so can be safely modified.
Then set the database host path - for AWS this is the RDS endpoint.
Then simply `source` it in your current session, and run `psql` to connect.

Note that you must have the AWS CLI installed and available in the session.
You should also authenticate to an **admin** role in whichever AWS account the DB resides in.
Admin access is currently required to assume a database role, even if the role provides read-only access to the DB.

Setting `PGSSLMODE` explicitly may be unnecessary, but is the recommended connection mode.
There is more information in the [AWS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html#PostgreSQL.Concepts.General.SSL).
