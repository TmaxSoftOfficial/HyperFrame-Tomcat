## HyperFrameOE-Tomcat

업로드된 바이너리는 HyperFrame Open Edition Tomcat 제품 설치를 위한 파일

## 설치 파일

### Tomcat

* Version : apache-tomcat-9.0.52
* Note : https://tomcat.apache.org/download-90.cgi

### Tomcat-Connectors

* Version : tomcat-connectors-1.2.48
* Note : https://tomcat.apache.org/download-connectors.cgi

## 설치 및 실행

### 1) Tomcat 압축 풀기

    $ tar -zxf apache-tomcat-9.0.52.tar.gz

### 2) Port 확인 및 변경

    $ vi ${TOMCAT_HOME}/conf/server.xml
    ...
    <Service name="Catalina">
        <Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    ...
    </Service>

### 3) Tomcat 실행

    $ cd ${TOMCAT_HOME}/bin/
    $ ./startup.sh
    
### 4) Tomcat 종료

    $ cd ${TOMCAT_HOME}/bin/
    $ ./shutdown.sh

## 디렉토리 구조

    $ cd /home/username/apache-tomcat/
        apache-tomcat
        ├── bin
        │    ├── bootstrap.jar       <---------------- tomcat 서버가 구동될 때 사용되는 main( ) 메소드 포함
        │    ├── tomcat-juil.jar
        │    ├── common-daemon.jar
        │    ├── catalina.sh         <---------------- CATALINA 서버의 제어 스크립트
        │    ├── ciphers.sh
        │    ├── configtest.sh       <---------------- CATALINA 서버의 설정 스크립트
        │    ├── daemon.sh
        │    ├── makebase.sh
        │    ├── setclasspath.sh     <---------------- JAVA_HOME 또는 JRE_HOME이 세팅되지 않았을 경우 세팅
        │    ├── shutdown.sh         <---------------- CATALINA 서버를 중지하는 스크립트
        │    ├── startup.sh          <---------------- CATALINA 서버를 시작하는 스크립트
        │    ├── tool-wrapper.sh
        │    └── version.sht
        ├── conf
        │    ├── catalina.policy
        │    ├── catalina.properties	
        │    ├── context.xml         <---------------- 세션, 쿠키 저장 경로등을 지정하는 설정 파일
        │    ├── logging.properties	
        │    ├── server.xml          <---------------- Tomcat 설정에서 가장 중요, Service, Connertor 등과 같은 주요 기능 설정 가능
        │    ├── tomcat-users.xml    <---------------- Tomcat의 manager 기능을 사용하기 위해 사용자 권한을 설정	
        │    └── web.xml             <---------------- Tomcat의 환경설정 파일	   
        └── common


## 로그 정보

### 1) 로그 경로

    # 서버상에서 발생한 모든 내용을 기록한 Log
    $ {TOMCAT_HOME}/logs/catalina.out
    
    # Tomcat에서 발생하는 Log
    $ {TOMCAT_HOME}/logs/catalina.yyyy-mm-dd.log
    
    # Tomcat Manager Web App Log
    $ {TOMCAT_HOME}/logs/manager.log 

## 환경 설정 파일 정보

### 1) 환경 설정 파일 경로

    $ {TOMCAT_HOME}/conf/server.xml
    
### 2) 환경 설정

* Port 변경
    
    $ vi ${TOMCAT_HOME}/conf/server.xml
    ...
    <Service name="Catalina">
        <Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    ...
    </Service>

* 프로토콜 별 Connector 추가 및 삭제


    ...
    <Service name="Catalina">
        # http Connector
        <Connector port="8080" protocol="HTTP/1.1" 
                   connectionTimeout="20000" 
                   redirectPort="8443" />
        # https Connector
        <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />
        # ajp Connector
        <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    ...
    </Service>
    ...

* Web Application 추가
    
    ...
    <Host name="localhost" appBase="webapps"
          unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        # https:localhost:Port/sample/, docBase = Page Directory 
        <Context path = "/sample" docBase = "/home/minjun/tomcat/server1/sample/" />
    </Host>
    ...











## 버전 확인

${TOMCAT_HOME}/home/username/apache-tomcat/lib > java -cp catalina.jar org.apache.catalina.util.ServerInfo

## 웹서버와 연동방법

### Apache 웹서버

#### 1. Apache와 Tomcat을 연동하기 위해서는 mod-jdk 플러그인이 필요
#### 2. 첨부한 tomcat-connectiors-1.2.43.src.tar.gz를 다운로드 받고 아래와 같은 명령어로 압축해제

      tar xvfz tomcat-connectors-1.2.43-src.tar.gz
      cd tomcat-connectors-1.2.43-src/native

#### 3. 위의 native 폴더에서 아래의 명령어로 configure를 적용

      ./configure --with-apxs=${APACHE_HOME}/bin/apxs
      
      make
      
      make install

make를 하고나서 ${APACHE_HOME}/modules 폴더에 mkd_jk.so 파일이 생성되었는지 확인

#### 4. Tomcat에서 AJP 프로토콜 확인

${TOMCAT_HOME}/conf의 server.xml에서 아래의 설정을 확인. 주석처리 되어있다면 주석해제

     <Connector protocol="AJP/1.3"
                    address="::1"
                    port="8009"
                    redirectPort="8443" />


#### 5. Apache의 worker 설정

Apache와 Tomcat의 포트를 연결하는 환경설정 파일을 작성 (${APACHE_HOME}/conf/workers.properties 파일을 생성)

<pre>
      worker.list=worker1

      worker.worker1.type=ajp13		      # AJP1.3 프로토콜을 사용
      worker.worker1.host=localhost	      # 톰캣은 local에서 돌고 있습니다.
      worker.worker1.port=8009	  	      # 연결할 톰캣의 포트 번호
</pre>

#### 6. Apache와 Tomcat 연동

${APACHE_HOME}/conf에서 httpd.conf 파일에 아래의 내용을 추가

<pre>
      LoadModule jk_module            modules/mod_jk.so
      <IfModule mod_jk.c>
              JkWorkersFile /usr/local/victolee/apache2.0.64/conf/workers.properties    # 실행파일
              JkLogFile /usr/local/victolee/apache2.0.64/logs/mod_jk.log                # 로그 경로
              JkLogLevel info                                                           # 로그레벨 설정
              JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"                                 # 로그 포맷
              JkShmFile /usr/local/victolee/apache2.0.64/logs/mod_jk.shm                # 공유파일
              JkMount /*.jsp worker1                                                    # /*.jsp 파일은 worker1에게 넘긴다         
      </IfModule>

</pre>
----------------------------------------

### Nginx 웹서버
${TOMCAT_HOME}/conf의 nginx.conf에서 아래의 설정을 확인.

<pre>
         server {
             listen       80;                            #Nginx Port
             server_name localhost;
            location / {
                root   html;
                index  index.html index.htm;
                proxy_pass http://127.0.0.1:8080;        #Tomcat Port
            }

</pre>

위의 proxy_pass http://127.0.0.1:8080; 부분과 같이 Tomcat의 Port 번호로 추가 해야 함

## 버전이력

apache-tomcat-9.0.52
