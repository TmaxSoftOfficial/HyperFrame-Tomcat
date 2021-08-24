# HyperFrameOE-Tomcat

- 업로드된 바이너리는 HyperFrame Open Edition Tomcat 제품 설치를 위한 파일임

# 사전작업

- Java 버전 8 이상 권장
- apache-tomcat-9.0.52.tar.gz 파일 필요
- 웹서버 연동을 위해서는 tomcat-connectors-1.2.48-src.tar.gz 파일 필요

# 설치방법

- apache-tomcat-9.0.52.tar.gz 압축 파일을 춤
- ${TOMCAT_HOME}/bin의 startup.sh 파일을 기동
- 웹어드민을 http:// ip:8080 주소로 접속해서 WEB UI 화면을 확인

# 실행/종료 명령어

- 실행: ${TOMCAT_HOME}/bin > ./startup.sh
- 종료: ${TOMCAT_HOME}/bin > ./shutdown.sh 

# 디렉토리 구조

# 로그 위치 

- ${TOMCAT_HOME}/logs/catalina.out : 서버상에서 발생한 모든 내용을 기록한 파일
- ${TOMCAT_HOME}/logs/catalina.yyyy-mm-dd.log : 톰캣에서 생기는 로그만을 기록
- ${TOMCAT_HOME}/logs/manager.log : Tomcat Manager Web App 로그

# 환경 설정 파일

- ${TOMCAT_HOME}/conf/server.xml
- <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" /> 에서 port 변경 가능

# 버전 확인
- 

# 웹서버와 연동방법

- Apache 웹서버

1. Apache와 Tomcat을 연동하기 위해서는 mod-jdk 플러그인이 필요
2. 첨부한 tomcat-connectiors-1.2.43.src.tar.gz를 다운로드 받고 아래와 같은 명령어로 압축해제

tar xvfz tomcat-connectors-1.2.43-src.tar.gz
cd tomcat-connectors-1.2.43-src/native

3. 위의 native 폴더에서 아래의 명령어로 configure를 적용

./configure --with-apxs=${APACHE_HOME}/bin/apxs
make
make install

make를 하고나서 ${APACHE_HOME}/modules 폴더에 mkd_jk.so 파일이 생성되었는지 확인

4. Tomcat에서 AJP 프로토콜 확인

${TOMCAT_HOME}/conf의 server.xml에서 아래의 설정을 확인. 주석처리 되어있다면 주석해제

<Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />

5. Apache의 worker 설정

Apache와 Tomcat의 포트를 연결하는 환경설정 파일을 작성 (${APACHE_HOME}/conf/workers.properties 파일을 생성)


worker.list=worker1

worker.worker1.type=ajp13		  # AJP1.3 프로토콜을 사용
worker.worker1.host=localhost	# 톰캣은 local에서 돌고 있습니다.
worker.worker1.port=8009	  	# 연결할 톰캣의 포트 번호

6. Apache와 Tomcat 연동

${APACHE_HOME}/conf에서 httpd.conf 파일에 아래의 내용을 추가

LoadModule jk_module            modules/mod_jk.so
<IfModule mod_jk.c>
        JkWorkersFile /usr/local/victolee/apache2.0.64/conf/workers.properties	# 실행파일
        JkLogFile /usr/local/victolee/apache2.0.64/logs/mod_jk.log			# 로그 경로
        JkLogLevel info							# 로그레벨 설정
        JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"			# 로그 포맷
        JkShmFile /usr/local/victolee/apache2.0.64/logs/mod_jk.shm		# 공유파일
        JkMount /*.jsp worker1						# /*.jsp 파일은 worker1에게 넘긴다         
</IfModule>




# 라이센스

# 버전이력

apache-tomcat-9.0.52
