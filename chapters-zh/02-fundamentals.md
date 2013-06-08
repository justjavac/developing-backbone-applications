# 基本原理

设计模式是一种可以提升应用程序的组织和架构的通用的开发方法。同过使用设计模式，我们可吸取众多开发者反复实践中总结出来的经验。

通常，开发者创建桌面和服务器类应用有丰富的设计模式供他们去选择，但是，在过去的仅仅几年中，这些模式已经应用到了客户端开发中。

在这一章中，我们将会探索改进的MVC模式以及如何使用Backbone.js框架在客户端开发中实现它。

## MVC

MVC(Model-View-Controller)是一种提倡通过分层来改进应用的设计模式。它强制通过第三个组件(Controller)来分离业务数据(Model)，用户界面(View)，控制器通常管理逻辑，用户输入，协调模型与视图间的通讯。这种模式最早是[Trygve Reenskaug](http://en.wikipedia.org/wiki/Trygve_Reenskaug)在Smalltaok-80(1979)中设计的，当初被称之为Model-View-Controller-Editor。1994,[“设计模式: 面向对象软件中可重用性元素”](http://www.amazon.co.uk/Design-patterns-elements-reusable-object-oriented/dp/0201633612) (“GOF”或者“四人帮”一书)中详细定义了MVC，这本书普及了它的应用。


### Smalltalk-80 MVC

随着时间的推移MVC模式变得更加笨重，非常有必要去了解下它早的设计初衷。在70年代，图形用户界面并不多见。有一种方法叫[Separated Presentation(表现分离)](http://martinfowler.com/eaaDev/uiArchs.html)，可以清晰的分离模仿现实世界概念(比如一张图片、一个人)的域对象和被渲染到用户屏幕的描述对象。

Smalltalk-80实现的MVC把这个概念贯彻的更深入，而且有目的性的把应用逻辑从用户界面中分离出来。它的观点是解耦应用这些部分也可以把模型重用到应用中其它的用户界面。这里有些非常有趣的关于Smalltalk-80's MVC架构的事情：

* 一个域元素被当做一个Model，而且用户界面(Views和Controllers)并不知道。
* 表现是View和控制器所关心的，但并不仅有一个view和controller。每个要展现到屏幕的元素都需要有一个View-Controller的组合，所以它们并没有正真的被分离。
* 在这个组合中，Controller扮演的是处理用户输入(比如键盘和鼠标点击事件)的角色，做些用户可感知的事情。
* 它依赖观察者模式在Model变化的时候来更新View。

有时，当开发者知道数十年前观察者模式(现在通常在发布/订阅系统中应用)也是MVC架构的一部分的时候，他们非常的惊讶。在Smalltalk-80的MVC中，View和Controller都观察了Model：Model改变的时候，View则做出响应。一个简单的例子就是基于股票市场数据的应用——因为它要展示实时的信息，所以在Models中的数据有任何改变都 要在View中立即刷新显示。

Martin Fowler在过去的些年中在写关于MVC起源[origins](http://martinfowler.com/eaaDev/uiArchs.html) 方面做了很多杰出的工作。如果你有兴趣了解更多关于Smalltalk-80 MVC的信息，推荐你阅读他的相关成果。

### MVC应用于Web

web严重依赖于HTTP协议，它是无状态的。意思就是说在浏览器和服务器之间没有不间断打开的连接；每个请求都在它们两者之间建立一个新的通讯信道。一旦请求的发起者(例如一个浏览器)获取到了响应连接就关闭。事实上，与许多基于原来的MVC思想开发出来的操作系统中的任何一个相比，这创建了一个完全不同的上下文。MVC的实现需要符合web的上下文。

一个尝试应用MVC到web上下文的服务器端web应用框架的例子是[Ruby On Rails](http://guides.rubyonrails.org/). 

![](img/rails_mvc.png)

它的核心就是我们预期的三个MVC组件——Model, View和Controller体系。在Rails中：

* Models负责应用中的数据，通常用于管理与特定数据库表交互的规则。通常一张表对应一个model，这些models中包含应用的业务逻辑。
* Views负责用户接口，通常采用发送给浏览器的HTML的方式。它们用于发起应用数据的请求。
* Controllers提供models和views之间的连接。它们的职责是处理来自浏览器的请求，从models拿取数据，然后把这些数据提供给views， views再把这些数据展现到浏览器。

虽然在Rails中这种类MVC有清晰的分离，实际上它使用了一种不同的模式[Model2](http://en.wikipedia.org/wiki/Model2)。有一条可以证明，Rails不是从model和controllers通知views，而是直接把model数据传递给view。

就是说，即便是对于从一个URL接受请求的服务器端工作流而言，生成HTML页面作为响应并且从界面分离业务逻辑有非常多的好处。同样道理，在服务器端框架中让UI清晰地与数据记录分离是非常有用的，同样在JavaScript中让UI清晰的与数据模型(models)相分离也非常有用。(后面会提到更多)。

其它服务器端的MVC(比如PHP [Zend](http://zend.com) )实现同样实现了前端控制器([Front Controller](http://en.wikipedia.org/wiki/Front_Controller_pattern))设计模式。这种模式把MVC堆栈层叠在一个单一入口背后。单一入口就是说所有HTTP请求(例如，`http://www.example.com`，`http://www.example.com/whichever-page/`等)更具服务器配置被路由到同一个处理器，不依赖于URL。

当前端控制器接受到一个HTTP请求它会分析然后决定调用哪个类(Controller)和方法(Action)。被选中的Controller Action接管进行处理和与对应的Model交互然后完成这个请求。Controller接受从Model返回的数据，载入对应的View，注入Model数据到View中，然后把响应返回给浏览器。

比如说，我们有一个blog，`www.example.com`，想要编辑一篇文章(通过`id=43`)，就请求`http://www.example.com/article/edit/43`:

在服务器端，前端控制器将分析URL然后调用Article Controller(对应到URL `/article/`的部分)及它的Edit Action(对应到URL的`/edit/`部分)。在Action中有一个调用，Articles Model和它的`Articles::getEntry(43)`方法(43对应到URI的`/43`部分)。它会从数据库返回blog文章的数据用于编辑。然后Article Controller会加载(`article/edit`) View， 它包含注入文章数据到编辑文章内容，标题和其它(元)数据表单的逻辑。最后，HTML的响应结果将返回给浏览器。

正如你想象的，当我们触发表单中的一个保存按钮时需要类似的流程来处理POST请求。POST action的URI可能类似于`/article/save/43`。请求会经过同样的Controller, 不过这次Save Action会调用(取决于`/save/` URI块)，文章的Model将调用`Articles::saveEntry(43)`把编辑的文章保存到数据库，并且浏览器会被重定向到`/article/edit/43`URI以便进一步编辑。

最后，如果用户请求`http://www.example.com/`，前端控制器将调用默认的Controller和Action；比如，Index Controller和它的Index action。在Index Action中有对Articles model的调用，其`Articles::getLastEntries(10)`方法会返回最新的10条blog文章。同时Controller也会加载blog/index View, 它包基本的列举blog文章的逻辑。

下面这张图展示了这种典型的服务器端MVC HTTP request/response生命周期：

![](img/webmvcflow_bacic.png)

服务器端接收一个HTTP请求然后路由到一个单一入口。在入口点，前端控制器分析这个请求并且基于它调用对应Controller的Action。这个过程叫路由选择。Action Model则被要求返回或者保存提交的数据。Model与数据源通讯(例如，数据库或者API)。一旦Model完成它的工作就返回数据给Controller, Controller然后加载对应的View。View使用提供的数据执行表示逻辑(遍历文章，输出标题，内容等) 。最后HTTP响应返回给浏览器。

### 客户端(Client-Side) MVC 和单页应用

一些[研究](http://radar.oreilly.com/2009/07/velocity-making-your-site-fast.html)表明改善延迟对于网站和app的使用有非常积极的影响。这与传统的以服务器为中心，从一个页面跳转到另一个页面需要全部重新载入的web app开发方式是相违背的。即便是有到位的缓存，浏览器仍然需要解析CSS，JavaScript, HTML并且渲染界面。

除了会返回给用户较多重复内容之外，这种方法也会影响延迟和一般的响应性的用户体验。在过去几年中改善这种延迟的趋势都朝着构建单页应(Single Page Applications——SPAs)的方向走——应用在载入一个初始化页面之后能够处理后续哦导航和数据请求，而不需要整个页面的重新载入。

当用户浏览到一个新的view时，view的附加内容需要通过XHR (XMLHttpRequest)去请求，通常与服务器端的REST API或端点通讯。[Ajax](https://en.wikipedia.org/wiki/Ajax_(programming))(Asynchronous JavaScript and XML) 可以异步与服务端通讯，所以数据可以在背后传输和处理，可以让用户不间断的与页面的其它部分交互。它提高了可用性和响应能力。

SPAs同样也可以使用浏览器的高级特性，比如当用户从一个view跳转到另一个view的时候可以使用[History API](http://diveintohtml5.info/history.html) 来更新地址栏的地址。这些URLs同样可以添加到书签和分享应用的状态，无需跳转到完整的新的页面。

典型的SPA由小块的逻辑实体接口组成，每个部分都有他们自己的UI，业务逻辑和数据。一个很好的例子就是购物应用中的购物篮，可以往里面添加元素， 购物篮可能是在页面右上角呈现给用户的一个盒子：

![](img/wireframe_e_commerce.png)

购物篮和它的数据呈现在HTML中。数据和它在HTML中关联的View会随着一起变化。曾经我们使用jQuery (或者类似的DOM操作库),一堆的Ajax调用和回调来保持他们两者的同步。那样经常产生结构不好，不易维护的代码。Bug频繁出现或不可避免。

对于需要快速，复杂和响应的Ajax支持的web应用复制了很多这样的逻辑在客户端上，极大的提高了代码的规模和复杂性。最终，把我们带向需要在客户端上实现MVC(或类似的架构)，以便更好的构建代码，在应用生命周期里更容易维护和扩展。

经过反复的尝试也演变，JavaScript开发者利用传统MVC模式的力量，开发出一些受MVC启发的JavaScript框架，比如Backbone.js。


### 客户端MVC - Backbone风格

我们通过一Todo应用示例来看下Backbone.js如何带来客户端MVC开发的好处。后面的章节我们会基于这个例子来探索Backbone的特性，不过目前我们只需要关心核心组件与MVC之间的联系。

示例中需要div元素来展现一个Todo列表。同时需要一个HTML模板，包含Todo标题，完成复选框的占位符，用于实例化一个Todo项实例。下面是相关的HTML：

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title></title>
  <meta name="description" content="">
</head>
<body>
  <div id="todo">
  </div>
  <script type="text/template" id="item-template">
    <div>
      <input id="todo_complete" type="checkbox" <%= completed ? 'checked="checked"' : '' %>>
      <%- title %>
    </div>
  </script>
  <script src="jquery.js"></script>
  <script src="underscore.js"></script>
  <script src="backbone.js"></script>
  <script src="demo.js"></script>
</body>
</html>
```

在Todo application (demo.js)中, Backbone Model 实例用于持有每个Todo项的数据：
```javascript
// Define a Todo Model
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  }
});

// Instantiate the Todo Model with a title, allowing completed attribute
// to default to false
var myTodo = new Todo({
  title: 'Check attributes property of the logged models in the console.'
});
```

Todo Model扩展自Backbone.Model，简单的定义了默认的两个数据属性。接下来的章节中你会发现Backbone Model提供了很多特性，不过这个简单的Model只是为了说明， 首先最重要的是Model是一个数据容器。

每个Todo实例都会通过TodoView渲染到页面上：

```javascript
var TodoView = Backbone.View.extend({

  tagName:  'li',

  // Cache the template function for a single item.
  todoTpl: _.template( $('#item-template').html() ),

  events: {
    'dblclick label': 'edit',
    'keypress .edit': 'updateOnEnter',
    'blur .edit':   'close'
  },

  // Called when the view is first created
  initialize: function () {
    this.$el = $('#todo');
    // Later we'll look at:
    // this.listenTo(someCollection, 'all', this.render);
    // but you can actually run this example right now by
    // calling TodoView.render();
  },

  // Re-render the titles of the todo item.
  render: function() {
    this.$el.html( this.todoTpl( this.model.toJSON() ) );
    // $el here is a reference to the jQuery element 
    // associated with the view, todoTpl is a reference
    // to an Underscore template and toJSON() returns an 
    // object containing the model's attributes
    // Altogether, the statement is replacing the HTML of
    // a DOM element with the result of instantiating a 
    // template with the model's attributes.
    this.input = this.$('.edit');
    return this;
  },

  edit: function() {
    // executed when todo label is double clicked
  },

  close: function() {
    // executed when todo loses focus
  },

  updateOnEnter: function( e ) {
    // executed on each keypress when in todo edit mode, 
    // but we'll wait for enter to get in action
  }
});

// create a view for a todo
var todoView = new TodoView({model: myTodo});
```

TodoView通过扩展自Backbone.View来定义并且使用一个对应的Model进行初始化。例子中，```render()```方法用了一个模板来构建Todo项的HTML存放到一个li元素内。每次```render()```调用都会使用当前的Model数据替换li的内容。因此，一个View实例使用对应的Model的属性来渲染DOM元素的内容。后面我们会讲到一个View可以把它的```render()```方法绑定到Model的change事件，当Model改变的时候就会触发View的重新渲染。

现在，我们已经看到了Backbone.Model实现了MVC的Model，Backbone.View实现了View。不过，正如我们前面提到的，对于Controllers，Backbone跟传统的MVC是不相同的——因为根本没有Backbone.Controller!

不过，Controller的功能已经包含在View里。回想一下，Controllers响应请求并且执行相应的事件，事件则有可能触发Model的改变并更新到View。在单页应用中，不同于传统意义上的请求，我们有事件。事件可以是传统的浏览器事件(比如，click)或者内部的应用事件比如Model changes。 

在我们的这个TodoView种，```events```属性就扮演了Controller配置的角色，定义了View的DOM元素内触发的事件如何路由到View内定义的事件处理方法。

在这个示例中events帮我们把Backbone关联到MVC模式，我们将会看到它们在SPA应用中扮演强大的角色。Backbone.Event是Backbone中一个基本的组件，混入到在Backbone.Model和Backbone.View之中，为它们提供丰富的事件管理功能。注意，传统的controller角色(Smalltalk-80风格)是由模板(template)执行的，而不是Backbone.View。

到这里我们就完成了与Backbone.js初次相遇。在这本书的后面部分我们将会探索这个框架在这些简单的结构基础之上的许多特性。不过，在此之前我们来看下JavaScript MV*框架的通用特性。

### 实现细节

一个SPA通过一个普通的HTTP请求和响应载入到浏览器。页面可能是一个简单的HTML文件，正如我们上面的例子一样，或者是一个由服务器端MVC构建的View。

一旦载入，客户端的Router就会拦截URLs并且触发客户端的逻辑，以替代发送一个新的请求道服务器端。下面这张图显示了Backbone实现的客户端MVC中典型的请求处理:

![](img/backbone_mvc.png)

URL路由，DOM事件(比如，鼠标点击)，以及Model事件(比如，属性changes)在View中所有触发器的处理逻辑(handling logic)。handlers会更新DOM和 Models，这有也可能触发其它事件。Models与数据源同步时有可能带来与后端服务器的通讯。

#### 模型(Models)

* Models的内置功能不同的框架会有所不同；不过，它们都共同支持属性验证，attributes代表Model的属性，比如Model的id标识。

* 在实际应用中使用model的时候通常我们还需要一种方式对model进行持久化。持久化保存可以让我们在对model进行编辑和更新的时候保存它的最新状态。比如在浏览器中使用本地存储，或者与数据库同步。

* 一个model有可能会有多个views来观察它的变化。*观察*意思是View注册了一个当Model有任何改变时的消息通知。这可以让View确保显示在频幕上的东西与model的数据保持一致。根据你的需求，你可能会创建单个View来显示所有的Model属性，或者创建单独的Views来显示不同的属性。重点是Model并不关心这些Views是如何组织的，它只是在必要的时候简单的更新它的数据并且通过框架的事件系统来通知更新。

* 对于现代MVC/MV*框架，提供一种模型组合的方法并不常见。在Backbone中，这些组合叫"Collections"(集合)。把模型组合来管理可以让我们编写应用逻辑时基于一个组合来通知，它包含了任何一个model的改变。这样也避免了手动去观察单个的model实例。后面我们会提到。Collections在执行夸多个model的计算时也非常有用。


#### 视图(Views)

* 用户与view进行交互，通常就是阅读或者编辑model的数据。 比如，在我们的这个todo应用案例中，todo model的视图展现发生在显示所有todo项列表的界面里。每个tod都用一个标题和完成复选框渲染。Model的编辑发生在“编辑”界面，用户选中一个特定的todo可以在form表单中编辑它的title。

* 在view里定义了一个```render()```方法，用JavaScript模板引擎 ([Underscore](http://underscorejs.org "Underscore.js")渲染和更新```this.el```引用的视图内容。

* 然后添加```render()```的回调作为Model的订阅者(subscribers)，这样view就可以在model改变的时候触发更新。

* 你可能想知道对用户交互行为这里是如何处理的。当用户点击view中的一个Todo项时，接下来该做什么并不是由view决定的，而是由Controller决定的。在Backbone里，通过添加一个事件监听到Todo元素，把点击的处理委托给一个事件处理器。

**模板(Templating)**

在支持MVC/MV*的JavaScript框架北京下，非常值得近距离的去审视下JavaScript模板和View之间的关系。

长期实践证明通过手工拼接字符串来创建大块的HTML片段是非常低效的。使用这种方式的开发者们经常会发现，他们遍历自己的数据，包裹在嵌套的div里面，然后使用过时的技术，比如```document.write```把所谓的'template'插入到DOM中。这种方式意味着必须使用标准的标签，脚本代码要放在页面内，而且很快就会变得难以阅读和维护，特别是对于构建大的应用来说。

JavaScript模板库(比如Handlebars.js or Mustache)通常用于view中定义模板，在HMTL标签中包含了一些模板变量。这些模板块可以保存在外部也可以保存在自定义类型(比如'text/template')的script标签里。变量通过变量语法(比如Underscore里的`<%= title %>`，Underscore里的`{{title}}`)来定义。

Javascript 模板库通常接受很多种格式的数据，包括JSON；序列化的格式始终是字符串。往模板中填充数据这种繁重的工作也由框架自身来完成。使用模板库有非常多的好处，特别是当模板存在在外部时，应用可以根据需要动态的加载模板。

让我们来比较下2个HTML模板的列子。一个使用流行的Handlebars.js库实现，另一个使用Underscore的'microtemplates'。

**Handlebars.js:**

```html
<div class="view">
  <input class="toggle" type="checkbox" {{#if completed}} "checked" {{/if}}>
  <label>{{title}}</label>
  <button class="destroy"></button>
</div>
<input class="edit" value="{{title}}">
```

**Underscore.js Microtemplates:**

```html
<div class="view">
  <input class="toggle" type="checkbox" <%= completed ? 'checked' : '' %>>
  <label><%- title %></label>
  <button class="destroy"></button>
</div>
<input class="edit" value="<%= title %>">
```

在Microtemplates中，你也可以使用双大括号(比如```{{}}```) (或者其它你认为爽的字符)。使用大括号的话，可以向下面这样设置Underscore的```templateSettings``` 属性:

```javascript
_.templateSettings = { interpolate : /\{\{(.+?)\}\}/g };
```

**关于导航和状态的注意事项**

值得关注的是，在传统web开发中，在独立的view之间导航需要刷新页面。而在单页应用中，通过ajax从服务器端获取数据，可以在同一个页面里动态的渲染一个新的view，因为不会自动更新URL，导航的角色就落到了"router"(路由)的身上，路由独立的管理应用状态(比如允许用户收藏一个他们浏览过的view)。然而，路由并不是MVC或者类MVC框架的一部分，所以在这部分我并不打算介绍更多的细节。

#### 控制器(Controllers)

在我们的Todo应用中，Controller负责处理在编辑View中用户对指定Todo的改变，当用户完成编辑时更新指定的Todo Model。

大部分JavaScript MVC框架可通过Controllers与传统MVC模式的说法区分开来。在我看来，这种变化的原因，可能Javascript框架作者最初参照了服务端MVC的观念(比如Ruby on Rails)。认识到这种方式跟服务器端的并不完全一样，而且MVC中的C在服务器端也不是用于解决管理状态问题。这是一个非常聪明的途径，但对于初学者来说更难以理解传统MVC模式和无Javascript框架中controller的特定意义。

那Backbone.js有Controller吗?并不真正有。Backbone的Views通常包含了"controller"的逻辑，而且Routers(后面会讨论)也用于帮助管理应用状态，但这两者都不是传统MVC模式中真正意义上的控制器。

在这方面，与官方文档或者网络博客中描述的相反，Backbone并不是正真的MVC框架。事实上，它更适合归类到MV*家族中，它有自己的实现架构。当然这并没有什么不对，只是帮助你区分和理解传统MVC与你在Backbone项目中的MV*。


## MVC给我们带来了什么?

总的来说，MVC模式可以帮助我们把应用的逻辑从界面中分离，使它们更容易修改和维护。基于这种分离的逻辑，对于数据的变化，用户界面，或者业务逻辑以及需要编写的单元测试都将非常的清晰。

### 深入探索MVC

现在，相信你对MVC模式已经有基本的了解了。为了满足大家的求知欲，这里我们将探索的更深入一点。

GoF (Gang of Four，四人组，　《Design Patterns: Elements of Reusable Object-Oriented Software》/《设计模式》一书的作者：Erich Gamma、Richard Helm、Ralph Johnson、John Vlissides)并没有把MVC提及为一种设计模式，而是把它当做"一组用于构建用户界面的类集合"。在他们看来，它其实是其它三个经典的设计模式的演变：观察者模式(Observer)(Pub/Sub), 策略模式(Strategy)和组合模式(Composite)。根据MVC在框架中的实现不同可能还会用到工厂模式(Factory)和装饰器(Decorator)模式。我在另一本免费的书"JavaScript Design Patterns For Beginners"中讲述了这些模式，如果你有兴趣可以阅读更多信息。

正如我们所讨论的，models表示应用的数据，而views处理屏幕上展现给用户的内容。为此，MVC在核心通讯上基于推送/订阅模型(惊讶的是在很多关于MVC的文章中并没有提及到)。当一个model变化时它对应用其它模块发出更新通知("publishes")，订阅者(subscriber)——通常是一个Controller，然后更新对应的view。观察者——这种自然的观察关系促进了多个view关联到同一个model。

对于感兴趣的开发人员想更多的了解解耦性的MVC(根据不同的实现)，这种模式的目标之一就是在一个主题和它的观察者之间建立一对多的关系。当这个主题改变的时候，它的观察者也会得到更新。Views和controllers的关系稍微有点不同。Controllers帮助views对不同用户的输入做不同的响应，是一个非常好的策略模式列子。


### 总结

已经回顾了经典的MVC模式，你现在应该明白了它是如何让开发者将一个应用清晰的分离开来。你应该也能区分出JavaScript MVC框架可能在实现上与原有模式的相似与不同。

当评定一个新的JavaScript MVC/MV*框架时，请记住——可以退一步想，看看它是如何实现Models, Views, Controllers或者其它备选方案，这样或许更能帮助你理解这个框架。

### 进一步阅读

如果你对Backbone.js使用的MVC变异模式感兴趣的话，可以阅读附录的MVP (Model-View-Presenter)章节。

## Fast facts

### Backbone.js

* 核心组件：Model, View, Collection, Router。采用自己的MV*风格。
* views与models之间基于事件驱动通讯。正如我们所看到的，它直接给每个mode每个属性添加事件监听，可以让开发者更细粒度的控制view的改变。
* 支持通过自定义事件，或者单独的Key-value observing (KVO) 库进行数据绑定。
* 支持RESTful接口，所以models可以轻易的与后端关联。
* 可扩展的事件系统。在Backbone中可以非常[精细](http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/)的支持pub/sub。
* 原型通过```new```关键字来实例化，很多开发者更喜欢这种方式。
* 模板框架无关性，不过默认提供了Underscore的micro-templating。Backbone可以与其他模板框架一起使用比如Handlebars。
* 为构建应用提供清晰和灵活的约定。Backbone不强制使用它的所有组件，有必须的组件就能正常工作。

### 被这些应用所使用

*Disqus*

Disqus选择Backbone.js来够坚挺他们的commenting widget。他们认为这对他们分布式web app的正确选择，Backbone体积小而且容易扩展。

![](img/disqus.png)


*Khan Academy*

提供可在任何地方都能使用的世界一流的免费教育，Khan使用Backbone来保持他们前端代码的模块化和有组织化。

![](img/khan-academy.png)


*MetaLab*

MetaLab使用Backbone为团队提供流程创建，任务管理的app。他们的工作空间使用Backbone创建任务的views，活动，账户，标签等等。

![](img/flow.png)

*Walmart Mobile*

Walmart选择Backbone来构建他们的移动web应用，在这个过程中创建了2个新的扩展框架——Thorax和Lumbar。本书的后面我么会讲到这两个框架。

![](img/walmart-mobile.png)


*AirBnb*

Airbnb使用Backbone开发他们的移动web app，并且在很多产品中使用Backbone。

![](img/airbnb.png)


*Code School*

Code School的课程挑战app从一开始就使用Backbone，利用了它提供的所有能力：routers, collections, models 以及复杂的事件处理。

![](img/code-school.png)

