# OauthSDK,简易集成第三方登录
[![Software license][ico-license]](LICENSE)
[![Latest development][ico-version-dev]][link-packagist]


-----



----


# OauthSDK是什么？

是第三方登录SDK。︿(￣︶￣)︿

## 安装

通过composer，这是推荐的方式，可以使用composer.json 声明依赖，或者直接运行下面的命令。

```php
    composer require "sujun/oauthsdk": "v1.0.1"

```

放入composer.json文件中

```php
    "require": {
         "sujun/oauthsdk": "v1.0.1"
    }
```

然后运行

```
composer update
```

#使用方式
分三步走。

# 1、登录处放置第三方登录入口
传type变量，根据此变量判断是QQ、微博、还是微信登录。
如下：login.html 页面
```
<a href="/login?type=qq">QQ登录</a>
<a href="/login?type=wechat">微信登录</a>
<a href="/login?type=sina">新浪登录</a>

```


#2、生成第三方登录授权地址
如下：login.php 页面
```

use OauthSDK\Oauth;

try {

    if (!in_array($_GET['type'], ['qq', 'wechat', 'sina'])) {
        throw new \Exception("暂不支持{$_GET['type']}方式登录", 500);
    }
    $sns = Oauth::getInstance($_GET['type'], array(
        //腾讯QQ登录配置
        'QQ' => array(
            'APP_KEY' => '***', //应用注册成功后分配的 APP ID
            'APP_SECRET' => '***', //应用注册成功后分配的KEY
            'CALLBACK' => '/callback.php?type=qq', //回调URL
        ),
        //新浪微博配置
        'SINA' => array(
            'APP_KEY' => '***', 
            'APP_SECRET' => '***', 
            'CALLBACK' => '/callback.php?type=sina', 
        ),
        //腾讯微信配置
        'WECHAT' => array(
            'APP_KEY' => '***', 
            'APP_SECRET' => '***', 
            'CALLBACK' => '/callback.php?type=wechat', 
        ),
    ));
    //跳转到授权页面
    header('Location: ' . $sns->getRequestCodeURL());

} catch (\Exception $e) {
    echo $e->getMessage();
    header('Location: /login' . );
}


```


#3、设置回调地址

```
use OauthSDK\Oauth;


try {

    $type = $_GET['type'];
    $code = $_GET['code'];

    if (!in_array($type, ['qq', 'wechat', 'sina'])) {
        throw new \Exception("参数错误", 500);
    }
    if(empty($type) || empty($code)) {
    	throw new \Exception('参数错误~',500);
    }
    $sns = Oauth::getInstance($type, array(
        //腾讯QQ登录配置
        'QQ' => array(
            'APP_KEY' => '***', //应用注册成功后分配的 APP ID
            'APP_SECRET' => '***', //应用注册成功后分配的KEY
            'CALLBACK' => '/callback.php?type=qq', //回调URL
        ),
        //新浪微博配置
        'SINA' => array(
            'APP_KEY' => '***', 
            'APP_SECRET' => '***', 
            'CALLBACK' => '/callback.php?type=sina', 
        ),
        //腾讯微信配置
        'WECHAT' => array(
            'APP_KEY' => '***', 
            'APP_SECRET' => '***', 
            'CALLBACK' => '/callback.php?type=wechat', 
        ),
    ));
    $tokenArr = $sns->getAccessToken($code, []);

    $openid = $tokenArr['openid'];
    $token = $tokenArr['access_token'];

    /*
    $checkData = select * from where openid=$tokenArr['openid'] and type_name = $type;
	if ($_check_data) {
	    //之前绑定
	    //跳至登录
	} */
  
    //获取当前登录用户信息
    if ($openid) {
        $userinfo = $sns->getUserInfo();
        //处理......
    } else {
        throw new \Exception("系统出错,请稍后再试！", 500);
    }

} catch (\Exception $e) {
    echo $e->errorMessage();
    exit;
}


```


至此完成第三方登录。很多人做法是用绑定的形式，即必须要有本站账号，这时要建一个表存着Openid关系。
表设置可如下:

```
DROP TABLE IF EXISTS `oauth_login`;
CREATE TABLE `oauth_login` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL COMMENT '本站用户',
  `openid` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '第三方用户id',
  `unionid` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '微信多平台唯一',
  `type_name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '类型:qq、wechat、sina,
  `access_token` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '访问令牌',
  `param` text COLLATE utf8_unicode_ci COMMENT '返回参数',
  `create_time` datetime DEFAULT NULL,
  `status` tinyint(1) NOT NULL COMMENT '0:正常，1：删除',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci COMMENT='第三方绑定登录用户表';


```


[ico-license]: https://img.shields.io/github/license/helei112g/payment.svg
[ico-version-dev]: https://img.shields.io/packagist/vpre/riverslei/payment.svg


[link-packagist]: https://packagist.org/packages/sujun/oauthsdk
