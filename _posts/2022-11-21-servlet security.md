---
author: author2
layout: post
title: Servlet security
category: middleware
tag: was
image: 
  path: /assets/img/blog/cyber-security.jpg
description: >
  본 문서는 Servlet specification 4.0에 기술된 Servlet security를 Servlet containter 구현 관점에서 재해석 했다. 인증과 권한 그리고 웹
  리소스의 보안적 제한에 대한 내용을 다룬다. 
sitemap: false
---

웹 어플리케이션은 응용프로그램 개발자에 의해 개발되며 이를 배포자에게 주던, 팔던간에 런타입 환경에 설치며, 응용프로그램 개발자는 보안 요구사항을 배포자와 배포 
시스템에 전달하게 된다. 이 정보는 애플리케이션의 배포 설명자, 애플리케이션 코드 내의 주석을 사용하거나 ServletRegistration.Dynamic 인터페이스의 
setServletSecurity 메서드를 통해 프로그래밍 방식으로 전달될 수 있다.

이 문서는 애플리케이션의 보안 요구 사항을 전달하기 위한 Servlet 컨테이너 보안 메커니즘과 인터페이스와 배포 설명자, 주석 및 프로그래밍 메커니즘에 대해 설명
한다.

[관련 주제]
> Session, Realm

## 서론

웹 애플리케이션에는 많은 사용자가 액세스할 수 있는 리소스가 포함되어 있다. 이러한 자원들은 종종 인터넷과 같은 보호되지 않은 개방형 네트워크를 통과한다. 
그러한 환경에서, 상당수의 웹 애플리케이션은 보안 요구 사항을 가질 것이다. 
상세 구현에 대한 차이가 있을 수 있지만 servlet 컨테이너 메커니즘과 인프라스트럭쳐에 대해 다음과 같은 공통된 특성이 요구된다.

* 인증: 통신요소가 접근 권한이 있는 특정 신원을 대신하여 행동하고 있음을 증명하는 수단

* 리소스 접근제한: 리소스와의 상호 작용은 무결성, 기밀성 또는 가용성 제약을 시행하기 위한 사용자 또는 프로그램 모음으로 제한

* 데이터 무결성: 전송중 제삼자에 의해 정보가 변경되지 않았음을 의미

* 무결성 혹은 데이터 프라버시: 정보에 접근할 권한이 있는 사용자에게만 정보를 제공하는 것을 의미

## 선언적 보안

선언적 보안은 애플리케이션 외부의 형태로 역할, 액세스 제어 및 인증 요구 사항을 포함하여 애플리케이션의 보안 모델이나 요구 사항을 표현하는 수단을 의미한다. 
배포 설명자는 웹 애플리케이션의 선언적 보안을 위한 주요 수단이다. 배포자는 애플리케이션의 논리적 보안 요구 사항을 런타임 환경에 특정한 보안 정책의 표현에 매핑
한다. 런타임에 서블릿 컨테이너는 보안 정책 표현을 사용하여 인증과 인증을 시행한다.

보안 모델은 웹 애플리케이션의 정적 콘텐츠 부분과 클라이언트가 요청한 애플리케이션 내의 서블릿 및 필터에 적용된다. 보안 모델은 서블릿이 RequestDispatcher
를 사용하여 foward 또는 include를 사용하여 정적 리소스 또는 서블릿을 호출할 때 적용되지 않는다.


## 프로그램적 보안

프로그래밍 방식 보안은 선언적 보안만으로는 애플리케이션의 보안 모델을 표현하기에 충분하지 않을 때 보안이 적용된 애플리케이션에서 사용된다. 프로그래밍 방식 보안
은 HttpServletRequest 인터페이스의 다음 method들로 구성됨:

### authenticate method

`boolean authenticate(HttpServletResponse response) throws java.io.IOException,ServletException`

이 메서드의 역할은 이름 그대로 인증이다. 이미 해당 요청이 이미 인증이 처리되어 getUserPrincipal, getRemoteUser, getAuthType 메서드의 반환 값이 
형성 된 경우 로그인을 시도 하지 않고 true를 반환한다. 그렇지 않은 경우 해당 servlet context에 설정된 login 메커니즘을 이용하여 로그인을 시도한다. 
해당 요청에 담긴 인증 정보(username, password)를 이용 인증을 처리하여 그 결과를 boolean type으로 반환한다. 

인증정보가 realm상의 사용자 인증 정보와 상이하여 login 메서드 수행 결과 ServletException이 발생한 경우 이 메서드는 false를 반환한다.

