## Prelude

![](img/logo.jpg)

以前，"富数据web应用"还是一种很矛盾的说法。今天，这中应用随处可见，以至于我们不得不掌握如何去构建它们。

传统方式，web应用把繁重的数据放到服务器端处理然后在完整的页面载入后把HTML推向浏览器。客户端的JavaScript对于提升用户体验所做的事情非常有限。现在这种关系反过来了——客户端应用再需要的时候从服务器端拉去原始数据然后渲染到浏览器。

想下那个Ajax的购物车，当添加商品到篮子里时不需要刷新页面。最初，jQuery成为处理这类范例的热门库。它的原理就是通过Ajax请求然后更新页面中的文本和其它内容。不过，这种使用jQuery的模式在客户端暗含里数据模型。服务器不再是唯一知道商品数量的地方了，it was a hint that there was a natural tension and pull of this evolution. 

随着客户端与服务器端交互的代码变多，客户端将变得跟加复杂。对客户端的一个经过深思的良好的架构已是必不可少了—— 你不能仅仅是缩减一些jQuery代码以期望能适应应用的增长。最有可能的是，UI的回调与业务逻辑纠缠在一起最终给你带来是一个噩梦，你的代码命中注定要被接手的那个倒霉鬼丢弃掉。

幸好，现在逐渐有很多JavaScript库可以帮助你改进代码的架构并且让代码更有可维护性，不用费太大力气就可以轻易的构建出绚丽的界面。[Backbone.js](http://documentcloud.github.com/backbone/) 很快成为非常流程的解决这种问题的方案之一，这本书我将带你深入了解它。

从基本原理开始，通过练习实践，学习如何构建结构清晰可维护的应用。如果你是一个开发者想知道如何编写易阅读，有组织，易扩展的代码——这本教程将对你有所帮助。

对我来说改善开发者的培养更为重要，这就是为什么这本书将遵循署名-非商业性使用-相同方式共享 3.0[协议](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/)发布。这就意味着你可以免费获取这本书的[拷贝](http://addyosmani.github.com/backbone-fundamentals/) 或者帮助(改进)(https://github.com/addyosmani/backbone-fundamentals/)它。 也非常欢迎修正原文中的素材，希望我们能一起为开发社区提供最新的资源帮助。


另外非常感谢 [Jeremy Ashkenas](https://github.com/jashkenas) 创建Backbone.js，[DocumentCloud](documentcloud.org)，以及[这些](https://github.com/addyosmani/backbone-fundamentals/contributors)社区成员的帮助，使得这个项目远比我想像中的要好。

## 面向读者

这本书面向的那些希望学习如何更好的构建客户端代码的中级开发者。已掌握JavaScript的基本原理和知识，不过必要的时候会在文中做些基本的描述。

## 致谢

我非常感激那些技术评审人员及其给力的工作，帮助校对和改进这本书。他们的知识，精力和激情促使这本书成文更好的学习资源and they continue to serve as a source of inspiration. Thanks go out to:

* [Marc Friedman](https://github.com/dcmaf)
* [Derick Bailey](https://github.com/derickbailey)
* [Jeremy Ashkenas](https://github.com/jashkenas)
* [Samuel Clay](https://github.com/samuelclay)
* [Mat Scales](http://github.com/wibblymat)
* [Alex Graul](https://github.com/alexgraul)
* [Dusan Gledovic](https://github.com/g6scheme)
* [Sindre Sorhus](https://github.com/sindresorhus)

我要感谢我亲爱的家人，感谢他们的耐心和支持我在这本书上的工作，我还有我杰出的编辑Mary Treseler。

## 相关人员

没有其它社区的开发者和作者的时间和精力的投入我是不可能完成这项工作的，我也要感谢他们：

* Derick and Marc (once again)
* [Ryan Eastridge](https://github.com/eastridge)
* [Jack Franklin](https://github.com/jackfranklin)
* [David Amend](https://github.com/raDiesle)
* [Mike Ball](https://github.com/mdb)
* [Uģis Ozols](https://github.com/ugisozols)
* [Björn Ekengren](https://github.com/Ekengren)

同时还有其它的杰出[贡献者](https://github.com/addyosmani/backbone-fundamentals/graphs/contributors)帮助使这个项目成为可能。

## 阅读

在阅读这本书前假定你已经有基本的JavaScript知识，并且一些很明确的话题将会被略过，比如对象字面量。如果你想学习关于语言的更多知识，建议你阅读下面这些资料：

* [Eloquent JavaScript](http://eloquentjavascript.net/)
* [JavaScript: The Definitive Guide](http://shop.oreilly.com/product/9780596805531.do) by David Flanagan (O’Reilly)
* [Effective JavaScript](http://www.informit.com/store/effective-javascript-68-specific-ways-to-harness-the-9780321812186) by David Herman (Pearson)
* [JavaScript: The Good Parts](http://shop.oreilly.com/product/9780596517748.do) by Douglas Crockford (O’Reilly)
* [Object-Oriented JavaScript](http://www.amazon.com/Object-Oriented-Javascript-Stoyan-Stefanov/dp/1847194141) by Stoyan Stefanov (Packt Publishing)

