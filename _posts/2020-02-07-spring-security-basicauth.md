---
title: "spring security"
date: 2020-02-07
categories: spring-security
---

1. spring security 설정  
```xml
    <!-- health check때문에 security 타지말라고 걸어놓은듯 -->
    <security:http pattern="/__*/**" security="none"/>

    <security:http pattern="/oauth/token" create-session="stateless" authentication-manager-ref="authenticationManager">
        <security:intercept-url pattern="/oauth/token" access="IS_AUTHENTICATED_FULLY"/>
        <security:anonymous enabled="false"/>
        <security:http-basic entry-point-ref="oauthAuthenticationEntryPoint"/>
        <security:custom-filter ref="clientCredentialsTokenEndpointFilter" before="BASIC_AUTH_FILTER"/> <!-- 디버깅해보니 이거 안탐..-->
        <security:access-denied-handler ref="oauthAccessDeniedHandler"/>
    </security:http>
    
    <security:http create-session="never" authentication-manager-ref="authenticationManager" use-expressions="true">
        <security:intercept-url pattern="/api/**" access="#oauth2.hasScope('vd') and @interworkOAuth2Validator.validate(authentication, request)"/>
        <security:intercept-url pattern="/error/**" access="permitAll"/>
        <security:intercept-url pattern="/**" access="denyAll"/>
        <security:anonymous enabled="false"/>
        <security:http-basic entry-point-ref="oauthAuthenticationEntryPoint"/>
        <security:custom-filter ref="resourceServerFilter" before="PRE_AUTH_FILTER"/>
        <security:access-denied-handler ref="oauthAccessDeniedHandler"/>
        <security:expression-handler ref="expressionHandler"/>
    </security:http>
```

2. accessToken 발급  



