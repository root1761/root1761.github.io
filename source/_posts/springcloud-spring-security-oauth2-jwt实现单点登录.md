---
title: springcloud+spring security oauth2+jwt实现单点登录
date: 2021-01-20 17:10:08
tags:
   - springcloud
   - oauth2
   - jwt
---
## 一、SSO基本介绍
 SSO（Single Sign-On，单点登录）是身份管理中的一部分。SSO的一种较为通俗的定义是：SSO是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。它包括可以将这次主要的登录映射到其他应用中用于同一个用户的登录的机制。它是目前比较流行的企业业务整合的解决方案之一。
 相关技术主要有：spring security ，shiro
## 二、spring security 基本实现
**1.引入pom依赖**

```java 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```
**2.添加配置文件**
```java
@Configuration
@EnableDiscoveryClient
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private Logger logger = LoggerFactory.getLogger(ActivitiConfig.class);

    @Override
    @Autowired
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService());
    }
    @Bean
    public UserDetailsService myUserDetailsService() {

        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();

        String[][] usersGroupsAndRoles = {
                {"salaboy", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam","GROUP_otherTeam"},

                {"ryandawsonuk", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                {"erdemedeiros", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                {"other", "password", "ROLE_ACTIVITI_USER", "GROUP_otherTeam"},
                {"admin", "password", "ROLE_ACTIVITI_ADMIN"},
        };
        for (String[] user : usersGroupsAndRoles) {
            List<String> authoritiesStrings = Arrays.asList(Arrays.copyOfRange(user, 2, user.length));
            logger.info("> Registering new user: " + user[0] + " with the following Authorities[" + authoritiesStrings + "]");
            inMemoryUserDetailsManager.createUser(new User(user[0], passwordEncoder().encode(user[1]),
                    authoritiesStrings.stream().map(s -> new SimpleGrantedAuthority(s)).collect(Collectors.toList())));
        }


        return inMemoryUserDetailsManager;
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();


    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```
启动测试：
<div style="width:70%;margin:auto">{% asset_img security1.png %}</div>

## 三、spring security oauth2
### 1.认证服务

**(1).引入pom依赖**

```java
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```
**(2).认证方式**

{% blockquote %}

授权码模式：authorization_code
客户端模式：client_credentials
简化模式： implicit
密码模式： password

{% endblockquote %}

**(3).认证服务访问路径**

{% blockquote %}
/oauth/authorize：授权端点。
/oauth/token：令牌端点。 
/oauth/confirm_access：用户确认授权提交端点。 
/oauth/error：授权服务错误信息端点。 
/oauth/check_token：用于资源服务访问的令牌解析端点。
/oauth/token_key：提供公有密匙的端点，如果你使用JWT令牌的话
{% endblockquote %}

**(4).认证服务配置**

```java

@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private TokenStore tokenStore;
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.tokenKeyAccess("permitAll()").//获取公钥开放
                checkTokenAccess("permitAll()").//用户信息开放
                allowFormAuthenticationForClients();//允许表单验证
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory().withClient("password").//标识客户端id
                authorizedGrantTypes("authorization_code","client_credentials","implicit","password", "refresh_token").
                accessTokenValiditySeconds(24*60*60*30*12).//access_token过期时间
                refreshTokenValiditySeconds(24*60*60*30*12*10).//refresh_token过期时间
                resourceIds("rid").//资源id
                scopes("all").//允许授权范围
                secret(passwordEncoder.encode("123"))//客户端密钥
                .autoApprove(false)
                .redirectUris("http://www.baidu.com");//回调地址
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore())//令牌存储
                .authenticationManager(authenticationManager)//令牌管理器
                .userDetailsService(userDetailsService);//用户信息user参数
    }
    @Bean
    public TokenStore tokenStore() {
        return new InMemoryTokenStore();//存储方式redis、jdbc、memory、jwt
     
    }
}

```
**(5).web安全配置**
```java
@Configuration
@EnableWebSecurity
@Slf4j
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserAuthenticationSuccessHandler userAuthenticationSuccessHandler;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }

    @Bean
    @ConditionalOnMissingBean(UserDetailsService.class)
    @Override
    protected UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        String[][] usersGroupsAndRoles = {
                {"salaboy", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam", "GROUP_otherTeam"},

                {"ryandawsonuk", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                {"erdemedeiros", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                {"other", "password", "ROLE_ACTIVITI_USER", "GROUP_otherTeam"},
                {"admin", "password", "ROLE_ACTIVITI_ADMIN"},
        };
        for (String[] user : usersGroupsAndRoles) {
            List<String> authoritiesStrings = Arrays.asList(Arrays.copyOfRange(user, 2, user.length));
            log.info("> Registering new user: " + user[0] + " with the following Authorities[" + authoritiesStrings + "]");
            inMemoryUserDetailsManager.createUser(new User(user[0], passwordEncoder().encode(user[1]),
                    authoritiesStrings.stream().map(s -> new SimpleGrantedAuthority(s)).collect(Collectors.toList())));
        }


        return inMemoryUserDetailsManager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests().antMatchers(HttpMethod.OPTIONS).permitAll()//permitAll:放过 authenticated：认证 denyAll：禁用
                .antMatchers("/oauth/**")//路径
                .permitAll().and().httpBasic().and()
                .csrf().disable();
    }
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```

