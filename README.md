# CSP_script

## 반복문을 사용 한 process 병렬 처리
1. cli script 적용 대상 list file 저장
2. parameter별 값이 상수인 경우 cli script 내에 작성하고 변수인 경우 변수 처리
3. 반복문을 통해 list file -> cli script 치환 및 검증/적용

### list file 작성
```
# cat list
i-test001 10.0.0.21
i-test002 10.0.0.22
i-test003 10.0.0.23
```

### cli script 작성
```
# cat create_instance
# https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
# 인스턴스 생성
aws ec2 run-instances \
--image-id ami-026d515222063994d \
--security-group-ids sg-09bbfae2826c79501 \
--subnet-id subnet-00d4d3f6565f9f712 \
--private-ip-address IP \
--tag-specifications 'ResourceType=instance,Tags=[{Key=webserver,Value=production}]' \
--instance-type t3.micro \ 
# --cpu-options “CoreCount=40, ThreadsPerCore=2” \ # HT Option
--disable-api-termination \
# --capacity-reservation-specification CapacityReservationTarget=CapacityReservationId \
--user-data """#cloud-boothhook #!/bin/bash hostnamectl set-hostname NAME""" &
```

### for문 검증
```
# for I_NAME in `cat list |awk '{print $1}'`;
do PVIP=`cat list |grep $I_NAME |awk '{print $2}'`;
cat create_instance |sed "s/NAME/$I_NAME/g" |sed "s/PVIP/$IP/g" > ./script_out;
cat ./script_out;
done;
```

### for문 script 실행
script 실행 전 script_out 파일에 실행 권한(+x) 부여 후 진행
```
# for I_NAME in `cat list |awk '{print $1}'`;
do PVIP=`cat list |grep $I_NAME |awk '{print $2}'`;
cat create_instance |sed "s/NAME/$I_NAME/g" |sed "s/PVIP/$IP/g" > ./script_out;
cat ./script_out;
./script_out;
done
```