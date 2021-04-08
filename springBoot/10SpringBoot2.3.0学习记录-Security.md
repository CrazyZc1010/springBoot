# 10. Security #  
如果Spring Security在类路径上，那么web应用程序在默认情况下是安全的。Spring Boot依赖于Spring Security的内容协商策略来决定是使用httpBasic还是formLogin。要向web应用程序添加method-level安全性，还可以使用所需设置添加@EnableGlobalMethodSecurity。其他信息可以在Spring安全参考指南中找到。  
默认的UserDetailsService只有一个用户。用户名是user，密码是随机的，在应用程序启动时在INFO级别打印，如下例所示：  

    Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35  

如果您微调了日志配置，请确保org.springframework.boot.autoconfigure.security类别设置为日志信息级别的消息。否则，不会打印默认密码。  
您可以通过提供spring.security.user.name和spring.security.user.password来更改用户名和密码。  
默认情况下，web应用程序中的基本功能包括：  

1. 一个具有内存存储的UserDetailsService（或者在WebFlux应用程序的情况下是ReactiveUserDetailsService）bean和一个具有生成的密码的用户（有关用户的属性，请参阅SecurityProperties.user）。  
1. 整个应用程序的基于表单的登录或HTTP基本安全性（取决于请求中的Accept头）（如果执行器位于类路径上，则包括执行器端点）。  
1. 用于发布身份验证事件的DefaultAuthenticationEventPublisher。  

您可以通过添加一个bean来提供不同的AuthenticationEventPublisher。  


## 10.1. MVC Security ##  
默认的安全配置在SecurityAutoConfiguration和UserDetailsServiceAutoConfiguration中实现。SecurityAutoConfiguration导入SpringBootWebSecurityConfiguration用于web安全和用户详细信息services autoconfiguration用于配置身份验证，这在非web应用程序中也是相关的。要完全关闭默认的web应用程序安全配置或组合多个Spring安全组件（如OAuth 2客户端和资源服务器），请添加WebSecurityConfigurerAdapter类型的bean（这样做不会禁用UserDetailsService配置或执行器的安全性）。   
 
要同时关闭UserDetailsService配置，可以添加UserDetailsService、AuthenticationProvider或AuthenticationManager类型的bean。  
  
访问规则可以通过添加自定义WebSecurity配置适配器来重写。Spring Boot提供了一些方便的方法，可以用来覆盖执行器端点和静态资源的访问规则。EndpointRequest可用于创建基于management.endpoints.web.base-path属性的RequestMatcher。PathRequest可用于为常用位置的资源创建RequestMatcher。  


## 10.2. WebFlux Security  ##   
与Spring MVC应用程序类似，您可以通过添加spring-boot-starter-security 依赖项来保护WebFlux应用程序。默认的安全配置在ReactiveSecurityAutoConfiguration和UserDetailsServiceAutoConfiguration中实现。ReactiveSecurityAutoConfiguration导入web安全的WebFluxSecurityConfiguration和UserDetailsServiceAutoConfiguration配置身份验证，这在非web应用程序中也是相关的。要完全关闭默认的web应用程序安全配置，可以添加WebFilterChainProxy类型的bean（这样做不会禁用UserDetailsService配置或执行器的安全性）。   

要同时关闭UserDetailsService配置，可以添加ReactiveUserDetailsService或ReactiveAuthenticationManager类型的bean。   

访问规则和多个Spring安全组件（如OAuth 2客户机和资源服务器）的使用可以通过添加自定义SecurityWebFilterChain bean进行配置。Spring Boot提供了一些方便的方法，可以用来覆盖执行器端点和静态资源的访问规则。EndpointRequest可用于创建基于management.endpoints.web.base-path属性的ServerWebExchangeMatcher。  

PathRequest可用于为常用位置的资源创建服务器webexchangematcher。  

例如，可以通过添加以下内容自定义安全配置：  

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    return http
    .authorizeExchange()
    .matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
    .pathMatchers("/foo", "/bar")
    .authenticated().and()
    .formLogin().and()
    .build();
    }  


----------
2020/4/2 20:03:26 

----------


## 10.3. OAuth2 ##  
OAuth2是Spring支持的一个广泛使用的授权框架。  


### 10.3.1. Client ###  
如果类路径上有spring-security-oauth2-client，那么可以利用一些自动配置来轻松设置oauth2/Open ID Connect客户端。此配置使用OAuth2ClientProperties下的属性。同样的属性适用于servlet和反应式应用程序。  
可以在spring.security.OAuth2.client前缀下注册多个OAuth2客户端和提供程序，如下例所示：  

    spring.security.oauth2.client.registration.my-client-1.client-id=abcd
    spring.security.oauth2.client.registration.my-client-1.client-secret=password
    spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
    spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
    spring.security.oauth2.client.registration.my-client-1.scope=user
    spring.security.oauth2.client.registration.my-client-1.redirect-uri=https://my-redirect-uri.com
    spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
    spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code
    
    spring.security.oauth2.client.registration.my-client-2.client-id=abcd
    spring.security.oauth2.client.registration.my-client-2.client-secret=password
    spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
    spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
    spring.security.oauth2.client.registration.my-client-2.scope=email
    spring.security.oauth2.client.registration.my-client-2.redirect-uri=https://my-redirect-uri.com
    spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
    spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code
    
    spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=https://my-auth-server/oauth/authorize
    spring.security.oauth2.client.provider.my-oauth-provider.token-uri=https://my-auth-server/oauth/token
    spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=https://my-auth-server/userinfo
    spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
    spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=https://my-auth-server/token_keys
    spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name  

