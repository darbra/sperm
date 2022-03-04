> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.nightteam.cn](https://bbs.nightteam.cn/forum.php?mod=viewthread&tid=1496&highlight=ast) ![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1616&size=middle)Nanda  
声明：本文内容仅供学习交流，严禁用于商业用途，请于 24 小时内删除。  
今天让我们玩一点有意思的东西，使用 **ast** 解决某 v5 加密的 js 代码。  
作为一个爬虫爱好者，可能你并不会使用 ast 处理混淆文件，但肯定是听过这个名字的，对吧？而且被传的神乎其神，又是编译原理，又是词法语法分析的。但实际上，底层的东西都已经有大佬封装好了，并不需要从头造轮子，而且 ast 对逆向工作中是真的很有效果！所以，最近我也是研究学习了一下，本次我就使用 babel 这一工具，为大家揭开 ast 的神秘面纱。  
既然是第一部曲，主要是让大家初步了解 ast，所以我们选择一个常规难度就好了，加密配置如下图：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=Mjk0fDA3YjBlYjVkfDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(153.86 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=Mjk0fDA3YjBlYjVkfDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

先上前后代码对比吧，还原前：  
/*  
 * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。;  
 * 已经打算把这个工具基础功能一直免费下去。还希望支持我。  
 * 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。  
 * 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。  
 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则  
 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。 */  
;var encode_version = 'sojson.v5'  
  , jrkqk = ''  
  , _0x3318 = ['eMO1woTDnRU=', '6L6h5pmt5LuD5Lq85Lq857Gi5Ymtwqh55pCG5LyT44OQ', '56us6ZaG5o2S6auk57ufDuKDp8KuwqzliYHlrpbig7NT5ZCXwrrigaJPw5bopp3lrb7ig4zCt+++heS/peWMheS+leeYlCB8w7bjga4=', 'Z3rCqsKQ', 'wq12EsOow7A=', 'wq3Di8Kpw63CisKZwr4=', 'RMOdwpc=', 'NTREw4FsCMKIQsKB', '54qF5p+/5Y6E772jw5LDkeS8meWsjOafneW+uOepq++9pei9v+itk+aXt+aMh+aKgOS7rOealOW3seS8vw==', '5Yis6Zia54ms5p225Y6H772nKMO85L2n5a+b5p+W5b2756iM', 'VMOBwofDtDscbsOBw7k=', 'eAglw6V9', 'dFUCw7LCsA==', 'wqzCij0=', '5YuX6Zip54ig5p6p5Y+J772cwqbDpOS+guWvgeadpOW+reermQ=='];  
(function(_0x1cab0b, _0x9fc93e) {  
    var _0x50aee6 = function(_0x5f6b93) {  
        while (--_0x5f6b93) {  
            _0x1cab0b['push'](_0x1cab0b['shift']());  
        }  
    };  
    _0x50aee6(++_0x9fc93e);  
}(_0x3318, 0x12d));  
var _0x4d94 = function(_0x387cf0, _0x46d6a3) {  
    _0x387cf0 = _0x387cf0 - 0x0;  
    var _0x1c7cd3 = _0x3318[_0x387cf0];  
    if (_0x4d94['initialized'] === undefined) {  
        (function() {  
            var _0x9672bc = typeof window !== 'undefined' ? window : typeof process === 'object' && typeof require === 'function' && typeof global === 'object' ? global : this;  
            var _0x47acfd = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';  
            _0x9672bc['atob'] || (_0x9672bc['atob'] = function(_0x4f114b) {  
                var _0xda79c1 = String(_0x4f114b)['replace'](/=+$/, '');  
                for (var _0x2456af = 0x0, _0xc64c02, _0x5e6923, _0x48f516 = 0x0, _0x448f43 = ''; _0x5e6923 = _0xda79c1['charAt'](_0x48f516++); ~_0x5e6923 && (_0xc64c02 = _0x2456af % 0x4 ? _0xc64c02 * 0x40 + _0x5e6923 : _0x5e6923,  
                _0x2456af++ % 0x4) ? _0x448f43 += String['fromCharCode'](0xff & _0xc64c02 >> (-0x2 * _0x2456af & 0x6)) : 0x0) {  
                    _0x5e6923 = _0x47acfd['indexOf'](_0x5e6923);  
                }  
                return _0x448f43;  
            }  
            );  
        }());  
        var _0x15dca8 = function(_0xa971bc, _0x5e580d) {  
            var _0x2a7b0e = [], _0x18c27c = 0x0, _0x8cce7, _0x21fe1e = '', _0x21ee82 ='';  
            _0xa971bc = atob(_0xa971bc);  
            for (var _0x465e88 = 0x0, _0x1826b5 = _0xa971bc['length']; _0x465e88 < _0x1826b5; _0x465e88++) {  
                _0x21ee82 += '%' + ('00' + _0xa971bc['charCodeAt'](_0x465e88)['toString'](0x10))['slice'](-0x2);  
            }  
            _0xa971bc = decodeURIComponent(_0x21ee82);  
            for (var _0x559b8b = 0x0; _0x559b8b < 0x100; _0x559b8b++) {  
                _0x2a7b0e[_0x559b8b] = _0x559b8b;  
            }  
            for (_0x559b8b = 0x0; _0x559b8b < 0x100; _0x559b8b++) {  
                _0x18c27c = (_0x18c27c + _0x2a7b0e[_0x559b8b] + _0x5e580d['charCodeAt'](_0x559b8b % _0x5e580d['length'])) % 0x100;  
                _0x8cce7 = _0x2a7b0e[_0x559b8b];  
                _0x2a7b0e[_0x559b8b] = _0x2a7b0e[_0x18c27c];  
                _0x2a7b0e[_0x18c27c] = _0x8cce7;  
            }  
            _0x559b8b = 0x0;  
            _0x18c27c = 0x0;  
            for (var _0x325076 = 0x0; _0x325076 < _0xa971bc['length']; _0x325076++) {  
                _0x559b8b = (_0x559b8b + 0x1) % 0x100;  
                _0x18c27c = (_0x18c27c + _0x2a7b0e[_0x559b8b]) % 0x100;  
                _0x8cce7 = _0x2a7b0e[_0x559b8b];  
                _0x2a7b0e[_0x559b8b] = _0x2a7b0e[_0x18c27c];  
                _0x2a7b0e[_0x18c27c] = _0x8cce7;  
                _0x21fe1e += String['fromCharCode'](_0xa971bc['charCodeAt'](_0x325076) ^ _0x2a7b0e[(_0x2a7b0e[_0x559b8b] + _0x2a7b0e[_0x18c27c]) % 0x100]);  
            }  
            return _0x21fe1e;  
        };  
        _0x4d94['rc4'] = _0x15dca8;  
        _0x4d94['data'] = {};  
        _0x4d94['initialized'] = !![];  
    }  
    var _0x281020 = _0x4d94['data'][_0x387cf0];  
    if (_0x281020 === undefined) {  
        if (_0x4d94['once'] === undefined) {  
            _0x4d94['once'] = !![];  
        }  
        _0x1c7cd3 = _0x4d94['rc4'](_0x1c7cd3, _0x46d6a3);  
        _0x4d94['data'][_0x387cf0] = _0x1c7cd3;  
    } else {  
        _0x1c7cd3 = _0x281020;  
    }  
    return _0x1c7cd3;  
};  
var a = {}  
  , b = {};  
(function(_0x117d09, _0x110647) {  
    var _0x4845bf = {  
        'XhuzS': _0x4d94('0x0', 'W7bQ'),  
        'iUTPo': _0x4d94('0x1', ']2T3')  
    };  
    _0x117d09[_0x4d94('0x2', 'WIC5')] = _0x4845bf['XhuzS'];  
    _0x110647['adinfo'] = _0x4845bf[_0x4d94('0x3', 'Wka7')];  
    _0x110647[_0x4d94('0x4', 'D8d!')] = '如果您的 JS 里嵌套了 PHP，JSP 标签，等等其他非 JavaScript 的代码，请提取出来再加密。这个工具不能加密 php、jsp 等模版内容';  
}(a, b));  
;(function(_0x182aaa, _0x4c3590, _0x55392e) {  
    var _0x4b40d0 = {  
        'gTfic': _0x4d94('0x5', 'oqkX'),  
        'GXIQH': function _0x38020c(_0x65ebe0, _0x2dc82b) {  
            return _0x65ebe0 === _0x2dc82b;  
        },  
        'xyVxG': _0x4d94('0x6', 'JsgM'),  
        'qeHKL': function _0x3e069c(_0x501f24, _0x41e6d1) {  
            return _0x501f24 + _0x41e6d1;  
        },  
        'EkYag': _0x4d94('0x7', '^7hL'),  
        'YZgLH': _0x4d94('0x8', 'g@wB')  
    };  
    _0x55392e = 'al';  
    try {  
        _0x55392e += _0x4b40d0['gTfic'];  
        _0x4c3590 = encode_version;  
        if (!(typeof _0x4c3590 !== _0x4d94('0x9', 'oqkX') && _0x4b40d0[_0x4d94('0xa', 'M*Wm')](_0x4c3590, _0x4b40d0['xyVxG']))) {  
            _0x182aaa[_0x55392e](_0x4b40d0[_0x4d94('0xb', '7glO')]('删除', _0x4b40d0['EkYag']));  
        }  
    } catch (_0x781e53) {  
        if ('aYN' === _0x4d94('0xc', '^7hL')) {  
            _0x182aaa[_0x55392e](_0x4d94('0xd', '9z79'));  
        } else {  
            _0x182aaa[_0x55392e](_0x4b40d0[_0x4d94('0xe', 'oqkX')]);  
        }  
    }  
}(window));  
;encode_version = 'sojson.v5';  
还原后：  
/* * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。;   
* 已经打算把这个工具基础功能一直免费下去。还希望支持我。   
* 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。   
* 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。  
 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则   
 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。   
 */  
   
 var a = {},  
    b = {};  
  
(function () {  
    a["info"] = "这是一个一系列 js 操作。";  
    b['adinfo'] = "站长接高级 “JS 加密” 和 “JS 解密” ，保卫你的 js。";  
    b["warning"] = '如果您的 JS 里嵌套了 PHP，JSP 标签，等等其他非 JavaScript 的代码，请提取出来再加密。这个工具不能加密 php、jsp 等模版内容';  
})();  
  
;  
  
(function () {  
    _0x55392e = 'al';  
  
    try {  
        _0x55392e += "ert";  
        _0x4c3590 = encode_version;  
  
        if (!(typeof _0x4c3590 !== "undefined" && _0x4c3590 === "sojson.v5")) {  
            window[_0x55392e]('删除' + "版本号，js 会定期弹窗，还请支持我们的工作");  
        }  
    } catch (_0x781e53) {  
        if ('aYN' === "aYN") {  
            window[_0x55392e]("删除版本号，js 会定期弹窗");  
        } else {  
            window[_0x55392e]("删除版本号，js 会定期弹窗");  
        }  
    }  
})();  
  
;  
encode_version = 'sojson.v5';  
是不是感觉效果很显著，那就认真看看本文是怎么实现的吧，哈哈。  
下面我来介绍下还原流程  
0，你肯定是要了解 ast 的概念和 babel 这一工具的具体使用的，我推荐几个链接，多读几遍肯定大有收获：  
a>[https://github.com/yacan8/blog/blob/master/posts/JavaScript%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91AST.md](https://github.com/yacan8/blog/blob/master/posts/JavaScript%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91AST.md) ：这是基础，多翻几遍，最好背过，才能更得心应手的使用 babel。  
b>[https://astexplorer.net/](https://astexplorer.net/) :  可视化的显示 ast 结构，开发时必不可少。  
1，分析结构  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MjkxfDA2MmQyOWVhfDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(39.3 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjkxfDA2MmQyOWVhfDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

总体，程序可分为两部分，上面是参数加密及转换的部分，在本例中，以_0x4d94 方法为出口，供下半部分调用，所以我们不用管上面，直接复制到 js 模块里然后导出_0x4d94 方法就好了。如下图所示：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MjkzfGFkOWRlYzU4fDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(32.57 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjkzfGFkOWRlYzU4fDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

2，结构已经清楚了，那我们就先用_0x4d94 方法把加密替换一下吧。  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=Mjk1fDc4NDgzNmEyfDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(83.59 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=Mjk1fDc4NDgzNmEyfDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

用到的依赖（babel 基础网上很多，我就不讲了哦，直接说代码了。）：  
const parser = require("@babel/parser");  
const template = require("@babel/template").default;  
const traverse = require("@babel/traverse").default;  
const t = require("@babel/types");  
const generator = require("@babel/generator").default;  
  
const path = require('path');  
const fs = require('fs')  
引入_0x4d94 方法：  
const {decryptStr, decryptStrFnName} = require('./module');  
读取原始 js 文件 && 转为 ast 语法树 && 遍历语法树寻找使用_0x4d94 方法的地方 && 替换 一气呵成！  
// 使用 parse 将 js 转为 ast 语法树  
const ast = parser.parse(jsStr);  
  
// 使用 traverse 遍历语法树，因为方法的调用为 CallExpression 类型，所以我们只对 type 为 CallExpression 的节点进行处理。  
// 类型的查看方式看代码后面的图。  
traverse(ast,{  
    CallExpression:funToStr  
})  
  
function funToStr(path) {  
    var curNode = path.node;  
  
    if(curNode.callee.name === decryptStrFnName && curNode.arguments.length === 2)  
    {  
        var strC = decryptStr(curNode.arguments[0].value, curNode.arguments[1].value);  
          
        // 将匹配到的位置 的 方法调用 使用 replaceWith 方法 替换为字符串。  
        path.replaceWith(t.stringLiteral(strC))  
  
    }  
}  
  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=Mjk3fDRmNGU2NDU5fDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(149.68 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=Mjk3fDRmNGU2NDU5fDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

3, 进行到此，第一部分的代码已经完全无用了，我们把 ast 再转回 js 代码看一下效果:  
使用 generator 将 ast 语法树转为 js 代码。  
let {code} = generator(ast);  
  
console.log(code);  
转后效果图：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=Mjk4fGQyNTM2OTgzfDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(73.22 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=Mjk4fGQyNTM2OTgzfDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

嗯，已经清晰很多了，但是可以看到_0x4845bf 或_0x4b40d0 这种对象里 定义字符串或方法的混淆还需要处理，那就继续吧.  
4，观察可见，调用方式如  
_0x4b40d0["qeHKL"]('删除', _0x4b40d0['EkYag'])  
或  
_0x4b40d0['gTfic']  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzAwfDgxNTRhYzNlfDE2NDY0MDcyNjh8MjIyMnwxNDk2&noupdate=yes)

**image.png** _(127.45 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzAwfDgxNTRhYzNlfDE2NDY0MDcyNjh8MjIyMnwxNDk2&nothumb=yes)

2020-4-30 01:43 上传

可以看到这次是 VariableDeclarator 类型了，所以我们只对 type 为 VariableDeclarator 的节点进行处理，代码如下（有一点复杂，耐心看。。）：  
traverse(ast,{  
    VariableDeclarator:callToStr  
})  
  
  
function callToStr(path) {  
    var node = path.node;  
  
    // 判断是否符合条件  
    if (!t.isObjectExpression(node.init))  
        return;  
  
    var objPropertiesList = node.init.properties;  
  
    if (objPropertiesList.length==0)  
        return;  
  
    var objName = node.id.name;  
  
    // 对定义的各个 方法 或 字符串 依次在作用域内查找是否有调用  
    objPropertiesList.forEach(prop => {  
        var key = prop.key.value;  
        if(!t.isStringLiteral(prop.value))  
        {  
        // 对方法属性的遍历  
          
            var retStmt = prop.value.body.body[0];  
  
            // 该 path 的最近父节点              
            var fnPath = path.getFunctionParent();  
            fnPath.traverse({  
                CallExpression: function (_path) {  
                    if (!t.isMemberExpression(_path.node.callee))  
                        return;  
                      
                    // 判断是否符合条件  
                    var _node = _path.node.callee;  
                    if (!t.isIdentifier(_node.object) || _node.object.name !== objName)  
                        return;  
                    if (!t.isStringLiteral(_node.property) || _node.property.value != key)  
                        return;  
  
                    var args = _path.node.arguments;  
  
                    // 二元运算                      
                    if (t.isBinaryExpression(retStmt.argument) && args.length===2)  
                    {  
                        _path.replaceWith(t.binaryExpression(retStmt.argument.operator, args[0], args[1]));  
                    }  
                    // 逻辑运算                      
                    else if(t.isLogicalExpression(retStmt.argument) && args.length==2)  
                    {  
                        _path.replaceWith(t.logicalExpression(retStmt.argument.operator, args[0], args[1]));  
                    }  
                    // 函数调用                      
                    else if(t.isCallExpression(retStmt.argument) && t.isIdentifier(retStmt.argument.callee))  
                    {  
                        _path.replaceWith(t.callExpression(args[0], args.slice(1)))  
                    }  
                }  
            })  
        }  
        else{  
        // 对字符串属性的遍历  
            var retStmt = prop.value.value;  
  
            // 该 path 的最近父节点            var fnPath = path.getFunctionParent();  
            fnPath.traverse({  
                MemberExpression:function (_path) {  
                    var _node = _path.node;  
                    if (!t.isIdentifier(_node.object) || _node.object.name !== objName)  
                        return;  
                    if (!t.isStringLiteral(_node.property) || _node.property.value != key)  
                        return;  
  
                    _path.replaceWith(t.stringLiteral(retStmt))  
                }  
            })  
  
        }  
  
    });  
  
    // 遍历过的对象无用了，直接删除。  
    path.remove();  
}  
ok，经过上面的处理，那些对象也被干掉了，让我们将 ast 语法树转为 js 代码，看一下现在 js 代码的样子  
/*  
 * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。;  
 * 已经打算把这个工具基础功能一直免费下去。还希望支持我。  
 * 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。  
 * 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。  
 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则  
 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。 */  
   
var a = {},  
    b = {};  
(function (_0x117d09, _0x110647) {  
  _0x117d09["info"] = "这是一个一系列 js 操作。";  
  _0x110647['adinfo'] = "站长接高级 “JS 加密” 和 “JS 解密” ，保卫你的 js。";  
  _0x110647["warning"] = '如果您的 JS 里嵌套了 PHP，JSP 标签，等等其他非 JavaScript 的代码，请提取出来再加密。这个工具不能加密 php、jsp 等模版内容';  
})(a, b);  
;  
(function (_0x182aaa, _0x4c3590, _0x55392e) {  
  _0x55392e = 'al';  
  try {  
    _0x55392e += "ert";  
    _0x4c3590 = encode_version;  
    if (!(typeof _0x4c3590 !== "undefined" && _0x4c3590 === "sojson.v5")) {  
      _0x182aaa[_0x55392e]('删除' + "版本号，js 会定期弹窗，还请支持我们的工作");  
    }  
  } catch (_0x781e53) {  
    if ('aYN' === "aYN") {  
      _0x182aaa[_0x55392e]("删除版本号，js 会定期弹窗");  
    } else {  
      _0x182aaa[_0x55392e]("删除版本号，js 会定期弹窗");  
    }  
  }  
})(window);  
;  
encode_version = 'sojson.v5';  
5, 是不是已经一目了然了？但是还有点小问题，像自执行函数里 a,b 那样的参数是没什么实际意义的，参数多了还影响理解代码逻辑，所以最好直接替换到方法里面去，所以我们继续处理：  
自执行函数的 type 是 ExpressionStatement，所以我们针对 ExpressionStatement 节点做处理：  
traverse(ast,{  
    ExpressionStatement:convParam  
})  
  
function convParam(path) {  
    var node = path.node;  
      
    // 判断是否是我们想修改的节点  
    if (!t.isCallExpression(node.expression))  
        return;  
  
    if (node.expression.arguments == undefined || node.expression.callee.params == undefined || node.expression.arguments.length> node.expression.callee.params.length)  
        return;  
  
    // 获取形参和实参  
    var argumentList = node.expression.arguments;  
    var paramList = node.expression.callee.params;  
    // 实参可能会比形参少，所以我们对实参进行遍历， 查看当前作用域内是否有该实参的引用  
    for (var i = 0; i<argumentList.length; i++)  
    {  
        var argumentName = argumentList_.name;  
        var paramName = paramList_.name;  
  
        path.traverse({  
            MemberExpression:function (_path) {  
                var _node = _path.node;  
                if (!t.isIdentifier(_node.object) || _node.object.name !== paramName)  
                    return;  
                      
                // 有对实参的引用则 将形参的名字改为实参的名字  
                _node.object.name = argumentName;  
            }  
        });  
    }  
    // 删除实参和形参的列表。  
    node.expression.arguments = [];  
    node.expression.callee.params = [];  
}  
都搞定了，就是最开始展示的那个结果了。让我们再来看一下：  
/* * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。; * 已经打算把这个工具基础功能一直免费下去。还希望支持我。 * 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。 * 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。 */var a = {},  
    b = {};  
  
(function () {  
    a["info"] = "这是一个一系列 js 操作。";  
    b['adinfo'] = "站长接高级 “JS 加密” 和 “JS 解密” ，保卫你的 js。";  
    b["warning"] = '如果您的 JS 里嵌套了 PHP，JSP 标签，等等其他非 JavaScript 的代码，请提取出来再加密。这个工具不能加密 php、jsp 等模版内容';  
})();  
  
;  
  
(function () {  
    _0x55392e = 'al';  
  
    try {  
        _0x55392e += "ert";  
        _0x4c3590 = encode_version;  
  
        if (!(typeof _0x4c3590 !== "undefined" && _0x4c3590 === "sojson.v5")) {  
            window[_0x55392e]('删除' + "版本号，js 会定期弹窗，还请支持我们的工作");  
        }  
    } catch (_0x781e53) {  
        if ('aYN' === "aYN") {  
            window[_0x55392e]("删除版本号，js 会定期弹窗");  
        } else {  
            window[_0x55392e]("删除版本号，js 会定期弹窗");  
        }  
    }  
})();  
  
;  
encode_version = 'sojson.v5';  
简单明了，这要是再看不懂就说不过去了吧!  
结尾：至此，对该加密的还原就告一段落了，可以看到还原后的代码完全足够让我们愉快的 debug 了。第一部曲就讲这些了, 至于反控制流平坦化、作用域管理等等使用 babel 也可以轻易的解决，二部曲我可能会分享到这些。至于第二部曲是写该加密的 绝对不可逆配置 呢，还是写 jsfuck 的还原呢，我还没想好，各位也可以留言提建议。。没什么意外的话，应该会在 5.1 假期写完分享出来。源码放在下面了，回复后再看哦，省的我感觉像打单机，哈哈。  
各位拜拜~  
[ttreply]  
[https://github.com/chencchen/webcrawler/tree/master/%E4%BD%BF%E7%94%A8ast%E5%AF%B9%E6%9F%90v5%E5%8A%A0%E5%AF%86%E8%BF%9B%E8%A1%8C%E8%BF%98%E5%8E%9F](https://github.com/chencchen/webcrawler/tree/master/%E4%BD%BF%E7%94%A8ast%E5%AF%B9%E6%9F%90v5%E5%8A%A0%E5%AF%86%E8%BF%9B%E8%A1%8C%E8%BF%98%E5%8E%9F)  
[/ttreply]  
__![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1842&size=middle)damen nb![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=196&size=middle)yzr nb![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=823&size=middle)fu9852531 厉害 ![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=191&size=middle)frank mark![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1449&size=middle) 努努 赞赞赞，楼主写的很详细  你并不是一个人在战斗 ![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=617&size=middle)franky 优秀，正想入门这一块![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=9&size=middle)花儿谢了 great![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=245&size=middle) 丶 Fallenoringash 秀啊![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1&size=middle) sml2h3 非常值得学习借鉴的案例