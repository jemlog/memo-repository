### 배포시에 반드시 설정 해줘야 하는 요소 

```javascript
if(process.env.NODE_ENV === 'production')
{
  app.use(morgan('combined'))  
  app.use(helmet({contentSecurityPolicy : false}))
  app.use(hpp())
}
else
{
  app.use(morgan('dev')); 
}

"start": "cross-env NODE_ENV=production PORT=80 pm2 start app.js -i 0",  // nginx 설치시 PORT=80은 불필요

```


### AWS 배포 방법 
✔ 배포하는 법 
```typescript
- sudo su     // 관리자 계정으로 전환, 경험상 꼭 필요하지는 않아보인다 
- sudo apt-get update  // 우분투의 시스템을 최신으로 업데이트 시켜주는 단계 
- sudo apt-get install -y build-essential  // nodejs를 설치하기전에 기본적으로 세팅해야 하는 것들 
- sudo apt-get install curl   

// 최근 경험상 우분투 18. 버전에서는 node 8버전이 깔리고 
   우분투 20. 버전에서는 가장 최신 node를 다운받으면 10버전이 깔린다 
   타입스크립트로 전환 후 라이브러리들 설치가 많아져서 8버전의 노드로는 
   활성화 시킬수 없는 것들이 매우 많다. 
   ✅ 'nodesource'사이트 들어가서 무조건 16. 버전 이상으로 받자 
- curl -fesL https://deb.nodesource.com/setup_16.x | sudo -E bash -- 
- sudo apt-get install -y nodejs
- 중요!! pm2는 꼭 sudo로 전역 설치를 해주자..! 
```


### 🐘 EC2에 Postgresql 설치하고 사용
```typescript
/// postgresql 설치
sudo apt-get install postgresql postgresql-contrib  // postgresql-contrib는 postgresql을 위한 확장판 

// 제일 처음 생성된 유저의 비밀번호를 내 DB (.env) 비밀번호랑 매칭 시켜주기 
sudo passwd postgres

// postgres 접속 
sudo su postgres

// 설정 파일 변경 
sudo vi /etc/postgresql/12/main/postgresql.conf
=> listen_addresses를 0.0.0.0 으로 변경해주자 , 모두가 접속 가능하다는 뜻이다 

// 설정 파일 변경2
sudo vi /etc/postgresql/12/main/pg_hba.conf
=> IPv4 local connection에서 127.0.0.1/32 잡혀있는걸 0.0.0.0/0 으로 변경 

// CONNECTIONS AND AUTHENTICATION의 listen_address = '*'으로 변경 
sudo su - postgres

psql -U postgres    

// 중요! 반드시 내가 .env에 설정한 것과 같아야한다.
alter user postgres password '원하는 비밀번호';


// 재시작
sudo service postgresql restart  || sudo systemctl restart postgresql

```
remove anonymous users 부터해서 
y n y y 

mysql -uroot -p  // 비밀번호 설정 나오면 로컬이랑 같도록!


https://github.com/jemlog/book_review_backend.git

폴더 들어가서 vim .env로 작성 
종료 후에는 cat .env 해줌 
그 뒤에 sudo npm i 해줌 

중요 여기서 설정하는 비밀번호와 원래 비밀번호 같아야함 

mysql -uroot -p 해서 들어간 후에 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '내 비밀번호'

npx sequelize db:create --env production

sudo npm start && sudo npx pm2 monit 

### 🐳 Docker mysql 실행 
- docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7
- ❗ 만약 문제가 생긴다면 꼭 -d 옵션을 때고 실행해보자 
- docker exec -it mysql mysql 
- create database 원하는 이름 CHARACTER SET utf8;
- grant all privileges on 원하는이름.* to 원하는이름@'%' identified by '원하는이름';
- flush privileges;
- quit
### 🎈 NGINX 설정
- sudo apt-get update && sudo apt-get install nginx
- nginx 실행 안되면 sudo service nginx start로 확인 
- /etc/nginx: 해당 디렉터리는 Nginx를 설정하는 디렉터리입니다.모든 설정을 이 디렉터리 안에서 합니다.
- /etc/nginx/nginx.conf: Ngnix의 메인 설정 파일로 Nginx의 글로벌 설정을 수정 할 수 있습니다.

- /etc/nginx/sites-available: 해당 디렉터리에서 프록시 설정 및 어떻게 요청을 처리해야 할지에 대해 설정 할 수 있습니다.

- /etc/nginx/sites-enabled: 해당 디렉터리는 sites-available 디렉터리에서 연결된 파일들이 존재하는 곳 입니다.이 곳에 디렉터리와 연결이 되어 있어야 nginx가 프록시 설정을 적용합니다.

- /etc/nginx/snippets: sites-available 디렉터리에 있는 파일들에 공통적으로 포함될 수 있는 설정들을 정의할 수 있는 디렉터리 입니다.

- /etc/nginx/sties-available 로 이동 
- sudo vi node-server(이름은 내 마음대로 가능)
```shell
server{
 listen 80;
 server_name '내 lightsail 서버 주소';
 location / {
    proxy_pass http://127.0.0.1:3001;
 }
}
```
- sudo ln -s /etc/nginx/sites-available/(내가 설정한 이름) /etc/nginx/sites-enabled
- ls -al로 확인
- sudo service nginx restart

// pull 안될때 사용하는 방법
# git stash && git pull origin master && git stash pop

