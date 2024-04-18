# Learn MLOps/DevOps
A self-study repo, where there is everything you need for a MLOps system. This could also apply for DevOps system.


## 1. How to use AWS

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
    aws iam list-attached-[group/user/role]-policies    --[group/user/role]-name <NAME> \
                                                        --scope <MANAGE_TYPE>
    ```
    - `MANAGE_TYPE`: we will use `All` for all policies including AWS-managed policies and the customer managed policies in your AWS account, `AWS` for only AWS-managed policies, and `Local` for only the customer managed policies in your AWS account.

## 2. How to use Docker

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