만약 해당 servlet context에 인증 관련 설정이 누락된 경우 역시 false를 반환한다.

[요약]
> authenticate method가 true를 반환하는 경우 
>> 1. 이미 해당 요청에 인증 처리되어 getUserPricipal이 null이 아닌 값을 리턴하는 경우 
>> 2. login 메서드가 예외 없이 수행 된 경우 

> authenticate method가 false를 반환하는 경우
>> 1. 인증 정보가 상이하여 login 메서드에서 ServletException을 throw 한 경우  
>> 2. 해당 servlet context에 login 설정이 없는 경우 

*servlet context login 설정 예시*

```xml
<!-- FORM based authentication --> 
<login-config> 
    <auth-method>FORM</auth-method> 
    <form-login-config> 
        <form-login-page>/login.jsp</form-login-page> 
        <form-error-page>/error.jsp</form-error-page> 
    </form-login-config> 
</login-config>
```

### login method

`void login(String username, String password) throws ServletException`

authenticate 메서드를 호출하면 servlet context 설정에 정의된 인증 메커니즘과 login 메커니즘을 총 동원해서 요청 정보에 인증 정보가 없다면 클라이언트
로 인증을 요청을 내려보내는 일부터 인증정보를 realm에 정의된 사용자 정보와 대조하여 인증을 처리하는 모든 일을 수행하게 된다. 이는 Servlet spce에 제약적
이어서 때로는 구현에 제약을 갖게 한다. 이를 해결할 수 있는 방법으로 login method를 사용할 수 있다. 이는 servlet context의 login 설정에 제약을 받
지 않는다. 어떤 방식으로던 username, password를 구해서 이 메서드를 호출하게 되면 그 야말로 로그인 처리가 된다.  
당연한 이야기지만 로그인이 성공하면 getUserPrincipal, getRemoteUser, getAuthType 메서드들은 null이 아닌 값을 반환하게 된다. 

이 메서드는 반환 값이 없다. id, password가 realm에서 정의한 사용자 인증정보와 달라 인증에 실패하게 되면 ServletException이 throw되며 그렇지 않은
경우 정상 수행된다.

### logout method

`void logout() throws ServletException`

이 메서드의 호출은 이미 로그인된 인증을 무효화 시킨다. 결과적으로 이 메서드 호출후에는 getUserPrincipal, getRemoteUser, getAuthType 메서드들은 
null을 반환하게 된다.

### getRemoteUser method

`String getRemoteUser()`

해당 요청 프로세스가 이미 로그인된 경우 username을 반환하는 메서드다. 만약 로그인 되지 않은 경우라면 null을 리턴한다.

### isUserInRole method

`boolean isUserInRole(java.lang.String role)`

인자로 주어진 역할 이름이 사용자에게 할당된 realm 역할 이름으로 존재하는지 확인하는 메서드다. 역할에 소속 되어 있지않거나 해당 요청 프로세스가 인증 되어 있
지 않은 경우 false를 반환하며 반대로 인자로 전달된 역할 이름이 사용자 역할 중 하나에 부합되면 true를 반환하게 된다. 

## getUserPrincipal method

`java.security.Principal getUserPrincipal()`

현재 인증 사용자의 `java.security.Principal` 객체를 반환한다. 만약 해당 요청의 사용자가 인증 정보를 가지고 있지 않은 경우(로그인 되지 않은 경우) 
null을 반환한다.

*보안 메서드에 대한 고찰*

> 지금까지 설명한 6개의 HttpServletRequest 인터페이스에 정의된 보안 메서드들의 의미를 정리 해보려 한다. 웹 응용프로그램을 개발하는 입장에서 선언적인 보
안정책을 통해서 웹 리소스에 대한 보안을 구현할 수 없거나 프로그램 로직에서 이를 판단하고 처리할 필요가 있을때 이 메서드들을 이용하여 원하는 보안 이슈를 해결 
할 수 있다. 다만 이런 프로그램적인 방법으로만 보안 정책을 구현하게 되면 배포 담당자를 통한 유연한 보안 설정이 불가능 해진다.

## 프로그램적인 보안정책 설정

## 역할

## 인증

## 인증 정보에 대한 서버 사이드 추적

## 보안 제한(제약) 명세하기



<script src="https://utteranc.es/client.js"
        repo="jchong-dalcomlab/jchong-dalcomlab.github.io"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

