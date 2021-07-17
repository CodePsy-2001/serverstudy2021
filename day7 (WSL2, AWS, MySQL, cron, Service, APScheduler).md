# day7 (WSL2, AWS, MySQL, cron, Service, APScheduler)

예전에 아나콘다 + 파이썬(유저3.7) + 파이썬(유저3.8, 사용자 지정 경로) + 파이썬(루트3.8) 가 동시에 설치된데다 환경변수까지 엉망진창인 상태를 정리하느라 개삽질을 해본 경험이 도움이 많이 되었다. 운영체제와 파일구조에 관한 기본지식을 쌓은 상태라서 배우는 게 그리 어렵지 않았다.

리눅스 재밌다. 컴퓨터공학의 기초부터 다시 쌓아올라가는 느낌이다.



### WSL2

- WSL2는 가상 네트워크에 리눅스를 하나 돌리는 방식이다. 파일 탐색기로 \\\\wsl$ 폴더에 접근하면 사용할 수 있다.
- /home/사용자명/ 폴더는 터미널 좌측에 path가 안 뜬다. 
  + /mnt/ 폴더에서 윈도우에 등록된 볼륨(c, d 등...)에 접근할 수 있다.

- 윈도우와 리눅스 파일시스템은 서로 권한이 다르다. 우선 윈도우 파일탐색기를 통해 리눅스 내부에 파일을 넣어준 다음, chmod 400 [파일명] 명령어를 사용해 리눅스에서도 그 파일을 사용가능하게 설정할 수 있다.
  + chmod 는 본래 파일 권한관리 명령어다. 4: 읽기, 2: 쓰기, 1: 실행하기
  + 400 = 4/0/0 (나에게만 읽기 권한)
  + 600 = 6/0/0 = 4+2/0/0 (나에게만 읽기, 쓰기 권한)
  + 741 = 7/4/1 = 4+3+1/4/1 (나는 읽기쓰기실행, 그룹은 읽기만, 전체는 실행만)
  + pem 파일은 내용을 수정할 필요가 없으므로 읽기 권한만 설정해주면 된다.



### AWS EC2 적응

- pem 파일만 있으면 손쉽게 AWS EC2의 파일시스템에 접근할 수 있다.
  + ssh -i "codepsy-aws-key.pem" ubuntu@ec2-3-35-136-145.ap-northeast-2.compute.amazonaws.com
- aws ec2 ubuntu의 기본 사용자명은 [ubuntu] 이다.

- scp 명령을 사용해 파일을 주고받을 수 있다.
  + 하지만 프로젝트 하나를 주고받을 때는 보통 git을 사용하는 게 훨씬 더 편하다. 단, github 비밀 리포지토리는 유료 사용자만 생성 가능하다.

- flask 서버를 AWS에서 연 채로 내 wsl 터미널을 종료했다면, 다시 AWS에 접속해 실행중인 flask task를 kill해줘야 한다.



### MySQL

- SQL은 DB를 다룰 수 있는 언어고, MySQL은 SQL을 다루는 기능을 제공하는 소프트웨어다.
  + SQL 기본 기능만 사용할 거라면, 사실 flask-sqlalchemy API를 사용해도 큰 문제는 없다. 공부하다보니 느끼는 건데 정말 별 차이 없는 것 같다. MySQL 그 자체가 중요한 게 아니라, SQL DB의 C/R/U/D를 cmd창에서 자유자재로 다룰 수 있는 게 중요한 거 아닐까?

- DB도 서버와 클라이언트가 나뉜다.
  + 보안을 위해 사용자(클라이언트) 권한 등을 나누기도 한다.

