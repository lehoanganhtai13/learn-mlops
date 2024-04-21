# Learn MLOps/DevOps
A self-study repo, where there is everything you need for a MLOps system. This could also apply for DevOps system.

### Agenda
1. [AWS](#aws)
2. [Docker](#docker)

## <a name="aws"></a> 1. How to use AWS

### AWS CLI
1. Credential configuration
    ```bash
    aws configure
    ```
- Output:
    - AWS Access Key ID: `ACCESS KEY ID`
    - AWS Secret Access Key: `SECRET ACCESS KEY`
    - Default region name: `REGION`
    - Default output format [None]: `ENTER`
2. IAM (Identity & Access management)
- Create group/user:
    ```bash
    aws iam create-[group/user] --[group/user]-name <NAME> --region <REGION>
    ```
- Create role:
    ```bash
    aws iam create-role --role-name <NAME> \
                        --assume-role-policy-document file://<LOCAL_PATH>/trust_policy.json \
                        --region <REGION> --description "<DESCRIPTION>"
    ```
    - `DESCRIPTION`: your description for your custom role, usually it will be "Allows `service-name` to call AWS services on your behalf.".
    - `LOCAL_PATH`: relative path to folder containing your trust policy file.
    - Trust policy file example for AWS Glue service:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "glue.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }
    ```
- Create policy:
    ```bash
    aws iam create-policy   --policy-name <NAME> \
                            --policy-document file://<LOCAL_PATH>/policy.json \
                            --region <REGION> --description "<DESCRIPTION>"
    ```
    - `DESCRIPTION`: your description for your custom policy.
    - `LOCAL_PATH`: relative path to folder containing your policy file.
    - Policy file example to get access to S3 bucket `BUCKET_NAME`:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "",
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket*",
                    "s3:PutBucket*",
                    "s3:GetBucket*"
                ],
                "Resource": [
                    "arn:aws:s3:::<BUCKET_NAME>"
                    "arn:aws:s3:::<BUCKET_NAME>/*"
                ]
            }
        ]
    }
    ```
- Add user to group:
    ```bash
    aws iam add-user-to-group --group-name <GROUP_NAME> --user-name <USER_NAME> --region <REGION>
    ```
- Attach group/user/role policy:
    ```bash
    aws iam attach-[group/user/role]-policy --[group/user/role]-name <NAME> \
                                            --policy-arn <ARN> \
                                            --region <REGION>
    ```
    - `NAME`: group, user, or role name to be attached policy.
    - `ARN`: the Amazon Resource Name of the policy to be attached. There are 2 types of ARN for this scenario:
        - ARN for AWS managed policies is `arn:aws:iam::aws:policy/<policy-name>`, for policies that are service role for example `AWSGlueServiceRole` will be `arn:aws:iam::aws:policy/service-role/<policy-name>`.
        - ARN for customer managed policies is `arn:aws:iam::<account-id-with-no-hyphens>:policy/<policy-name>`.
- Detach group/user/role policy:
    ```bash
    aws iam detach-[group/user/role]-policy --[group/user/role]-name <NAME> \
				 	                        --policy-arn <ARN>
    ```
    - `NAME`: group, user, or role name to be detached policy.
    - `ARN`: the Amazon Resource Name of the policy to be detached. There are 2 types of ARN for this scenario:
        - ARN for AWS managed policies is `arn:aws:iam::aws:policy/<policy-name>`, for policies that are service role for example `AWSGlueServiceRole` will be `arn:aws:iam::aws:policy/service-role/<policy-name>`.
        - ARN for customer managed policies is `arn:aws:iam::<account-id-with-no-hyphens>:policy/<policy-name>`.
- Delete group/user/role:
    ```bash
    aws iam delete-[group/user/role] --[group/user/role]-name <NAME>
    ```
    - `NAME`: group, user, or role name to be deleted.
- Detach group/user/role policy:
    ```bash
    aws iam delete-policy --policy-arn <ARN>
    ```
    - `ARN`: the Amazon Resource Name of the policy to be detached. There are 2 types of ARN for this scenario:
        - ARN for AWS managed policies is `arn:aws:iam::aws:policy/<policy-name>`, for policies that are service role for example `AWSGlueServiceRole` will be `arn:aws:iam::aws:policy/service-role/<policy-name>`.
        - ARN for customer managed policies is `arn:aws:iam::<account-id-with-no-hyphens>:policy/<policy-name>`.
- List groups/users/roles/policies:
    ```bash
    aws iam list-[groups/users/roles/policies]
    ```
- Remove user from group:
    ```bash
    aws iam remove-user-from-group --group-name <GROUP_NAME> --user-name <USER_NAME>
    ```
- List all policies attached to group/user/role:
    ```bash
    aws iam list-attached-[group/user/role]-policies --[group/user/role]-name <NAME> --scope <MANAGE_TYPE>
    ```
    - `MANAGE_TYPE`: we will use `All` for all policies including AWS-managed policies and the customer managed policies in your AWS account, `AWS` for only AWS-managed policies, and `Local` for only the customer managed policies in your AWS account.

3. Amazon S3 (Storage)
- Create bucket
    ```bash
    aws s3 mb s3://<BUCKET_NAME> --region <REGION>
    ```

- Remove bucket
    ```bash
    aws s3 rb s3://<BUCKET_NAME> --force
    ```

- Create a folder
    ```bash
    aws s3api put-object --bucket <BUCKET_NAME> --key <FOLDER_PATH> --content-length 0
    ```
    - `FOLDER_PATH`: key name of the object you're creating in the S3 bucket. If it is a single folder, it will be `folder-name`. If it is subfolder, it will be `path-to-folder-containing-subfolder/subfolder-name`.

- List S3 objects/buckets
    ```bash
    aws s3 ls <S3Uri> --recursive --region <REGION> --summarize --human-readable
    ```
    - `S3Uri`: it could be known as the path to the objects or buckets in S3, for a bucket it will be `s3://<bucket-name>`, for a folder in bucket it will be `s3://<bucket-name>/<path-to-the-specific-folder>`.

- Copy/Move S3 objects, files
    ```bash
    aws s3 cp/mv <SOURCE> <TARGET> \
                --recursive \
                --include "<INCLUDED_FILE_PATTERNS>" \
                --exclude "<EXCLUDED_FILE_PATTERNS>"
    ```
    - `SOURCE`: can be either a local file/directory path or an S3 object/bucket path.
        - A local file path: This is the path to a file on your local machine that you want to copy or move. For example, `./<file-name>.txt` or `/home/user/<path-to-folder>/<file-name>.txt`.
        - A local directory path: This is the path to a directory on your local machine that you want to copy or move. For example, `./<folder-name>` or `/home/user/<path-to-folder>`.
        - An S3 object path: This is the path to an object in an S3 bucket that you want to copy or move. For example, `s3://<bucket-name>/<file-name>.txt` or `s3://<bucket-name>/<path-to-folder>/<file-name>.txt`.
        - An S3 bucket path: This is the path to an S3 bucket, potentially with a prefix, from which you want to copy or move all matching objects. For example, `s3://<bucket-name>` or `s3://<bucket-name>/<path-to-folder>`.
    - `TARGET`: can be either a local file/directory path or an S3 object/bucket path.
        - A local directory path: This is the path to a directory on your local machine where you want to copy or move the source file or directory to. For example, `./<path-to-folder>` or `/home/user/<path-to-folder>`.
        - An S3 object path: This is the path where you want to copy or move the source file to in an S3 bucket. For example, `s3://<bucket-name>/<file-name>.txt` or `s3://<bucket-name>/<path-to-folder>/<file-name>.txt`.
        - An S3 bucket path: This is the path to an S3 bucket, potentially with a prefix, where you want to copy or move all matching source objects to. For example, `s3://<bucket-name>` or `s3://<bucket-name>/<path-to-folder>`.
    - `INCLUDED_FILE_PATTERNS` and `EXCLUDED_FILE_PATTERNS`: patterns to specify which files to include or exclude when copying or moving files, if `SOURCE` is path to a folder. For example, to include all `.txt` files, you would use `*.txt`, or if exclude or `.jpg` files in `temp` folder, you would use `temp/*.jpg`.
    - `--recursive`: used only when `SOURCE` is not path to a specific file.

4. AWS Lake Formation (Data Lake management)
- Add administrator for data lake
    ```bash
    aws lakeformation put-data-lake-settings --data-lake-settings '{ "DataLakeAdmins": [{ "DataLakePrincipalIdentifier": "arn:aws:iam::<ACCOUNT_ID>:user/<USER_NAME>" }] }' \
                                            --cli-input-json file://<LOCAL_PATH>/settings.json	
    ```
    - `LOCAL_PATH`: relative path to folder containing your settings file.
    - Setting file example:
    ```json
    {
        "DataLakeSettings": {
            "CreateDatabaseDefaultPermissions": [],
            "CreateTableDefaultPermissions": []
        }
    }
    ```

- Register storage as resource for lake formation
    ```bash
    aws lakeformation register-resource --resource-arn arn:aws:s3:::<BUCKET_NAME>/<PATH_TO_FOLDER>/ --use-service-linked-role
    ```

- Grant data lake permissions for database and its tables
    ```bash
    aws lakeformation grant-permissions --principal '{ "DataLakePrincipalIdentifier": "arn:aws:iam::<ACCOUNT_ID>:[user/role]/<[USER/ROLE]_NAME>" }' \
                                        --cli-input-json file://<LOCAL_PATH>/permissions.json
    ```
    - `LOCAL_PATH`: relative path to folder containing your permissions file.
    - Permissions file example for a specific table in database, use only 1 type `Table` or `TableWithColumns` based on access level demands of different users or roles:
    ```json
    {
        "Resource": {
            "Table": {
                    "DatabaseName": "<DATABASE_NAME>",
                    "Name": "<TABLE_NAME>"
            },
            "TableWithColumns": {
                    "DatabaseName": "<DATABASE_NAME>",
                    "Name": "<TABLE_NAME>",
                    "ColumnNames": ["<COLUMN_1>", "<COLUMN_2> ..."]
            }
        },
        "Permissions": ["SELECT" "<PERMISSION_2>" ...]
    }
    ```

5. AWS Glue (ETL)
- Create database for data cataloging
    ```bash
    aws glue create-database --database-input '{"Name": "<DATABASE_NAME>", "LocationUri": "s3://<BUCKET_NAME>/<PATH_TO_FOLDER>", "Description": "<DESCRIPTION>"}'
    ```
    - `DESCRIPTION`: description for your database.

- Create crawler with specific targets, role, and optional schedule
    ```bash
    aws glue create-crawler --name <CRAWLER_NAME> --role <IAM_ROLE> --database-name <DATABASE_NAME> \
                                                --targets '{ "S3Targets": [{ "Path": "s3://<BUCKET_NAME>/<PATH_TO_FOLDER>/" }] }' \
						                        --recrawl-policy '{ "RecrawlBehavior": "<CRAWL_BEHAVIOR>" }' \
                                                --schedule 'cron(<CRON_EXPRESSION>)'
    ```
    - `CRAWL_BEHAVIOR`: it can be `CRAWL_EVERYTHING` to crawl all the data sources, including the ones it has previously crawled; `CRAWL_NEW_FOLDERS_ONLY` to only crawl the new data sources since the last crawl; `CRAWL_EVENT_MODE` to crawl only the changes identified by Amazon S3 events.
    - `CRON_EXPRESSION`: a string to define the schedule for when to start the crawler in format `Minutes Hours Day-of-month Month Day-of-week Year`, with `?` to specify "no specific value", and `*` to specify "every value" of a specific  field.
        - `Minutes`: An integer from 0 to 59.
        - `Hours`: An integer from 0 to 23.
        - `Day-of-month`: An integer from 1 to 31.
        - `Month`: An integer from 1 to 12.
        - `Day-of-week`: An integer from 1 to 7 (1 is Sunday, and 7 is Saturday).
        - `Year`: A four-digit integer.

- Run crawler
    ```bash
    aws glue start-crawler --name <CRAWLER_NAME>
    ```

- Get list of tables in a specified database
    ```bash
    aws glue get-tables --database-name <DATABASE_NAME>
    ```

6. Amazon Kinesis Data Firehose (data real-time streaming)
- Create a Firehose delivery stream to S3
    ```bash
    aws firehose create-delivery-stream --delivery-stream-name <STREAM_NAME> \
                                        --delivery-stream-type <STREAM_TYPE> \
                                        --s3-destination-configuration '{
                                            "RoleARN": "arn:aws:iam::<ACCOUND_ID>:role/<ROLE_NAME>",
                                            "BucketARN": "arn:aws:s3:::<BUCKET_NAME>",
                                            "Prefix": "<PATH_TO_FOLDER>/",
                                            "BufferingHints": {
                                                "SizeInMBs": <BUFFER_SIZE>,
                                                "IntervalInSeconds": <BUFFER_INTERVAL>
                                            }
                                        }'
    ```
    - `STREAM_TYPE`: type of the delivery stream. It can be one of the following values:
        - `DirectPut`: data records are delivered to your destination with no additional processing.
        - `KinesisStreamAsSource`: the delivery stream uses a `Kinesis Data Stream` as a source.
    - `PATH_TO_FOLDER`: the prefix that Firehose applies to the data record object names in S3, which simulates a folder structure in the S3 bucket where the streamed data is stored.
    - `BUFFER_SIZE`: size of the buffer, in MBs, used for buffering incoming data before delivering it to the S3 bucket, the value should be between 1 and 128. The data is delivered to the S3 bucket when either the buffer size reaches or the time period elapses.
    - `BUFFER_INTERVAL`: the time, in seconds, for buffering incoming data before delivering it to the S3 bucket, the value should be between 60 (1 minute) and 900 (15 minutes).
    - `ROLE_NAME`: name of the IAM role that has permissions to access the S3 bucket. IAM role for Kinesis can be created based on [here](https://github.com/lehoanganhtai13/data-lake-for-smart-farming-system-using-aws-services/tree/main/kinesis_policy).

- Delete a delivery stream and its data
    ```bash
    aws firehose delete-delivery-stream --delivery-stream-name <STREAM_NAME>
    ```

- Describe the specified delivery stream and its status
    ```bash
    aws firehose describe-delivery-stream --delivery-stream-name <STREAM_NAME>
    ```

- List your delivery streams in alphabetical order of their names
    ```bash
    aws firehose list-delivery-streams
    ```

7. AWS IoT Core (IoT communication via MQTT)
- Create a thing
    ```bash
    aws iot create-thing --thing-name <THING_NAME>
    ```

- Create an IoT policy
    ```bash
    aws iot create-policy --policy-name <POLICY_NAME> --policy-document file://<LOCAL_PATH>/policy.json
    ```
    - `LOCAL_PATH`: relative path to folder containing your IoT policy file.
    - IoT policy file example:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "iot:Publish",
                    "iot:Receive",
                    "iot:PublishRetain"
                ],
                "Resource": [
                    "arn:aws:iot:<REGION>:<ACCOUNT_ID>:topic/<IOT_TOPIC>"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iot:Subscribe"
                ],
                "Resource": [
                    "arn:aws:iot:<REGION>:<ACCOUNT_ID>:topicfilter/<IOT_TOPIC>"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iot:Connect"
                ],
                "Resource": [
                    "arn:aws:iot:<REGION>:<ACCOUNT_ID>:client/<CLIENT_ID>"
                ]
            }
        ]
    }
    ```

- Creates a 2048-bit RSA key pair and issues an X.509 certificate using the issued public key to create an IoT MQTT client
    ```bash
    aws iot create-keys-and-certificate --set-as-active \
                                        --certificate-pem-outfile "<THING_NAME>.cert.pem" \
                                        --public-key-outfile "<THING_NAME>.public.key" \
                                        --private-key-outfile "<THING_NAME>.private.key"
    ```
    - File `root-CA.crt` - root certificate authority - can be downloaded here [here](https://www.amazontrust.com/repository/AmazonRootCA1.pem).

- List all certificates info (ARN, ID, status, created date)
    ```bash
    aws iot list-certificates
    ```

- Attach policy to a thing/group of thing/certificate
    ```bash
    aws iot attach-policy --policy-name <POLICY_NAME> --target "<ARN>"
    ```
    - `ARN`: the Amazon Resource Name of the target to be attached, usually is ARN of a certificate in this format `arn:aws:iot:<region>:<account-id>:cert/<certificate-id>`, if for a thing it would be `arn:aws:iot:<region>:<account-id>:thing/<thing-name>`, or `arn:aws:iot:<region>:<account-id>:thinggroup/<group-name>` for group of thing. 

- Attach a certificate to a thing
    ```bash
    aws iot attach-thing-principal --thing-name <THING_NAME> --principal "arn:aws:iot:<REGION>:<ACCOUNT_ID>:cert/<CERTIFICATE_ID>"
    ```

- Get a ATS endpointto the Amazon Web Services account making the call.
    ```bash
    aws iot describe-endpoint --endpoint-type iot:Data-ATS
    ```

- Connect to AWS IoT, subscribe and publish to a topic (Example code using AWS Python SDK)
    ```python
    import AWSIoTPythonSDK.MQTTLib as AWSIoTPyMQTT
    import time
    import json

    # Callback when the subscribed topic receives a message
    def customCallback(client, userdata, message):
        print("Received a new message: ")
        print(message.payload)
        print("from topic: ")
        print(message.topic)
        print("--------------\n\n")

    # Initialize the AWS IoT MQTT Client
    myAWSIoTMQTTClient = AWSIoTPyMQTT.AWSIoTMQTTClient("<CLIENT_ID>")
    myAWSIoTMQTTClient.configureEndpoint("<ENDPOINT>", 8883)
    myAWSIoTMQTTClient.configureCredentials("<ROOT_CA_PATH>", "<PRIVATE_KEY_PATH>", "<CERTIFICATE_PATH>")

    # Connect to AWS IoT
    myAWSIoTMQTTClient.connect()

    # Subscribe to a topic
    myAWSIoTMQTTClient.subscribe("<TOPIC_NAME>", 1, customCallback)

    # Publish messages to the topic
    for i in range(10):
        message = {"message": "Hello, world!", "sequence": i}
        myAWSIoTMQTTClient.publish("<TOPIC_NAME>", json.dumps(message), 1)
        print(f'Sent {i}')
        time.sleep(1)  # Publish a message every second

    # Disconnect from the topic
    myAWSIoTMQTTClient.unsubscribe("<TOPIC_NAME>")
    myAWSIoTMQTTClient.disconnect()
    ```


- Create a rule for AWS IoT to listen to MQTT topics and invoke actions when certain conditions are met
    ```bash
    aws iot create-topic-rule --rule-name <RULE_NAME> --topic-rule-payload file://<LOCAL_PATH>/IoT_rule.json
    ```
    - `LOCAL_PATH`: relative path to folder containing your IoT rule file.
    - IoT rule file example to trigger action `kinesis` and `republish` for error, you can optionally choose the approriate action for specific scenario, it could be `s3`, `lambda`, `dynamoDB`, etc:
        - `ACTION_ROLE_NAME`: name of the IAM role that has permissions to access the Amazon Kinesis Firehose stream.
        - `ERROR_ACTION_ROLE_NAME`: name of the IAM role that has permissions to republish the message to another MQTT topic `ERROR_TOPIC_NAME`.
        - `qos`: Quality of Service (QoS) level for the MQTT subscription
            - QoS 0: the message is delivered at most once. This is the fastest mode of transfer, but there's no guarantee that the message is received.
            - QoS 1: the message is delivered at least once. The sender stores the message until it gets an acknowledgement from the receiver. This ensures that the message is received, but there might be duplicates.
            - QoS 2: the message is delivered exactly once. This is the safest, but slowest mode of transfer.
    ```json
    {
        "sql": "SELECT * FROM '<TOPIC_NAME>'",
        "actions": [{
            "firehose": {
                "roleArn": "arn:aws:iam::<ACCOUNT_ID>:role/<ACTION_ROLE_NAME>",
                "deliveryStreamName": "<STREAM_NAME>",
                "separator": "\n",
                "batchMode": false
            }
        }],
        "errorAction": {
            "republish": {
                "roleArn": "arn:aws:iam::<ACCOUNT_ID>:role/<ERROR_ACTION_ROLE_NAME>",
                "topic": "<ERROR_TOPIC_NAME>",
                "qos": 1
            }
        },
        "ruleDisabled": false,
        "awsIotSqlVersion": "2016-03-23"
    }
    ```

- Detach the certificate from the thing
    ```bash
    aws iot detach-thing-principal --thing-name <THING_NAME> --principal "arn:aws:iot:<REGION>:<ACCOUNT_ID>:cert/<CERTIFICATE_ID>"
    ```

- Deactivate the certificate
    ```bash
    aws iot update-certificate --certificate-id <CERTIFICATE_ID> --new-status INACTIVE
    ```

- Detach the policy from the certificate
    ```bash
    aws iot detach-policy --policy-name <POLICY_NAME> --target "arn:aws:iot:<REGION>:<ACCOUNT_ID>:cert/<CERTIFICATE_ID>"
    ```

- Delete the certificate
    ```bash
    aws iot delete-certificate --certificate-id <CERTIFICATE_ID>
    ```

- Delete the IoT policy
    ```bash
    aws iot delete-policy --policy-name <POLICY_NAME>
    ```

- Delete the thing
    ```bash
    aws iot delete-thing --thing-name <THING_NAME>
    ```

- Delete IoT rule
    ```bash
    aws iot delete-topic-rule --rule-name <IOT_RULE_NAME>
    ```

## <a name="docker"></a> 2. How to use Docker

### Docker commands
1. Run container

2. Stop running container

3. Build Docker image

### Docker Compose commands
1. Run multi containers (services) of an application
    ```bash
    docker-compose up --build -d
    ```
- `--detach`, `-d`: run containers in the background and leave them running,
- `--no-color`: produce monochrome output.
- `--quiet-pull`: pull without printing progress information.
- `--no-deps`: Don't start linked services.
- `--force-recreate`: recreate containers even if their configuration and image haven't changed.
- `--no-recreate`: if containers already exist, don't recreate them.
- `--no-build`: don't build an image, even if it's missing.
- `--no-start`: don't start the services after creating them.
- `--build`: build images before starting containers.
- `--remove-orphans`: remove containers for services not defined in the Compose file.
- `--timeout TIMEOUT`: use this timeout in seconds for container shutdown when attached or when containers are already running (default is 10).

2. Stop running all the containers
    ```bash
    docker-compose down --rmi all
    ```
- `--rmi all`: remove all images used by any service.
- `--rmi local`: remove only images that don't have a custom tag set by `image` field.
- `--volumes`, `-v`: remove all volumes defined in `volumes` section of the Compose file, but not the named volumes declared in the `docker-compose.yml` file or any other `.yml` files with option `-f`.
- `--remove-orphans`: remove containers for services not defined in the Compose file.
- `--timeout TIMEOUT`, `-t TIMEOUT`: specify a shutdown timeout in seconds (default is 10).

3. Build Docker image seperately
    ```bash
    docker-compose build
    ```
- `--compress`: compress the build context using gzip.
- `--force-rm`: always remove intermediate containers.
- `--no-cache`: do not use cache when building the image to force Docker to run the command again instead of using the cached result.
- `--pull`: always attempt to pull a newer version of the image. When you build a Docker image, it's often based on another image, specified in the `FROM` directive in your Dockerfile. Docker will pull this base image from a Docker registry (like Docker Hub) the first time you build your image.
- `--message`, `-m`: set commit message for the image.
- `SERVICE ...`: optionally specify service(s) to build. If no services are specified, all services defined in the Docker Compose file are built.

