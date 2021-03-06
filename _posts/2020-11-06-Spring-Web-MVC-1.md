---
title: Spring Web MVC 1
author: Jiny
date: 2020-11-06 15:14:00 +0800
categories: [Java, Spring]
tags: [web, springmvc]
toc: false
---

# **Web MVC**

---

## 💿 **Servlet**

> 클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술

- 특징
    - 클라이언트의 요청에 대해 동적으로 작동하는 web-app Component
    - html을 사용하여 요청에 응답
    - Java Thread를 이용하여 동작한다.
    - MVC 패턴에 Controller로 이용된다.
    - Http 서비스를 지원하는 javax.servlet.http.HttpServlet Class를 상속받는다.
    - UDP보다 처리 속도가 느리다.
    - Html 변경 시 Servlet을 재 컴파일해야한느 단점이 있다.

웹서버는 정적인 페이지만을 제공합니다. 따라서 동적인 페이지를 제공하기 위해서는 웹서버는 다른 곳에 도움을 요청하여 동적인 페이지를 작성해야 함
웹서버가 동적인 페이지를 제공할 수 있도록


여기서 도와주는 어플리케이션이 서블릿이며, 동적인 페이지를 생성하는 어플리케이션이 CGI입니다.

___

## 💿 Servlet Container 동작 방식

