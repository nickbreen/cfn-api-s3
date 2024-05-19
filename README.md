```shell
docker-compose down
```

```shell
docker-compose up --wait
```

```shell
docker-compose restart
```

```shell
docker-compose logs --follow
```

# Web

```shell
. .venv.awslocal/bin/activate
awslocal cloudformation delete-stack --stack-name web
awslocal cloudformation wait stack-delete-complete --stack-name web
awslocal cloudformation validate-template --template-body file://web.yaml
awslocal cloudformation create-stack --stack-name web --template-body file://web.yaml
awslocal cloudformation wait stack-create-complete --stack-name web
awslocal cloudformation describe-stacks --stack-name web
# Deploy website
bucket=$(awslocal cloudformation describe-stacks --stack-name web --output text --query "Stacks[].Outputs[?OutputKey == 'Bucket'].OutputValue")
awslocal s3 sync www/ s3://${bucket?}/
# check website
url=$(awslocal cloudformation describe-stacks --stack-name web --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -f "${url?}/index.html" && xdg-open "${url?}/index.html"
deactivate
```

# SQS
## Localstack
```shell
. .venv.awslocal/bin/activate
awslocal cloudformation delete-stack --stack-name accepted
awslocal cloudformation wait stack-delete-complete --stack-name accepted
awslocal cloudformation validate-template --template-body file://accepted.yaml
awslocal cloudformation create-stack --stack-name accepted --template-body file://accepted.yaml
awslocal cloudformation wait stack-create-complete --stack-name accepted
awslocal cloudformation describe-stacks --stack-name accepted
# check it
url=$(awslocal cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -sSfD/dev/stderr --header 'Content-Type: application/json' --header 'Accept: application/json' --data @- "${url?}/queue" <<< '{"f":1}'  | jq .
queue=$(awslocal cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'Queue'].OutputValue")
curl -sSf --header 'Accept: application/json' "localhost.localstack.cloud:4566/_aws/sqs/messages?QueueUrl=${queue}" | jq .
deactivate
```

## AWS
```shell
. .venv.awslocal/bin/activate
aws cloudformation delete-stack --stack-name accepted
aws cloudformation wait stack-delete-complete --stack-name accepted
aws cloudformation validate-template --template-body file://accepted.yaml
aws cloudformation create-stack --stack-name accepted --template-body file://accepted.yaml --capabilities CAPABILITY_IAM
aws cloudformation wait stack-create-complete --stack-name accepted
aws cloudformation describe-stacks --stack-name accepted
# check it 
url=$(aws cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -sSfD/dev/stderr --header 'Content-Type: application/json' --header 'Accept: application/json' --data @- "${url?}/queue" <<< '{"f":1}' | jq .
queue=$(aws cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'Queue'].OutputValue")
aws sqs receive-message --queue-url $queue --max-number-of-messages 10 | jq .
deactivate
```

# Bazel

Stopped immediately by a bug with `rules_python`: https://github.com/bazelbuild/rules_python/issues/1659.

# Utter Python Bullshit

My system python is 3.12, which `awsebcli` is not compatible with: https://github.com/aws/aws-elastic-beanstalk-cli-setup/issues/153.

So we need this crap: https://github.com/pyenv/pyenv.

So know that we're building python from source for any given version... just follow `pyenv`' instructions. Sigh.

```shell
. .env
pyenv install 3.11
pyenv version
```

```shell
python -m venv .venv.eb
. .venv.eb/bin/activate
pip install awsebcli
eb --version
deactivate
```

```shell
python -m venv .venv.awslocal
. .venv.awslocal/bin/activate
pip install awscli-local awscli
awslocal --version
deactivate
```

OMG.