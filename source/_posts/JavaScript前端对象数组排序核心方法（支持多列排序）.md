---
title: JavaScript前端对象数组排序核心方法（支持多列排序）
date: 2019-06-05 10:23:20
tags:
- JavaScript
categories:
---
开箱即用工具代码：
```javascript
// 缓存当前数据
var tempDataArray = [{name:"linc",age:28},{name:"linda",age:26}];
// 存储排序后的数据
var currentData = [];
// 排序数组，支持多列排序
var orderArr = [{colName:"name",orderDir:"desc"}];
// 生成排序脚本
var sortData = _generateByStr(orderArr);
currentData = tempDataArray.concat();
currentData.sort(eval(sortData));
// 生成多列排序字符串方法
function _generateByStr(tempDataArray) {
	var arr = tempDataArray.concat();
	if (arr == null || arr.length == 0) {
		return "";
	} else {
		if (arr.length > 1) {
			var a = arr[0];
			arr.shift();
			return "_sortBy('" + a.colName + "','" + a.orderDir + "',"
					+ _generateByStr(arr) + ")";
		} else {
			return "_sortBy('" + arr[0].colName + "','"
					+ arr[0].orderDir + "')";
		}
	}
}
// 排序主方法
function _sortBy(name, dir, minor) {
	return function(o, p) {
		var a, b;
		if (o && p && typeof o === 'object' && typeof p === 'object') {
			a = o[name];
			b = p[name];
			if (a === b) {
				return typeof minor === 'function' ? minor(o, p) : 0;
			}
			if (typeof a === typeof b) {
				if (dir == "asc") {
					return a < b ? -1 : 1;
				} else {
					return a > b ? -1 : 1;
				}
			}
			if (dir == "desc") {
				return typeof a < typeof b ? -1 : 1;
			} else {
				return typeof a > typeof b ? -1 : 1;
			}
		} else {
			throw("error");
		}
	};
}
```