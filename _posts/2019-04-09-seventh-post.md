---
title: RestControllerAdvice
author: Sophosss
layout: post
---

### @RestControllerAdvice

我们可以利用该注解实现异常全局统一处理，而不必在单个controller中去处理。

当我们定义了一个自定义返回参数格式时，希望得到统一的返回，如果在运行时发现了异常，也希望将异常统一返回。那么，如何让我们的异常得到期望的返回格式，这里就需要用到了`@ControllerAdvice`或者`@RestControllerAdvice`。如果全部异常处理返回 Json，那么可以使用 `@RestControllerAdvice` 代替` @ControllerAdvice` ，这样在方法上就可以不需要添加 `@ResponseBody`。

下面给出我的样例demo：

1. 创建自定义异常类

   ```java
   public class ADFException extends RuntimeException {
   
       /**
        * 具体的错误描述
        */
       private final String errorDetail;
       private final ErrorCode errorCode;
   
       public ADFException() {
           this.errorCode = null;
           this.errorDetail = null;
       }
   
       public ADFException(ErrorCode errorCode) {
           this.errorCode = errorCode;
           this.errorDetail = null;
       }
   
       public ADFException(ErrorCode errorCode, String errorDetail) {
           this.errorDetail = errorDetail;
           this.errorCode = errorCode;
       }
   
       public String getErrorDetail() {
           return errorDetail;
       }
   
       public ErrorCode getErrorCode() {
           return errorCode;
       }
   
   }
   ```

   Spring 对于 Runtime Exception 类的异常才会进行事务回滚，所以我们一般自定义异常都继承该异常类。

   

2. 创建错误代码枚举类

   ```java
   public enum ErrorCode {
   
       DELETE_GROUP_APP_FAILURE_EXCEPTION("DELETE_GROUP_APP_FAILURE_EXCEPTION", "在分组中删除应用失败"),
       ADD_GROUP_APP_FAILURE_EXCEPTION("ADD_GROUP_APP_FAILURE_EXCEPTION", "在分组中添加应用失败"),
       APP_EXIST_EXCEPTION("APP_EXIST_EXCEPTION", "应用已经存在"),
       DELETE_GROUP_FAILURE_EXCEPTION("DELETE_GROUP_FAILURE_EXCEPTION", "删除自定义分组失败"),
       UPDATE_GROUP_FAILURE_EXCEPTION("UPDATE_GROUP_FAILURE_EXCEPTION", "更新自定义分组失败"),
       ADD_GROUP_FAILURE_EXCEPTION("ADD_GROUP_FAILURE_EXCEPTION", "新建自定义分组失败"),
       GROUP_NAME_EXIST_EXCEPTION("GROUP_NAME_EXIST_EXCEPTION", "分组名称已经存在"),
       SYSTEM_EXCEPTION("SYSTEM_EXCEPTION", "系统异常"),
       USER_NOT_LOGIN_EXCEPTION("USER_NOT_LOGIN_EXCEPTION", "用户未登录");
   
       private String code;
       private String desc;
   
       ErrorCode(String code, String desc) {
           this.code = code;
           this.desc = desc;
       }
   
       public String getCode() {
           return code;
       }
   
       public String getDesc() {
           return desc;
       }
   
       public static String getDescByValue(String value) {
           for (ErrorCode enums : ErrorCode.values()) {
               if (enums.getCode() == value) {
                   return enums.getDesc();
               }
           }
           return "";
       }
   
   }
   ```

   

3. 创建全局异常处理类

   ```java
   @RestControllerAdvice
   public class ADFExceptionHandler {
       private final static Logger logger = LoggerFactory.getLogger(ADFExceptionHandler.class);
   
       @ExceptionHandler(value = Exception.class)
       @ResponseBody
       public ResponseResult handle (Exception e) {
           
           ResponseResult result = new ResponseResult();
           result.setSuccessful(false);
           
           if (e instanceof ADFException) { // 系统异常
               ADFException exception = (ADFException)e;
               logger.error(exception.getErrorCode().getDesc() + exception.getMessage());
               result.setResultHint(exception.getErrorCode().getDesc());
           } else { // 未知异常
               logger.error(ErrorCode.SYSTEM_EXCEPTION.getDesc() + e.getMessage());
               result.setResultHint(ErrorCode.SYSTEM_EXCEPTION.getDesc());
           }
           return result;
       }
   }
   ```

   