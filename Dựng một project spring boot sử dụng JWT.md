**Cách hoạt động của JWT:**

JWT theo mình hiểu là công nghệ để xác thực người dùng trong việc sử dụng REST APIs ngoài ra còn có nhiều cách xác thực người dùng khác như sử dụng Session, Cookie hay sử dụng trực tiếp username và password

Sơ đồ minh họa:

![ảnh](https://user-images.githubusercontent.com/33534455/88456249-66094d00-cea6-11ea-8055-c0d031e63550.png)

Khi người dùng đăng nhập lần đầu vào hệ thống, người dùng cần gửi thông tin chứa mật khẩu cho server.

Server kiểm tra tính hợp lệ của thông tin đó. Nếu hợp lệ, server sẽ gửi một đoạn token chứa thông tin của người dùng để sử dụng trong hệ thống vào 1 chuỗi đã được mã hóa gọi là token

Những lần request sau lên server, người dùng sẽ không cần gửi thông tin về mật khẩu nữa. Thay vào đó chuỗi token sẽ được gắn vào phần header của request đến server. Server sẽ kiểm tra sự hợp lệ của chuỗi này và sẽ trả về respone tương ứng

**Ưu điểm:** 

Khi so sách với việc xác thực bằng session thì JWT có một số ưu điểm như:

- [ ] Có tính mở rộng cao, đa thiết bị do token có thể lưu ở bất cứ đâu còn sessionID thì lưu tại cookie của trình duyệt
- [ ] Tiết kiệm tài nguyên cho server do thông tin người dùng lưu tại client

----------------------------------------------------------------------------------------------------

**Tạo project Spring boot- Security**

Thêm các dependency:

```xml
<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.auth0/auth0 -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>auth0</artifactId>
    <version>1.19.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.auth0/auth0-spring-security-api -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>auth0-spring-security-api</artifactId>
    <version>1.4.0</version>
</dependency>
```

Các dependency được sử dụng cho việc sử dụng jwt để xác thực người dùng trong REST-APIs

Cấu trúc project như sau. Chú ý phần khoanh đỏ

![ảnh](https://user-images.githubusercontent.com/33534455/88455494-e6787f80-ce9f-11ea-8dd5-051cfeb21238.png)

Tạo config cho Spring Security: (tạm thời **chưa chú ý** đến **UserDetailServiceImpl**)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailServiceImpl userService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                .antMatchers("/").permitAll()
                .anyRequest().authenticated();
        http.addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }
    
    @Bean
    public JWTAuthenticationFilter jwtAuthenticationFilter() {
        return new JWTAuthenticationFilter();
    }
```

> .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)

Có tác dụng khai báo cho Spring Security ko sử dụng session để xác thực người dùng sau khi người dùng đã login thành công vào hệ thống. Điều này cần thiết là vì khi xác thực jwt thành công lần đầu, Spring sẽ sử dụng session của client đó để xác thực cho những lần request sau. Điều này ko đảm bảo an toàn.

> .antMatchers("/").permitAll()
> .anyRequest().authenticated();

Tất cả các request có path "/" thì ko cần xác thực. Còn những request có path khác thì bắt buộc phải xác thực.

> http.addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);

Có tác dụng như 1 filter để kiểm tra tất cả các request trc khi đến controller kể cả request có path = "/"

Tiếp tục đến class **JWTAuthenticationFilter**

```java
@Slf4j
public class JWTAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Autowired
    private UserDetailServiceImpl userDetailService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        if (request.getServletPath().equals("/login")&&request.getMethod().equals("POST")) {
            String username = request.getParameter("username");
            String password = request.getParameter("password");
            CustomUserDetail user = (CustomUserDetail) userDetailService.loadUserByUsername(username);
            System.out.println(tokenProvider.generateToken(user));
        } else {
            try {
                String jwt = getJwtFromRequest(request);
                if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
                    String idUser = tokenProvider.getUserFromJWT(jwt);
                    UserDetails userDetails = userDetailService.loadUserById(idUser);
                    if (userDetails != null) {
                        UsernamePasswordAuthenticationToken authentication
                                = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                        authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                        SecurityContextHolder.getContext().setAuthentication(authentication);
                    }
                }
            } catch (Exception e) {
                SecurityContextHolder.clearContext();
            }
        }
        filterChain.doFilter(request, response);
    }

    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

**JWTAuthenticationFilter** được extend từ **OncePerRequestFilter** có tác dụng kiểm tra từng request một trc khi đến controller. Sử dụng method **doFilterInternal** để thực hiện công việc kiểm tra này.

**UsernamePasswordAuthenticationToken** được kế thừa từ **Authentication** có tác dụng sử dụng Username và Password để xác thực người dùng. Username và Password được lấy từ DB hoặc dựa vào Claims của token sau khi được decode 

> UsernamePasswordAuthenticationToken authentication
>                                 = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
>
> SecurityContextHolder.getContext().setAuthentication(authentication);

Sau khi xác thực thành công một người dùng nó sẽ đưa quyền xác thực vào cho **SecurityContextHolder**

Cuối cùng là **JwtTokenProvider**

```java
@Component
@Slf4j
public class JwtTokenProvider {

    static final long EXPIRATION_TIME = 604800000L;

    private static final String SECRET = "A4343434343434343";

    public String generateToken(CustomUserDetail userDetail) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + EXPIRATION_TIME);
        return Jwts.builder()
                .setSubject(userDetail.getUser().get_id())
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, SECRET)
                .compact();
    }

    public String getUserFromJWT(String token) {
        return Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String authToken) {
        try {
            System.out.println(Jwts.parser().setSigningKey(SECRET).parseClaimsJws(authToken));
            return true;
        } catch (ExpiredJwtException | UnsupportedJwtException | MalformedJwtException | SignatureException | IllegalArgumentException e) {
            log.error("ERROR: ", e);
        }
        return false;
    }
}
```

Lớp này có nhiệm vụ là tạo token khi có requet gửi username và password vào "/login" với method POST, thực hiện kiểm tra tính hợp lệ của token, và cuối cùng là lấy ra các thông tin cần thiết của người dùng đã được mã hóa trong token để sử dụng cho việc thực hiện gọi APIs

Để Spring security kiểm tra và thực hiện xác thực được người dùng ta cần có thêm **UserDetails** và **UserDetailsService**. Đây là 2 interface build-in của Spring Security.

```java
@Getter
@Setter
@ToString
public class CustomUserDetail implements UserDetails {

    private User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

Implements **UserDetails** để tạo ra 1 người dùng có một số hành vi cơ bản như có đã hết hạn hay chưa đã được kích hoạt hay chưa, có những role gì ...

```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {

    private UserRepo userRepo;

    @Autowired
    public UserDetailServiceImpl(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        List<User> userList = userRepo.findByUsername(username);
        if (userList.size() == 0) {
            return null;
        } else {
            CustomUserDetail customUserDetail = new CustomUserDetail();
            customUserDetail.setUser(userList.get(0));
            return customUserDetail;
        }
    }

    public UserDetails loadUserById(String id) {
        CustomUserDetail customUserDetail = new CustomUserDetail();
        customUserDetail.setUser(userRepo.findBy_id(id));
        return customUserDetail;
    }
    
}
```

Implements **UserDetailsService** để tạo ra 1 lớp có chứ năng lấy thông tin của **UserDetails** 

Clone project về chạy thôi👏👏👏

DONE

