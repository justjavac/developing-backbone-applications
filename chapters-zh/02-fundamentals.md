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

TodoView通过扩展自Backbone.View来定义并且使用一个对应的Model进行初始化。In our example, the ```render()``` method uses a template to construct the HTML for the Todo item which is placed inside a li element. Each call to ```render()``` will replace the content of the li element using the current Model data. Thus, a View instance renders the content of a DOM element using the attributes of an associated Model. Later we will see how a View can bind its ```render()``` method to Model change events, causing the View to re-render whenever the Model changes.

So far, we have seen that Backbone.Model implements the Model aspect of MVC and Backbone.View implements the View. However, as we noted earlier, Backbone departs from traditional MVC when it comes to Controllers - there is no Backbone.Controller!

Instead, the Controller responsibility is addressed within the View. Recall that Controllers respond to requests and perform appropriate actions which may result in changes to the Model and updates to the View. In a single-page application, rather than having requests in the traditional sense, we have events. Events can be traditional browser DOM events (e.g., clicks) or internal application events such as Model changes. 

In our TodoView, the ```events``` attribute fulfills the role of the Controller configuration, defining how events occurring within the View's DOM element are to be routed to event-handling methods defined in the View.

While in this instance events help us relate Backbone to the MVC pattern, we will see them playing a much larger role in our SPA applications. Backbone.Event is a fundamental Backbone component which is mixed into both Backbone.Model and Backbone.View, providing them with rich event management capabilities. Note that the traditional controller role (Smalltalk-80 style) is performed by the template, not by the Backbone.View.

This completes our first encounter with Backbone.js. The remainder of this book will explore the many features of the framework which build on these simple constructs. Before moving on, let's take a look at common features of JavaScript MV* frameworks.

### Implementation Specifics

An SPA is loaded into the browser using a normal HTTP request and response. The page may simply be an HTML file, as in our example above, or it could be a view constructed by a server-side MVC implementation.

Once loaded, a client-side Router intercepts URLs and invokes client-side logic in place of sending a new request to the server. The picture below shows typical request handling for client-side MVC as implemented by Backbone:

![](img/backbone_mvc.png)

URL routing, DOM events (e.g., mouse clicks), and Model events (e.g., attribute changes) all trigger handling logic in the View. The handlers update the DOM and Models, which may trigger additional events. Models are synced with Data Sources which may involve communicating with back-end servers.

#### Models

* The built-in capabilities of Models vary across frameworks; however, it's common for them to support validation of attributes, where attributes represent the properties of the Model, such as a Model identifier.

* When using Models in real-world applications we generally also need a way of persisting Models. Persistence allows us to edit and update Models with the knowledge that their most recent states will be saved somewhere, for example in a web browser's localStorage data-store or synchronized with a database.

* A Model may have multiple Views observing it for changes. By *observing* we mean that a View has registered an interest in being informed whenever an update is made to the Model. This allows the View to ensure that what is displayed on screen is kept in sync with the data contained in the model. Depending on your requirements, you might create a single View displaying all Model attributes, or create separate Views displaying different attributes. The important point is that the Model doesn't care how these Views are organized, it simply announces updates to its data as necessary through the framework's event system.

* It is not uncommon for modern MVC/MV* frameworks to provide a means of grouping Models together. In Backbone, these groups are called Collections. Managing Models in groups allows us to write application logic based on notifications from the group when a Model within the group changes. This avoids the need to manually observe individual Model instances. We'll see this in action later in the book. Collections are also useful for performing any aggregate computations across more than one model.


#### Views

* Users interact with Views, which usually means reading and editing Model data. For example, in our Todo application, Todo Model viewing happens in the user interface in the list of all Todo items. Within it, each Todo is rendered with its title and completed checkbox. Model editing is done through an "edit" View where a user who has selected a specific Todo edits its title in a form.

