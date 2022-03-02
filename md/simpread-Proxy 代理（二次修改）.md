> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyMjQ3OTE5MA==&mid=2247483738&idx=1&sn=1d53007e7805d88e5841a582718f0e16&chksm=e82d9763df5a1e75e59b27c3b6772b78eeb2b9ccd219df09362b8a4ac7f455766d8e76f6b362&mpshare=1&scene=1&srcid=030205ef041Dql3QXB3GY2bT&sharer_sharetime=1646203661515&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

### Proxy（二次修改）

> `Proxy` 二次封装，js 逆向补环境，修复了昨天的 **bug**，挺尴尬的

```
(function () {
    function set_traverse_object(tarrget, obj, recursion_layers) {
        recursion_layers -= 1;
        console.log();
        for (let prop in obj) {
            const value = obj[prop];
            const tg_name = `${tarrget}.${prop.toString()}`;
            const value_type = get_value_type(value);
            if (value && value_type === "object" && recursion_layers >= 1) {
                set_traverse_object(tg_name, value, recursion_layers);
                continue
            }
            if (value && value.toString() !== '[object Object]') {
                console.log(`setter  hook->${tg_name};  value-> ${value};  typeof-> ${value_type}`);
                continue
            }
            console.log(`setter  hook->${tg_name};  value-> ${value};  typeof-> ${value_type}\n`)

        }
    }

    function new_handel(target_name, obj, number) {
        returnnewProxy(obj, my_handler(target_name, number))
    }

    function get_value_type(value) {
        if (Array.isArray(value)) {
            return'Array'
        }
        returntypeof value;
    }

    function my_handler(target_name, number) {
        return {
            set: function (obj, prop, value) {
                const value_type = get_value_type(value);
                const tg_name = `${target_name}.${prop.toString()}`;

                if (value && value_type === "object") {
                    set_traverse_object(tg_name, value, number)
                } else {
                    console.log(`setter  hook->${tg_name};  value-> ${value};  typeof-> ${value_type}`)
                }
                returnReflect.set(obj, prop, value);
            },
            get: function (obj, prop) {
                const tg_name = `${target_name}.${prop.toString()}`;
                const value = Reflect.get(obj, prop);
                let value_type = get_value_type(value);
                if (value && value_type === 'object') {
                    return new_handel(tg_name, value, number)
                }
                console.log(`getter  hook->${tg_name};  value-> ${value};  typeof-> ${value_type}\n`);
                return value
            }
        }
    }

    window = newProxy(window, my_handler(Object.keys({window})[0], 30));
}());


```

> 代理效果如下，深层代理，更直观

```
getter  hook->window.document.cookie;  value-> undefined;  typeof-> undefined

getter  hook->window.location.href;  value-> https://mobile.pinduoduo.com/;  typeof-> string

getter  hook->window.location.port;  value-> ;  typeof-> string

{ addEventListener: [Function: addEventListener] } addEventListener
getter  hook->window.document.addEventListener;  value-> function () {
    };  typeof-> function

getter  hook->window.document.addEventListener;  value-> function () {
    };  typeof-> function

getter  hook->window.document.addEventListener;  value-> function () {
    };  typeof-> function


```