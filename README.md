# Django 배포하기
`Nginx`, `Gunicorn`, `EC2`, `MySQL` 을 사용하여 배포하기

순서
1. `Nginx`, `Gunicorn` 연동
2. `Amazon EC2` 인스턴스 생성
3. `Amazon RDS` 사용하지 않고 `Amazon EC2` 인스턴스에 `MySQL` 직접 설치

# Amazon EC2
1. 인스턴스 생성
    1. 이름 설정
    2. 애플리케이션 및 OS 이미지(Amazon Machine Image) : Ubuntu Server SSD Volumne Type(프리 티어)
    3. 인스턴스 유형 : t2.micro => 추후 데이터량에 따라 변경해야 함
    4. 키 페어 생성
    ```bash
    mkdir ~/.ssh/
    mv "키페어 파일" ~/.ssh/
    ```

2. EC2 서버에 원격으로 접속하기
    1. `ssh -i [키페어 경로] [유저 이름]@[퍼블릭 DNS 주소]`
        * init user name : ununtu   
        * public DNS Address : 생성된 인스턴스에서 확인 가능
        

3. EC2 기본 세팅
    1. `sudo apt-get update` :  패키지 정보 업데이트
    2. `sudo apt-get dist-upgrade` : 패키지 의존성 검사 및 업그레이드
    3. `sudo apt-get install python3-pip`

4. Github
    > 파일 업로드 & 버젼 관리를 위해 github를 이용해 Clone 함
    1. `sudo chown -R ubuntu:ununtu /srv/` : 클론을 위해 폴더 소유자 변경
    2. `cd /srv`
    3. `git clone [레포지토리 주소]`