* We define a ```render()``` utility within our View which is responsible for rendering the contents of the ```Model``` using a JavaScript templating engine (provided by Underscore.js) and updating the contents of our View, referenced by ```this.el```.

* We then add our ```render()``` callback as a Model subscriber, so the View can be triggered to update when the Model changes.

* You may wonder where user interaction comes into play here. When users click on a Todo element within the View, it's not the View's responsibility to know what to do next. A Controller makes this decision. In Backbone, this is achieved by adding an event listener to the Todo's element which delegates handling of the click to an event handler.

**Templating**

In the context of JavaScript frameworks that support MVC/MV*, it is worth looking more closely at JavaScript templating and its relationship to Views.

It has long been considered bad practice (and computationally expensive) to manually create large blocks of HTML markup in-memory through string concatenation. Developers using this technique often find themselves iterating through their data, wrapping it in nested divs and using outdated techniques such as ```document.write``` to inject the 'template' into the DOM. This approach often means keeping scripted markup inline with standard markup, which can quickly become difficult to read and maintain, especially when building large applications.

JavaScript templating libraries (such as Mustache or Handlebars.js) are often used to define templates for Views as HTML markup containing template variables. These template blocks can be either stored externally or within script tags with a custom type (e.g 'text/template'). Variables are delimited using a variable syntax (e.g `<%= title %>` for Underscore and `{{title}}` for Handlebars).

JavaScript template libraries typically accept data in a number of formats, including JSON; a serialisation format that is always a string. The grunt work of populating templates with data is generally taken care of by the framework itself. This has several benefits, particularly when opting to store templates externally which enables applications to load templates dynamically on an as-needed basis.

Let's compare two examples of HTML templates. One is implemented using the popular Handlebars.js library, and the other uses Underscore's 'microtemplates'.

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

You may also use double curly brackets (i.e ```{{}}```) (or any other tag you feel comfortable with) in Microtemplates. In the case of curly brackets, this can be done by setting the Underscore ```templateSettings``` attribute as follows:

```javascript
_.templateSettings = { interpolate : /\{\{(.+?)\}\}/g };
```

**A note on Navigation and State**

It is also worth noting that in classical web development, navigating between independent views required the use of a page refresh. In single-page JavaScript applications, however, once data is fetched from a server via Ajax, it can be dynamically rendered in a new view within the same page. Since this doesn't automatically update the URL, the role of navigation thus falls to a "router", which assists in managing application state (e.g., allowing users to bookmark a particular view they have navigated to). As routers are neither a part of MVC nor present in every MVC-like framework, I will not be going into them in greater detail in this section.

#### Controllers


In our Todo application, a Controller would be responsible for handling changes the user made in the edit View for a particular Todo, updating a specific Todo Model when a user has finished editing.

It's with Controllers that most JavaScript MVC frameworks depart from the traditional interpretation of the MVC pattern. The reasons for this vary, but in my opinion, Javascript framework authors likely initially looked at server-side interpretations of MVC (such as Ruby on Rails), realized that the approach didn't translate 1:1 on the client-side, and so re-interpreted the C in MVC to solve their state management problem. This was a clever approach, but it can make it hard for developers coming to MVC for the first time to understand both the classical MVC pattern and the "proper" role of Controllers in other JavaScript frameworks.

So does Backbone.js have Controllers? Not really. Backbone's Views typically contain "Controller" logic, and Routers are used to help manage application state, but neither are true Controllers according to classical MVC.

In this respect, contrary to what might be mentioned in the official documentation or in blog posts, Backbone isn't truly an MVC framework. It's in fact better to see it a member of the MV* family which approaches architecture in its own way. There is of course nothing wrong with this, but it is important to distinguish between classical MVC and MV* should you be relying on discussions of MVC to help with your Backbone projects.

## What does MVC give us?

To summarize, the MVC pattern helps you keep your application logic separate from your user interface, making it easier to change and maintain both. Thanks to this separation of logic, it is more clear where changes to your data, interface, or business logic need to be made and for what your unit tests should be written. 