![image](https://t1.daumcdn.net/cfile/tistory/993A7F335A04179D20)

1. Client가 URL을 입력하면 HTTP Request가 Servlet Container로 전송
2. 요청을 전송받은 Servlet Container는 HttpServletRequest, HttpResponse 객체를 생성
3. web.xml을 기반으로 사용자가 요청한 URL이 어느 서블릿에 대한 요청인지 찾음
4. 해당 서블릿에서 service 메소를 호출한 후 Client의 GET, POST에 따라 doGet() doPost()를 호출
5. doGet() or doPost() 메소드는 동적인 페이지를 생성한 후 HttpServletResponse 객체에 응답을 보냄
6. 응답이 끝나면 HttpServletRequest, HttpServletResponse 두 객체를 소멸

___

## 💿 **Servlet 생명주기**

- 서블릿 컨테이너가 서블릿 인스턴스의 init() 메소드를 호출하여 초기화 한다.
  - 최초 요청을 받았을 때 한번 초기화 하고 나면 그 다음 요청부터는 이 과정을 생략한다.
- 서블릿이 초기화 된 다음부터 클라이언트의 요청을 차리할 수 있다. 각 요청은 별도의 쓰레드로 처리하고 이때 서블릿 인스턴스의 service() 메소드를 호출한다.
  - 이 안에서 HTTP 요청을 받고 Client로 보낼 HTTP 응답을 만든다.
  - service()는 보통 HTTP Method에 따라 doGet(), doPost() 등으로 처리를 위임한다.
  - 따라서 보통 doGet() 또는 doPost()를 구현한다.
- 서블릿 컨테이너 판단에 따라 해당 서블릿을 메모리에서 내려야 할 시점에 destroy()를 호출한다.

___

## 💿 **Servlet Mapping**

![image](https://t1.daumcdn.net/cfile/tistory/9980B8475C66121F35)

위는 배포서술자(Deployment Descriptor)를 사용한 방법
- web.xml 파일을 만들고 web-app 태그 내부에 서블릿을 정의
- 이것들은 서블릿으로 톰캣이 실행하게 됩니다.

하지만 servlet3.0부터 Annotation을 지원하면서 배포서술자 없이도 서블릿 매핑 가능

![image](https://t1.daumcdn.net/cfile/tistory/997687475C66122035)

클래스 위에 어노테이션을 이용해서 URL 매핑을 완료했고 HttpServlet 클래스를 상속받은 클래스를 만들고 여기서는 doGet() 메소드를 오버라이드

그런데 out.println() 으로 HTML 코드를 생성하는 것은 작성하기도 불편하고 가독성이 떨어집니다. 그리고 오류가 발생했을 때나 새로운 요구사항이 추가 되었을 때 수정을 하기도 어려울 수 밖에 없습니다. 자바 코드 안에서 HTML 태그를 넣는것이 아니라 반대로 HTML 안에 자바 코드를 넣을 수 있다면 이런 문제들이 해결될 수 있을 것입니다. 그 역할을 해 주는 것이 바로 JSP 입니다.

___

## 💿 **CGI & WAS**

>  별도로 제작된 웹서버와 프로그램 간의 교환방식. 

- 어떠한 프로그래밍언어로도 구현이 가능하며 별도로 만들어 놓은 프로그램에 HTML의 GET, POST 방식으로 Client의 데이터를 환경변수로 전달하고, 프로그램의 표준 출력 결과를 클라이언트에게 전송하는 것


### **WAS와 CGI**

> 웹서버: 요청에 대한 정적인 리소스를 응답하는 목적으로 만들어진 서버

- Apache, Nginx 등이 있음

> CGI (Common Gate Interface): 웹서버와 WAS 간의 데이터를 주고받는 규칙을 통일할 규약. 이에 따라 만들어진 프로그램을 CGI 프로그램이라 함(옛날에 나옴)

> WAS (Web Application Server): 동적인 처리를 하고 웹서버로 리턴하는 목적으로 만든 서버

- 웹서버와 WAS는 다른 언어, 체계로 만들어졌기 때문에 데이터 교환이 어려움
- Tomcat이나 Zeus등이 있음 

### **WAS의 실행 유형**

1. Interpreter
   - 형태: .php, .pl, .js, .py 등
   - Script 엔진이 필요: 웹서버가 스크립트 엔진을 실행하고 스크립트 엔진이 스크립트를 해석 -> 데이터 리턴
2. Compile
   - 기계어가 최종본, 웹서버가 바로 실행
3. 자바와 CGI 프로그램
   - .class로 된 웹앱: .class는 바이트코드(jvm에 의해 기계어로 번역됨)
   - 웹서버가 웹앱 실행 불가: .class는 jvm에 의해 번역, 실행되어야 함
   - 웹서버가 jvm을? -> jvm은 운영체제에 종속적
   - 그러면 jvm이 실행되고 그 위에서 자바 프로그램 관리서버(Tomcat)을 만들고 실행하자
   - Tomcat: JSP 환경을 포함하고 있는 Servlet 컨테이너(관리서버)

### **따라서**

![image](https://t1.daumcdn.net/cfile/tistory/99C8853359BA7F4810)

1. Client로 부터 웹서버가 요청을 받으면 자바 프로그램 관리 서버(CGI) 실행
2. 자바 프로그램 관리 서버(CGI)는 자바 프로그램(jvm위에서) 실행
3. 리턴받은 값을 자바 프로그램 관리 서버가 결과값을 웹서버로: 데이터 전달해주는 역할 수행


### **서블릿 컨테이너란**

1. 자바 프로그램 관리 서버의 이름
2. 자바 프로그램을 서블릿이라 함: .class, 웹앱
3. 요청이 오면 호출(생성, 실행)뿐 아니라 소멸까지 관리: lifeCycle 관리
  - 스크립트의 경우 ㅇㅇㅇ엔진이라 함
4. 여러 요청: 파나의 프로세스에서 멀티 쓰레드 처리등 app 관리 기능까지
5. 대표적으로 톰캣(아파치 아님), 제우스, 웹로직 등이 있음
6. WAS라소 함: 웹서버로 부터 요청을 전달받고 웹앱(서블릿)을 실행시키고 얻은 결과값을 웹서로 전달해 주는 역할


### **Http를 이용한 Server 와 Client의 통신**

1. 클라이언트는 정보를 얻기 위해 서버로 HTTP 요청을 전송하고, 
2. 서버는 이를 해석하여 정적 자원에 대한 요청일 경우 자원을 반환해주고, 그렇지 않은 경우 CGI 프로그램을 실행시켜 해당 결과를 리턴
3. 이때 서버는 CGI 프로그램에게 요청을 전달해주고, 결과를 전달받기 위한 파이프라인을 연결합니다. 
4. CGI 프로그램은 입력에 대한 서비스를 수행하고, 결과를 클라이언트에게 전달하기 위해 결과 페이지에 해당하는 MIME 타입의 컨텐츠데이터를 웹 서버와 연결된 파이프라인에 출력하여 서버에 전달합니다. 
5. 서버는 파이프라인을 통해 CGI 프로그램에서 출력한 결과 페이지의 데이터를 읽어, HTTP 응답헤더를 생성하여 데이터를 함께 반환해줍니다.

___

##  💿 **Tomcat Container**

###  **Stand-alone servlet containers(Tomcat의 기본 모드)**

- 내장된 웹서버의 기능을 사용하는 것
- 기능면에서 JavaWebServer의 부분인 Serlvet 컨테이너와 Java 근간 웹 서버를 사용

### **In-process servlet containers**

- Servlet 컨테이너는 웹서버 플러그인과 Java 컨테이너 구현
- 웹서버 플러그인은 웹서버의 주소 공간 내에 JVM을 열고 그 안에서 JAVA 컨테이너가 실행되도록 한다.
- 다중 스레드의 단일 프로세스 서버에 적당하고 퍼포먼스도 좋지만 확장성에 한계가 있음

### **Out-of-process servlet containers**

- 웹서버 플러그인과 웹서버의 외부 JVM에서 실행하는 JAVA 컨테이너 구현
- 웹서버 플러그인과 JAVA 컨테이너 JVM은 몇몇 IPC(보통은 TCP/IP 소켓)를 사용해서 통신
- Out-of-process 엔진의 반응 시간은 in-process 만큼 좋지 않지만 out-of-process 엔진은 확장성과 안전성 면은 In-process보다 좋다.

___

## 💿 **Servlet 구조**

![image](https://t1.daumcdn.net/cfile/tistory/998111475C66121E35)

- 서블릿: 
  - 웹서버는 정적인 페이지를 제공하는 역할만 할 수 있기 때문에 실시간으로 작성된 페이지를 제공하거나 서버 상에 데이터를 저장하는 일은 웹서버가 할 수 없음
  - 따라서 웹서버는 요청을 servlet에 요청을 전달하기만 하면 됨

- 컨테이너:
  - 요청이 들어오면 요청을 처리할 새로운 스레드를 만들어야 하고 서블릿에서 필요한 메서드를 호출해야 함
  - 웹서버는 요청을 컨테이너로 보내고 Container는 서블릿을 찾아 필요한 메서드를 호출(doGet(), doPost())