- cheatsheet 라고 검색하면, MySQL **CREATE TABLE**에 필요한 모든 속성을 한방에 알아볼 수 있다.
  + [MySQL cheatsheet (devhints.io)](https://devhints.io/mysql)

- MySQL -> databases(use로 선택) -> tables

- desc (이름); 명령어로 해당 개체의 속성을 쉽게 알아볼 수 있다. (show는 해당 목록을 보여준다)

- topic 이라는 이름의 table을 생성했을 때:

  + __INSERT INTO__ topic (title,description,created,author,profile) __VALUES__('MySQL','I Love MySQL', now(),'CodePsy-2001','developer');

  + __SELECT__ * from topic; (* 은 전체를 불러온다는 뜻)
  + select id, title, description from topic; (특정 column만 가져오기)
  + select title from topic __WHERE__ author='bsw0716'; (조건 붙이기)
  + select title form topic where author='CodePsy-2001'; __ORDER BY__ id DESC; (역정렬하기)
  + select  * from topic __LIMIT__ 2; (2개만 불러오기)
  + __UPDATE__ topic __SET__ description='brand new description', title='new title'; (table 내용을 모두 바꿔버림. 취소도 안됨!)
  + __update__ topic __set__ description='brand new description', title='new title' __WHERE__ id=2; (where에서 지정해준 조건만 바꿈. __UPDATE를 할 때마다 WHERE문이 빠지진 않았는지 손가락으로 가리키며 입으로 읊는 습관을 만들자.__ 지적확인이라 한다.)
  + __DELETE__ from topic WHERE id=5; (update랑 사용방법이 거의 똑같다.)

- 테이블 분리하기
  + **RENAME TABLE** topic **TO** topic_backup; (일단 백업하고)
  + CREATE TABLE 로 DB model을 2개 생성해서
  + JOIN 하면 된다 (관계형 데이터베이스 생성)

- JOIN
  + SELECT * FROM topic **LEFT JOIN** author **ON** topic.author_id = author.id;
  + (ON 뒤에 오는 조건에 맞춰서 LEFT JOIN 결과를 반환)
  + JOIN의 종류: [Joins in MySQL | Learn Top 6 Most Useful Types of Joins in MySQL (educba.com)](https://www.educba.com/joins-in-mysql/) (이중 Full Join은 MySQL이 지원하지 않는다.)

- 기타
  + EXPLAIN - 명령 맨 앞에 달아주면, 해당 명령이 실행할 결과를 예측해서 보여준다.
    [MySQL Explain 실행계획 사용법 및 분석 - Useful Guide (nomadlee.com)](https://nomadlee.com/mysql-explain-sql/)
  + CREATE VIEW - SQL 쿼리의 결과값을 임시 테이블로 저장해서 사용할 수 있다.
    사용이 끝나면 DROP VIEW를 통해 명시적으로 삭제해줘야 한다.
  + JOIN은 집합이다. **for if 문이 아니다! (중요)**
    그래서 정규표현식처럼 이해하는 게 더 좋다.
  + DB 다루기에 필요한 가장 최소한의 명령어만 공부했지만, MySQL 명령어는 생각보다 아주 많다. 책을 하나 사서 공부해보면 좋을 것 같다.



### Cron

- $ crontab {-e(추가) | -l(확인) | -r(삭제)} 
  + (* * * * * /root/backup.sh
    매분, 매시간, 매일, 매월, 매요일 /root/backup.sh 실행
  + (0 4 * * 1-5 /root/backup.sh
    4시마다, 월~금요일에만 실행
  + (*/10 4 * * * /root/backup.sh
    10분 간격, 4시에만 실행
  + 예제) 원펫 지표 분석기를 격일 오전 4시마다 실행하려면?
    $ crontab -e
    (0 4 */2 * * /경로/지표분석기.확장자

- 직접 /etc/crontab 에 접근해 수정해서 사용하면 제대로 작동하지 않을 때가 많다. 실제로는 /var/spool/cron/crontabs/ 폴더에 crontab 파일의 수정사항이 즉각적으로 반영되어야 하는데, 이 작업에 sudo 권한이 필요하기 때문이다.
- #로 주석을 달 수 있다.
- crontab 명령 등록 후 반드시 cron(1분마다 예약된 작업 확인하는 프로세스)을 재실행해야 한다.
  + service cron restart
  + 그냥 service cron start 하면 기존 명령 리스트를 그대로 들고 다시 시작한다.



### cron에 파일 넣기

- $ vi script.sh 등으로 파일 생성 (.sh 는 윈도우에서의 .cmd 와 비슷)
- $ chmod +x script.sh 등으로 실행 권한 변경 (x: execute)
- 그냥 쉘에서 인터프리터와 파일명을 넣으면 자동으로 실행함
  + crontab 으로 등록한 파일에 어떤 일을 할지 써넣으면 된다.
    $ python3 getDAU.py
  + 파일에 shebang을 써줬으면 인터프리터를 굳이 골라줄 필요가 없다.
    $ getDAU.py



### Service (.service file)

- 운영체제 부팅과 동시에 백그라운드에서 실행되는 작업

- sudo 권한으로 /etc/systemd/system/ 위치에 원하는 service file 생성



[Unit]

Description= 이 유닛에 대한 설명

Requires/Wants/BindsTo/PartsOf/Conflicts= 이 유닛에 대해 의존성, 상호성, 또는 배타적 관계 설정

Before/After= 이 유닛이 실행되기 전/후에 실행하고 후/전에 종료하게 됨

[Service]

ExecStart= 실행할 파일이 위치한 전체 경로

WorkingDirectory= 파일의 작업이 이뤄질 경로 (지정 안하면 루트경로에서 실행함)

Restart=[no|on-success|on-failure|on-watchdog|on-abort|always] (서비스 자동 재시작 옵션)



예) AWS Freetier에서 서버를 호스팅할 경우, 서버가 재부팅될때마다 해줘야 하는 작업을 .service에 등록해줘야 한다. 네트워크 상태 체크하고, 오류 검사하고, flask 실행하고, 필요한 곳에 권한 부여하고, 등등등...



- sudo systemctl start 서비스명 (서비스 시작)
  sudo systemctl status 서비스명 (서비스 상태 체크)
  sudo systemctl enable 서비스명 (서비스 등록, 이제 부팅때마다 실행됨)
- service에 후술할 APScheduler file을 실행시키게 만들면 cron을 사용하는 것과 동일한 효과를 얻으면서, 프로그램 개발은 더욱 쉽게 할 수 있음.



### APScheduler

공식 도큐먼트: [Advanced Python Scheduler — APScheduler 3.7.0 documentation](https://apscheduler.readthedocs.io/en/latest/#)

- 프로그램 내부 루프가 아닌, 굳이 스케줄러가 필요한 이유는?
  코드의 실행속도에 영향을 받지 않고, 꾸준히 일정한 시간마다 특정 명령을 실행시키기 위해.
- 운영체제 자체 스케줄러가 아닌, 굳이 인터프리터로 작동하는 스케줄러가 필요한 이유는?
  각 운영체제별로 서비스를 따로 등록시키면 프로그램의 이식성이 낮아짐. 프로그램 자체에 그 프로그램이 언제 실행되어야 하는지에 대한 정보가 같이 등록되어 따라가는 게 이상적임.
- 사용방법 자체는 flask에 비해 그렇게 어렵지 않아 보이는데(전체적인 작성 방식이 flask랑 유사함), 우선 logging을 배운 다음 APScheduler를 적용하는 게 맞는 것 같음. 필요한 만큼만 학습해서 사용하면 될 것 같음.
- **결론:** Service -> Cron, APScheduler -> Flask 등 인터프리터 프로그램 -> 서버 작동
  뭔가 사소한 오류가 발생하면 프로그램 단에서 로그에 기록하고
  중대한 오류가 발생해서 인터프리터가 아예 꺼져버리면 Service와 APScheduler로 문제해결
  정상작동 중일 때도 꾸준히 status 체크해줘야 함 (갑자기 유저가 몰리지는 않나 등)
- 서비스란? 최악의 상황에도 꾸준히 고객에게 제공되어야만 하는 것. 실시간으로 살아있는 서비스를 만들고 유지하려면 그에 관련한 지식이 필요함.

