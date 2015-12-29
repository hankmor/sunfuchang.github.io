---
layout: post
category : 技术摘录
tagline: 
tags : [mybatis, SQL, 动态]
excerpt : 
title_cn: mybatis——动态SQL
description: 最近开发微信活动的时候，发现分享到朋友圈和发送给朋友功能不稳定，时而有效时而无效，打开debug模式查看，发现多数报的数签名错误，仔细看了官方文档，最终问题出在ticket的缓存上。
---
{% include JB/setup %}

最近开发微信活动的时候，发现分享到朋友圈和发送给朋友功能不稳定，时而有效时而无效，打开·debug·模式查看，发现多数报的数签名错误，仔细看了官方文档，最终问题出在ticket的缓存上。

## 1、签名：

* `url`：
需要根据不同的页面动态获取，`url`不能进行`encodeURIComponent`，否则验签会失败
`url`不能包括微信添加的#后边的部分，所以应该处理为：`window.location.href.split('#')[0]`
签名用的`url`必须是调用JS接口页面的完整URL。
* `nonceStr`、`timestamp`：应该动态生成，而不能hardcode
签名用的`noncestr`和`timestamp`必须与`wx.config`中的`nonceStr`和`timestamp`相同
* 为安全考虑，签名必须在后台进行，其他调用js在前台进行。

## 2、ticket和accesstoken：
* `accesstoken`：同调用微信其他接口的`accesstoken`，必须全局缓存，以免影响其他业务，即是说：微信所有业务应该用同一个`accesstoken`去调用微信接口，而不能自己刷新`accesstoken`。
* `ticket`：同`accesstoken`一样，必须全局缓存，方式很多，可以放到数据库，或者放到缓存。目前`ticket`的有效时间为2小时，所以2小时内`ticket`未过期时，不能重复获取，否则可能导致`ticket`获取次数超过限额，导致`sign`失败。

## 3、注意代码执行顺序
首先应该获取签名，签名获取后在调用`wx.config`方法，然后再执行`wx.ready`、`wx.error`方法。

## 4、其他
每个页面加载完成后都应该重新从后台获取签名信息，避免签名失败

具体开发步骤详见<a target="_blank" href="http://mp.weixin.qq.com/wiki/11/74ad127cc054f6b80759c40f77ec03db.html">官方文档</a>

## 5、官方常见问题及处理方法：

调用config 接口的时候传入参数 debug: true 可以开启debug模式，页面会alert出错误信息。以下为常见错误及解决方法：
### 1、invalid url domain
当前页面所在域名与使用的appid没有绑定，请确认正确填写绑定的域名，如果使用了端口号，则配置的绑定域名也要加上端口号（一个appid可以绑定三个有效域名，见 目录1.1.1）。
### 2、invalid signature签名错误。
建议按如下顺序检查：
1. 确认签名算法正确，可用 http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign 页面工具进行校验。
2. 确认config中nonceStr（js中驼峰标准大写S）, timestamp与用以签名中的对应noncestr, timestamp一致。
3. 确认url是页面完整的url(请在当前页面alert(location.href.split('#')[0])确认)，包括'http(s)://'部分，以及'？'后面的GET参数部分,但不包括'#'hash后面的部分。
4. 确认 config 中的 appid 与用来获取 jsapi_ticket 的 appid 一致。
5. 确保一定缓存access_token和jsapi_ticket。
6. 确保你获取用来签名的url是动态获取的，动态页面可参见实例代码中php的实现方式。如果是html的静态页面在前端通过ajax将url传到后台签名，前端需要用js获取当前页面除去'#'hash部分的链接（可用location.href.split('#')[0]获取,而且需要encodeURIComponent），因为页面一旦分享，微信客户端会在你的链接末尾加入其它参数，如果不是动态获取当前链接，将导致分享后的页面签名失败。

### 3、the permission value is offline verifying
这个错误是因为config没有正确执行，或者是调用的JSAPI没有传入config的jsApiList参数中。建议按如下顺序检查：
1. 确认config正确通过。
2. 如果是在页面加载好时就调用了JSAPI，则必须写在wx.ready的回调中。
3. 确认config的jsApiList参数包含了这个JSAPI。

### 4、permission denied
该公众号没有权限使用这个JSAPI，或者是调用的JSAPI没有传入config的jsApiList参数中（部分接口需要认证之后才能使用）。

### 5、function not exist
当前客户端版本不支持该接口，请升级到新版体验。

### 6、为什么6.0.1版本config:ok，但是6.0.2版本之后不ok
因为6.0.2版本之前没有做权限验证，所以config都是ok，但这并不意味着你config中的签名是OK的，请在6.0.2检验是否生成正确的签名以保证config在高版本中也ok。

