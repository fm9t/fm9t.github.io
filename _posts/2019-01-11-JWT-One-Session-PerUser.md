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

###IdentityServer4 添加iat
IdentityServer4生成的JWT里有可能没有iat内容，此时可以使用以下方式添加：

    services.AddIdentityServer(
        .......
    )
    ......
    .AddProfileService<ProfileService>();

    ProfileService Class:
    public class ProfileService : IProfileService
    {
        private long GetUnixTimestamp()
        {
            return (DateTime.Now.ToUniversalTime().Ticks - 
                621355968000000000L) / 10000000L;
        }

        public async Task GetProfileDataAsync(
            ProfileDataRequestContext context)
        {
            try
            {
                Claim issueAt = new Claim(JwtClaimTypes.IssuedAt,
                    GetUnixTimestamp().ToString());

                context.IssuedClaims.Add(issueAt);
            }
            catch(Exception error)
            {
                //log the error
            }
        }

        public async Task IsActiveAsync(IsActiveContext context)
        {
            context.IsActive = true;
        }
    }