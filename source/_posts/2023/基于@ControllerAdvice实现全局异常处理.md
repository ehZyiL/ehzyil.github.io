---
title: '基于@ControllerAdvice实现全局异常处理'
author: ehzyil
tags:
  - SpringBoot
  - 异常
categories:
  - 技术
date: 2023-10-17
---
### RestControllerAdvice注解与全局异常处理

#### 1.前置知识

`@RestControllerAdvice` 和 `@ExceptionHandler` 是 Spring 框架中用于处理全局异常的注解。

1. **@RestControllerAdvice**:

   `@RestControllerAdvice` 是一个组合注解，结合了 `@ControllerAdvice` 和 `@ResponseBody`。它用于全局处理控制器层（Controller）抛出的异常，可以将处理结果直接以 JSON 格式返回给客户端。

   具体用途包括但不限于：

   - 全局异常处理：捕获所有 Controller 层抛出的异常，统一进行处理，避免异常直接传递到客户端。
   - 全局数据绑定：可以在这里对请求参数进行全局的预处理或校验。
   - 全局数据预处理：可以在这里对返回的数据进行统一处理。

2. **@ExceptionHandler**:

   `@ExceptionHandler` 是一个局部异常处理的注解，它可以用在控制器层的方法上，用于处理指定类型的异常。

   具体用途包括但不限于：

   - 处理特定类型的异常：将指定类型的异常捕获，然后执行相应的处理逻辑。
   - 提供自定义的异常处理逻辑：可以根据业务需求编写特定的异常处理代码。

   #### 2.举例

举例：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Internal Server Error");
    }

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFoundException(UserNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body("User Not Found");
    }
}
```

在上面的例子中，`GlobalExceptionHandler` 类使用 `@RestControllerAdvice` 注解来指示它是一个全局异常处理器。然后，使用 `@ExceptionHandler` 注解来定义处理特定类型异常的方法。例如，`handleException` 方法处理所有类型的异常，而 `handleUserNotFoundException` 方法只处理 `UserNotFoundException` 类型的异常。

总的来说，`@RestControllerAdvice` 和 `@ExceptionHandler` 的组合允许你在应用程序中建立一套全局的异常处理机制，提高了代码的可维护性和健壮性。

#### 3.代码实现

**自定义业务异常类ServiceException**

```java
package com.ehzyil.exception;

import lombok.Getter;

import static com.ehzyil.enums.StatusCodeEnum.FAIL;

/**
 * 业务异常
 **/
@Getter
public final class ServiceException extends RuntimeException {

    /**
     * 返回失败状态码
     */
    private Integer code = FAIL.getCode();

    /**
     * 返回信息
     */
    private final String message;

    public ServiceException(String message) {
        this.message = message;
    }

}
```



**全局异常处理类GlobalExceptionHandler.java**

```java
package com.ehzyil.handler;

import cn.dev33.satoken.exception.DisableServiceException;
import cn.dev33.satoken.exception.NotLoginException;
import cn.dev33.satoken.exception.NotPermissionException;
import cn.dev33.satoken.exception.NotRoleException;
import com.ehzyil.exception.ServiceException;
import com.ehzyil.model.vo.Result;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.Objects;

import static com.ehzyil.enums.StatusCodeEnum.*;

/**
 * @author ehyzil
 * @Description 全局异常处理
 * @create 2023-09-2023/9/30-22:47
 */
@RestControllerAdvice

public class GlobalExceptionHandler {
    /**
     * 处理业务异常
     *
     * @param e
     * @return
     */
    @ExceptionHandler(value = ServiceException.class)
    public Result<?> handleServiceException(ServiceException e) {
        return Result.fail(e.getMessage());
    }

    /**
     * 处理Assert异常
     */
    @ExceptionHandler(value = IllegalArgumentException.class)
    public Result<?> handleIllegalArgumentException(IllegalArgumentException e) {
        return Result.fail(e.getMessage());
    }

    /**
     * 处理参数校验异常
     */
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public Result<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        return Result.fail(VALID_ERROR.getCode(), Objects.requireNonNull(e.getBindingResult().getFieldError()).getDefaultMessage());
    }

    /**
     * 处理权限不足
     */
    @ExceptionHandler(value = NotPermissionException.class)
    public Result<?> handleNotPermissionException() {
        return Result.fail("权限不足");
    }

    /**
     * 处理账号封禁
     */
    @ExceptionHandler(value = DisableServiceException.class)
    public Result<?> handleDisableServiceExceptionException() {
        return Result.fail("此账号已被禁止访问服务");
    }

    /**
     * 处理无此角色异常
     */
    @ExceptionHandler(value = NotRoleException.class)
    public Result<?> handleNotRoleException() {
        return Result.fail("权限不足");
    }

    /**
     * 处理SaToken异常
     */
    @ExceptionHandler(value = NotLoginException.class)
    public Result<?> handlerNotLoginException(NotLoginException nle) {
        // 判断场景值，定制化异常信息
        String message;
        if (nle.getType().equals(NotLoginException.NOT_TOKEN)) {
            message = "未提供token";
        } else if (nle.getType().equals(NotLoginException.INVALID_TOKEN)) {
            message = "token无效";
        } else if (nle.getType().equals(NotLoginException.TOKEN_TIMEOUT)) {
            message = "token已过期";
        } else {
            message = "当前会话未登录";
        }
        // 返回给前端
        return Result.fail(UNAUTHORIZED.getCode(), message);
    }

    /**
     * 处理系统异常
     */
    @ExceptionHandler(value = Exception.class)
    public Result<?> handleSystemException() {
        return Result.fail(SYSTEM_ERROR.getCode(), SYSTEM_ERROR.getMsg());
    }

}
```

