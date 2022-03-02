> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5ODgyOTUzNw==&mid=2247484039&idx=1&sn=0de1ee2e3d201b4b5e2823c1b7975b5a&chksm=febf7209c9c8fb1fb4f8feedd37e223a35887526548564462bd846cc41b954541fe53ac5e0ed&mpshare=1&scene=1&srcid=0302cSUvBnRqvw2ComEobKZP&sharer_sharetime=1646203623636&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

1，首先要说的肯定是 Proxy 了，介绍就不说了，直接上代码：

```
window = new Proxy(global, {
    get: function (target, key, receiver) {
        console.log("window.get", key, target[key]);
        if (key=="location"){
            location = new Proxy(target[key], {
                get: function (_target, _key, _receiver) {
                    console.log("window.get", key, _key, _target[_key]);
                    if (_key=="port"){console.log("关注公众号【妄为写代码】")}
                    return _target[_key];
                }
            })
        }
        return target[key];
    },
    set: function (target, key, value, receiver) {
        console.log("window.set", key, value);
        target[key] = value;
    }
});
window.a = {};
window.a;
window.location = {a: 2};
window.location.a;
window.b = {a: 2};
window.b.a;
location.port;
console.log("--------------");
window.location.port;

```

node 环境执行结果：  

![](https://mmbiz.qpic.cn/mmbiz_png/PicMqQs6T6MNerXH5LUe0WPbwcutymn72M4H3yicw6ibWh4NYhQyOrqScX6zZvMLNBt3P5nSHRQ4gI6hmOtKqqzDQ/640?wx_fmt=png)

重点关注【嵌套 Proxy】和【重复 Proxy】

2，对象属性的 hook 方式

在浏览器中执行：  

![](https://mmbiz.qpic.cn/mmbiz_png/PicMqQs6T6MNerXH5LUe0WPbwcutymn72NpUd6aVfbRdf5j3icqr0EsEPM49wmyjY77pYNADx5DRlxibqTEvC4HsQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PicMqQs6T6MNerXH5LUe0WPbwcutymn72C2ydR961aTluSG8ibICClkNiaUJypLI0FHGxJCwtXP5l1XZl6S3hU0cw/640?wx_fmt=png)

重点关注【未在固定范围的新增属性】和【对比两种方式的 location.port】和【多层属性的获取 window.location.port】

3，这个监控的作用就不用说了吧，就是大家常说的缺哪补哪需要用到的，现在补环境的场景越来越多了，一些知名 js 反爬产品，就可以用这个思路，环境补的好，可以到处用，还能省好多事，一举多得。

嗯，我也准备学大家开始【佛系】社群运营了，大家可以扫码进群一起交流技术，nice to meet you.

![](https://mmbiz.qpic.cn/mmbiz_png/PicMqQs6T6MNerXH5LUe0WPbwcutymn72c05JlHHVWUoEQCabAghOvicFz9OsF3lbH8h5uDm0W7ZJYDYiav5CeawQ/640?wx_fmt=png)