### 7、在iOS和Android都无法分享
请确认公众号已经认证，只有认证的公众号才具有分享相关接口权限，如果确实已经认证，则要检查监听接口是否在wx.ready回调函数中触发

### 8、服务上线之后无法获取jsapi_ticket，自己测试时没问题。
因为access_token和jsapi_ticket必须要在自己的服务器缓存，否则上线后会触发频率限制。请确保一定对token和ticket做缓存以减少2次服务器请求，不仅可以避免触发频率限制，还加快你们自己的服务速度。目前为了方便测试提供了1w的获取量，超过阀值后，服务将不再可用，请确保在服务上线前一定全局缓存access_token和jsapi_ticket，两者有效期均为7200秒，否则一旦上线触发频率限制，服务将不再可用。

### 9、uploadImage怎么传多图
目前只支持一次上传一张，多张图片需等前一张图片上传之后再调用该接口

### 10、没法对本地选择的图片进行预览
chooseImage接口本身就支持预览，不需要额外支持

### 11、通过a链接(例如先通过微信授权登录)跳转到b链接，invalid signature签名失败
后台生成签名的链接为使用jssdk的当前链接，也就是跳转后的b链接，请不要用微信登录的授权链接进行签名计算，后台签名的url一定是使用jssdk的当前页面的完整url除去'#'部分

### 12、出现config:fail错误
这是由于传入的config参数不全导致，请确保传入正确的appId、timestamp、nonceStr、signature和需要使用的jsApiList

### 13、如何把jsapi上传到微信的多媒体资源下载到自己的服务器
请参见文档中uploadVoice和uploadImage接口的备注说明

### 14、Android通过jssdk上传到微信服务器，第三方再从微信下载到自己的服务器，会出现杂音
微信团队已经修复此问题，目前后台已优化上线

### 15、绑定父级域名，是否其子域名也是可用的
是的，合法的子域名在绑定父域名之后是完全支持的

### 16、在iOS微信6.1版本中，分享的图片外链不显示，只能显示公众号页面内链的图片或者微信服务器的图片，已在6.2中修复

### 17、是否需要对低版本自己做兼容
jssdk都是兼容低版本的，不需要第三方自己额外做更多工作，但有的接口是6.0.2新引入的，只有新版才可调用
### 18、该公众号支付签名无效，无法发起该笔交易
请确保你使用的jweixin.js是官方线上版本，不仅可以减少用户流量，还有可能对某些bug进行修复，拷贝到第三方服务器中使用，官方将不对其出现的任何问题提供保障，具体支付签名算法可参考 JSSDK微信支付一栏

### 19、目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题已在Android6.2中修复

### 20、uploadImage在chooseImage的回调中有时候Android会不执行
Android6.2会解决此问题，若需支持低版本可以把调用uploadImage放在setTimeout中延迟100ms解决

### 21、require subscribe错误说明你没有订阅该测试号，该错误仅测试号会出现

### 22、getLocation返回的坐标在openLocation有偏差
因为getLocation返回的是gps坐标，openLocation打开的腾讯地图为火星坐标，需要第三方自己做转换，6.2版本开始已经支持直接获取火星坐标

### 23、查看公众号（未添加）: "menuItem:addContact"不显示
目前仅有从公众号传播出去的链接才能显示，来源必须是公众号

### 24、ICP备案数据同步有一天延迟，所以请在第二日绑定

## 6、示例：

### 1、后台获取ticket方法：
<pre>
/**
 * 缓存微信jssdk调用的ticket。
 *
 * @param appid
 * @return
 * @throws IOException
 */
public String cacheJsSDKTicket(String appid) throws IOException {
    Assert.hasLength(appid);
    String cachedTicket = spyMemcachedClient.get(WechatConst.Cache.CACHE_KEY + "TICKET");
    if (cachedTicket != null && !"".equals(cachedTicket)) {
        LOG.info("Ticket exists, return directly ...");
        return cachedTicket;
    }
    LOG.info("Ticket not exists, get from weixin ...");
    String accessToken = weChatApi.getAccessToken(appid);
    String ticket = weChatApi.getJsApiTicket(accessToken);
    // 7200秒过期
    int timeOut = 7200;
    spyMemcachedClient.safeSet(WechatConst.Cache.CACHE_KEY + "TICKET", timeOut, ticket);
    return ticket;
}
</pre>

这里我将ticket全局缓存到memorycache中。

### 2、后台签名方法：

