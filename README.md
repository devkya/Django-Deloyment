# Django 배포하기
`Nginx`, `Gunicorn`, `Amazon EC2` +`Amazon RDS(MySQL)` 을 사용하여 배포하기


## Amazon EC2
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


## pyenv & pipenv
> Ubuntu에서 pyenv 패키지를 활용하여 python 버젼 관리
1. `pyenv install [version]` : 해당하는 파이썬 버전 install
2. `pyenv local [installed version]` 버젼 변경
3. `pipenv install` : 프로젝트에 생성되어있는 pipfile install
4. `pipenv shell` : pipenv 가상환경 실행


## Gunicorn
1. `pipenv install gunicorn` : 가상환경에 gunicorn 설치
2. `vi /etc/systemd/system/gunicorn.service` : 서비스 등록(서버가 재시작될때 Gunicorn도 실햄되도록 설정)
    ```
    [UNIT]
    Description=gunicron daemon
    After=network.target

    [Service]
    User=ubuntu
    Group=ununtu
    WorkingDirectory=/srv/Django-Deloyment/backend
    ExecStart=/home/ubuntu/.local/share/virtualenvs/Django-Deloyment-7ElGeabW/gunicorn --bind 0.0.0.0:8000 backend.wsgi:application

    [Install]
    WantedBy=multi-user.target
    ```
3. `sudo systemctl daemon-reload` : 시스템 데몬 재시작
    ```
    sudo systemctl start gunicorn # 서비스 실행하기
    sudo systemctl enable gunicorn # 서버 재시작시 자동으로 실행
    sudo systemctl status gunicorn.service # 실행한 서비스 상태 보기

    sudo systemctl stop gunicorn # 서비스 중지
    sudo systemctl restart gunicorn # 서비스 재시작
    ```

4. error 로그는 `/var/log/syslog`에서 확인 가능

## EC2 보안 그룹
1. 포트를 열기 위해 `보안 그룹` => `Edit indound rules` => Custom TCP 8000 port 추가
2. `settings.py` ALLOWED_HOSTS = ["*"]

## Nginx
1. `sudo apt-get install nginx` : 서버에 nginx 설치
2. `sudo vi /etc/nginx/nginx.conf` : user 변경(ubuntu)
3. `manage.py`가 있는 디렉토리에 `.conf/nginx/[site name].conf` 생성
    ```
    server {
        listen 80;
        server_name *.compute.amazonaws.com;
        charset utf-8;
        client_max_body_size 128M;
 
    location / {
        proxy_pass  http://0.0.0.0:8000;
        }
    }


    ```
4. `sudo cp -f /srv/Django-Deloyment/backend/.confing/nginx/backend.conf /etc/nginx/sites-available/backend.conf` => `git pull` 이후 폴더에 있는 conf 파일 복사
5. `sudo ln /etc/nginx/sites-available/backend.conf  /etc/nginx/sites-enabled/backend.conf` => `sites-enabled`로 링크
6. `sudo service nginx restart`
6. 데몬 새로 고침(daemon-reload) 후 `gunicorn` 다시 실행

## STATIC & MEDIA
1. `STATIC_ROOT = os.path.join(BASE_DIR, 'static')` : `settings.py` 추가
2. `python manage.py collecstatic`
3. `backend.conf` 수정


## Domain 연결하기

## Https 적용하기

## Amazon RDS
1. [Amazon RDS](https://ap-northeast-2.console.aws.amazon.com/rds/home) 접속
2. 데이터베이스 생성
3. Django 연동
    * `pipenv install mysqlclient` => python 3.8 에러 발생 => python 3.10 정상 설치
