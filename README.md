# 香港永居日期计算器

> 💡 [https://7years.atom.im](https://7years.atom.im)

申请核实香港永久性居民身份证可以在满7年的日子到来之前提前一个月时间提交。但是一个月具体是多少天呢？众说纷纭，包括但不限于29天、30天、31天。实际上，如果说提前，到底哪天算满7年呢？一定是7年前哪天入境就今年哪天算满7年吗？不一定是。2017年入境今年满7年的日子正好就是入境的日子，比如2017年7月6号入境，2024年7月6号就满7年。但是如果是2016年7月6号入境，申请系统里面满7年的日子要往后一天，是7月7号。

看了下香港永居在线申请系统，发现检查是否满7年、是否允许提交这些操作都是直接用js脚本完成的，检查7年的代码在5.19a8fa481e3b181d2299.js里面。`check7yr()`函数调用了`rop2b2d_7yr()`函数来判断两个日期之间的天数，如果天数大于等于2556.75（7年时间）就直接pass继续到后面的提交申请材料步骤，否则再判断一下是否大于等于2526.3125，如果大于的话就弹出警告但是可以继续提交，如果小于这个数就直接报错不允许提交申请了。

```javascript
e.calDateInDay = function(e, t) {
    return Math.abs(t.getTime() - e.getTime()) / 864e5
}

e.rop2b2d_7yr = function(e, t) {
    return ["2b", "2c"].includes(e) ? t >= 2556.75 ? {
        stat: "pass"
    } : t >= 2526.3125 ? {
        stat: "passWwarn"
    } : {
        stat: "fail"
    } : {
        stat: "pass"
    }
}

e.prototype.check7yr = function(e) {
    if (!["2b", "2d"].includes(e)) return !0;
    for (var t = 0, n = 0; n <= this.p4a; n++) {
        var o = J.isEmpty(this.part4A(n).controls.startDay.value) ? 0 : Number.parseInt(this.part4A(n).controls.startDay.value),
            r = J.isEmpty(this.part4A(n).controls.startMth.value) ? 0 : Number.parseInt(this.part4A(n).controls.startMth.value) - 1,
            i = this.part4A(n).controls.startYr.value,
            a = J.isEmpty(this.part4A(n).controls.endDay.value) ? 0 : Number.parseInt(this.part4A(n).controls.endDay.value),
            l = J.isEmpty(this.part4A(n).controls.endMth.value) ? 0 : Number.parseInt(this.part4A(n).controls.endMth.value) - 1,
            s = this.part4A(n).controls.endYr.value;
        t += J.calDateInDay(new Date(Number.parseInt(s), l, a), new Date(Number.parseInt(i), r, o))
    }
    var d = J.rop2b2d_7yr(this.rop145Model.catCode, t);
    return "pass" == d.stat || ("passWwarn" == d.stat ? (this.commonService.errorPopUpWarning(J.get7yrMessage("rop.2ab.warn", this.translateservice)), !0) : (this.commonService.errorPopUp(J.get7yrMessage("rop.2ab.error", this.translateservice)), !1))
}
```
这个2526.3125天数就是个门槛，跟满7年的天数相差30.438天。这个跟在之前在永居申请系统里试出的结果是一样的，提前31天不允许提交，30天可以提交申请。

现在有了这两个magic number了，后面的日期就好计算了，写了个网页来快速计算。7月6号加1天就是7月7号这样。首次入境小白条上的入境日期分别加上向上取整得到的2527和2557两个数算出最早提前申请日期和满7年日期。

需要注意的是最早入境日期是指用学生签、工作签、投资移民签证、受养人签证等符合条件的签证入境的日期。旅游签及其他特定非永居性签证入境的日期不算。

