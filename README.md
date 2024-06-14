# CSP별 운영용 cli script 제어 방법
## 용어 설명
* 선번장
    * [NAME] [IP]
    * 공백을 구분자로 지정
    * 구분자가 공백이 아닌 경우 -F 옵션을 사용하여 구분자 지정
    * 작업 대상을 list-up 하여 선번장 파일로 저장
* DNS 등록
    * Hybrid Cloud 투입 장비의 경우 TOSS 인시던트 발행하여 DNS 등록 필요
- - -
## 업무 프로세스
* VM 생성
    1. 선번장 작성
    2. cli script 작성
    3. 반복문을 사용하여 배포전 검증
    4. 실행
* VM 제거
    1. 선번장 작성
    2. cli script 작성
    3. 반복문을 사용하여 배포전 검증
    4. 실행
* 서버 점검
    1. 추후 Ansible yml file 업데이트 예정
* H/T 옵션 변경
    1. 선번장 작성
    2.  
- - -
# EXAMPLE - AWS EC2 생성
## 반복문을 통한 process 병렬 처리 제어 방법
1. cli script 적용 대상 list file 저장
2. parameter별 값이 상수인 경우 cli script 내에 작성하고 변수인 경우 변수 처리
3. 반복문을 통해 list file -> cli script 치환 및 검증/적용
- - -
## 선번장(list file) 작성
```
# cat list
i-test001 10.0.0.21
i-test002 10.0.0.22
i-test003 10.0.0.23
```
- - -
## cli script 작성
parameter별 상수인 값(subnet-id, image-id 등등)은 작성하고 변수인 값(instance name, private IP 등)은 변수 처리
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
- - -
## for문 검증
변수 치환이 된 script_out 파일 확인을 통해 치환이 제대로 이루어 졌는지 확인
```
# for I_NAME in `cat list |awk '{print $1}'`;
do PVIP=`cat list |grep $I_NAME |awk '{print $2}'`;
cat create_instance |sed "s/NAME/$I_NAME/g" |sed "s/PVIP/$IP/g" > ./script_out;
cat ./script_out;
echo '';
done;
```
- - -
## for문 script 실행
script 실행 전 script_out 파일에 실행 권한(+x) 부여 후 진행
```
# for I_NAME in `cat list |awk '{print $1}'`;
do PVIP=`cat list |grep $I_NAME |awk '{print $2}'`;
cat create_instance |sed "s/NAME/$I_NAME/g" |sed "s/PVIP/$IP/g" > ./script_out;
cat ./script_out;
./script_out;
done
```