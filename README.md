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
bin/aws cloudformation delete-stack --stack-name web
bin/aws cloudformation wait stack-delete-complete --stack-name web
```

```shell
bin/aws cloudformation validate-template --template-body file://web.yaml
bin/aws cloudformation create-stack --stack-name web --template-body file://web.yaml
bin/aws cloudformation wait stack-create-complete --stack-name web
bin/aws cloudformation describe-stacks --stack-name web
```

```shell
bucket=$(bin/aws cloudformation describe-stacks --stack-name web --output text --query "Stacks[].Outputs[?OutputKey == 'Bucket'].OutputValue")
bin/aws s3 sync www/ s3://${bucket?}/
```

```shell
url=$(bin/aws cloudformation describe-stacks --stack-name web --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -f "${url?}/index.html" && xdg-open "${url?}/index.html"
```

# SQS

```shell
bin/aws cloudformation delete-stack --stack-name accepted
bin/aws cloudformation wait stack-delete-complete --stack-name accepted
```

```shell
bin/aws cloudformation validate-template --template-body file://accepted.yaml
bin/aws cloudformation create-stack --stack-name accepted --template-body file://accepted.yaml
bin/aws cloudformation wait stack-create-complete --stack-name accepted
bin/aws cloudformation describe-stacks --stack-name accepted
```

```shell
url=$(bin/aws cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -sSfD/dev/stderr --header 'Content-Type: application/json' --header 'Accept: application/json' --data @- "${url?}/queue" <<< '{"f":1}'  | jq .
queue=$(bin/aws cloudformation describe-stacks --stack-name accepted --output text --query "Stacks[].Outputs[?OutputKey == 'Queue'].OutputValue")
curl -sSf --header 'Accept: application/json' "localhost.localstack.cloud:4566/_aws/sqs/messages?QueueUrl=${queue}" | jq .
```
