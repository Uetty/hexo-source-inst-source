## spring项目构建



### 框架搭建

1. springboot-starter
2. mybatis、mybatis-config，及pagerHelper
3. log4j
4. job
5. 消息队列
6. shiro权限控制
7. 不同开发环境和生产环境配置文件分离
8. 事务



### 细项

1. 公共BaseController（公共注入与公共出参）

2. 公共异常处理exceptionhandler、自定义异常

3. 

   

   

   

#### 统一返回码枚举类（同时考虑国际化）

国际化文件，在`application.yml`中配置`spring.message.basename=i18n/message`表明前缀

resources文件夹下建i18n文件夹，并创建message.properties等国际化文件

```
public enum ResponseCode implements CodeableEnum {

    /** 成功 */ SUCCESS(1),
    /** 失败 */ ERROR(100_000),
    /* 100_001 - 100_999 系统级权限错误 */
    /* 登录失败 */ LOGIN_FAILED(100_001),
    /** 无权限 */ USER_NOT_PERMIT(100_002),

    /* 101_000 - 101_999 模块一定义的错误 */
    ;

    int code; // 返回码
    private static final String I18N_MESSAGE_KEY = "reponse.message.code_";

    ResponseCode(int code) {
        this.code = code;
    }

    @Override
    public String toString() {
        return "ResponseCode{" +
                "name=" + name() +
                ",code=" + code +
                '}';
    }

    private static MessageSource messageSource;

    @Override
    public int getCode() {
        return this.code;
    }

    public String getDefaultMessage() {
        return getDefaultMessage(Locale.CHINESE);
    }

    public String getDefaultMessage(Locale locale) {
        if (ApplicationVariable.APPLICATION_CONTEXT == null) {
            return null;
        }
        try {
            if (messageSource == null) {
                messageSource = ApplicationVariable.APPLICATION_CONTEXT.getBean(MessageSource.class);
            }
            return messageSource.getMessage(I18N_MESSAGE_KEY + code, null, locale);
        } catch (Exception ignore) {}
        return null;
    }
}
```

**BaseConsole.class**

```
protected Locale getLocale() {
	return httpServletRequest.getLocale();
}
```



#### mybatis枚举类处理

枚举类继承一个自定义接口，并定义对这个接口的解析类，在mybatis配置文件中定义映射

**CodeableEnum.class**

```
public interface CodeableEnum {
    int getCode();
}
```

**CodeableEnumTypeHandler.class**

```
public class CodeableEnumTypeHandler<T extends Enum<T>> extends BaseTypeHandler<T> {

    private final Class<T> type;

    public CodeableEnumTypeHandler(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Type argument cannot be null");
        }
        this.type = type;
    }

    @Override
    public void setNonNullParameter(PreparedStatement pstmt, int i, T t, JdbcType jdbcType) throws SQLException {
        if (isCodeable()) {
            CodeableEnum codeableEnum = (CodeableEnum) t;
            pstmt.setInt(i, codeableEnum.getCode());
        } else {
            pstmt.setInt(i, t.ordinal());
        }
    }

    private boolean isCodeable() {
        return CodeableEnum.class.isAssignableFrom(type);
    }

    @Override
    public T getNullableResult(ResultSet rs, String columnName) throws SQLException {
        int code = rs.getInt(columnName);
        if (code == 0 && rs.wasNull()) {
            return null;
        }
        return EnumUtil.valueByCodeOrOrdinal(code, type);
    }

    @Override
    public T getNullableResult(ResultSet rs, int i) throws SQLException {
        int code = rs.getInt(i);
        if (code == 0 && rs.wasNull()) {
            return null;
        }
        return EnumUtil.valueByCodeOrOrdinal(code, type);
    }

    @Override
    public T getNullableResult(CallableStatement cs, int i) throws SQLException {
        int code = cs.getInt(i);
        if (code == 0 && cs.wasNull()) {
            return null;
        }
        return EnumUtil.valueByCodeOrOrdinal(code, type);
    }
}
```

**mybatis-config.xml**

```
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeHandlers>
        <!-- 枚举变量指定处理器 -->
        <typeHandler handler="com.uetty.enums.CodeableEnumTypeHandler"
                     javaType="com.uetty.enums.CodeableEnum"/>

    </typeHandlers>
</configuration>
```