**(6).请求方式**
授权码模式：
客户端获取code值
/oauth/authorize?client_id=password&response_type=code&scope=all&redirect_uri=http://www.baidu.com

{% blockquote %}

client_id：客户端准入标识。
response_type：授权码模式固定为code。 
scope：客户端权限。 
redirect_uri：跳转uri，当授权码申请成功后会跳转到此地址，并在后边带上code参数（授权码）

{% endblockquote %}
<div style="margin:auto;width:90%">{% asset_img code.png%}</div>

密码模式:
<div style="margin:auto;width:90%">{% asset_img password.png%}</div>

客户端模式：

<div style="margin:auto;width:90%">{% asset_img client.png%}</div>

简化模式：

浏览器访问：http://localhost:9006/oauth/authorize?client_id=password&response_type=token&scope=all&redirect_uri=http://www.baidu.com

### 2.资源服务

**(1).引入pom依赖**
```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```
**(2).资源服务配置**
```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources
                .resourceId("rid").//资源id（必须和认证服务一致）
                .stateless(true);
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/**/current/get","/**/oauth/**","/swagger-ui.html","/doc.html","/**/v2/api-docs/**","/error","/**/swagger-resources/**","/webjars/**")
                .permitAll()
                .antMatchers("/**")//路径
                .hasAnyRole("ADMIN")//用户角色权限
                .anyRequest()
                .authenticated();
    }

}

```
## 四、oauth2集成redis
**1.引入pom依赖**
```java
      <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
```
**2.yml 添加配置**
```bash
spring:
  redis:
    host: 192.168.42.146
    port: 6379
    database: 0
    jedis:
      pool:
        max-active: 8
        max-wait: -1
        max-idle: 500
        min-idle: 0
```
**3.修改认证配置**
```java

@Autowired
private RedisConnectionFactory redisConnectionFactory;
  @Bean
    public TokenStore tokenStore() {
            return new RedisTokenStore(redisConnectionFactory);//存储方式redis、jdbc、memory、jwt
     
    }

```
效果如下：

<div style="margin:auto;width:90%">{% asset_img redis.png%}</div>

## 五、oauth2集成jwt

### 1.认证服务

**(1).引入pom依赖**
```java
  <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-jose</artifactId>
 </dependency>
```
**(2).生成密钥**
keytool -genkey -alias youlai -keyalg RSA -keypass 000000 -keystore youlai.jks -storepass 00000
{% blockquote %}

-genkey：生成密钥
-alias：别名
-keyalg：密钥算法
-keypass：密钥口令
-keystore：生成密钥库的存储路径和名称
-storepass：密钥库口令

{% endblockquote %}
将生成的youlai.jks添加到resource下

**(3).修改配置文件**
```bash
encrypt:
  key-store:
    location: classpath:youlai.jks
    secret: 000000
    alias: youlai
    password: 0000
```
**(4).修改认证服务**
```java
@Autowired
    @Autowired
   private KeyProperties keyProperties;
 @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    /*    endpoints.tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);*/
        TokenEnhancerChain enhancerChain=new TokenEnhancerChain();
        final List<TokenEnhancer> objects = Arrays.asList(jwtAccessTokenConverter(),consumerTokenEnhancer());
        enhancerChain.setTokenEnhancers(objects);
        endpoints.tokenEnhancer(enhancerChain)
                .accessTokenConverter(jwtAccessTokenConverter())
                .authenticationManager(authenticationManager)//认证管理器
                .tokenStore(tokenStore)//令牌存储
                .userDetailsService(userDetailsService)//用户信息user参数
                // refresh_token有两种使用方式：重复使用(true)、非重复vice
                //                .tokenEnhancer(enhancerChain)//自定义jwt令牌中的携带使用(false)，默认为true
                //      1.重复使用：access_token过期刷新时， refresh token过期时间未改变，仍以初次生成的时间为准
                //      2.非重复使用：access_token过期刷新时， refresh_token过期时间延续，在refresh_token有效期内刷新而无需失效再次登录
                .reuseRefreshTokens(false);
    }
    public ConsumerTokenEnhancer consumerTokenEnhancer(){
        return new ConsumerTokenEnhancer(expiraTime,redisConnectionFactory);
}
    @Bean
    public TokenStore tokenStore() {
        //return new InMemoryTokenStore();
        return new JwtTokenStore(jwtAccessTokenConverter());
        //return new RedisTokenStore(redisConnectionFactory);
    }

  @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setKeyPair(keyPair()); //对称秘钥，资源服务器使用该秘钥来验证
        return converter;
    }

    @Bean
    public KeyPair keyPair(){
    return  (new KeyStoreKeyFactory(this.keyProperties.getKeyStore().getLocation(), this.keyProperties.getKeyStore().getSecret().toCharArray())).getKeyPair(this.keyProperties.getKeyStore().getAlias(), this.keyProperties.getKeyStore().getPassword().toCharArray());

}
```
### 2.资源服务
```java
@Autowired
    private TokenStore tokenStore;
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources
                .resourceId("rid").tokenStore(tokenStore)
                .stateless(true);
    }
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter(){
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        Resource resource = new ClassPathResource("publicKey.txt");//秘钥
        String publicKey;
        try {
            publicKey = new String(FileCopyUtils.copyToByteArray(resource.getInputStream()));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        converter.setVerifierKey(publicKey);
        return converter;
    }
```
