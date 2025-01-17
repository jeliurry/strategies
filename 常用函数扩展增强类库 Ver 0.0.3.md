
> 策略名称

常用函数扩展增强类库 Ver 0.0.3

> 策略作者

张超





> 源码 (javascript)

``` javascript
/*
 * @Project: 官方扩展增强类库
 * @Version: Ver 0.0.3
 * @Author: RedSword <redsword@gmail.com>
 * @Description:收集整理常用增强函数
 * @Date: 2021-05-06 11:02:29
 * @LastEditors: RedSword
 * @LastEditTime: 2021-09-07 11:35:52
 * @Copyright:: Copyright © 2020 FMZ Quant
 * 代码规范参考: https://github.com/fex-team/styleguide/blob/master/javascript.md
 */

/**
 * @Title: 转北京时间
 * @description: 将托管者所在服务器时间转为北京时间
 * @param {time} localtime 所在时间
 * @return {time} 北京时间
 * @example
 *  var localDate = $.ToBJTime(new Date());
 *  Log(localDate)  //Thu May 06 2021 11:19:45 GMT+0000
 *  Log(localDate.getTime())    //1620299985696
 */
$.ToBJTime = function (localDate) {
	return new Date(localDate.getTime() + localDate.getTimezoneOffset() * 60000 + 3600000 * 8);
};

/**
 * 将日期格式化成指定格式的字符串
 * @param date 要格式化的日期，不传时默认当前时间，也可以是一个时间戳
 * @param fmt 目标字符串格式，支持的字符有：y,M,d,q,w,H,h,m,S，默认：yyyy-MM-dd HH:mm:ss
 * @returns 返回格式化后的日期字符串
 */
$.FormatDate = function (date, fmt) {
	date = !date ? new Date() : date;
	date = typeof date === "number" ? new Date(date) : date;
	fmt = fmt || "yyyy-MM-dd HH:mm:ss";
	var obj = {
		y: date.getFullYear(), // 年份，注意必须用getFullYear
		M: date.getMonth() + 1, // 月份，注意是从0-11
		d: date.getDate(), // 日期
		q: Math.floor((date.getMonth() + 3) / 3), // 季度
		w: date.getDay(), // 星期，注意是0-6
		H: date.getHours(), // 24小时制
		h: date.getHours() % 12 == 0 ? 12 : date.getHours() % 12, // 12小时制
		m: date.getMinutes(), // 分钟
		s: date.getSeconds(), // 秒
		S: date.getMilliseconds(), // 毫秒
	};
	var week = ["天", "一", "二", "三", "四", "五", "六"];
	for (var i in obj) {
		fmt = fmt.replace(new RegExp(i + "+", "g"), function (m) {
			var val = obj[i] + "";
			if (i == "w") return (m.length > 2 ? "星期" : "周") + week[val];
			for (var j = 0, len = val.length; j < m.length - len; j++) val = "0" + val;
			return m.length == 1 ? val : val.substring(val.length - m.length);
		});
	}
	return fmt;
};
/**
 * @Title: 获取永久缓存
 * @description:当缓存不存在时,写入缓存,如果存在就返回数据,一般记录初始价格使用
 * @param {string} key 唯一标示
 * @param {*} data 要保存的数据
 * @return {*}
 *
 */
$.ForeverCache = function (key, data) {
	key = "ForeverCache" + key;
	var getDate = _G(key);
	if (getDate == null) {
		_G(key, data);
		return data;
	}
	return getDate;
};

/**
 * @title: 当天缓存
 * @description:获取当天的缓存,每天北京时间0点0分重置数据
 * @param {string} key 唯一标示
 * @param {*} data 要保存的数据
 * @return {*}
 */
$.TodayCache = function (key, data) {
	key = "TodayCache" + key;
	var today = $.ToBJTime(new Date()).getDate();
	if (_G("TodayCacheToday_" + key) !== today) {
		_G("TodayCacheToday_" + key, today);
		_G(key, data);
		return data;
	} else {
		return _G(key);
	}
};

/**
 * @title: 定时执行函数
 * @description: 指定时间间隔,来执行函数
 * @param {int} second 间隔时间,为秒
 * @param {string} key 标识
 * @param {function} fun 要执行的函数
 * @return {*} 定时执行函数的结果
 * @example
 * //每60秒执行一次nowTime()函数
 * Log($.ExecuteFuncForTime(60,"myTime",nowTime))
 */
$.ExecuteFuncForTime = function (second, key, fun) {
	var endSecond = second * 1000;
	var nowTime = new Date().getTime();
	if (_G("funcForTime_" + key) == null || nowTime - _G("funcForTime_" + key) > endSecond) {
		var data = fun();
		_G("funcForTime_" + key, nowTime);
		_G(key, data);
		return data;
	} else {
		return _G(key);
	}
};

/**
 * @title: Object排序
 * @description:根据指定的标识符,对Object进行排序
 * @param {object} obj 要排序的对象
 * @param {string} key 标识符
 * @return {object}
 * @example
 *  var obj = { aa: { f: 2 }, bb: { f: 1 }, cc: { f: 3 } };
 *	Log(sortobjkey(obj,'f')) //{"cc":{"f":3},"aa":{"f":2},"bb":{"f":1}}
 */
$.SortObjectKey = function (obj, key) {
	var o = {};
	Object.keys(obj)
		.map(function (k, i) {
			return [k, obj[k]];
		})
		.sort(function (a, b) {
			k = key;
			if (a[1][k] > b[1][k]) return -1;
			if (a[1][k] < b[1][k]) return 1;
			return 0;
		})
		.forEach(function (a) {
			o[a[0]] = a[1];
		});
	return o;
};

/**
 * @title:策略运行时间
 * @description:记录第一次运行时间并计算已经运行的时间数
 * @return {object} 对象
 * @example
 * Log($.RunTime())
 * //{"RunSeconds":18170,"NowUnix":1622821609901,"NowTime":"2021-06-04 23:46:49","FormatTime":"运行时间: 0 天 5 时 2 分 50 秒","StartTime":"2021-06-04 18:43:59"}
 */
$.RunTime = function () {
	var startTime = new Date($.ForeverCache("startTime", new Date()));
	var between = $.TimeBetween(startTime, new Date());
	var timestamp = parseInt(UnixNano() / 1000000);
	return {
		RunSeconds: parseInt((timestamp - startTime.getTime()) / 1000), //运行秒数
		NowUnix: timestamp, //当间时间戳
		NowTime: _D($.ToBJTime(new Date())), //当前时间
		FormatTime: "运行时间: " + between.days + " 天 " + between.hours + " 时 " + between.minutes + " 分 " + between.seconds + " 秒", //格式化后的运行时间
		StartTime: _D($.ToBJTime(startTime)), //开始时间
	};
};

$.TimeBetween = function (startDate, endDate) {
	let delta = Math.abs(endDate - startDate) / 1000;
	const isNegative = startDate > endDate ? -1 : 1;
	return [
		["days", 24 * 60 * 60],
		["hours", 60 * 60],
		["minutes", 60],
		["seconds", 1],
	].reduce((acc, [key, value]) => ((acc[key] = Math.floor(delta / value) * isNegative), (delta -= acc[key] * isNegative * value), acc), {});
};
function main() {}

```

> 策略出处

https://www.fmz.com/strategy/277946

> 更新时间

2021-09-07 11:40:21