<pre>
@Override
    public String signJsSDK(String ticket, String url, String timestamp, String nonceStr) throws NoSuchAlgorithmException {
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("noncestr", nonceStr);
    map.put("jsapi_ticket", ticket);
    map.put("timestamp", timestamp);
    map.put("url", url);
    String[] ss = {"noncestr", "jsapi_ticket", "timestamp", "url"};
    Arrays.sort(ss);
    String signStr = "";
    for (String s : ss) {
        signStr += s + "=" + map.get(s) + "&";
    }
    signStr = signStr.substring(0, signStr.length() - 1);
    return new SHA1().getDigestOfString(signStr.getBytes());
}
</pre>

### 3、获取签名方法：
public void getJsSdkInfo(CspServiceContext serviceContext) throws IOException, NoSuchAlgorithmException {
    MessageObject mo = serviceContext.getRequestData();
    String url = (String) mo.getValue("url");
    String appid = (String) mo.getValue("appid");
    String timestamp = Long.toString(System.currentTimeMillis() / 1000);
    String nonceStr = UUID.randomUUID().toString(); 
    // 缓存ticket
    String ticket = memoryCacheManager.cacheJsSDKTicket(appid);
    // 签名
    String sign = weChatApi.signJsSDK(ticket, url, timestamp, nonceStr); 
    Map<String, Object> resMap = new HashMap<String, Object>();
    resMap.put("appid", appid);
    resMap.put("ticket", ticket);
    resMap.put("sign", sign);
    resMap.put("nonceStr", nonceStr);
    resMap.put("timestamp", timestamp);

    Response resp = new Response(resMap);
    serviceContext.setResponseData(resp);
    serviceContext.setResult(Result.FAULT_RESULT);
}
</pre>

### 4、前台获取jssdk：

<pre>
function _wechatConfig(o) {
        wx.config({
            debug: debug, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: o.appid, // 必填，公众号的唯一标识
            timestamp: o.timestamp, // 必填，生成签名的时间戳
            nonceStr: o.nonceStr, // 必填，生成签名的随机串
            signature: o.sign,// 必填，签名，见附录1
            jsApiList: [
                'onMenuShareTimeline',
                'onMenuShareAppMessage'
            ] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
        });
    }
 
    function _getWechatJsSdkInfo(callback) {
        var s = W.Storage.get(W.location.href + "_" + area);
            var appid ='wx7e80e8f93543a5c8';
            ajaxJsonCall('/wechat/service/WeChat.getJsSdkInfo.json', {
                url: W.location.href.split('#')[0],
                appid: appid
            }, function (data) {
                if (data.rtnCode === "000000") {
                    var o = data.responseData;
                    W.Storage.set(W.location.href + "_" + area, o, jsSdkTimeout);
                    callback(o);
                } else {
                    W.Storage.remove(W.location.href + "_" + area);
                }
            }, false);
    }

_getWechatJsSdkInfo(_wechatConfig);

wx.checkJsApi({
    jsApiList: [
        'getLocation',
        'onMenuShareTimeline',
        'onMenuShareAppMessage'
    ],
    success: function (res) {
        //alert(JSON.stringify(res));
    }
});
 
//var url = "www.zaichengdu.com" + app_path;
var url = curDomain;
function _shareAppMessage() {
    // 页面加载后设置微信分享给朋友的内容
    wx.onMenuShareAppMessage({
        title: '圣诞老人送礼啦，现金红包人人领！', // 分享标题
        desc: '“圣诞夺包”35000份礼包等你拆！', // 分享描述
        link: encodeURI(curDomain + '/christmas/service/ChristmasSockOnline.home.do?area=cd'),//encodeURI(window.location.href.replace('&from=ad', '')), // 分享链接
        imgUrl: url + '/public/christmas/img/shorejoin.jpg', // 分享图标
        type: '', // 分享类型,music、video或link，不填默认为link
        dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
        success: function () {
            // 用户确认分享后执行的回调函数
            //Message.toast.success("分享成功！").appear();
        },
        cancel: function () {
            // 用户取消分享后执行的回调函数
            //alert('cancel');
        }
    });
}
 
function _shareTimeline() {
    // 设置分享到朋友圈的内容
    wx.onMenuShareTimeline({
        title: '圣诞老人送礼啦，现金红包人人领！', // 分享标题
        link: encodeURI(curDomain + '/christmas/service/ChristmasSockOnline.home.do?area=cd'),//encodeURI(window.location.href.replace('&from=ad', '')), // 分享链接
        imgUrl: url + '/public/christmas/img/shorejoin.jpg', // 分享图标
        success: function () {
            // 用户确认分享后执行的回调函数
            Message.toast.success("分享成功！").appear();
        },
        cancel: function () {
            // 用户取消分享后执行的回调函数
            //alert('cancel');
        }
    });
}
 
wx.ready(function () {
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
    //alert('success');
    _shareAppMessage();
    _shareTimeline();
});
 
wx.error(function (res) {
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
    //alert('error');
});
</pre>