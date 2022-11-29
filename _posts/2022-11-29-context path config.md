---
author: author2
layout: post
title: context path를 결정하는 3가지 방법
category: middleware
tag: was
image: 
  path: /assets/img/blog/context-path.png
description: >
  WAS라고 부르는 java web application server에 web application을 배포하면 context path 라고하는 경로가 생성된다. 이 문서는 이를 결정하는 3
  가지 요소에 대해 정리하려 한다.
sitemap: false
---

구현체 마다 다르지만 Web application server(이하 server)에는 복수의 host를 구성할 수 있으며 하나의 host내에는 복수의 web application을 배포
할 수 있다. web application은 Servlet API상에서는 context라는 개념으로 대입된다. (Web application == context) 하나의 host내에서 각각의
context를 구분하기 위한 방법으로 context path를 부여 받는다. 다음 두 가지 url을 예로 설명한다.

**_context path_**
> http://www.dalcomlab.com/admin/Users.jsp
> http://www.dalcomlab.com/document/ServletSpec.jsp

www.dalcomlab.com host에는 admin이라는 web application과 document라는 web application이 배포되어 있음을 가정한다. 이 url중 "/admin"
그리고 "/document"에 해당하는 부분 경로가 context path다. 이 경로는 Servlet API의 `ServletContext#getContextPath`를 이용하여 얻을 수 있
다. 이 글은 이 context 경로를 지정하는 3가지 방법을 설명하고자 한다.


### 배포 파일이름으로 context path를 결정: 가장 쉬운 방법

각 host(어떤 구현체는 domain이라고도 부름)마다 web application을 배포하는 기본 디렉터리가 있다. (여기에 배포하기 싫으면 이 방법은 못쓴다.) 여기에 
웹 어플리케이션 아카이브 파일(확장자 war)이나 웹 어플리케이션 디렉터리를 복사하고 server 시작 시키면 확장자를 제외한 파일 이름과 동일한 context path가
부여 된다. 

**_참고> 구현체 별 web application 기본 배포 디렉터리_**

> FlexusAP, TOMCAT, jetty는 server 디렉터리 하위에 webapps
> glassfish는 server 디렉터리 하위에 glassfish/domains/domain1/autodeploy

### web application 배포 서술자(deployment descripter)에 default-context-path 지정

web application의 배포 서술자 즉 물리적으로 web.xml의 최상이 element는 web-app로 시작한다. web application에 대한 모든 배포 정보를 기술할 수
있음을 의미한다. web-app 하위 요소로 default-context-path를 지정 할 수 있는데 이 값이 존재하고 다른 방법으로 context path가 설정되지 않았다면 
이 값으로 context path를 부여한다. 여기서 다른 방법이란 다음에 설명할 <u>context config를 이용한 방법</u>이다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <display-name>EXAM-CONTEXT-PATH</display-name>

    <default-context-path>/context-path-test</default-context-path>

    <!-- 중략 -->
    
</web-app>
```

**_참고> TOMCAT default-context-path 동작하지 않음_**

> 필자가 TOMCAT 9.0.62를 대상으로 테스트 해본 결과 web.xml에 해당 설정을 했지만 설정 값이 아닌 배포한 파일 이름으로 context path가 설정되었다.

### context config를 이용한 context path설정

context config는 web application 상위 설정이다. 다시 말해 Servlet spec을 벗어나는 설정이며 각 vender별로 각각의 설정 방식이 존재한다. 대부분의
구현체는 context 설정에 web application가 배포된 물리적인 위치(디렉터리)와 더불어 context path를 필수 설정으로 요구한다. 이 값을 설정하게 되면 앞
에서 설명한 두 항목에 대한 설정은 무의미 하게된다.

**FlexusAP의 context.conf 파일 context path 설정**

아래 설정 파일은 FlexusAP의 context.conf context path 설정 예시다. 이 설정은 server.conf에 host 설정 하위에도 구성할 수 있다. (context 
설정 요소의 이름으로 지정)

```json
context["/context-path-test"] : {
    # 중략
}
```

**TOMCAT의 context.xml 파일 context path 설정**

아래 설정 파일은 FlexusAP의 context.xml context path 설정 예시다. 이 설정은 server.xml에 host 설정 하위에도 구성할 수 있다. (Context 
element의 path 속성)

```xml
<Context docBase="contextPathTest" path="/context-path-test" reloadable="true">
    <!-- 중략 -->
</Context>
````

