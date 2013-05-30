# 前言

弗兰克·劳埃德·赖特(Frank Lloyd Wright)曾说“你不能成为一个建筑师，但是你可以打开门和窗户，走向你所看到的光明。” 在这本书中，我希望分享一些如何改进web应用架构的光明，打开通往更有可维护性，可阅读性应用之门。

所有架构的目的都是把东西构建得更好；在我们的例子中，创建持久的，能使自己和将来我们离开之后维护我们代码的开发者都非常愉悦的代码。希望我们的架构简单而漂亮。

当代的JavaScript框架和库可以给我们的项目提供架构和组织结构，从一开始就建立起可维护的基础。它们构建在开发者不得不解决混乱的回调的考验和痛苦之上，类似于你现在或者可能将来不久会面临的问题。

当使用jQuery开发应用时，缺失块就是一种构建和组织代码的方式。使用一团糟的jQuery选择器和回调来创建JavaScript应用非常容易，所有地方都拼命让数据在UI的HTML，JavaScript逻辑，数据调用的API之间保持同步。

没有什么帮助驯服这些乱起八糟的东西的话，你可能会串起一个独立的插件或者库来实现这些功能或者一切从头开始，并需要自己维护。Backbone就为你解决了这一问题，它提供一种清晰的方式组织代码，把职责分隔到可辨认的容易维护的块中。

在"Developing Backbone.js Applications"这一书中我和一些其他经验丰富的作者将给你展示如何通过流行的JavaScript库，Backbone.js来改进你的web应用架构。

### 什么是MVC?

现在有大量的框架给开发者提供一种简单的方式，MVC(Model-View-Controller)模式的变种，来组织他们的代码。MVC把我们在应用中关心的问题分隔成三部分：

* Models描述了特定域信息和应用中的数据。可以认为是一“类”你可以模拟的数据 — 比如一个用户、照片或者Todo笔记。Models 应该要通知它的所有观察者它当前的状态(比如Views)。
* Views通常被认为是应用中的用户界面(比如你的标记和模板)。但不一定是。Views需要知道已经存在的Models以便于观察它们，但不用直接跟它们通讯。
* Controllers控制了应用中的输入(比如点击，用户操作)和视图(可以看作是输出)。当一个Controller更新一个Model的状态时(比如编辑Todo的内容)，它不会直接告诉View。这就是View和Model之间自然的观察关系。

JavaScript‘MVC’框架可以帮助我们不用总是严格的遵循上面的模式来组织代码。有些框架会在View中包含Controller的功能(比如 Backbone.js)，有些框架会整合他们自己认为更有效的组件。

出于这种原因我们把这类框架称之为MV*模式，就是，你可能有View和Model，但是更可能还包含其它东西。

### 什么是Backbone.js?

![](img/backbonejsorg.png)

Backbone.js是一个构建client端代码的轻量级JavaScript框架。它可以非常容易的管理和解耦应用，使你在长远中更容易维护代码。

开发者通常使用Backbone.js创建单页应用或者SPA。简单的说，这些应用可以让浏览器在client端对数据的改变做出响应而不用从服务器端完整地加载你的标记，意味着不用整个重新刷新页面。

在我写这本书的时候Backbone.js是一个成熟、流行的框架，有庞大的开发者社区，丰富的插件和扩展。它被Disqus、Walmart、SoundCloud还有Foursquare用来构建伟大的应用。

### 你何时需要一个Javascript MV*框架?

当用JavaScript构建一个单页应用的时候，不管它包含一个复杂的用户界面还是简单的，尝试减少创建新Views时的HTTP请求，你可能会发现自己通过MV*框架，比如Backbone.js，创建了很多小块。

在起初，写个避免嵌套式代码的框架并不难，但是同样说写些关于Backbone的标准也不重要是不对的。

如何架构一个应用比尝试组合DOM操作库，模板和路由来的更重要。成熟的MV*框架通常不仅包含可能你发现自己会写的一些模块，还包含在之后的过程中你可能会发现的问题的解决方案。这其中节省的时间你不可低估。

所以，你何处需要MV*框架而何处不需要?

如果你要开发一个只需要跟API或者后台数据服务通讯的应用，应用随着数据在浏览器中的变化会有偏重量级的展现和控制，你将会发现MV*框架非常有用。这类比较好的例子就是GMail和Google Docs。

这种应用通常下载一个包含所有脚本、样式、用户常用任务标记的载体，然后在后台完成添加一系列的行为。你可以在阅读email或者要写的一个文档之间来回切换而根本不需要这个应用去重新渲染整个页面。

但是，如果你要构建一个大部分繁琐的视图/页面依然依赖服务器端的应用，只需要一点Javascript或用jQuery实现一点交互，MV框架就可能有点过重了。当然有复杂的Web应用，对视图的局部刷新可以有效的结合单页应用的方式，但不管怎样，坚持简单的原则总会让你驾驭自如。

