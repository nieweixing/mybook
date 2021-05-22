# aws cli命令行操作

## 首先获取账号的access

```
Access key ID,Secret access key
AKIATRxxxxxxxxJX5WL,ZP2enHUH3XegoGhZxxxxxxxxfDMZo43ohz
```

## 登录ec2的实例下载awscli

### pip安装awscli

```
ssh -i xxxxx.pem ec2-user@10.0.64.11
sudo -i
yum -y install python-pip
pip install awscli
```

### 2进制包安装awscli

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## 登录awscli

```
[root@ip-10-0-64-11 ~]# aws configure
AWS Access Key ID [None]: AKIATRMLTGCFIY5WRHGG
AWS Secret Access Key [None]: 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
Default region name [None]: cn-northwest-1
Default output format [None]:   可以不输入
```

## 从S3桶下载对应的文件

```
aws s3 ls
aws s3 cp s3://xxxx/jobs.tgz /root
```

## 其他的awscli命令

### EC2

#### 挂载 EBS

* linux 

```
查看块设备： lsblk
格式化磁盘： sudo mkfs -t ext4 /dev/xvdb
挂载卷： sudo mount /dev/xvdb /mnt/mydir
卸载卷： sudo umount /dev/xvdb
```
* windows 

```
diskpart
san policy=onlineall
list disk
disk yourdiskid
attributes disk clear readonly
online disk
```

#### ec2操作

```
aws ec2 describe-instances
aws ec2 describe-instances --instance-ids "instanceid1" "instanceid2"
aws ec2 start-instances --instance-ids "instanceid1" "instanceid2"
aws ec2 stop-intances --instance-ids "instanceid1" "instanceid2"
aws ec2 run-instances --image-id ami-b6b62b8f --security-group-ids sg-xxxxxxxx --key-name mytestkey --block-device-mappings "[{\"DeviceName\": \"/dev/sdh\",\"Ebs\":{\"VolumeSize\":100}}]" --instance-type t2.medium --count 1 --subnet-id subnet-e8330c9c --associate-public-ip-address 
(Note: 若不指定subnet-id则会在默认vpc中去选，此时若指定了非默认vpc的安全组会出现请求错误。如无特殊要求，建议安全组和子网都不指定，就不会出现这种问题。)
查看region与AZ


aws ec2 describe-region
aws ec2 describe-availability-zones --region region-name
```


#### 查看ami

```
aws ec2 describe-images
```

#### key-pair

```
aws ec2 create-key-pair --key-name mykeyname
```

#### 安全组

```
aws ec2 create-security-group --group-name mygroupname --description mydescription --vpc-id vpc-id (若不指定vpc，则在默认vpc中创建安全组)
aws ec2 authorize-security-group-ingress --group-id sg-xxxxyyyy --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-xxxxyyyy --protocol tcp --port 9999 --source-group sg-xxxxxxxx
```

### AutoScaling

```
列出AS组 
aws autoscaling describe-auto-scaling-groups
列出AS实例 
aws autoscaling describe-auto-scaling-instances --instance-ids [instance-id-1 instance-id-2 ...]
从组中分离实例 
aws autoscaling detach-instances --auto-scaling-group-name myasgroup --instance-ids instanceid1 instanceid2 [--should-decrement-desired-capacity|--no-should-decrement-desired-capacity]
附加实例到组 
aws autoscaling attach-instances --auto-scaling-group-name myasgroup --instance-ids instanceid1 instanceid2
挂起AS流程 
aws autoscaling suspend-process --auto-scaling-group-name myasgroup --scaling-processes AZRebalance|AlarmNotification|...
删除AS组 
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name myasgroup
```
### S3


#### 查看

```
aws s3 ls
aws s3 ls s3://bucket
aws s3 ls s3://bucket/prefix
```

#### 拷贝

```
aws s3 cp /to/local/path s3://bucket/prefix
aws s3 cp s3://bucket/prefix /to/local/path
aws s3 cp s3://bucket1/prefix1 s3://bucket2/prefix2
```

#### 同步

```
aws sync [--delete] /to/local/dir s3://bucket/prefixdir
aws sync [--delete] s3://bucket/prefixdir /to/local/dir
aws sync [--delete] s3://bucket1/prefixdir1 s3://bucket2/prefixdir2
```

#### 手动分片上传

文件分片 

```
split -b 40m myfile myfile-part-
```

创建分片上传任务 

```
aws s3api create-multipart-upload --bucket bucketname --key prefix
记录返回值

{
    "Bucket": "bucketname", 
    "UploadId": "uploadeid", 
    "Key": "prefix"
}
```

上传分片

```
aws s3api upload-part --bucket bucketname --key prefix --part-number [分片上传编号(e.g. 1,2,3...)] --body myfile-[x] --upload-id uploadid
```

列出已上传分片，创建分片结构文件 

```
aws s3api list-parts --bucket bucketname --key prefix --upload-id uploadid
```

将上命令结果中的parts部分保存为 temp 文件 

```
{"Parts": [ 
{ 
"PartNumber": 1, 
"ETag": "\"xxxxxxx\"" 
}, 
{ 
"PartNumber": 2, 
"ETag": "\"xxxxxxxx\"" 
} 
] 
} 
```

结束分片上传任务 

```
aws s3api complete-multipart-upload --multipart-upload file://temp --bucket bucketname --key prefix --upload-id uploadid
```


### AWSCLI访问阿里云OSS

```
aws configure --p aliyun #设置key与secret,其他默认
aws configure set s3.addressing_style virtual --p aliyun
aws s3 ls --endpoint-url [url/(e.g. http://oss-cn-hangzhou.aliyuncs.com)] --p aliyun
```

### IAM

#### Role操作 

```
aws iam create-role MY-ROLE-NAME --assum-role-policy-document file://path/to/trustpolicy.json
aws iam put-role-policy --role-name MY-ROLE-NAME --policy-name MY-PERM-POLICY --policy-document file://path/to/permissionpolicy.json
aws iam create-instance-profile --instance-profile-name MY-INSTANCE-PROFILE
aws iam add-role-to-instance-profile --instance-profile-name MY-INSTANCE-PROFILE --role-name MY-ROLE-NAME
```

### AUTO-SCALING

##### 查看信息 

```
aws autoscaling describe-auto-scaling-groups
aws autoscaling describe-auto-scaling-instances
```

### kinesis

#### 创建流 

```
aws kinesis create-stream –stream-name mystream –shard-count
```
#### 列出流 

```
aws kinesis list-streams
```

#### 获取指定流的分片迭代器 

```
aws kinesis get-shard-iterator –stream-name mystream –shard-id shard-1 –shard-iterator-type TRIM_HORIZON
```
#### 发送数据到流 

```
aws kinesis put-record –stream-name mystream –partition-key mykey –data test
```

#### 获取流数据

```
aws kinesis get-records –shard-iterator myiterator
```
