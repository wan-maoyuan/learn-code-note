## spring boot controller层全局异常的统一处理

- 先定义好所有的状态码
```java
@Getter
public enum ResultCode{
    //操作成功
    SUCCESS(200, "请求成功"),
    //操作失败
    FAILED(401, "操作失败"),
    //未授权
    UNAUTHORIZED(403, "未授权");

    private Integer code;
    private String msg;

    ResultCode(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

}
```

- 在定义好统一的前端数据返回的格式
```java
@Getter
public class ResultData<T> implements Serializable {
    /**
     * 返回状态码：默认200
     */
    private Integer code;
    /**
     * 返回结果信息
     */
    private String message;
    /**
     * 返回数据
     */
    private T data;

    public void setResultCode(ResultCode resultCode){
        this.code = resultCode.getCode();
        this.message = resultCode.getMsg();
    }

    public void setData(T data){
        this.data = data;
    }

}
```

- 定义一个自己的异常类，或者第三方等等其他的异常类
```java
@Getter
public class UserException extends RuntimeException {

    private int code;
    private String msg;

    // 默认异常使用APP_ERROR状态码
    public UserException(String message) {
        super(message);
        this.code = ResultCode.FAILED.getCode();
        this.msg = ResultCode.FAILED.getMsg();
    }

}
```

- 最后定义全局的异常处理类
```java
@RestControllerAdvice
public class ControllerExceptionAdvice {

    /**
     * 全局认证异常处理
     * @param e
     * @return
     */
    @ExceptionHandler(AuthenticationException.class)
    public ResultData<String> authenticationExceptionHandler(AuthenticationException e) {
        ResultData<String> resultData = new ResultData();
        resultData.setResultCode(ResultCode.FAILED);
        resultData.setData(e.getMessage());
        return resultData;
    }

    /**
     * 全局用户信息异常处理
     * @param e
     * @return
     */
    @ExceptionHandler(UserException.class)
    public ResultData<String> userExceptionHandler(UserException e) {
        ResultData<String> resultData = new ResultData();
        resultData.setResultCode(ResultCode.FAILED);
        resultData.setData(e.getMessage());
        return resultData;
    }

}
```

## 使用方法,直接抛出异常即可
```java
    @SneakyThrows
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        log.info("登录的用户名 = {}， 密码 = {}", authentication.getName(), authentication.getCredentials().toString());
        // 获取用户登录时输入的用户名
        String username = authentication.getName();
        // 根据用户名查询系统中的用户信息
        UserDao userDao = userMapper.selectOne(new QueryWrapper<UserDao>()
                .eq("UserName", username));
        // 如果用户列表为空，说明没有匹配的用户，抛出 UsernameNotFoundException
        if (Objects.isNull(userDao)) {
            throw new UsernameNotFoundException(String.format("没有找到该用户[%s]!", username));
        }
        // 如果用户没有设置启用或禁用状态，或者用户被设为禁用，则抛出 DisabledException
        Optional<Boolean> enabled = Optional.of(userDao.isEnabled());
        if (!enabled.orElse(false)) {
            throw new DisabledException(String.format("用户[%s] 账户未启用!", username));
        }
        // 如果用户没有过期状态或过期状态为 true 则抛出 AccountExpiredException
        Optional<Boolean> expired = Optional.of(userDao.isAccountNonExpired());
        if (expired.orElse(true)) {
            throw new AccountExpiredException(String.format("用户[%s] 账号信息过期!", username));
        }
        // 如果用户没有锁定状态或锁定状态为 true 则抛出 LockedException
        Optional<Boolean> locked = Optional.of(userDao.isAccountNonLocked());
        if (locked.orElse(true)) {
            throw new LockedException(String.format("用户[%s] 账号被锁定!", username));
        }
        // 如果用户登录时输入的密码和系统中密码匹配，则返回一个完全填充的 Authentication 对象
        if (passwordEncoder.matches(authentication.getCredentials().toString(), userDao.getPassword())){
            loadRedis(userDao.getUserId());
            return new UsernamePasswordAuthenticationToken(authentication, authentication.getCredentials(), new ArrayList<>());
        }
        // 如果密码不匹配则返回 null（此处可以抛异常，试具体应用场景而定）
        return null;
    }
```