### Delving Deeper into MVC

Right now, you likely have a basic understanding of what the MVC pattern provides, but for the curious, we'll explore it a little further.

The GoF (Gang of Four) do not refer to MVC as a design pattern, but rather consider it a "set of classes to build a user interface." In their view, it's actually a variation of three other classical design patterns: the Observer (Publish/Subscribe), Strategy, and Composite patterns. Depending on how MVC has been implemented in a framework, it may also use the Factory and Decorator patterns. I've covered some of these patterns in my other free book, "JavaScript Design Patterns For Beginners" if you would like to read about them further.

As we've discussed, Models represent application data, while Views handle what the user is presented on screen. As such, MVC relies on Publish/Subscribe for some of its core communication (something that surprisingly isn't covered in many articles about the MVC pattern). When a Model is changed it "publishes" to the rest of the application that it has been updated. The "subscriber," generally a Controller, then updates the View accordingly. The observer-viewer nature of this relationship is what facilitates multiple Views being attached to the same Model.

For developers interested in knowing more about the decoupled nature of MVC (once again, depending on the implementation), one of the goals of the pattern is to help define one-to-many relationships between a topic and its observers. When a topic changes, its observers are updated. Views and Controllers have a slightly different relationship. Controllers facilitate Views' responses to different user input and are an example of the Strategy pattern.

### Summary

Having reviewed the classical MVC pattern, you should now understand how it allows developers to cleanly separate concerns in an application. You should also now appreciate how JavaScript MVC frameworks may differ in their interpretation of MVC, and how they share some of the fundamental concepts of the original pattern.

When reviewing a new JavaScript MVC/MV* framework, remember - it can be useful to step back and consider how it's opted to approach Models, Views, Controllers or other alternatives, as this can better help you understand how the framework is intended to be used.

### Further reading

If you are interested in learning more about the variation of MVC which Backbone.js uses, please see the MVP (Model-View-Presenter) section in the appendix.

## Fast facts

### Backbone.js

* Core components: Model, View, Collection, Router. Enforces its own flavor of MV*
* Event-driven communication between Views and Models. As we'll see, it's relatively straight-forward to add event listeners to any attribute in a Model, giving developers fine-grained control over what changes in the View
* Supports data bindings through manual events or a separate Key-value observing (KVO) library
* Support for RESTful interfaces out of the box, so Models can be easily tied to a backend
* Extensive eventing system. It's [trivial](http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/) to add support for pub/sub in Backbone
* Prototypes are instantiated with the ```new``` keyword, which some developers prefer
* Agnostic about templating frameworks, however Underscore's micro-templating is available by default. 
* Clear and flexible conventions for structuring applications. Backbone doesn't force usage of all of its components and can work with only those needed.

### Used by

*Disqus*

Disqus chose Backbone.js to power the latest version of their commenting widget. They felt it was the right choice for their distributed web app, given Backbone's small footprint and ease of extensibility.

![](img/disqus.png)


*Khan Academy*

Offering a web app that aims to provide free world-class education to anyone anywhere, Khan use Backbone to keep their frontend code both modular and organized.

![](img/khan-academy.png)


*MetaLab*

MetaLab created Flow, a task management app for teams using Backbone. Their workspace uses Backbone to create task views, activities, accounts, tags and more.

![](img/flow.png)

*Walmart Mobile*

Walmart chose Backbone to power their mobile web applications, creating two new extension frameworks in the process - Thorax and Lumbar. We'll be discussing both of these later in the book.

![](img/walmart-mobile.png)


*AirBnb*

Airbnb developed their mobile web app using Backbone and now use it across many of their products.

![](img/airbnb.png)


*Code School*

Code School's course challenge app is built from the ground up using Backbone, taking advantage of all the pieces it has to offer: routers, collections, models and complex event handling.

![](img/code-school.png)

