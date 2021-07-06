---
title: OOS与OAuth2.0
date: 2021-07-5 15:38:06
---

### OOS（Single Sign On）
相互信任的系统之间，实现一次登录之后各个服务之间无需重复登录。需要抽离单独的认证授权服务，用户在Authentication Server获取token，访问每个系统资源都需要携带token，
被访问系统拿token去Authentication Server认证，完成资源的访问与权限。

### OAuth2.0（Open Authority）
用于实现第三方授权，实现访问第三方资源，各个系统之间可以不存在关系，每个接入的系统作为第三方服务的client，常用的授权方式1.授权码模式（Authorization Code）2.密码模式（Resource owner password Credentials）
