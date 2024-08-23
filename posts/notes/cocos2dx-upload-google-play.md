---
title: Cocos2dx－上传Google Play遇到的问题
author: Finger
tags:
  - Cocos2d-x
  - Javascript
categories:
  - Note
date: 2017-01-27 20:40:00
---

#### 一、OpenSSL版本有多个安全漏洞，建议您尽快更新 OpenSSL

***
因为项目使用的是cocos2d-x 3.3版本。因此cocos2d-x第三方库比较旧。上传Google Play 后台时被rejected。邮件原文如下：

```
Vulnerability
-----------------------------------------------------------
**OpenSSL**

The vulnerabilities were addressed in OpenSSL 1.0.2f/1.0.1r. To confirm your OpenSSL version, you can do a grep search for:

\$ unzip -p YourApp.apk | strings | grep "OpenSSL"

You can find more information and next steps in this Google Help Center article.
-----------------------------------------------------------

To confirm you’ve upgraded correctly, submit the updated version of your app to the Play Console and check back after five hours to make sure the warning is gone.

While these vulnerabilities may not affect every app that uses this software, it’s best to stay up to date on all security patches. Make sure to update any libraries in your app that have known security issues, even if you're not sure the issues are relevant to your app.

Apps must also comply with the Developer Distribution Agreement and Developer Program Policies.

If you feel we have made this determination in error, please reach out to our developer support team.

Best,

The Google Play Team
```

**解决方案**

邮件写的很清楚，主要是OpenSSL的库版本太低，存在漏洞。cocos论坛上也有很多同学遇到类似的问题，传送门：http://discuss.cocos2d-x.org/t/the-vulnerabilities-were-addressed-in-openssl-1-0-2f-1-0-1r/35766/9

具体做法，就是从cocos的第三方库下载最新的curl库替换当前项目的curl。

https://github.com/cocos2d/cocos2d-x-3rd-party-libs-bin


#### 二、Google Play支付(in-app Billing)


**注意：在测试之前一定要注意后台APK版本号、签名，一定要与测试用的APK版本号签名一致**


1、错误提示

```

1. "需要验证身份，您需要登录自己google账号" // 没有成功发布apk测试(Alpha)版本
2. "无法购买您要买的商品" // 已成功发布Alpah版本。需要等待生效

```

2、系统异常

```
java.lang.IllegalStateException: Can't start async operation (launchPurchaseFlow)

// 解决方法
// 1、mHelper.flagEndAsync() 修改成public
// 2、在launchPurchaseFlow() 之前调用flagEndAsync()
----------------------------------------------------
if (mHelper != null) {
    try {
        mHelper.flagEndAsync();
        mHelper.launchPurchaseFlow(this, item, RC_REQUEST, mPurchaseFinishedListener, "");
    }       
    catch(IllegalStateException ex){
        Toast.makeText(this, "Please retry in a few seconds.", Toast.LENGTH_SHORT).show();
    }
}
```



