---
layout:     post
title:      Shiro Quickstart
subtitle:   Shiro Quickstart
date:       2019-07-17
author:    Sunsj
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Java
---

> 准备配置文件 模拟用户和角色

shiro.ini

```lang=ini
[users]
# username = password , role1 , role2  
root = secret, admin
guest = guest, guest
presidentskroob = 12345, president
darkhelmet = ludicrousspeed, darklord, schwartz
lonestarr = vespa, goodguy, schwartz

[roles]
#  role =  permissions
admin = *
schwartz = lightsaber:*
goodguy = winnebago:drive:eagle5


```

#### 1、securityManager 装配

```lang=java
 SecurityUtils.setSecurityManager(securityManager);
```

#### 2、获取Subject 主体

get the currently executing user: 

```lang=java

Subject currentUser = SecurityUtils.getSubject();

```

#### 3、处理shiro的session
> Do some stuff with a Session (no need for a web or EJB container!!!)

```lang=java
Session session = currentUser.getSession();
session.setAttribute("someKey", "aValue");
String value = (String) session.getAttribute("someKey");
if (value.equals("aValue")) {
    log.info("Retrieved the correct value! [" + value + "]");
}

```

#### 4、登录当前用户，以便检查角色和权限：
> let's login the current user so we can check against roles and permissions:

```lang=java
// 判断是否认证
if (!currentUser.isAuthenticated()) {
            // 设置用户Token 
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);
            try {
                // 登陆验证
                currentUser.login(token);
            } catch (UnknownAccountException uae) {
                // 用户不存在异常
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {
                // 密码错误
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {
                // 用户被锁定
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            catch (AuthenticationException ae) {
                // 其他异常 AuthenticationException 是以上几个异常共同的爷爷辈
                //unexpected condition?  error?
            }
}
// 再次判断是否认证 
System.out.println("currentUser.isAuthenticated():" + currentUser.isAuthenticated());   // 输出: currentUser.isAuthenticated():true

System.out.println("currentUser.getPrincipal():" + currentUser.getPrincipal()); // 输出: currentUser.getPrincipal():lonestarr
```

#### 5、测试是否有某role

> test a role:

```lang=java
if (currentUser.hasRole("schwartz")) {
    log.info("May the Schwartz be with you!");
} else {
    log.info("Hello, mere mortal.");
}

```

#### 5、测试某 permission 是否被允许

> test a typed permission 

```lang=java
// (not instance-level) 没有精确到实例
if (currentUser.isPermitted("lightsaber:wield")) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}


// Instance Level permission:  精确到实例,实例ID为`eagle5`  
if (currentUser.isPermitted("winnebago:drive:eagle5")) {
    log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
            "Here are the keys - have fun!");
} else {
    log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
}


```

#### 6、退出登陆

```lang=java

//all done - log out!
currentUser.logout();

```