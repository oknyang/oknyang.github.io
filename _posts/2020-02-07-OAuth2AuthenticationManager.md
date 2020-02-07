---
title: "spring OAuth2AuthenticationManager"
date: 2020-02-07
categories: spring-security
---

1. spring security 설정  
```xml
    <!-- health check때문에 security 타지말라고 걸어놓은듯 -->
    <security:http pattern="/__*/**" security="none"/>

    <!-- 인증 -->
    <security:http pattern="/oauth/token" create-session="stateless" authentication-manager-ref="authenticationManager">
        <security:intercept-url pattern="/oauth/token" access="IS_AUTHENTICATED_FULLY"/>
        <security:anonymous enabled="false"/>
        <security:http-basic entry-point-ref="oauthAuthenticationEntryPoint"/>
        <security:custom-filter ref="clientCredentialsTokenEndpointFilter" before="BASIC_AUTH_FILTER"/>
        <security:access-denied-handler ref="oauthAccessDeniedHandler"/>
    </security:http>
    
    <!-- 인가 -->
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

2. 인증  
/oauth/token으로 요청이 들어오면 custom-filter 부분의 clientCredentialsTokenEndPointFilter 부분부터 타는듯.  
기본 인증 필터(BASIC_AUTH_FILTER) 태우기 전에 커스텀 필터를 태워라. 라는 뜻이겠지...?  
ClientCredentialsTokenEndpointFilter를 까보면 attemptAuthentication 메소드가 핵심이란게 보임.  
POST전송이어야 하고, 인증이 있으면 그냥 리턴하고..(여기서 인증이 어떻게 튀어나오는지는 아직 잘 이해 안됨.)  인증 정보가 없으면 clientId와 clientSecret으로 UsernamePasswordAuthenticationToken을 생성하여 AuthoricationManager.authenticate에 전달한다. 

```java

	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException, IOException, ServletException {

		if (allowOnlyPost && !"POST".equalsIgnoreCase(request.getMethod())) {
			throw new HttpRequestMethodNotSupportedException(request.getMethod(), new String[] { "POST" });
		}

		String clientId = request.getParameter("client_id");
		String clientSecret = request.getParameter("client_secret");

		// If the request is already authenticated we can assume that this
		// filter is not needed
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authentication != null && authentication.isAuthenticated()) {
			return authentication;
		}

		if (clientId == null) {
			throw new BadCredentialsException("No client credentials presented");
		}

		if (clientSecret == null) {
			clientSecret = "";
		}

		clientId = clientId.trim();
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(clientId,
				clientSecret);

		return this.getAuthenticationManager().authenticate(authRequest);

	}
```


