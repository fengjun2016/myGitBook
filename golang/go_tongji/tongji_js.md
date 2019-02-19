# 打点服务器统计js实现上报服务功能
```js
$(function(){
	/**
	 * 实现js打点服务器上报功能
	 */
	var lock = true;
	$.get("http://localhost:8888/gxcms/dig", {
		"time"	: gettime(),
		// "ip"	: getip(),
		"url"	: geturl(),
		"refer"	: getrefer(),
		"ua"	: getuser_agent(),
	}, function(){
		lock = false;   //加锁操作 避免js打点上报操作重复多次
	});
});


/**
gettime(); //js获取当前时间
getip(); //js获取客户端ip
geturl(); //js获取客户端当前url
getrefer(); //js获取客户端当前页面的上级页面的url
getuser_agent(); //js获取客户端类型
**/

function gettime(){
	var nowDate = new Date();
	return nowDate.toLocaleString();
}

function geturl(){
	return window.location.href;
}

// function getip(){
// 	return returnCitySN["cip"]+','+returnCitySN["cname"];
// }

function getrefer(){
	return document.referrer;
}

function getcookie(){
	return document.cookie;
}

function getuser_agent(){
	return navigator.userAgent;
}
```