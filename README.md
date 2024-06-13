# CSP_script

### 반복문
```
# cat list          # list file 작성/구분자는 공백으로
i-test001 10.0.0.21
i-test002 10.0.0.22
i-test003 10.0.0.23

# cat create_instance
# https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
# 인스턴스 생성
aws ec2 run-instances \
--image-id AMI-ID \
--security-group-ids SG-ID \
--subnet-id SUBNET-ID \
--private-ip-address PVIP \
--tag-specifications TAG \
--instance-type INSTANCE_TYPE \
# --cpu-options “CoreCount=40, ThreadsPerCore=2” \ # HT Option
--disable-api-termination \
--capacity-reservation-specification CapacityReservationTarget=CapacityReservationId \
--user-data """#cloud-boothhook #!/bin/bash hostnamectl set-hostname HOSTNAME"""&%

# for I_NAME in `cat list |awk '{print $1}'` \
do PVIP=`cat list |grep $I_NAME |awk '{print $2}'` \
cat create_instance |sed "s/
