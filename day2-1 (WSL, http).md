# day2-1 (WSL2, http)

시작시간, 종료시간이 명확하지 않을 수 있습니다.





### WSL2 적용 (오후 1:00 ~ 오후 2:00)

듀얼부팅보다 WSL 기술이 훨씬 낫다는 조언을 들었다. C드라이브 내에 리눅스 파일구조를 마운트하고, 우분투 터미널을 사용할 수 있도록 세팅할 수 있다. 이미 SSD와 램을 사버렸는데, 어차피 나중에 듀얼부팅도 필요해지지 않을까... 하는 생각이 든다.

GUI가 없으니 적응하기 조금 어렵다. 왜 기본 명령어나 vi 등에 익숙해져야 한다고 말해주신지 알겠다. 시스템 서비스 등록도, Python 가상환경 만드는 법도, SQL DB 다루는 법도 기본은 다 알고 있는데, GUI가 사라지니까 진입장벽이 갑자기 높아진 느낌이다.

앞으로 해야 할 일:

1. 기본 명령어 및 vi에 익숙해지기
2. sudo apt로 venv/nginx 등 라이브러리 다운로드
3. python가상환경에서 gunicorn을 통한 서버 활성화(worker 등에 대한 이해)
4. systemctl 등록: service 파일을 등록하고 컴퓨터 재부팅시 자동시작되도록 등록
5. nginx 기본 설정(server_name, 포트 등에 대한 이해)
6. mysql 설치 및 기본 내용(사용자 추가, 권한 부여/제거, DB/테이블 생성, SELECT/UPDATE/DELETE/INSERT에 대한 내용)





### HTTP 기초 (오후 4:00 ~ 오후 5:30)

이 부분은 이론적 이해가 중요하다. 공부한 내용을 간략히 요약해 적어본다.

- http는 서버/클라이언트 모델을 따른다

  Client가 '요청'을 보내는 경우에만 Server가 '응답'하는 단방향적 통신이다. 서버가 클라이언트에 요청을 보낼 수는 없다. (WSGI도 클라이언트가 서버에 특정 요청을 보내는 형태다. '이 내용의 댓글을 추가해주세요~'. 이후 보통 리다이렉트로 새로고침해준다.)

  실시간 연결이 아니고, 클라이언트가 요청을 보낸 뒤 서버가 응답을 보내면 그 즉시 연결이 종료된다. (그래서 웹 파일을 덤프해서 보내면 서버가 꺼져도 사용자가 웹앱을 이용할 수 있다.)

  + Socket 통신은 서버와 클라이언트가 특정 포트를 통해 실시간 통신한다. http와 달리 실시간 스트리밍 서비스, 온라인 멀티플레이 게임 등에 적합하다.

    

- http는 무상태(stateless)이다

  클라이언트로부터 전달받은 요청을 응답한 뒤에는 연결을 완전히 끊어버린다. 때문에, 서버는 통상적으로 클라이언트의 이전 상태를 알 수 없다. 이러한 특징을 무상태(stateless)라고 말한다.

  특수한 정보를 유지하기 위해 Cookie를 저장할 수도 있다. (쿠키클리커 같은 idle류 웹게임은 사용자의 현재 상태를 웹브라우저 쿠키에 저장해둔다.)

  + 쿠키는 얼마든지 참조, 변조될 수 있다. 사용자의 개인정보과 같은 중요한 정보는 아예 저장하지 말거나, 꼭 필요한 경우 암호화해두어야 한다.

    

