---
title: 接入jira的oauth认证
date: 2018-02-16 15:45:28
tags:
- jira
- oauth
---

jira是一个敏捷开发的软件，主要用户管理需求和工作流。

有时候需要外部系统使用jira里的对应用户的权限，来对里面的需求单和工作流进行修改。
这个时候就要使用oauth来从jira获取授权。

## Oauth

先简单说一下Oauth流程。

[]

如图所示，主要就是三步。
1）使用consumer key和secret，以及公钥，发起请求得到一个request token，生成一个url。
2）之后用户在浏览器使用该url跳转到jira进行授权。
3）使用该授权的request token，去申请一个access token。后续使用access token来进行rest api的操作。
   如果request token过期或者授权失败，则重复第一步。


## Jira 配置

jira上面首先要对客户端（consumer）进行配置。（如果使用中文版，这一步可能会踩坑，jira的中文翻译包完全是机翻）

这里直接给出截图参考。

1） 进入应用程序链接（application link），添加一个新的链接。

[]

2） 选择进入的程序。这里刷新很慢可以喝杯咖啡等待一下。

[]

3） 配置公钥，key，和secret

[]

收工。

## 认证代码

核心代码如下。另外官方也给出了一些实例代码。

官方代码可以参考以下链接

https://bitbucket.org/atlassian_tutorial/atlassian-oauth-examples/


（django代码）

```
import os
import sys
import base64
import urlparse

from tlslite.utils import keyfactory
import oauth2 as oauth
from django.conf import settings
from django.http import HttpResponse,HttpResponseRedirect

# 配置在Jira,另外回调的Url也要在Jira上配置
consumer_key = settings.CONSUMER_KEY
consumer_secret = settings.CONSUMER_SECRET
# 私钥
pem_file = os.path.join(BASE_DIR, 'website/common/jira.pem')

# Oauth 认证Url
request_token_url = '{0}/plugins/servlet/oauth/request-token'.format(jira_url)
access_token_url = '{0}/plugins/servlet/oauth/access-token'.format(jira_url)
authorize_url = '{0}/plugins/servlet/oauth/authorize'.format(jira_url)


#签名加密方法
class SignatureMethod_RSA_SHA1(oauth.SignatureMethod):
    name = 'RSA-SHA1'

    def signing_base(self, request, consumer, token):
        if not hasattr(request, 'normalized_url') or request.normalized_url is None:
            raise ValueError("Base URL for request is not set.")

        sig = (
            oauth.escape(request.method),
            oauth.escape(request.normalized_url),
            oauth.escape(request.get_normalized_parameters()),
        )

        key = '%s&' % oauth.escape(consumer.secret)
        if token:
            key += oauth.escape(token.secret)
        raw = '&'.join(sig)
        return key, raw

    def sign(self, request, consumer, token):
        """Builds the base signature string."""
        key, raw = self.signing_base(request, consumer, token)

        with open(pem_file, 'r') as f:
            data = f.read()
        privateKeyString = data.strip()

        privatekey = keyfactory.parsePrivateKey(privateKeyString)
        signature = privatekey.hashAndSign(raw)

        return base64.b64encode(signature)


def request_token(request):
    """
    Step 1: 生成url，向RP申请request token
    """
    consumer = oauth.Consumer(consumer_key, consumer_secret)
    client = oauth.Client(consumer)
    client.set_signature_method(SignatureMethod_RSA_SHA1())
    resp, content = client.request(request_token_url, "POST")
    # consumer key或者name有问题
    if resp['status'] != '200':
        # Exception: Invalid response 401: oauth_problem=consumer_key_unknown
        print "Invalid response %s: %s" % (resp['status'],  content)
        return {'status':False, 'status': resp['status'], 'content':content, 'message':'请求被拒绝，请确认配置的client key和name是否正确'}

    request_token = dict(urlparse.parse_qsl(content))

    # print "Request Token:"
    # print "    - oauth_token        = %s" % request_token['oauth_token']
    # print "    - oauth_token_secret = %s" % request_token['oauth_token_secret']
    # print

    # Step 2: 浏览器重定向到url，等待授权，之后使用session的request_token，获取access_token

    url = "%s?oauth_token=%s" % (authorize_url, request_token['oauth_token'])
    request.session['request_token'] = request_token['oauth_token']
    request.session['request_token_secret'] = request_token['oauth_token_secret']

    return {'status':True, 'redirect_url':url, 'request_token':request_token['oauth_token'], 'request_secret':request_token['oauth_token_secret'] }

def access_token(request):
    """
    Step 3: 使用授权的request_token,申请真正的access_token,
            把access_token保存在数据库
    """

    request_token = request.session.get('request_token')
    request_token_secret = request.session.get('request_token_secret')
    username = request.user.username

    #session 中不存在token，重定向到申请页面
    if not request_token or not request_token_secret:
        return {'status':False, 'message':'请求授权信息丢失，请重新连接Jira授权'}
    
    consumer = oauth.Consumer(consumer_key, consumer_secret)
    token = oauth.Token(request_token, request_token_secret)
    # token.set_verifier(request.GET['oauth_verifier'])
    client = oauth.Client(consumer, token)
    client.set_signature_method(SignatureMethod_RSA_SHA1())
   
    resp, content = client.request(access_token_url, "POST")
    if resp['status'] != '200':
        # Invalid response 401: oauth_problem=token_rejected
        return {'status':False, 'message':'请求授权信息过期:{0}，请重新连接Jira授权'.format(content)}

    access_token = dict(urlparse.parse_qsl(content))
    # 如果到这一步 用户没有点击授权
    # Access Token: {'oauth_problem': 'permission_unknown'}
    if access_token.get('oauth_problem') or  not access_token.get('oauth_token'):
        return {'status':False, 'message':'请求Access Token失败，请确认是否在Jira上点击了‘允许’'}

    # print "Access Token:"
    # print "    - oauth_token        = %s" % access_token['oauth_token']
    # print "    - oauth_token_secret = %s" % access_token['oauth_token_secret']
    # print
    print "You may now access protected resources using the access tokens above."
    print

    # 测试授权正确, 使用user页面测试
    accessToken = oauth.Token(access_token['oauth_token'], access_token['oauth_token_secret'])
    client = oauth.Client(consumer, accessToken)
    client.set_signature_method(SignatureMethod_RSA_SHA1())

    resp, content = client.request("{0}/rest/api/2/user?username={1}".format(jira_url, username), "GET")
    if resp['status'] != '200':
        print resp, content
        return {'status':False, 'message':'请求授权信息已失效:{0}，请重新连接Jira授权'.format(content)}

    # save db
    old = None
    expire_time = datetime.now() + timedelta(seconds=int(access_token['oauth_expires_in']))
    try:
        old = UserJiraToken.objects.get(username = username)
    except:
        # new
        UserJiraToken(username = username, access_token = access_token['oauth_token'], 
          access_secret=access_token['oauth_token_secret'], expire_time=expire_time).save()
    else:
        # update
        old.access_token = access_token['oauth_token']
        old.access_secret = access_token['oauth_token_secret']
        old.expire_time = expire_time
        old.save()

    return {
      'status':True,
      'access_token': access_token['oauth_token'],
      'access_secret': access_token['oauth_token_secret'],
    }


```

## 其他判断

对于没有token的用户需要给出提示，并让其跳转到jira上进行赋权，跳出网站之后，也要有响应的恢复操作可以让用户跳回原站点。

如果把鉴权判断放在提交表单时的操作，还要注意恢复表单的填写项。

请求连接的客户端还需要判断token是否过期，如果过期，jira会返回403




