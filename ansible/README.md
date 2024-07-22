# Ansible 활용 방법

## 📝 File

### 1. ansible.cfg
* 별도의 디렉토리마다 config 파일 생성 가능
  * ansible 명령어를 수행하는 현재 디렉토리의 ansible.cfg 파일을 우선으로 읽음
  * 현재 디렉토리에 ansible.cfg 파일이 없다면, /etc/ansible/ansible.cfg 를 default로 참조
```
[defaults]
inventory = ./inventory #사용할 inventory 파일 경로
host_key_checking = False #호스트 키 확인 비활성화
ask_pass = True #비밀번호 입력란 표시 (-k 옵션과 동일)
remote_user = ca*v12*** #원격 서버에 접속하는 username

# remote_user가 sudo 권한을 상속받기 위한 설정
[privilege_escalation]
become=True  #playbook 설정에 덮어씌워짐
become_method=sudo
become_user=root
become_ask_pass=False #remote_user가 sudo 명령어 사용을 위해 password가 필요하다면, True로 설정하여 입력해줘야 함
```


### 2. inventory
* 원격서버로 사용할 서버들을 INI 파일 형식으로 작성
  * 호스트 그룹으로 그룹화 가능
  * 정규표현식과 같이 [0001:0100] 등의 syntax 사용 가능
* 예시)
```
[aws_servers]
ws***[0001:0200]

[azure_servers]
z*******[0001:0100]
z*******[0101:0200]

[oci_servers]
o****[0001:100]
```
* DNS 등록이나 /etc/hosts 등록되어있지 않을 시에는 아래와 같이 IP주소로 작성
```
[aws_servers]
10.0.10.[1:50]
10.0.20.[1:100]
```
* ```$ ansible-inventory --graph``` 명령어를 통해 현재 참조하는 인벤토리와 호스트 그룹별 서버 목록 확인 가능

### 3. Playbook
* ansible 모듈을 통한 task를 스크립트로 작성한 파일
* 예시)
```
- name: Copy configuration file playbook1
  hosts: aws_servers
  become: yes
  tasks:
    - name: Copy configuration file task1
      copy:
        src: /tmp/test
        dest: /tmp/test
    - name: Copy configuration file task2
      copy:
        src: /tmp/test2
        dest: /tmp/
```
  * remote_user의 권한 중요

## ⌨️ Command
* -k 옵션 사용 시 password를 통해 ssh 인증 가능
  * password는 ansible.cfg의 remote_user의 원격 서버 password
* ```$ ansible-inventory --graph```
  * 현재 config 파일에서 참조하는 인벤토리와 호스트 그룹별 서버 목록 확인 가능
* ```$ ansible-playbook [FILE_PATH]```
  * playbook 스크립트 실행
* ```$ ansible {my_target_host} -m {ad_hoc_module} -u {my_user_name} ```
  * 모듈을 통한 간단한 명령 가능 (ex. ping)
---
# ✅ 서버 모니터링
ping_check.yaml 플레이북을 실행한 뒤, 마지막 결과 요약(PLAY RECAP)을 보는 방식으로 간단하게 SSH 접속 점검 가능
* ping_check.yaml
```
- name: Server Ping Check
  hosts: TARGET_HOST_GROUP
  gather_facts: no  #해당 옵션을 통해 원격 서버에서 필요한 정보를 가져오는 시간을 줄여 빠르게 SSH 접속만 검증 가능
  tasks:
  - name: simple ping check
    ping:
```

# 🖥️ 서버 검증 스크립트 활용 방법

## EXAMPLE - 재배포 서버 통합 점검
* 가정 상황 : TOSS 인시던트를 통해 wstest0251 ~ wstest0300 서버 OS 7.9 → 8.6 업데이트 및 H/T off 요청 수신
  * 재배포 완료 시점에서 서버 통합 검증 Playbook 활용

### 작업 환경으로 이동 (DS Confluence내 가이드 문서 참조)
* ```$ ssh p*k*0022```
  * Ansible 컨트롤 서버에 접속
* ```$ cd /user/cloudtest/USERS/ys***ng/workspace/ansible-validation (검증 작업 디렉토리)```
  * 작업 디렉토리로 이동

### ansible.cfg 수정 및 확인
* ```$ vi ansible.cfg```
  * remote_user = [현재 VWP에 로그인 되어있는 LDAP 계정으로 수정]

### 인벤토리 수정 및 확인
* ```$ vi inventory```
  * Ansible 인벤토리에 검증할 Host List를 그룹으로 생성
예시)
```
[aws_temp]
wstest[0251:0300]
```
* ```$ ansible-inventory --graph```
  * 인벤토리 검증 (aws_temp 호스트 그룹의 hostname 확인)

### 검증 내용에 따른 Playbook 수정 후 명령 실행
* 현 예시에서는 SSH 접속, OS 및 커널 버전, Core 수, Mount 검증을 위해 통합 검증 플레이북 실행
* ```$ vi server_check.yaml```
  * hosts: TARGET_HOST_GROUP -> 점검 대상 호스트 그룹(인벤토리 파일 내)으로 수정
  * 서버에 설정되어 있어야 할 값에 대한 정보 수정 (서버에서 아래 명령어 실행 시 나타나는 값)
    * EXPECTED_RHEL_VERSION (cat /etc/redhat-release)
    * EXPECTED_KERNEL_VERSION (uname -r)
    * EXPECTED_CORE_NUM (nproc)
    * EXPECTED_MOUNT_DIR (df로 확인할 mount 디렉토리)
* ```$ echo “----------” >> server_check.log```
  * 실행 전, Output 파일 내 추가될 내용 구분을 위한 라인 첨부
* ```$ ansible-playbook -k server_check.yaml```
  * 플레이북 실행
* 검증 이후 조치 필요 사항
  * ```cat server_check.log```
  * 검증에 실패한 모든 서버(server_check.log에 기록된 hostname)에 접속하여 상세 점검 필요

