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

## 목차

[서론](#서론) 

[선언적 보안](#선언적-보안) 

[프로그램적 보안](#프로그램적-보안) 

[프로그램적인 보안정책 설정](#프로그램적인-보안정책-설정) 

[역할](#역할) 

[인증](#인증) 

[인증 정보에 대한 서버 사이드 추적](#인증-정보에-대한-서버-사이드-추적) 

[보안 제한(제약) 명세하기](#보안-제한제약-명세하기) 

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

### getUserPrincipal method

`java.security.Principal getUserPrincipal()`

현재 인증 사용자의 `java.security.Principal` 객체를 반환한다. 만약 해당 요청의 사용자가 인증 정보를 가지고 있지 않은 경우(로그인 되지 않은 경우) 
null을 반환한다.

*보안 메서드에 대한 고찰*

> 지금까지 설명한 6개의 HttpServletRequest 인터페이스에 정의된 보안 메서드들의 의미를 정리 해보려 한다. 웹 응용프로그램을 개발하는 입장에서 선언적인 보
안정책을 통해서 웹 리소스에 대한 보안을 구현할 수 없거나 프로그램 로직에서 이를 판단하고 처리할 필요가 있을때 이 메서드들을 이용하여 원하는 보안 이슈를 해결 
할 수 있다. 다만 이런 프로그램적인 방법으로만 보안 정책을 구현하게 되면 배포 담당자를 통한 유연한 보안 설정이 불가능 해진다.

## 프로그램적인 보안정책 설정

이 섹션에서는 Servlet 컨테이너에 의해 시행되는 보안 제약을 구성하기 위해 Servlet spec에서 제공된 주석과 API방식을 설명한다.

### annotation 방식의 Servlet 보안제약 설정

Servlet containter에 보안 정책을 설정할 수 있는 방법은 크게 web.xml에 선언하는 방식과 servlet class에 선언하는 annotation방식이 존재한다.
web.xml에 선언하는 보안정책은 요청 url을 중심으로 해당 리소스에 대한 보안 제약사항을 체크하는 방식이다. 이후 등장한 annotation방식은 servlet class
가 보안 설정에 주인이 된다. 이로 인해 개념적으로 많은 혼란이 야기되는데 이 단원에서 차차 자세히 설명하기로 한다.

#### ServletSecurity annotation

이 어노테이션은 하나의 HttpConstraint와 복수개의 HttpMethodConstraint 하위 annotation을 구성요소로 갖을 수 있다. 적어도 하나는 있어야 이를 사용
하는 의미가 있다. 

![ServletSecurity anno](/assets/img/blog/ServletSecurity-anno.png)

ServletSecurity annotation 하위에 구성되는 HttpMethodConstraint annotation은 특정 Http method 에 대한 서블릿 보안 제약을 설정한다. 
하나의 @HttpMethodConstraint 는 하나의 http protocol method에 대한보안 제한을 지정 할 수 있다. 예를 들어 GET, POST 두 가지에 대한 보안제한을
지정해야 한다면 @HttpMethodConstraint 두 나열해야 한다. 
HttpConstraint annotation은 메서드를 지정하는 멤버가 없다. 그래서 ServletSecurity당 하나만 쓸수 있다. @HttpMethodConstraint에서 설정한 보
안 제한을 제외한 나머지 http protocol method에 대한 보안제한 정책을 설정한다. 

만약 @HttpMethodConstraint 하나도 없다면 모든 http protocol method에 대한 보안 제한 정책이 된다.

다음 annotation 예시는 다음과 같은 보안 제약을 수행한다.
GET에 대해서는 역할 보안제약을 두지 않고(제한 없이 요청 프로세스 처리), 나머지 method에 대해서 Administrator 역할 보안 제약을 확인 해야한다.

```java
@ServletSecurity(value=@HttpConstraint(rolesAllowed = "Administrator"),
    httpMethodConstraints = @HttpMethodConstraint(value = "GET", emptyRoleSemantic = EmptyRoleSemantic.PERMIT))
```
<@HttpMethodConstraint, @HttpConstraint 같이 사용하는 예>

다음 annotation 예시는 다음과 같은 보안 제약을 수행한다. 
모든 http protocol method에 대해 Manager 역할 있어야 요청 프로세스를 처리할 수 있다. 

```java
@ServletSecurity(value=@HttpConstraint(rolesAllowed={"Manager"}, transportGuarantee = TransportGuarantee.NONE))
```
<@HttpConstraint 만 사용하는 예>

*Empty Role Semantic (빈 역할에 대한 정책)*
> 이 정책은 @HttpMethodConstraint의 emptyRoleSemantic 그리고 @HttpConstraint의 value 프로퍼티에 설정한다. 이는 rolesAllowed가 
비어 있을때 해당 http protocol method의 요청을 어떻게 처리 할 것인가에 대한 정책 설정이다. PERMIT은 허용을 의미하고 DENY는 거부를 의미한다.

*web.xml의 Empty Role Semantic (빈 역할에 대한 정책)*
> web.xml에 빈 역할 보안 정책은 <auth-constraint> 를 어떻게 기술했느냐에 의해 결정된다. 아애 해당 xml element를 기술하지 않은 경우 PERMIT에
해당하는 동작을 해야하고, element는 있으나 아무런 <role-name>도 기술되지 않은 경우 즉 <auth-constraint/> 라고 기술한 경우 DENY에 해당하는 처리
를 하게된다.

##### @HttpMethodConstraint annotation

| 프로퍼티 | 타입 | 설명 | 기본값 | 필수여부 |
|---|---|---|---|:---:|
| `value` | java.lang.String | Http protocol method 이름 | N/A | O |
| `emptyRoleSemantic` | ServletSecurity.EmptyRoleSemantic | 빈 역할에 대한 정책 | PERMIT | X |
| `rolesAllowed` | java.lang.String[] | 허용된 역할 | empty | X |
| `transportGuarantee` |  ServletSecurity.TransportGuarantee | 전송 데이터 보호 옵션 | NONE | X |

##### @HttpMethodConstraint annotation

| 프로퍼티 | 타입 | 설명 | 기본값 | 필수여부 |
|---|---|---|---|:---:|
| `value` | ServletSecurity.EmptyRoleSemantic | 빈 역할에 대한 정책 | PERMIT | X |
| `rolesAllowed` | java.lang.String[] | 허용된 역할 | empty | X |
| `transportGuarantee` |  ServletSecurity.TransportGuarantee | 전송 데이터 보호 옵션 | NONE | X |

*HTTP protocol method*

다음은 이 단원에서 여러번 거론된 Http protocol method에 종류다. 
> GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE, PATCH 

각 method에 대한 자세한 의미는 다음 링크를 클릭하여 확인 하도록 한다.

[HTTP request methods in mdn web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

## 역할

보안 역할은 애플리케이션 개발가 정의한 사용자의 논리적 그룹이다. 애플리케이션이 배포되면, 역할은 배포자에 의해 런타임 환경의 주체나 그룹에 매핑된다. 
서블릿 컨테이너는 주체의 보안 속성을 기반으로 들어오는 요청과 관련된 주체에 대해 선언적 또는 프로그래밍 보안을 시행한다. 이것은 다음 방법 중 하나로 발생할 수 
있다:

1. 배포자는 운영 환경의 사용자 그룹에 보안 역할을 매핑다. 호출 주체가 속한 사용자 그룹은 보안 속성에서 검색된다. 배포자가 보안 역할을 매핑한 사용자 그룹에 
속한 경우에만 보안 역할이 있다.

2. 배포자는 보안 정책 도메인의 주요 이름에 보안 역할을 매핑했다. 이 경우, 호출 주체의 주요 이름은 보안 속성에서 검색된다. 이름이 보안 역할에 매핑된 주체 
이름과 동일한 경우에만 보안 역할이 있다.

## 인증

인증(authentication)은 Web application server 관점에서 인가(authorization)과 함께 중요한 한축을 이루는 요소가 인증이다. Servlet spec에서
다음 4가지 인증 방법에 대해 설명하고 있다. 

* HTTP Basic Authentication
* HTTP Digest Authentication
* HTTPS Client Authentication
* Form Based Authentication



### HTTP Basic Authentication

Basic 인증은 HTTP/1.0의 사양으로 username, password를 기반으로 한 인증 메커니즘이다. 
web server는 web client에 사용자 인증을 요청한다. 이때 Web server는 로그인 요청(401 UNAUTHORIZED, 사실 기술적으로 응답임)을 내려 보낼때 다음
과 같은 응답 헤더를 포함시킨다.  

> WWW-Authenticate: BASIC realm="[realmName]" 

이 응답을 사람이 알아듣기 표편한 표현으로 하자면 다음과 같을 것이다.
*"당신이 요청한 웹 리소스는 인증이 필요한데 거기에 해당하는 영역 이름은 [realmName]이야"* 
이런 응답을 받은 브라우저는 사용자가 Basic 인증을 진행할 수 있도록 다음과 같은 UI를 나타낸다.

![Basice authentication](/assets/img/blog/basic-login-ux.png)

그리고 브라우저는(웹 클라이언트)는 위 화면을 통해 사용자로부터 사용자 이름과 비밀번호를 얻어 웹 서버로 전송한다. 

*Basic 인증의 문제점*

Basic 인증시 사용되는 사용자 암호는 간단한 base64 encoding으로 server에 전송된다. 이는 안전하지 않으며 이 문제를 해결하려면 SSL을 사용하여 모든 전송
데이터 자체를 안전하게 처리 해야 한다.

## 인증 정보에 대한 서버 사이드 추적

## 보안 제한(제약) 명세하기



<script src="https://utteranc.es/client.js"
        repo="jchong-dalcomlab/jchong-dalcomlab.github.io"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

