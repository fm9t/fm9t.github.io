---
layout:     post
title:      JWT分发新Token时失效旧Token
subtitle:   JWT分发新Token时失效旧Token
date:       2019-01-11
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - C#
    - JWT
---
# JWT分发新Token时失效旧Token
JWT是无状态的，但其中可以带上iat字段(即Issue At时间)，每次用户申请token时，记录下该用户的last iat时间，后续token校验时，检查token的iat时间，如果iat时间小于用户的last iat时间，则token无效。