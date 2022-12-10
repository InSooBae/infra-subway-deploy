<p align="center">
    <img width="200px;" src="https://raw.githubusercontent.com/woowacourse/atdd-subway-admin-frontend/master/images/main_logo.png"/>
</p>
<p align="center">
  <img alt="npm" src="https://img.shields.io/badge/npm-%3E%3D%205.5.0-blue">
  <img alt="node" src="https://img.shields.io/badge/node-%3E%3D%209.3.0-blue">
  <a href="https://edu.nextstep.camp/c/R89PYi5H" alt="nextstep atdd">
    <img alt="Website" src="https://img.shields.io/website?url=https%3A%2F%2Fedu.nextstep.camp%2Fc%2FR89PYi5H">
  </a>
  <img alt="GitHub" src="https://img.shields.io/github/license/next-step/atdd-subway-service">
</p>

<br>

# 인프라공방 샘플 서비스 - 지하철 노선도

<br>

## 🚀 Getting Started

### Install
#### npm 설치
```
cd frontend
npm install
```
> `frontend` 디렉토리에서 수행해야 합니다.

### Usage
#### webpack server 구동
```
npm run dev
```
#### application 구동
```
./gradlew clean build
```
<br>

## 미션

* 미션 진행 후에 아래 질문의 답을 README.md 파일에 작성하여 PR을 보내주세요.

### 0단계 - pem 키 생성하기

1. 서버에 접속을 위한 pem키를 [구글드라이브](https://drive.google.com/drive/folders/1dZiCUwNeH1LMglp8dyTqqsL1b2yBnzd1?usp=sharing)에 업로드해주세요

2. 업로드한 pem키는 무엇인가요.
   -  insoobae-key

### 1단계 - 망 구성하기
1. 구성한 망의 서브넷 대역을 알려주세요
- 대역 : 
  - 외부망-a(insoobae-public-a) : 192.168.64.0/26
  - 외부망-b(insoobae-public-b) : 192.168.64.64/26
  - 내부망-a(insoobae-internal-a): 192.168.64.128/27
  - 관리망-b(insoobae-bastion-b): 192.168.64.160/27

2. 배포한 서비스의 공인 IP(혹은 URL)를 알려주세요

- URL : 3.39.221.17(http://insoobae-test.kro.kr)



---

### 2단계 - 배포하기
1. TLS가 적용된 URL을 알려주세요

- URL : https://insoobae.n-e.kr/

---

### 3단계 - 배포 스크립트 작성하기

1. 작성한 배포 스크립트를 공유해주세요.

```bash
#!/bin/bash

## 변수 설정

txtrst='\033[1;37m' # White
txtred='\033[1;31m' # Red
txtylw='\033[1;33m' # Yellow
txtpur='\033[1;35m' # Purple
txtgrn='\033[1;32m' # Green
txtgra='\033[1;30m' # Gray

EXE_PATH=/home/ubuntu/nextstep/infra-subway-deploy/
BRANCH=$1
PROFILE=$2

echo -e "${txtylw}=======================================${txtrst}"
echo -e "${txtgrn}  << 스크립트 🧐 >>${txtrst}"
echo -e "${txtylw}=======================================${txtrst}"

## 저장소 pull
function pull() {
	echo -e "${txtylw}=======================================${txtrst}"
	echo -e "${txtgrn}  << git switch $BRANCH >> ${txtrst}"
	git switch $BRABCH

	git pull origin $BRANCH
}

function check_df() {
        git fetch
        master=$(git rev-parse $BRANCH)
        remote=$(git rev-parse origin/$BRANCH)

        if [[ $master == $remote ]]; then
                echo -e "[$(date)] Nothing to do!!! 😫"
                exit 0
        else
                pull
        fi
}
## gradle build
function app_build() {
        echo -e "${txtylw} Gradle build... ${txtrst}"
        echo -e ""

        ./gradlew clean build

        echo -e ""
}

## 프로세스 pid를 찾는 명령어
function find_process() {
      echo -e "${txtylw} Find process... ${txtrst}"
      echo -e ""

      jar_file_name=$(basename ./build/libs/*.jar)
      ps_pid=$(pgrep -f $jar_file_name)
      if [ -z "$ps_pid" ]; then
	      echo -e "${txtgrn} >> There is no existing process. Reruning... ${txtrst}"
	      app_start
      else
	      echo -e "${txtgrn} >> Find existing process: $ps_pid"
      fi
      echo -e ""
}
## 프로세스를 종료하는 명령어
function kill_process() {
      	echo -e "${txtylw} Kill process $ps_pid... ${txtrst}"
      	echo -e ""
  
      	sudo kill $ps_pid
      	echo -e " >> Successfully killed process. ${txtrst}"

      	echo -e ""
}

function app_start() {
      	echo -e "${txtpur}[$(date)] APP 시작.${txtrst}"
	file=$(sudo find ./build/libs/* -name "*.jar")
      	sudo nohup java -jar -Dspring.profiles.active=$PROFILE $file 1> ../spring-server.log 2>&1 &
      	java_pid="$(pgrep -f java)"
      	if [ -z "$java_pid" ]
      	then
	    	echo -e "${txtred}[$(date)] APP 시작 실패.${txtrst}"
	    	exit 1
      	fi
}

function deploy() {
	find_process
	check_df
	find_process
	kill_process
	app_build
	app_start
}
cd $EXE_PATH
deploy
```