一个软件(框架)发展的成熟度不是简单的因其存在了多长时间。而是这个框架的可靠程度和它扮演的角色的重要程度。它在解决通用问题上是否有效？随着开发人员使用它构建更大、更复杂的应用框架是否持续改进？


### 为什么考虑Backbone.js?

Backbone提供了一个最小集的数据结构(Models, Collections)和用户接口(Views, URLs)这些对于构建动态的JavaScript应用非常有用的基本实体。它并不是武断的，意思就是说你可以自由和灵活的以你自己认为舒服的方式来构建web应用。你也可以使用它规定的体系结构，或者扩展它以适合你的需求。

这个框架不关注widgets或者是另一种构建对象的方式——它只提供在应用中处理和查询数据的一套工具。它也不规定使用某个特定的模板引擎——不过你可以自由的使用由Underscore.js(其依赖项之一)提供的Micro-templating，views(视图)也可以使用你自己选择的模板方案绑定到HTML结构。


这里有[大量的](http://backbonejs.org/#examples)应用使用Backbone构建，很显然它扩张的很好。Backbone同样也可以跟其它框架很好的一起工作。 意味着你可以嵌入同AngularJS编写的Backbone widgets到你的应用中，把它跟TypeScript一起使用，或者仅仅使用它里面个别class(比如Models)作为简单apps里的数据支撑。

使用Backbone来构建应用不会有什么性能上的缺陷。它避免了循环调用，双向绑定，恒定轮询数据结构检查更新，并且劲量保持简单。不过，你想反其道而行之，你可以在它之上来实现这些东西。Backbone不会阻止你的。

有了充满生气的插件社区和扩展作者，如果你想实现一些Backbone缺失的行为，可以通过一个补充的项目来做。Backbone对其源代码提供非常有阅读性的文档，任何人都可以容易的理解其幕后发生了些什么。

经过超过2年半的发展，Backbone已是一个成熟的库，并将继续提供构建更好web应用的极简方案。我会定期的使用它，希望你也会像我一样发现它是一个有用的工具库。


### 设置期望

这本书的目的是能创建一个权威和集中的信息库，帮助那些在实际应用中使用Backbone的开发者。如果你有认为需要改进或者扩展的话题，请自由的提交问题(或更好一个pull请求)到这本书的[GitHub页面](https://github.com/addyosmani/backbone-fundamentals)。不久你将会帮助到其它开发者避免你曾经遇到的问题。

这本书的话题包括MVC理论，如何使用Backbone的Models, Views, Collections,以及Routers构建应用。同样也会有更高级的话题比如使用Backbone.js时的模块开发和AMD(使用RequireJS)，常见问题的处理比如嵌套views，使用Backbone和jQuery Mobile如何解决路由问题，等等。

每个章节里你分别可以学到：

<i>第2章, 基本原理</i> 追溯MVC设计模式的历史，介绍在Backbone.js和其它JavaScript框架中它是如何实现的。

<i>第3章, Backbone基本要素</i> 包括Backbone.js framework框架的核心和主要特性，以及使用它时需要知道的一些技术和技巧。

<i>第4章, 练习1: Todos - 第一个Backbone.js App</i>step-by-step带你开发一个简单的客户端Todo List应用。

<i>第5章, 练习2: Book Library - 第一个RESTful Backbone.js App</i> 指导你完成开发开发一个图书馆应用，通过服务器端的REST API把model持久化保存。

<i>第6章, Backbone扩展</i> 讲述了Backbone.Marionette和Thorax，给Backbone.js添加有利于开发大型应用特性的两个扩展框架。

<i>第7章, 常见问题和解决方案</i> 回顾了一些使用Backbone.js时可能会遇到的问题及解决方式。

<i>第8章, 模块化开发</i> 看看AMD模块和RequireJS可以如何用于模块你的代码。

<i>第9章, 练习3: Todos - 第一个模块化Backbone + RequireJS App</i>带你使用RequireJS重写练习1中创建的app ，使它模块化。

<i>第10章, 分页Backbone请求&集合</i> 教你如何使用Backbone.Paginator插件来分页Collections的数据。

<i>第11章, Backbone Boilerplate和Grunt BBB</i> 介绍通过样板代码构建一个新的Backbone.js应用的强大工具。

<i>第12章, 移动应用</i> 解决使用Backbone和jQuery Mobile时会引发的问题。

<i>第13章, Jasmine</i> 如何使用Jasmine测试框架对Backbone代码进行单元测试。

<i>第14章, QUnit</i> 讨论如何使用QUnit做单元测试。

<i>第15章, SinonJS</i> 讨论如何使用SinonJS对Backbone apps进行单元测试。

<i>第16章, 资源</i> 提供附加的Backbone相关的资源参考。

<i>第17章, 总结</i>对Backbone.js开发的世界做一个概括。

<i>第18章, 附录</i> 回到设计模式的讨论，通过MVC与Model-View-Presenter (MVP)模式的对比pattern以及探讨Backbone.js如何与这两者想关联。同时也包含演练如何从头写一个Backbone类似的框架和一些其它话题。