对于支持OpenID Connect发现的OpenID Connect提供程序，可以进一步简化配置。提供程序需要配置一个颁发者uri，该uri是它断言为其颁发者标识符的uri。例如，如果提供的颁发者uri是“https://example.com”，则将向“https://example.com/.well-known/OpenID-Configuration”发出OpenID提供程序配置请求。结果应该是OpenID提供程序配置响应。以下示例显示如何使用颁发者uri配置OpenID连接提供程序：  

    spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/  

默认情况下，Spring Security的OAuth2LoginAuthenticationFilter只处理匹配/login/oauth2/code/*的url。如果要自定义重定向uri以使用不同的模式，则需要提供配置来处理该自定义模式。例如，对于servlet应用程序，您可以添加自己的WebSecurity配置适配器，类似于以下内容：  

    public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .anyRequest().authenticated()
    .and()
    .oauth2Login()
    .redirectionEndpoint()
    .baseUri("/custom-callback");
    }
    }  

**公用提供程序的OAuth2客户端注册**  
对于常见的OAuth2和OpenID提供者，包括Google、Github、Facebook和Okta，我们提供了一组提供者默认值（分别是Google、Github、Facebook和Okta）。  

如果不需要自定义这些提供程序，可以将provider属性设置为需要推断其默认值的属性。此外，如果客户端注册的密钥与默认支持的提供程序匹配，那么Spring Boot也会推断出这一点。  

换句话说，以下示例中的两个配置使用Google提供程序：  

    spring.security.oauth2.client.registration.my-client.client-id=abcd
    spring.security.oauth2.client.registration.my-client.client-secret=password
    spring.security.oauth2.client.registration.my-client.provider=google
    
    spring.security.oauth2.client.registration.google.client-id=abcd
    spring.security.oauth2.client.registration.google.client-secret=password    

### 10.3.2 资源服务器 ###  
如果类路径上有spring-security-oauth2-resource-server，那么spring Boot可以设置oauth2资源服务器。对于JWT配置，需要指定JWK Set URI或OIDC Issuer URI，如下例所示：  

    spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys  

    spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/  

如果授权服务器不支持JWK Set URI，则可以使用用于验证JWT签名的公钥配置资源服务器。这可以使用spring.security.oauth2.resourceserver.jwt.public-key-location属性完成，其中的值需要指向包含PEM编码x509格式的公钥的文件。  
同样的属性适用于servlet和反应式应用程序。  
或者，您可以为servlet应用程序定义自己的JwtDecoder bean，或者为反应性应用程序定义一个反应性JwtDecoder bean。  
在使用不透明令牌而不是jwt的情况下，可以配置以下属性以通过内省验证令牌：  

    spring.security.oauth2.resourceserver.opaquetoken.introspection-uri=https://example.com/check-token
    spring.security.oauth2.resourceserver.opaquetoken.client-id=my-client-id
    spring.security.oauth2.resourceserver.opaquetoken.client-secret=my-client-secret
    
同样，相同的属性也适用于servlet和反应式应用程序。  
或者，您可以为servlet应用程序定义自己的OpaqueTokenIntrospector bean，或者为反应性应用程序定义一个ReactiveOpaqueTokenIntrospector。 

### 10.3.3 授权服务器 ###  
目前，Spring Security不支持实现OAuth 2.0授权服务器。但是，这个功能可以从Spring Security OAuth项目获得，该项目最终将被Spring Security完全取代。在此之前，您可以使用spring-security-oauth2-autoconfigure模块轻松设置oauth2.0授权服务器；有关说明，请参阅其文档。  


## 10.4. SAML 2.0 ##  

### 10.4.1. Relying Party ###  
如果您的类路径上有spring-security-saml2-service-provider，那么您可以利用一些自动配置来轻松地设置saml2.0依赖方。此配置使用Saml2RelyingPartyProperties下的属性。  
依赖方注册表示标识提供程序IDP和服务提供程序SP之间的成对配置。可以在spring.security.saml2.relying party前缀下注册多个依赖方，如下例所示：  

    spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].private-key-location=path-to-private-key
    spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].certificate-location=path-to-certificate
    spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.verification.credentials[0].certificate-location=path-to-verification-cert
    spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.entity-id=remote-idp-entity-id1
    spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.sso-url=https://remoteidp1.sso.url
    
    spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].private-key-location=path-to-private-key
    spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].certificate-location=path-to-certificate
    spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.verification.credentials[0].certificate-location=path-to-other-verification-cert
    spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.entity-id=remote-idp-entity-id2
    spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.sso-url=https://remoteidp2.sso.url

## 10.5 执行机构安全 ##  
出于安全目的，默认情况下，除/health和/info之外的所有执行器都将被禁用。management.endpoints.web.exposure.include属性可用于启用执行器。  
如果Spring Security位于类路径上，并且没有其他WebSecurityConfigurerAdapter，那么除了/health和/info之外的所有执行器都由Spring Boot auto configuration进行保护。如果您定义了一个自定义的WebSecurityConfigurerAdapter，Spring Boot自动配置将退出，您将完全控制执行器访问规则。  
在设置management.endpoints.web.exposure.include之前，请确保暴露的执行器不包含敏感信息，和/或通过将其置于防火墙后或通过类似于Spring Security的方式进行保护。  

### 10.5.1 跨站点请求伪造保护  ###  
由于Spring Boot依赖于Spring Security的默认值，所以CSRF保护默认打开。这意味着当使用默认安全配置时，需要POST（关闭和记录器端点）、PUT或DELETE的执行器端点将获得403禁止错误。  
我们建议仅在创建非浏览器客户端使用的服务时才完全禁用CSRF保护。  
有关CSRF保护的其他信息，请参见《Spring安全参考指南》。


    