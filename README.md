# Kinesisの作成
Twitterのタイムラインデータを送信する先となるAmazon Kindesisのストリームを作成する

```sh
$ aws kinesis create-stream --stream-name twitter-stream \
--shard-count 1
```

作成されたこと確認

```sh
$ aws kinesis list-streams
```

# データ保存先のDynamoDBのテーブルも作成しておく

```sh
$ aws dynamodb create-table --table-name tweet-data \
--attribute-definitions AttributeName=id,AttributeType=N \
AttributeName=timestamp,AttributeType=N \
--key-schema AttributeName=id,KeyType=HASH \
AttributeName=timestamp,KeyType=RANGE \
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

# スクリプトで利用するライブラリをインストールしておく。boto3はAWSSDK。

```sh
$ pip install boto3
$ pip install TwitterAPI
```

Twitterのクレデンシャル情報を記述。

```credentials.py
customer_key = "XXXXXXXXXXXXXXXX"
customer_secret = "XXXXXXXXXXXXXXXX"
access_token_key = "XXXXXXXXXXXXXXXX"
access_token_secret = "XXXXXXXXXXXXXXXX"
```
# IAMRoleの作成

```
$ aws iam create-role --role-name process-tweet-data-role \
--assume-role-policy-document file://path_to/trustpolicy.json
```

作成したIAMロールのARNを確認し、控えておく。

# IAMロールにポリシーを適用する

permission.jsonのarnを自分のアカウントのものに変更すうｒ


# permission.jsonの権限をIAMロールに割り当てる

```
$ aws iam put-role-policy --role-name process-tweet-data-role \
--policy-name dynamodb-access --policy-document \
file://path_to/permission.json
```

# デプロイパッケージの作成

コードを保存したディレクトリと同じディレクトリにzipファイルを作成

```
$ zip process-tweet-data.zip process-tweet-data.py
```

# Lambdaファンクションを作成する

```
$ aws lambda create-function --function-name process-tweet-data \
--zip-file fileb://path_to/process-tweet-data.zip \
--role arn:aws:iam:111122223333:role/process-tweet-data-role \
--handler process-tweet-data.lambda_handler --funtime python3.6 \
--environment Variables={TABLE_NAME=tweet-data}
```
