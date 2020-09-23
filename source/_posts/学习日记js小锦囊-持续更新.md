---
title: 学习日记js小锦囊-持续更新
date: 2018-05-02 22:15:32
tags: js
categories: 
  - js
  - 学习日记
---
#### 两中数字格式化的方法
> 数字格式化一 e.g. 23422039->2342.2W

```javascript
/**
  * 2342.2W
  * @param {number} count 需要格式化的数字
  */
function CounFormat (count) {
  var countString = count.toString()
  var tmp = ''
  var result = ''
  if (countString.length >= 5) {
    tmp = countString.slice(-4, -3) === '0' ? '' : countString.slice(-4, -3)
    if (tmp.slice(tmp.length + 1, 1) === 0) {
      result = countString.slice(0, -4).concat('w')
    } else {
      tmp === '' ? result = countString.slice(0, -4).concat('w') : result = countString.slice(0, -4).concat('.' + tmp + 'w')
    }
    return result
  } else {
    return count
  }
}
```



> 数字格式化二 e.g. 23232222->23,232,222
```javascript
/**
  * 千位符数字格式化23,232,222
  * @param {number} num 需要格式化的数字
  * @param {int} precision 保留小数位
  * @param {String} separator 分隔符
  */
function CounFormat2 (num, precision, separator) {
  var parts
  if (!isNaN(parseFloat(num)) && isFinite(num)) {
    num = Number(num)
    num = (typeof precision !== 'undefined' ? num.toFixed(precision) : num).toString()
    parts = num.split('.')
    parts[0] = parts[0].toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1' + (separator || ','))
    return parts.join('.')
  }
  return NaN
}
	```
	
#### 时间格式化的方法	
> 时间格式化 e.g. 1245278300->2009-6-18 06:38
```javascript
/**
  * 时间格式化 2018-09-09 12:02:22
  * @param {number} timespan 时间戳
  */
function formatMsgTime (timespan) {
  timespan = timespan * 1000
  var dateTime = new Date(timespan)
  var year = dateTime.getFullYear()
  var month = dateTime.getMonth() + 1
  var day = dateTime.getDate()
  var hour = dateTime.getHours() ? (dateTime.getHours() < 10 ? '0' + dateTime.getHours() : dateTime.getHours()) : '00'
  var minute = dateTime.getMinutes() ? (dateTime.getMinutes() < 10 ? '0' + dateTime.getMinutes() : dateTime.getMinutes()) : '00'
  // var second = dateTime.getSeconds()
  var now = new Date()
  // typescript转换写法
  var nowNew = now.getTime()
  // var nowNew = Date.parse(now.toDateString())
  var milliseconds = 0
  var timeSpanStr
  milliseconds = nowNew - timespan
  if (milliseconds <= 1000 * 60 * 1) {
    timeSpanStr = '刚刚'
  } else if (1000 * 60 * 1 < milliseconds && milliseconds <= 1000 * 60 * 60) {
    timeSpanStr = Math.round((milliseconds / (1000 * 60))) + '分钟前'
  } else if (1000 * 60 * 60 * 1 < milliseconds && milliseconds <= 1000 * 60 * 60 * 24) {
    timeSpanStr = Math.round(milliseconds / (1000 * 60 * 60)) + '小时前'
  } else if (1000 * 60 * 60 * 24 < milliseconds && milliseconds <= 1000 * 60 * 60 * 24 * 15) {
    timeSpanStr = Math.round(milliseconds / (1000 * 60 * 60 * 24)) + '天前'
  } else if (milliseconds > 1000 * 60 * 60 * 24 * 15 && year === now.getFullYear()) {
    timeSpanStr = month + '-' + day + ' ' + hour + ':' + minute
  } else {
    timeSpanStr = year + '-' + month + '-' + day + ' ' + hour + ':' + minute
  }
  return timeSpanStr
}
```

#### 视频时长格式化
```javascript
/**
   * 视频时长格式化
   * @param {number} seconds
   */
  function secondsToHour (seconds) {
    var h = parseInt(seconds / 3600)
    var m = parseInt((seconds % 3600) / 60)
    var s = parseInt(seconds % 60)
    h = h ? h + ':' : ''
    m = m ? (m < 10 ? '0' + m + ':' : m + ':') : '00:'
    s = s ? (s < 10 ? '0' + s : s) : '00'
    return h + m + s
  }
```

#### 文案截取
```javascript
/**
   * 文案截取
   * @param {number} num 截取个数
   * @param {String} string 拼接字符串
   * @param {String} symbol 拼接字符串
   * stringLen("asdsfdfsdf", 80, "...")
   */
  function stringLen (string, num, symbol) {
    var len = 0
    for (var i = 0; i < string.length; i++) {
      if (len >= num) {
        return string.slice(0, i) + symbol
      }
      if (string.charCodeAt(i) > 127 || string.charCodeAt(i) === 94) {
        len++
      } else {
        len += 1 / 2
      }
    }
    return string
  }
```

#### 对比版本号
```javascript
/**
   * 对比版本号，大于：true 小于：false
   * @param {String} v1 2.1.1
   * @param {String} v2 2.2.2
   */
  function versions(v1, v2) {
    var arr1 = v1.replace(/[-_]/g, '.').split('.')
    var arr2 = v2.replace(/[-_]/g, '.').split('.')
    var len = Math.max(arr1.length, arr2.length)
    for (var i = 0; i < len; i++) {
      arr1[i] = !arr1[i] ? 0 : arr1[i]
      arr2[i] = !arr2[i] ? 0 : arr2[i]
      if (parseInt(arr1[i]) > parseInt(arr2[i])) {
        return true
      } else if (parseInt(arr1[i]) < parseInt(arr2[i])) {
        return false
      }
    }
    return true
  }
```

#### 倒计时
```javascript
/**
   * 倒计时
   * @param {number} msd 截止时间
   */
  function MillisecondToDate (msd) {
    var now = new Date().getTime()
    var time = parseFloat(msd - now) / 1000
    if (time !== null && time !== '') {
      var timeDay = parseInt(time / 86400.0)
      var timeH = parseInt(time / 3600.0 - 24 * timeDay)
      var timeM = parseInt((parseFloat(time / 3600.0) - parseInt(time / 3600.0)) * 60)
      var timeS = parseInt((parseFloat((parseFloat(time / 3600.0) - parseInt(time / 3600.0)) * 60) - parseInt((parseFloat(time / 3600.0) - parseInt(time / 3600.0)) * 60)) * 60)
      timeDay = timeDay < 10 ? '0' + timeDay : timeDay
      timeH = timeH < 10 ? '0' + timeH : timeH
      timeM = timeM < 10 ? '0' + timeM : timeM
      timeS = timeS < 10 ? '0' + timeS : timeS
    }
    return {
      'day': timeDay.toString(),
      'hour': timeH.toString(),
      'minute': timeM.toString(),
      'second': timeS.toString()
    }
  }
```