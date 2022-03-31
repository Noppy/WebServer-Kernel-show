# WebServer-Kernel-show
Kernel様の



## 手順
## (1)事前設定
### (1)-(a) 作業環境の準備
下記を準備します。
* bashが利用可能な環境(LinuxやMacの環境)
* aws-cliのセットアップ
* AdministratorAccessポリシーが付与され実行可能な、aws-cliのProfileの設定
### (1)-(b) CLI実行用の事前準備
これ以降のAWS-CLIで共通で利用するパラメータを環境変数で設定しておきます。
```shell
export PROFILE=<設定したプロファイル名称を指定。デフォルトの場合はdefaultを設定>
export REGION=ap-northeast-1
```
## (2)環境のデプロイ(CloudFormation利用)
### (2)-(a)gitのclone
```shell
git clone https://github.com/Noppy/WebServer-Kernel-show.git
cd WebServer-Kernel-show.git
```

### (2)-(b) CloudFormationによるデプロイ
```shell
#最新のAmazon Linux2のAMI IDを取得します。
AL2_AMIID=$(aws --profile ${PROFILE} --region ${REGION} --output text \
    ec2 describe-images \
        --owners amazon \
        --filters 'Name=name,Values=amzn2-ami-kernel-5.10-hvm-2.?.????????.?-arm64-gp2' \
                  'Name=state,Values=available' \
        --query 'reverse(sort_by(Images, &CreationDate))[:1].ImageId' ) ;

CFN_STACK_PARAMETERS='
[
  {
    "ParameterKey": "AmiId",
    "ParameterValue": "'"${AL2_AMIID}"'"
  }
]'

aws --profile ${PROFILE} --region ${REGION} cloudformation create-stack \
    --stack-name KernelShowWeb \
    --template-body "file://./cfn/web_server.yaml" \
    --parameters "${CFN_STACK_PARAMETERS}" \
    --capabilities CAPABILITY_IAM ;
```

## srpm2htmlの使い方
こちらを参照
- https://nopipi.hatenablog.com/entry/2017/06/11/161709

## srpm2htmlで作成したhtmlをCF公開用のS3にアップする
