```shell
docker-compose down
```

```shell
docker-compose up --wait
```

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
bin/aws cloudformation describe-stacks --stack-name web --output text
url=$(bin/aws cloudformation describe-stacks --stack-name web --output text --query "Stacks[].Outputs[?OutputKey == 'URL'].OutputValue")
curl -f "${url?}/index.html" && xdg-open "${url?}/index.html"
```