- URL vs URI

  URL : 웹상에서 파일이 존재하는 특정 위치 (html, jpg 등)

  URI : 서버에 전달해주는 자원 식별자 (서버 내부에서 이를 처리해 특정 파일을 보여준다)

  URI의 일반적인 형태

  scheme:[// [user [:password] @] host [:port] ]   [/path] [?query] [#fragement]

  예) [https://www.youtube.com/watch?v=8hFzH3gN7O4] (오토나시 코토리 - 하늘)
  
  https - https 프로토콜(scheme)을 사용해
  
  www.youtube.com - YouTube의 도메인 주소(host)에서
  
  /watch - watch라는 접근 대상 중에서
  
  ?v=8hFzH3gN7O4 - v(video의 약자로 추정) 중에서 8hFzH3gN7O4 라는 id값을 가진 영상을 볼 수 있도록 해주세요 __(요청)__
  
  ▶ WSGI 등의 기술을 활용해 html 본문을 만들어서 클라이언트에게 __(응답)__
  
  
  
- Proxy

  대리인. 웹 브라우저에서 원하는 주소를 입력해 요청을 보냈을 때, 내 IP 주소를 가지고 바로 해당 웹서버로 접속하지 않고, __프록시 서버__로 요청을 전송한다. 프록시 서버는 대리인을 데리고 웹서버에 접속해 데이터를 주고받은 뒤, 다시 프록시 서버를 통해 나의 웹브라우저에게 응답을 보낸다.

  + 프록시 서버에 캐시를 저장해 인터넷 통신 속도를 높일 수 있다.
  + 웹서버는 프록시 서버의 주소만을 알 수 있으므로, 클라이언트의 익명성을 보장해준다. (해외 IP를 차단해둔 서버에도 프록시를 사용해 접근할 수 있게 된다.)
  + 프록시는 아니지만, 공유기를 사용해 인터넷 보안을 향상시킬 수 있는 것과도 일맥상통한다. 해커가 공유기의 IP는 알 수 있지만, 내 컴퓨터의 특정 IP는 알 수 없기 때문이다. VPN은 가상 사설망 서비스로, 공개 Proxy에 비해 암호화 서비스 등을 제공한다.

  



### HTTP 프로토콜 이해 (오후 5:40 ~ 오후 6:30)

1. connect
2. request & response (HTTP 요청/응답 메시지를 주고받는다)
3. close




- HTTP 요청 메시지

   시작줄(start line) + 헤더(header) + 바디(body)

   + 시작줄: 요청 메소드 + 요청 목표 + HTTP 버전

     요청 메소드: (GET, POST, PUT, DELETE, HEAD, OPTIONS, TRACE)

     예) GET 뒤에 URI를 집어넣어서 서버에게 정보를 요청한다

     - GET /servlet/query?=a10&b=90 HTTP/1.1 ()

     GET과 HEAD는 원칙적으로 Safe Method여야 하는데, 이를 호출한다고 해서 서버 측의 데이터에 변화가 있어서는 안된다는 뜻이다.

   + 헤더: key와 value로 이루어짐

     공통: Date, Connection, Content-Length, Cache-Control, Content-Type, Content-Language, Content-Encoding 등

     요청: Host, User-Agent(사용자 통계 등에 활용), Accept, Cookie, Origin, If-Modified-Since, Authorization(API 요청에 활용) 등

     - Host: www.sk.com

       User-agent: mozilla/4.0

       Accept-language: kr

   + 바디: 데이터 전송용. POST, PUT, CONNECT, OPTIONS, PATCH 등.

     가장 자주 쓰이는 GET 요청에는 body가 포함되지 않는다.

     

- HTTP 응답 메시지

   마찬가지로, 시작줄(start line) + 헤더(header) + 바디(body)

   + 시작줄: HTTP 버전 + 상태코드 + 상태 텍스트

     200 OK, 400 Bad Request, 403 Forbidden, 404 Not Found, 408 Request Timeout, 500 Internal Server Error, 502 Bad Gateway 등이 있다.
     
     - HTTP/1.0 200 OK
     
   + 헤더: key와 value로 이루어짐

     응답: Server, Access-Control-Allow-Origin, Allow, Content-Disposition, Location(403 에러시 로그인 화면으로 보내기에 활용), Content-Security-Policy 등
     
     - Connection: Keep-Alive
     
       Content-Encoding: gzip
     
       Content-Type: text/html; charset=utf-8
     
       Date: Mon, 18 Hul 2016 16:06:00 GMT
     
       Server: Apache
     
       Set-Cookie: mykey=myvalue; Path=/; secure
   
  + 바디: 마찬가지로, 데이터 전송용. HTML 정보를 보내는 등.
  

