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
KEYNAME="CHANGE_KEY_PAIR_NAME"  #環境に合わせてキーペア名を設定してください。  

#最新のAmazon Linux2のAMI IDを取得します。
AL2_AMIID=$(aws --profile ${PROFILE} --region ${REGION} --output text \
    ec2 describe-images \
        --owners amazon \
        --filters 'Name=name,Values=amzn2-ami-hvm-2.0.????????.?-x86_64-gp2' \
                  'Name=state,Values=available' \
        --query 'reverse(sort_by(Images, &CreationDate))[:1].ImageId' ) ;

CFN_STACK_PARAMETERS='
[
  {
    "ParameterKey": "AmiId",
    "ParameterValue": "'"${AL2_AMIID}"'"
  },
  {
    "ParameterKey": "KeyName",
    "ParameterValue": "'"${KEYNAME}"'"
  }
]'

aws --profile ${PROFILE} --region ${REGION} cloudformation create-stack \
    --stack-name KernelShowWeb \
    --template-body "file://./cfn/web_server.yaml" \
    --parameters "${CFN_STACK_PARAMETERS}" \
    --capabilities CAPABILITY_IAM ;
```
## (3)WebServerのデプロイ(Ansible利用)
### (3)-(a) WebServerへのログイン
```shell
WebIp=$(aws --profile ${PROFILE} --region ${REGION} --output text \
    cloudformation describe-stacks \
        --stack-name KernelShowWeb \
        --query 'Stacks[].Outputs[?OutputKey==`WebServer1PubIP`].[OutputValue]')

ssh ec2-user@${WebIp}
```
### (3)-(b) Ansibleとwebserver用playbookのgit clone
```shell
sudo amazon-linux-extras install ansible2

git clone https://github.com/Noppy/ansible-BuildWebServer.git
cd ansible-BuildWebServer/
```
### (3)-(c) Create Inventory File
```shell
cat > inventory << EOL
[webservers]
127.0.0.1 ansible_connection=local
EOL
```
### (3)-(d) Execute ansible play-book
```shell
#Nitro系の場合
EbsDevName="/dev/nvme2"
PartitionDevName="/dev/nvme2n1"
#Xen系の場合
EbsDevName="/dev/xvdb"
PartitionDevName="/dev/xvdb1"

#セットアップ
ansible-playbook site.yml --extra-vars "EbsDevName=${EbsDevName} PartitionDevName=${PartitionDevName}" -i inventory
```

