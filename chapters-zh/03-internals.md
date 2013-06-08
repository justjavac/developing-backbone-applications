# Backbone基本构成

在这一章中，你将学习到Backbone的基本元素，models、views、collections和routers。这并不是说这些内容就替代的官方文档，这会在你开始使用它构建应用前帮助你理解Backbone背后的一些核心观念。

### 开始

在投入代码示例前，我们先来定义一些样板标签，可以指定Backbone的依赖项。这些样板可以无需修改标题就能在很多情况下复用，而且可以帮你轻松的运行示例代码。

你可以选择把下面代码复制到你的编辑器中，把script标签里的注释替换成任何想要运行的示例代码就可以了：

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript" src="http://documentcloud.github.com/underscore/underscore-min.js"></script>
<script type="text/javascript" src="http://documentcloud.github.com/backbone/backbone-min.js"></script>
<script type="text/javascript">
  // Your code goes here
</script>
</body>
</html>
```

你可以保存文件后在浏览器中浏览你的页面。不过，如果你更喜欢使用在线编辑器的话，[jsFiddle](http://jsfiddle.net/jnf8B/)或者[jsBin](http://jsbin.com/iwiwox/1/edit)，上面的样板代码也同样可用。

大部分示例都可以直接在浏览器developer tools的console中运行，如果你加载了上面的样板HTML页面的话，这样Backbone的依赖项才是可用。 

![](img/devtools.png)
 

## 模型(Models)

Backbone的models包含了应用中的交互是数据，以及数据的相关逻辑。比如，我们可以用一个model来代表一个todo对象，包含了它的标题(todo的内容)，已完成标识(todo当前的状态)。

Models可以通过继承`Backbone.Model`来创建：

```javascript
var Todo = Backbone.Model.extend({});

// We can then create our own concrete instance of a (Todo) model
// with no values at all:
var todo1 = new Todo();
// Following logs: {}
console.log(JSON.stringify(todo1));

// or with some arbitrary data:
var todo2 = new Todo({
  title: 'Check the attributes of both model instances in the console.',
  completed: true
});

// Following logs: {"title":"Check the attributes of both model instances in the console.","completed":true}
console.log(JSON.stringify(todo2));
```

#### 初始化

`initialize()`方法在当一个model创建一个新的实例是调用。它是可选的，不过你最好像下面这样去使用它，后面你会发现其好处的原因。

```javascript
var Todo = Backbone.Model.extend({
  initialize: function(){
      console.log('This model has been initialized.');
  }
});

var myTodo = new Todo();
// Logs: This model has been initialized.
```

**默认值**

当你想给model设置默认属性时(比如，当用户不会提供一份完整的数据时)，可以用`defaults`属性。

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  }
});

// Now we can create our concrete instance of the model
// with default values as follows:
var todo1 = new Todo();

// Following logs: {"title":"","completed":false}
console.log(JSON.stringify(todo1));

// Or we could instantiate it with some of the attributes (e.g., with custom title):
var todo2 = new Todo({
  title: 'Check attributes of the logged models in the console.'
});

// Following logs: {"title":"Check attributes of the logged models in the console.","completed":false}
console.log(JSON.stringify(todo2));

// Or override all of the default attributes:
var todo3 = new Todo({
  title: 'This todo is done, so take no action on this one.',
  completed: true
});

// Following logs: {"title":"This todo is done, so take no action on this one.","completed":true} 
console.log(JSON.stringify(todo3));
```

#### Getters & Setters

**Model.get()**

`Model.get()`提供了简单的对模型属性的访问。 

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  }
});

var todo1 = new Todo();
console.log(todo1.get('title')); // empty string
console.log(todo1.get('completed')); // false

var todo2 = new Todo({
  title: "Retrieved with model's get() method.",
  completed: true
});
console.log(todo2.get('title')); // Retrieved with model's get() method.
console.log(todo2.get('completed')); // true
```

如果你想读取或者复制model的所有数据，可以使用它的`toJSON()`方法。这个方法复制其属性作为一个对象返回(不是JSON 字符串，虽然其名字有点像)。(当使用`JSON.stringify()`传入一个对象调用`toJSON()`方法的结果时，它字符串化`toJSON()`的返回值，而不是原始的对象。前面这个例子使用这个特性改进下，调用`JSON.stringify()`来log model的实例。)

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  }
});

var todo1 = new Todo();
var todo1Attributes = todo1.toJSON();
// Following logs: {"title":"","completed":false} 
console.log(todo1Attributes);

var todo2 = new Todo({
  title: "Try these examples and check results in console.",
  completed: true
});

// logs: {"title":"Try these examples and check results in console.","completed":true}
console.log(todo2.toJSON());
```

**Model.set()**

`Model.set()`给model设置包含一个或多个属性的hash对象。当这些属性任何一个改变model的状态时，"change"事件就会触发。每个属性的Change事件都可以触发和绑定(比如 `change:name`, `change:age`)。

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  }
});

// Setting the value of attributes via instantiation
var myTodo = new Todo({
  title: "Set through instantiation."
});
console.log('Todo title: ' + myTodo.get('title')); // Todo title: Set through instantiation.
console.log('Completed: ' + myTodo.get('completed')); // Completed: false

// Set single attribute value at a time through Model.set():
myTodo.set("title", "Title attribute set through Model.set().");
console.log('Todo title: ' + myTodo.get('title')); // Todo title: Title attribute set through Model.set().
console.log('Completed: ' + myTodo.get('completed')); // Completed: false

// Set map of attributes through Model.set():
myTodo.set({
  title: "Both attributes set through Model.set().",
  completed: true
});
console.log('Todo title: ' + myTodo.get('title')); // Todo title: Both attributes set through Model.set().
console.log('Completed: ' + myTodo.get('completed')); // Completed: true
```

**直接访问**

Models提供了一个`.attributes`属性，是包含model内部状态的一个hash。通常是以JSON对象的形式，与在服务器端的model数据基本相似，不过也可以是其它形式。

通过`.attributes`属性来给model设置值会绕过绑定在model上的触发器(triggers或者说事件)。

改变model属性的时候传入`{silent:true}`并不会避免特定的`"change:attr"`事件，而是完全沉默：

```javascript
var Person = new Backbone.Model();
Person.set({name: 'Jeremy'}, {silent: true});

console.log(!Person.hasChanged(0));
// true
console.log(!Person.hasChanged(''));
// true
```

请记住，最好使用`Model.set()`或者像前面一样直接实例化来设置model的属性。

#### 监听model的变化

如果你想在Backbone model改变接收到通知可以监听它的change事件。可以在`initialize()`函数中添加监听：

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  },
  initialize: function(){
    console.log('This model has been initialized.');
    this.on('change', function(){
        console.log('- Values for this model have changed.');
    });
  }
});

var myTodo = new Todo();

myTodo.set('title', 'The listener is triggered whenever an attribute value changes.');
console.log('Title has changed: ' + myTodo.get('title'));


myTodo.set('completed', true);
console.log('Completed has changed: ' + myTodo.get('completed'));

myTodo.set({
  title: 'Changing more than one attribute at the same time only triggers the listener once.',
  completed: true
});

// Above logs:
// This model has been initialized.
// - Values for this model have changed.
// Title has changed: The listener is triggered whenever an attribute value changes.
// - Values for this model have changed.
// Completed has changed: true
// - Values for this model have changed.
```

在Backbone model中也可以监听个别的属性变化。下面这个示例，当指定属性(Todo model的title)修改时打印日志。

```javascript
var Todo = Backbone.Model.extend({
  // Default todo attribute values
  defaults: {
    title: '',
    completed: false
  },

  initialize: function(){
    console.log('This model has been initialized.');
    this.on('change:title', function(){
        console.log('Title value for this model has changed.');
    });
  },

  setTitle: function(newTitle){
    this.set({ title: newTitle });
  }
});

var myTodo = new Todo();

// Both of the following changes trigger the listener:
myTodo.set('title', 'Check what\'s logged.');
myTodo.setTitle('Go fishing on Sunday.');

// But, this change type is not observed, so no listener is triggered:
myTodo.set('completed', true);
console.log('Todo set as completed: ' + myTodo.get('completed'));

// Above logs:
// This model has been initialized.
// Title value for this model has changed.
// Title value for this model has changed.
// Todo set as completed: true
```


#### Validation

Backbone支持通过`Model.validate()`对model进行验证，可以在设置model的属性前对值进行校验。默认情况下，验证会在model调用`save()`或者调用`set()`传入`{validate:true}`参数时触发。

```javascript
var Person = new Backbone.Model({name: 'Jeremy'});

// Validate the model name
Person.validate = function(attrs) {
  if (!attrs.name) {
    return 'I need your name';
  }
};

// Change the name
Person.set({name: 'Samuel'});
console.log(Person.get('name'));
// 'Samuel'

// Remove the name attribute, force validation
Person.unset('name', {validate: true});
// false
```

上面，我们同样在unset()`方法时做验证，它可以从内部model的hash中删除一个属性。

验证函数可以简单也可以看需要而极为复杂。如果属性值验证通过，`.validate()`方法可以不返回任何值。如果验证失败，需要返回一个自定义错误(error)。

如果返回error：

* 将触发一个`invalid`事件，model的`validationError`属性被设置成`.validate()`返回的值。
* `.save()`将不会继续，服务器端的model属性页不会被修改。

下面是一个更复杂点的验证的例子：

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    completed: false
  },

  validate: function(attribs){
    if(attribs.title === undefined){
        return "Remember to set a title for your todo.";
    }
  },

  initialize: function(){
    console.log('This model has been initialized.');
    this.on("invalid", function(model, error){
        console.log(error);
    });
  }
});

var myTodo = new Todo();
myTodo.set('completed', true, {validate: true}); // logs: Remember to set a title for your todo.
console.log('completed: ' + myTodo.get('completed')); // completed: false
```

**提示**: 传递给`validate`函数的`attributes`对象表示完成当前的`set()`或者`save()`之后的属性。这个对象与当前的model属性和传递给这个操作的参数都是不同的。因为它是浅拷贝，所以在该函数内不会修改传入的Number, String, 或者Boolean类型的值，但是可以改变嵌套对象的属性值。

这里有一个相关的[例子](http://jsfiddle.net/2NdDY/7/)(by @fivetanley)。


## 视图(Views)

Backbone中的Views不包含应用中的标记，但是它们定义models如何呈现给用户的逻辑。通常通过JavaScript模板来完成(比如：Mustache, jQuery-tmpl等)。view的`render()`方法可以绑定到model的`change()`事件上,这样view就可以保持更新而不用刷新整个页面。


#### 创建一个views

创建一个view跟前面创建一个model一样类似的简单。 通过扩展自`Backbone.View`创建一个view。在前面的章节中，我们介绍了下面的这个TodoView示例；现在我们进一步的来看下它是如何工作的。

```javascript
var TodoView = Backbone.View.extend({

  tagName:  'li',

  // Cache the template function for a single item.
  todoTpl: _.template( "An example template" ),

  events: {
    'dblclick label': 'edit',
    'keypress .edit': 'updateOnEnter',
    'blur .edit':   'close'
  },

  // Re-render the titles of the todo item.
  render: function() {
    this.$el.html( this.todoTpl( this.model.toJSON() ) );
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

var todoView = new TodoView();

// log reference to a DOM element that corresponds to the view instance
console.log(todoView.el); // logs <li></li>
```

#### 什么是el?

`el`(上面例子中最后一行打印的值)是view的一个核心属性。那什么是`el`，它是如何定义的？

`el`通常是DOM元素的引用，所有views都必须有一个。所有view的内容都一次性插入这个DOM，可以让让浏览器执行最小化的重绘，渲染更快。

有2种方式给view指定一个DOM元素：元素在页面中已存在，或者一个新创建的元素，开发者手动添加。
如果元素已经存在，你可以设置`el`为一个css选择器或者直接对DOM的引用。

如果要给view创建一个新的元素，设置view属性的任意组合：`tagName`, `id`和`className`。框架会为你创建一个新的元素，并且可以通过`el`属性来引用这个元素。如果没有指定`tagName`的值，默认会是`div`。

上面这个例子中，`tagName`设为'li'，就会创建一个li元素。下面这个例子会创建一个ul元素，并且包含id和class属性:

```javascript
var TodosView = Backbone.View.extend({
  tagName: 'ul', // required, but defaults to 'div' if not set
  className: 'container', // optional, you can assign multiple classes to this property like so: 'container homepage'
  id: 'todos', // optional
});

var todosView = new TodosView();
console.log(todosView.el); // logs <ul id="todos" class="container"></ul>
```

上面的代码创建下面的DOM元素，但不会添加到DOM中。

```html
<ul id="todos" class="container"></ul>
```
如果这个元素已经存在页面中，你可以把`el`设为匹配该元素的CSS选择器。

```javascript
el: '#footer'
```

当创建view时`el`设置为一个存在的元素：

```javascript
var todosView = new TodosView({el: $('#footer')});
```

提示: 当声明View时，`options`, `el`, `tagName`, `id`和`className`都可以定义成函数，如果你期望它们在运行时返回特定的值。

**$el和$()**

View的逻辑通常要在`el`和它嵌套的元素上调用jQuery和Zepto的函数。Backbone通过定义的`$el`属性和`$()`函数可以很容易的做到这点。`view.$el`属性等同于`$(view.el)` ，`view.$(selector)`等同于`$(view.el).find(selector)`。TodosView例子的render方法就看到`this.$el`用于设置元素的HTML，`this.$()`用于查找具有'edit'class名称的子元素。

**setElement**

如果你想把一个已有的Backbone view应用到一个不同的DOM元素上，可以使用`setElement`覆盖this.el需要改变DOM的引用，重新绑定事件到新的元素(并且从老的元素上解除事件绑定)。 

`setElement` 会为你创建缓存的`$el`引用， 把事件委托从老的元素上转移到新的元素上。

```javascript

// We create two DOM elements representing buttons
// which could easily be containers or something else
var button1 = $('<button></button>');
var button2 = $('<button></button>');

// Define a new view
var View = Backbone.View.extend({
      events: {
        click: function(e) {
          console.log(view.el === e.target);
        }
      }
    });

// Create a new instance of the view, applying it
// to button1
var view = new View({el: button1});

// Apply the view to button2 using setElement
view.setElement(button2);

button1.trigger('click'); 
button2.trigger('click'); // returns true
```

"el"属性表示view将会渲染到的其中的标签；要让view最终渲染到页面，需要把它作为一个新元素添加或者追加到一个已有的元素中去。

```javascript

// 我们也可以像下面这样给setElement提供一行的标签(仅仅为了演示它是可以的):
var view = new Backbone.View;
view.setElement('<p><a><b>test</b></a></p>');
view.$('a b').html(); // 输出"test"
```

**理解 `render()`**

`render()`是一个可选方法，定义模板的渲染逻辑。在这里示例中我们会用Underscore的micro-templating，但是你要记得，你也可以使用其它的模板框架。示例中将使用下面的HTML标签：

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
      <%= title %>
    </div>
  </script>
  <script src="underscore-min.js"></script>
  <script src="backbone-min.js"></script>
  <script src="jquery-min.js"></script>
  <script src="example.js"></script>
</body>
</html>
```

Underscore的`_.template`方法把JavaScript模板编译成方法， 在渲染的时候执行。在上面这个view中，通过ID `item-template`获取模板标记，传给`_.template()`去编译，并且当view创建时保存在todoTpl属性上。

`render()`方法中，`toJSON()`方法把model的属性进行编码，然后传给模板。模板返回使用model的title,completed数据对表达式进行计算之后的结果标签。然后把结果设为`el`($el访问)元素HTML内容。

转眼间!在短短几行代码之内，填充模板，给你一个完成数据填充的标签集合。


Backbone通用的惯例是在`render()`末尾返回`this`。这有很多好处：

* 可以让views轻易的在其它父views中重复使用
* 创建一个元素列表而不用单独每个渲染和绘制，只渲染一次填充整个列表。

后面我们会尝试实现它。一个最简单的没有使用ItemView的ListView，其`render`方法可以这样：

```javascript

var ListView = Backbone.View.extend({
  render: function(){
    this.$el.html(this.model.toJSON());
  }
});
```

已足够简单了。现在，假定我们要使用ItemView来构建真个items，加强list行为。ItemView可以像下面这样：

```javascript

var ItemView = Backbone.View.extend({
  events: {},
  render: function(){
    this.$el.html(this.model.toJSON());
    return this;
  }
});

```
注意`render`末尾`return this;`的用处。这中普通的模式可以让我们把它作为子view重复使用。我们也可以利用它在呈现之前做预渲染(pre-render)。需要对ListView的`render`方法做些修改:

```javascript

var ListView = Backbone.View.extend({
  render: function(){

    // 假定items是model暴露的需要呈现的list
    var items = this.model.get('items');

    // Loop through each our items using the Underscore
    // _.each iterator
    _.each(items, function(item){

      // 创建一个新的ItemView实例，传入指定的model项
      var itemView = new ItemView({ model: item });
      // itemView的DOM元素渲染之后追加到ListView的el中。
	  // 这里'return this'可帮助在render之后访问到它的输出("el")
      this.$el.append( itemView.render().el );
    }, this);
  }
});
```

**`events` hash**


Backbone `events` hash可以允许我们添加事件监听到`el`——用户自定义的选择器，或者未指定选择器的时直接监听到`el`。事件以`{"事件名称 选择器": "回调函数"}`的格式表示，支持大量的DOM事件，包括`click`, `submit`, `mouseover`, `dblclick` 还有更多。

```javascript

// A sample view
var TodoView = Backbone.View.extend({
  tagName:  'li',

  // with an events hash containing DOM events
  // specific to an item:
  events: {
    'click .toggle': 'toggleCompleted',
    'dblclick label': 'edit',
    'click .destroy': 'clear',
    'blur .edit': 'close'
  },
```

不是特别明显的是，Backbone用jQuery的`.delegate()`来提供事件代理的支持，但有些改进，`this`始终指向当前的view对象。需要记住的是，events属性中指定的回调函数名称必须在view范围内有一个对应函数。

需要声明的是，委托jQuery事件意味着你不需要担心特定的元素是否已经渲染到DOM。通常使用jQuery绑定事件时你需要担心元素是否一直存在DOM中。

TodoView示例中，edit回调当用户双击`el`元素内的label元素时触发，updateOnEnter在每个'edit'类元素keypress时调用，close在'edit'类元素失去焦点时执行。每个回调函数内都可以使用`this`来引用TodoView对象。

你也可以自己使用`_.bind(this.viewEvent, this)`来绑定方法，实际上跟在events key-value对立面做的是一样的。下面使用`_.bind`当model change的时候重新渲染view。

```javascript

var TodoView = Backbone.View.extend({
  initialize: function() {
    this.model.bind('change', _.bind(this.render, this));
  }
});
```

`_.bind`一次只能指定一个方法，但是支持返回作为绑定函数，意思是说可以在匿名函数上使用`_.bind`。


## 集合(Collections)

Collections是Models的集合，通过扩展自`Backbone.Collection`来创建。

Normally, when creating a collection you'll also want to define a property specifying the type of model that your collection will contain, along with any instance properties required.

In the following example, we create a TodoCollection that will contain our Todo models:

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var TodosCollection = Backbone.Collection.extend({
  model: Todo
});

var myTodo = new Todo({title:'Read the whole book', id: 2});

// pass array of models on collection instantiation
var todos = new TodosCollection([myTodo]);
console.log("Collection size: " + todos.length); // Collection size: 1
```

#### Adding and Removing Models

The preceding example populated the collection using an array of models when it was instantiated. After a collection has been created, models can be added and removed using the `add()` and `remove()` methods:

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var TodosCollection = Backbone.Collection.extend({
  model: Todo,
});

var a = new Todo({ title: 'Go to Jamaica.'}),
    b = new Todo({ title: 'Go to China.'}),
    c = new Todo({ title: 'Go to Disneyland.'});

var todos = new TodosCollection([a,b]);
console.log("Collection size: " + todos.length);
// Logs: Collection size: 2

todos.add(c);
console.log("Collection size: " + todos.length);
// Logs: Collection size: 3

todos.remove([a,b]);
console.log("Collection size: " + todos.length);
// Logs: Collection size: 1

todos.remove(c);
console.log("Collection size: " + todos.length);
// Logs: Collection size: 0
```

Note that `add()` and `remove()` accept both individual models and lists of models.

Also note that when using `add()` on a collection, passing `{merge: true}` causes duplicate models to have their attributes merged in to the existing models, instead of being ignored.

```javascript
var items = new Backbone.Collection;
items.add([{ id : 1, name: "Dog" , age: 3}, { id : 2, name: "cat" , age: 2}]);
items.add([{ id : 1, name: "Bear" }], {merge: true });
items.add([{ id : 2, name: "lion" }]); // merge: false
 
console.log(JSON.stringify(items.toJSON()));
// [{"id":1,"name":"Bear","age":3},{"id":2,"name":"cat","age":2}]
```

#### Retrieving Models

There are a few different ways to retrieve a model from a collection. The most straight-forward is to use `Collection.get()` which accepts a single id as follows:

```javascript
var myTodo = new Todo({title:'Read the whole book', id: 2});

// pass array of models on collection instantiation
var todos = new TodosCollection([myTodo]);

var todo2 = todos.get(2);

// Models, as objects, are passed by reference
console.log(todo2 === myTodo); // true
```

In client-server applications, collections contain models obtained from the server. Anytime you're exchanging data between the client and a server, you will need a way to uniquely identify models. In Backbone, this is done using the `id`, `cid`, and `idAttribute` properties.

Each model in Backbone has an `id`, which is a unique identifier that is either an integer or string (e.g., a UUID). Models also have a `cid` (client id) which is automatically generated by Backbone when the model is created. Either identifier can be used to retrieve a model from a collection. 

The main difference between them is that the `cid` is generated by Backbone; it is helpful when you don't have a true id - this may be the case if your model has yet to be saved to the server or you aren't saving it to a database.

The `idAttribute` is the identifying attribute of the model returned from the server (i.e., the `id` in your database). This tells Backbone which data field from the server should be used to populate the `id` property (think of it as a mapper). By default, it assumes `id`, but this can be customized as needed. For instance, if your server sets a unique attribute on your model named "userId" then you would set `idAttribute` to "userId" in your model definition.

The value of a model's idAttribute should be set by the server when the model is saved. After this point you shouldn't need to set it manually, unless further control is required. 

Internally, `Backbone.Collection` contains an array of models enumerated by their `id` property, if the model instances happen to have one. When `collection.get(id)` is called, this array is checked for existence of the model instance with the corresponding `id`.

```javascript
// extends the previous example

var todoCid = todos.get(todo2.cid);

// As mentioned in previous example, 
// models are passed by reference
console.log(todoCid === myTodo); // true
```

#### Listening for events

As collections represent a group of items, we can listen for `add` and `remove` events which occur when models are added to or removed from a collection. Here's an example:

```javascript
var TodosCollection = new Backbone.Collection();

TodosCollection.on("add", function(todo) {
  console.log("I should " + todo.get("title") + ". Have I done it before? "  + (todo.get("completed") ? 'Yeah!': 'No.' ));
});

TodosCollection.add([
  { title: 'go to Jamaica', completed: false },
  { title: 'go to China', completed: false },
  { title: 'go to Disneyland', completed: true }
]);

// The above logs:
// I should go to Jamaica. Have I done it before? No.
// I should go to China. Have I done it before? No.
// I should go to Disneyland. Have I done it before? Yeah!
```

In addition, we're also able to bind to a `change` event to listen for changes to any of the models in the collection.

```javascript
var TodosCollection = new Backbone.Collection();

// log a message if a model in the collection changes
TodosCollection.on("change:title", function(model) {
    console.log("Changed my mind! I should " + model.get('title'));
});

TodosCollection.add([
  { title: 'go to Jamaica.', completed: false, id: 3 },
]);

var myTodo = TodosCollection.get(3);

myTodo.set('title', 'go fishing');
// Logs: Changed my mind! I should go fishing
```

jQuery-style event maps of the form `obj.on({click: action})` can also be used. These can be clearer than needing three separate calls to `.on` and should align better with the events hash used in Views:

```javascript

var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var myTodo = new Todo();
myTodo.set({title: 'Buy some cookies', completed: true});

myTodo.on({
   'change:title' : titleChanged,
   'change:completed' : stateChanged
});

function titleChanged(){
  console.log('The title was changed!');
}

function stateChanged(){
  console.log('The state was changed!');
}

myTodo.set({title: 'Get the groceries'});
// The title was changed! 
```

Backbone events also support a [once()](http://backbonejs.org/#Events-once) method, which ensures that a callback only fires one time when a notification arrives.It is similar to Node's [once](http://nodejs.org/api/events.html#events_emitter_once_event_listener), or jQuery's [one](http://api.jquery.com/one/). This is particularly useful for when you want to say "the next time something happens, do this".

```javascript
// Define an object with two counters
var TodoCounter = { counterA: 0, counterB: 0 };
// Mix in Backbone Events
_.extend(TodoCounter, Backbone.Events);

// Increment counterA, triggering an event
var incrA = function(){ 
  TodoCounter.counterA += 1; 
  TodoCounter.trigger('event'); 
};

// Increment counterB
var incrB = function(){ 
  TodoCounter.counterB += 1; 
};

// Use once rather than having to explicitly unbind
// our event listener
TodoCounter.once('event', incrA);
TodoCounter.once('event', incrB);

// Trigger the event once again
TodoCounter.trigger('event');

// Check out output
console.log(TodoCounter.counterA === 1); // true
console.log(TodoCounter.counterB === 1); // true
```

`counterA` and `counterB` should only have been incremented once.

#### Resetting/Refreshing Collections

Rather than adding or removing models individually, you might want to update an entire collection at once. `Collection.set()` takes an array of models and performs the necessary add, remove, and change operations required to update the collection.

```javascript
var TodosCollection = new Backbone.Collection();

TodosCollection.add([
    { id: 1, title: 'go to Jamaica.', completed: false },
    { id: 2, title: 'go to China.', completed: false },
    { id: 3, title: 'go to Disneyland.', completed: true }
]);

// we can listen for add/change/remove events
TodosCollection.on("add", function(model) {
  console.log("Added " + model.get('title'));
});

TodosCollection.on("remove", function(model) {
  console.log("Removed " + model.get('title'));
});

TodosCollection.on("change:completed", function(model) {
  console.log("Completed " + model.get('title'));
});

TodosCollection.set([
    { id: 1, title: 'go to Jamaica.', completed: true },
    { id: 2, title: 'go to China.', completed: false },
    { id: 4, title: 'go to Disney World.', completed: false }
]);

// Above logs:
// Removed go to Disneyland.
// Completed go to Jamaica.
// Added go to Disney World.
```

If you need to simply replace the entire content of the collection then `Collection.reset()` can be used:

```javascript
var TodosCollection = new Backbone.Collection();

// we can listen for reset events
TodosCollection.on("reset", function() {
  console.log("Collection reset.");
});

TodosCollection.add([
  { title: 'go to Jamaica.', completed: false },
  { title: 'go to China.', completed: false },
  { title: 'go to Disneyland.', completed: true }
]);

console.log('Collection size: ' + TodosCollection.length); // Collection size: 3

TodosCollection.reset([
  { title: 'go to Cuba.', completed: false }
]);
// Above logs 'Collection reset.'

console.log('Collection size: ' + TodosCollection.length); // Collection size: 1
```

Another useful tip is to use `reset` with no arguments to clear out a collection completely. This is handy when dynamically loading a new page of results where you want to blank out the current page of results.

```javascript
myCollection.reset();
```

Note that using `Collection.reset()` doesn't fire any `add` or `remove` events. A `reset` event is fired instead as shown in the previous example. The reason you might want to use this is to perform super-optimized rendering in extreme cases where individual events are too expensive.

Also note that listening to a [reset](http://backbonejs.org/#Collection-reset) event, the list of previous models is available in `options.previousModels`, for convenience.

```javascript
var Todo = new Backbone.Model();
var Todos = new Backbone.Collection([Todo])
.on('reset', function(Todos, options) {
  console.log(options.previousModels);
  console.log([Todo]);
  console.log(options.previousModels[0] === Todo); // true
});
Todos.reset([]);
```

An `update()` method is available for Collections (which is also available as an option to fetch) for "smart" updating of sets of models. This method attempts to perform smart updating of a collection using a specified list of models. When a model in this list isn't present in the collection, it is added. If it is, its attributes will be merged. Models which are present in the collection but not in the list are removed.

```javascript
var theBeatles = new Collection(['john', 'paul', 'george', 'ringo']);

theBeatles.update(['john', 'paul', 'george', 'pete']);

// Fires a `remove` event for 'ringo', and an `add` event for 'pete'.
// Updates any of john, paul and georges's attributes that may have
// changed over the years.
```

#### Underscore utility functions

Backbone takes full advantage of its hard dependency on Underscore by making many of its utilities directly available on collections:

**`forEach`: iterate over collections**

```javascript
var Todos = new Backbone.Collection();

Todos.add([
  { title: 'go to Belgium.', completed: false },
  { title: 'go to China.', completed: false },
  { title: 'go to Austria.', completed: true }
]);

// iterate over models in the collection
Todos.forEach(function(model){
  console.log(model.get('title'));
});
// Above logs:
// go to Belgium.
// go to China.
// go to Austria.
```

**`sortBy()`: sort a collection on a specific attribute**

```javascript
// sort collection
var sortedByAlphabet = Todos.sortBy(function (todo) {
    return todo.get("title").toLowerCase();
});

console.log("- Now sorted: ");

sortedByAlphabet.forEach(function(model){
  console.log(model.get('title'));
});
// Above logs:
// go to Austria.
// go to Belgium.
// go to China.
```

**`map()`: iterate through a collection, mapping each value through a transformation function**

```javascript
var count = 1;
console.log(Todos.map(function(model){
  return count++ + ". " + model.get('title');;
}));
// Above logs:
//1. go to Belgium.
//2. go to China.
//3. go to Austria.
```

**`min()`/`max()`: retrieve item with the min or max value of an attribute**

```javascript
Todos.max(function(model){
  return model.id;
}).id;

Todos.min(function(model){
  return model.id;
}).id;
```

**`pluck()`: extract a specific attribute**

```javascript
var captions = Todos.pluck('caption');
// returns list of captions
```

**`filter()`: filter a collection**

*Filter by an array of model IDs*

```javascript
var Todos = Backbone.Collection.extend({
  model: Todo,
  filterById: function(ids){
    return this.models.filter(
      function(c) { 
        return _.contains(ids, c.id); 
      })
  }
});
```

**`indexOf()`: return the item at a particular index within a collection**

```javascript
var People = new Backbone.Collection;

People.comparator = function(a, b) {
  return a.get('name') < b.get('name') ? -1 : 1;
};

var tom = new Backbone.Model({name: 'Tom'});
var rob = new Backbone.Model({name: 'Rob'});
var tim = new Backbone.Model({name: 'Tim'});

People.add(tom);
People.add(rob);
People.add(tim);

console.log(People.indexOf(rob) === 0); // true
console.log(People.indexOf(tim) === 1); // true
console.log(People.indexOf(tom) === 2); // true
```

**`any()`: Confirm if any of the values in a collection pass an iterator truth test**

```javascript
Todos.any(function(model){
  return model.id === 100;
});

// or
Todos.some(function(model){
  return model.id === 100;
});
```

**`size()`: return the size of a collection**

```javascript
Todos.size();

// equivalent to
Todos.length;
```

**`isEmpty()`: determine whether a collection is empty**

```javascript
var isEmpty = Todos.isEmpty();
```

**`groupBy()`: group a collection into groups of like items**

```javascript
var Todos = new Backbone.Collection();

Todos.add([
  { title: 'go to Belgium.', completed: false },
  { title: 'go to China.', completed: false },
  { title: 'go to Austria.', completed: true }
]);

// create groups of completed and incomplete models
var byCompleted = Todos.groupBy('completed');
var completed = new Backbone.Collection(byCompleted[true]);
console.log(completed.pluck('title'));
// logs: ["go to Austria."]
```

In addition, several of the Underscore operations on objects are available as methods on Models.

**`pick()`: extract a set of attributes from a model**

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var todo = new Todo({title: 'go to Austria.'});
console.log(todo.pick('title'));
// logs {title: "go to Austria"}
```

**`omit()`: extract all attributes from a model except those listed**

```javascript
var todo = new Todo({title: 'go to Austria.'});
console.log(todo.omit('title'));
// logs {completed: false}
```

**`keys()` and `values()`: get lists of attribute names and values**

```javascript
var todo = new Todo({title: 'go to Austria.'});
console.log(todo.keys());
// logs: ["title", "completed"]

console.log(todo.values());
//logs: ["go to Austria.", false]
```

**`pairs()`: get list of attributes as [key, value] pairs**

```javascript
var todo = new Todo({title: 'go to Austria.'});
var pairs = todo.pairs();

console.log(pairs[0]);
// logs: ["title", "go to Austria."]
console.log(pairs[1]);
// logs: ["completed", false]
```

**`invert()`: create object in which the values are keys and the attributes are values**

```javascript
var todo = new Todo({title: 'go to Austria.'});
console.log(todo.invert());

// logs: {go to Austria.: "title", false: "completed"}
```

The complete list of what Underscore can do can be found in its official [docs](http://documentcloud.github.com/underscore/).

#### Chainable API

Speaking of utility methods, another bit of sugar in Backbone is its support for Underscore’s `chain()` method. Chaining is a common idiom in object-oriented languages; a chain is a sequence of method calls on the same object that are performed in a single statement. While Backbone makes Underscore's array manipulation operations available as methods of Collection objects, they cannot be directly chained since they return arrays rather than the original Collection.

Fortunately, the inclusion of Underscore's `chain()` method enables you to chain calls to these methods on Collections.

The `chain()` method returns an object that has all of the Underscore array operations attached as methods which return that object. The chain ends with a call to the `value()` method which simply returns the resulting array value. In case you haven’t seen it before, the chainable API looks like this:

```javascript
var collection = new Backbone.Collection([
  { name: 'Tim', age: 5 },
  { name: 'Ida', age: 26 },
  { name: 'Rob', age: 55 }
]);

var filteredNames = collection.chain() // start chain, returns wrapper around collection's models
  .filter(function(item) { return item.get('age') > 10; }) // returns wrapped array excluding Tim
  .map(function(item) { return item.get('name'); }) // returns wrapped array containing remaining names
  .value(); // terminates the chain and returns the resulting array

console.log(filteredNames); // logs: ['Ida', 'Rob']
```

Some of the Backbone-specific methods do return `this`, which means they can be chained as well:

```javascript
var collection = new Backbone.Collection();

collection
    .add({ name: 'John', age: 23 })
    .add({ name: 'Harry', age: 33 })
    .add({ name: 'Steve', age: 41 });

var names = collection.pluck('name');

console.log(names); // logs: ['John', 'Harry', 'Steve']
```

## RESTful Persistence

Thus far, all of our example data has been created in the browser. For most single page applications, the models are derived from a data set residing on a server. This is an area in which Backbone dramatically simplifies the code you need to write to perform RESTful synchronization with a server through a simple API on its models and collections.

**Fetching models from the server**

`Collections.fetch()` retrieves a set of models from the server in the form of a JSON array by sending an HTTP GET request to the URL specified by the collection's `url` property (which may be a function). When this data is received, a `set()` will be executed to update the collection.

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var TodosCollection = Backbone.Collection.extend({
  model: Todo,
  url: '/todos'
});

var todos = new TodosCollection();
todos.fetch(); // sends HTTP GET to /todos
```

**Saving models to the server**

While Backbone can retrieve an entire collection of models from the server at once, updates to models are performed individually using the model's `save()` method. When `save()` is called on a model that was fetched from the server, it constructs a URL by appending the model's id to the collection's URL and sends an HTTP PUT to the server. If the model is a new instance that was created in the browser (i.e., it doesn't have an id) then an HTTP POST is sent to the collection's URL. `Collections.create()` can be used to create a new model, add it to the collection, and send it to the server in a single method call.

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var TodosCollection = Backbone.Collection.extend({
  model: Todo,
  url: '/todos'
});

var todos = new TodosCollection();
todos.fetch();

var todo2 = todos.get(2);
todo2.set('title', 'go fishing');
todo2.save(); // sends HTTP PUT to /todos/2

todos.create({title: 'Try out code samples'}); // sends HTTP POST to /todos and adds to collection
```

As mentioned earlier, a model's `validate()` method is called automatically by `save()` and will trigger an `invalid` event on the model if validation fails.

**Deleting models from the server**

A model can be removed from the containing collection and the server by calling its `destroy()` method. Unlike `Collection.remove()` which only removes a model from a collection, `Model.destroy()` will also send an HTTP DELETE to the collection's URL.

```javascript
var Todo = Backbone.Model.extend({
  defaults: {
    title: '',
    completed: false
  }
});

var TodosCollection = Backbone.Collection.extend({
  model: Todo,
  url: '/todos'
});

var todos = new TodosCollection();
todos.fetch();

var todo2 = todos.get(2);
todo2.destroy(); // sends HTTP DELETE to /todos/2 and removes from collection
```

Calling `destroy` on a Model will return `false` if the model `isNew`:

```javascript
var Todo = new Backbone.Model();
console.log(Todo.destroy());
// false
```

**Options**

Each RESTful API method accepts a variety of options. Most importantly, all methods accept success and error callbacks which can be used to customize the handling of server responses. 

Specifying the `{patch: true}` option to `Model.save()` will cause it to use HTTP PATCH to send only the changed attributes (i.e partial updates) to the server instead of the entire model i.e `model.save(attrs, {patch: true})`:

```javascript
// Save partial using PATCH
model.clear().set({id: 1, a: 1, b: 2, c: 3, d: 4});
model.save();
model.save({b: 2, d: 4}, {patch: true});
console.log(this.syncArgs.method);
// 'patch'
```

Similarly, passing the `{reset: true}` option to `Collection.fetch()` will result in the collection being updated using `reset()` rather than `set()`.

See the Backbone.js documentation for full descriptions of the supported options.

## Events

Events are a basic inversion of control. Instead of having one function call another by name, the second function is registered as a handler to be called when a specific event occurs.

The part of your application that has to know how to call the other part of your app has been inverted. This is the core thing that makes it possible for your business logic to not have to know about how your user interface works and is the most powerful thing about the Backbone Events system.

Mastering events is one of the quickest ways to become more productive with Backbone, so let's take a closer look at Backbone's event model.

`Backbone.Events` is mixed into the other Backbone "classes", including:

* Backbone
* Backbone.Model
* Backbone.Collection
* Backbone.Router
* Backbone.History
* Backbone.View

Note that `Backbone.Events` is mixed into the `Backbone` object. Since `Backbone` is globally visible, it can be used as a simple event bus:

```javascript
Backbone.on('event', function() {console.log('Handled Backbone event');});
Backbone.trigger('event'); // logs: Handled Backbone event
```

#### on(), off(), and trigger()

`Backbone.Events` can give any object the ability to bind and trigger custom events. We can mix this module into any object easily and there isn't a requirement for events to be declared before being bound to a callback handler.

Example:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

// Add a custom event
ourObject.on('dance', function(msg){
  console.log('We triggered ' + msg);
});

// Trigger the custom event
ourObject.trigger('dance', 'our event');
```

If you're familiar with jQuery custom events or the concept of Publish/Subscribe, `Backbone.Events` provides a system that is very similar with `on` being analogous to `subscribe` and `trigger` being similar to `publish`.

`on` binds a callback function to an object, as we've done with `dance` in the above example. The callback is invoked whenever the event is triggered.

The official Backbone.js documentation recommends namespacing event names using colons if you end up using quite a few of these on your page. e.g.:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function dancing (msg) { console.log("We started " + msg); }

// Add namespaced custom events
ourObject.on("dance:tap", dancing);
ourObject.on("dance:break", dancing);

// Trigger the custom events
ourObject.trigger("dance:tap", "tap dancing. Yeah!");
ourObject.trigger("dance:break", "break dancing. Yeah!");

// This one triggers nothing as no listener listens for it
ourObject.trigger("dance", "break dancing. Yeah!");
```

A special `all` event is made available in case you would like notifications for every event that occurs on the object (e.g., if you would like to screen events in a single location). The `all` event can be used as follows:


```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function dancing (msg) { console.log("We started " + msg); }

ourObject.on("all", function(eventName){
  console.log("The name of the event passed was " + eventName);
});

// This time each event will be caught with a catch 'all' event listener
ourObject.trigger("dance:tap", "tap dancing. Yeah!");
ourObject.trigger("dance:break", "break dancing. Yeah!");
ourObject.trigger("dance", "break dancing. Yeah!");
```

`off` removes callback functions that were previously bound to an object. Going back to our Publish/Subscribe comparison, think of it as an `unsubscribe` for custom events.

To remove the `dance` event we previously bound to `ourObject`, we would simply do:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function dancing (msg) { console.log("We " + msg); }

// Add namespaced custom events
ourObject.on("dance:tap", dancing);
ourObject.on("dance:break", dancing);

// Trigger the custom events. Each will be caught and acted upon.
ourObject.trigger("dance:tap", "started tap dancing. Yeah!");
ourObject.trigger("dance:break", "started break dancing. Yeah!");

// Removes event bound to the object
ourObject.off("dance:tap");

// Trigger the custom events again, but one is logged.
ourObject.trigger("dance:tap", "stopped tap dancing."); // won't be logged as it's not listened for
ourObject.trigger("dance:break", "break dancing. Yeah!");
```

To remove all callbacks for the event we pass an event name (e.g., `move`) to the `off()` method on the object the event is bound to. If we wish to remove a specific callback, we can pass that callback as the second parameter:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function dancing (msg) { console.log("We are dancing. " + msg); }
function jumping (msg) { console.log("We are jumping. " + msg); }

// Add two listeners to the same event
ourObject.on("move", dancing);
ourObject.on("move", jumping);

// Trigger the events. Both listeners are called.
ourObject.trigger("move", "Yeah!");

// Removes specified listener
ourObject.off("move", dancing);

// Trigger the events again. One listener left.
ourObject.trigger("move", "Yeah, jump, jump!");
```

Finally, as we have seen in our previous examples, `trigger` triggers a callback for a specified event (or a space-separated list of events). e.g.:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function doAction (msg) { console.log("We are " + msg); }

// Add event listeners
ourObject.on("dance", doAction);
ourObject.on("jump", doAction);
ourObject.on("skip", doAction);

// Single event
ourObject.trigger("dance", 'just dancing.');

// Multiple events
ourObject.trigger("dance jump skip", 'very tired from so much action.');
```

`trigger` can pass multiple arguments to the callback function:

```javascript
var ourObject = {};

// Mixin
_.extend(ourObject, Backbone.Events);

function doAction (action, duration) {
  console.log("We are " + action + ' for ' + duration ); 
}

// Add event listeners
ourObject.on("dance", doAction);
ourObject.on("jump", doAction);
ourObject.on("skip", doAction);

// Passing multiple arguments to single event
ourObject.trigger("dance", 'dancing', "5 minutes");

// Passing multiple arguments to multiple events
ourObject.trigger("dance jump skip", 'on fire', "15 minutes");
```

#### listenTo() and stopListening()

While `on()` and `off()` add callbacks directly to an observed object, `listenTo()` tells an object to listen for events on another object, allowing the listener to keep track of the events for which it is listening. `stopListening()` can subsequently be called on the listener to tell it to stop listening for events:

```javascript
var a = _.extend({}, Backbone.Events);
var b = _.extend({}, Backbone.Events);
var c = _.extend({}, Backbone.Events);

// add listeners to A for events on B and C
a.listenTo(b, 'anything', function(event){ console.log("anything happened"); });
a.listenTo(c, 'everything', function(event){ console.log("everything happened"); });

// trigger an event
b.trigger('anything'); // logs: anything happened

// stop listening
a.stopListening();

// A does not receive these events
b.trigger('anything');
c.trigger('everything');
```

`stopListening()` can also be used to selectively stop listening based on the event, model, or callback handler.

If you use `on` and `off` and remove views and their corresponding models at the same time, there are generally no problems. But a problem arises when you remove a view that had registered to be notified about events on a model, but you don't remove the model or call `off` to remove the view's event handler. Since the model has a reference to the view's callback function, the JavaScript garbage collector cannot remove the view from memory. This is called a "ghost view" and is a form of memory leak which is common since the models generally tend to outlive the corresponding views during an application's lifecycle. For details on the topic and a solution, check this [excellent article](http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/) by Derick Bailey. 

Practically, every `on` called on an object also requires an `off` to be called in order for the garbage collector to do its job. `listenTo()` changes that, allowing Views to bind to Model notifications and unbind from all of them with just one call - `stopListening()`.

The default implementation of `View.remove()` makes a call to `stopListening()`, ensuring that any listeners bound using `listenTo()` are unbound before the view is destroyed.

```javascript
var view = new Backbone.View();
var b = _.extend({}, Backbone.Events);

view.listenTo(b, 'all', function(){ console.log(true); });
b.trigger('anything');

view.listenTo(b, 'all', function(){ console.log(false); });
view.remove(); // stopListening() implicitly called
b.trigger('anything');
// logs: true
```

#### Events and Views

Within a View, there are two types of events you can listen for: DOM events and events triggered using the Event API. It is important to understand the differences in how views bind to these events and the context in which their callbacks are invoked.

DOM events can be bound to using the View's `events` property or using `jQuery.on()`. Within callbacks bound using the `events` property, `this` refers to the View object; whereas any callbacks bound directly using jQuery will have `this` set to the handling DOM element by jQuery. All DOM event callbacks are passed an `event` object by jQuery. See `delegateEvents()` in the Backbone documentation for additional details.

Event API events are bound as described in this section. If the event is bound using `on()` on the observed object, a context parameter can be passed as the third argument. If the event is bound using `listenTo()` then within the callback `this` refers to the listener. The arguments passed to Event API callbacks depends on the type of event. See the Catalog of Events in the Backbone documentation for details.

The following example illustrates these differences:

```html
<div id="todo">
    <input type='checkbox' />
</div>
```

```javascript
var View = Backbone.View.extend({

    el: '#todo',

    // bind to DOM event using events property
    events: {
        'click [type="checkbox"]': 'clicked',
    },

    initialize: function () {
        // bind to DOM event using jQuery
        this.$el.click(this.jqueryClicked);

        // bind to API event
        this.on('apiEvent', this.callback);
    },

    // 'this' is view
    clicked: function(event) {
        console.log("events handler for " + this.el.outerHTML);
        this.trigger('apiEvent', event.type);
    },

    // 'this' is handling DOM element
    jqueryClicked: function(event) {
        console.log("jQuery handler for " + this.outerHTML);
    },

    callback: function(eventType) {
        console.log("event type was " + eventType);
    }

});

var view = new View();
```

## Routers

In Backbone, routers provide a way for you to connect URLs (either hash fragments, or
real) to parts of your application. Any piece of your application that you want
to be bookmarkable, shareable, and back-button-able, needs a URL.

Some examples of routes using the hash mark may be seen below:

```javascript
http://example.com/#about
http://example.com/#search/seasonal-horns/page2
```

An application will usually have at least one route mapping a URL route to a function that determines what happens when a user reaches that route. This relationship is defined as follows:

```javascript
'route' : 'mappedFunction'
```

Let's define our first router by extending `Backbone.Router`. For the purposes of this guide, we're going to continue pretending we're creating a complex todo application (something like a personal organizer/planner) that requires a complex TodoRouter.

Note the inline comments in the code example below as they continue our lesson on routers.

```javascript
var TodoRouter = Backbone.Router.extend({
    /* define the route and function maps for this router */
    routes: {
        "about" : "showAbout",
        /* Sample usage: http://example.com/#about */

        "todo/:id" : "getTodo",
        /* This is an example of using a ":param" variable which allows us to match
        any of the components between two URL slashes */
        /* Sample usage: http://example.com/#todo/5 */

        "search/:query" : "searchTodos",
        /* We can also define multiple routes that are bound to the same map function,
        in this case searchTodos(). Note below how we're optionally passing in a
        reference to a page number if one is supplied */
        /* Sample usage: http://example.com/#search/job */

        "search/:query/p:page" : "searchTodos",
        /* As we can see, URLs may contain as many ":param"s as we wish */
        /* Sample usage: http://example.com/#search/job/p1 */

        "todos/:id/download/*documentPath" : "downloadDocument",
        /* This is an example of using a *splat. Splats are able to match any number of
        URL components and can be combined with ":param"s*/
        /* Sample usage: http://example.com/#todos/5/download/files/Meeting_schedule.doc */

        /* If you wish to use splats for anything beyond default routing, it's probably a good
        idea to leave them at the end of a URL otherwise you may need to apply regular
        expression parsing on your fragment */

        "*other"    : "defaultRoute"
        /* This is a default route that also uses a *splat. Consider the
        default route a wildcard for URLs that are either not matched or where
        the user has incorrectly typed in a route path manually */
        /* Sample usage: http://example.com/# <anything> */,

        "optional(/:item)": "optionalItem",
        "named/optional/(y:z)": "namedOptionalItem"
        /* Router URLs also support optional parts via parentheses, without having
           to use a regex.  */
    },

    showAbout: function(){
    },

    getTodo: function(id){
        /*
        Note that the id matched in the above route will be passed to this function
        */
        console.log("You are trying to reach todo " + id);
    },

    searchTodos: function(query, page){
        var page_number = page || 1;
        console.log("Page number: " + page_number + " of the results for todos containing the word: " + query);
    },

    downloadDocument: function(id, path){
    },

    defaultRoute: function(other){
        console.log('Invalid. You attempted to reach:' + other);
    }
});

/* Now that we have a router setup, we need to instantiate it */

var myTodoRouter = new TodoRouter();
```


Backbone offers an opt-in for HTML5 pushState support via `window.history.pushState`. This permits you to define routes such as http://backbonejs.org/just/an/example. This will be supported with automatic degradation when a user's browser doesn't support pushState. Note that it is vastly preferred if you're capable of also supporting pushState on the server side, although it is a little more difficult to implement.

**Is there a limit to the number of routers I should be using?**

Andrew de Andrade has pointed out that DocumentCloud, the creators of Backbone, usually only use a single router in most of their applications. You're very likely to not require more than one or two routers in your own projects; the majority of your application routing can be kept organized in a single router without it getting unwieldy.

#### Backbone.history

Next, we need to initialize `Backbone.history` as it handles `hashchange` events in our application. This will automatically handle routes that have been defined and trigger callbacks when they've been accessed.

The `Backbone.history.start()` method will simply tell Backbone that it's okay to begin monitoring all `hashchange` events as follows:

```javascript
var TodoRouter = Backbone.Router.extend({
  /* define the route and function maps for this router */
  routes: {
    "about" : "showAbout",
    "search/:query" : "searchTodos",
    "search/:query/p:page" : "searchTodos"
  },

  showAbout: function(){},

  searchTodos: function(query, page){
    var page_number = page || 1;
    console.log("Page number: " + page_number + " of the results for todos containing the word: " + query);
  }
});

var myTodoRouter = new TodoRouter();

Backbone.history.start();

// Go to and check console:
// http://localhost/#search/job/p3   logs: Page number: 3 of the results for todos containing the word: job
// http://localhost/#search/job      logs: Page number: 1 of the results for todos containing the word: job 
// etc.
```

Note: To run the last example, you'll need to create a local development environment and test project, which we will cover later on in the book.

If you would like to update the URL to reflect the application state at a particular point, you can use the router's `.navigate()` method. By default, it simply updates your URL fragment without triggering the `hashchange` event:

```javascript
// Let's imagine we would like a specific fragment (edit) once a user opens a single todo
var TodoRouter = Backbone.Router.extend({
  routes: {
    "todo/:id": "viewTodo",
    "todo/:id/edit": "editTodo"
    // ... other routes
  },

  viewTodo: function(id){
    console.log("View todo requested.");
    this.navigate("todo/" + id + '/edit'); // updates the fragment for us, but doesn't trigger the route
  },

  editTodo: function(id) {
    console.log("Edit todo opened.");
  }
});

var myTodoRouter = new TodoRouter();

Backbone.history.start();

// Go to: http://localhost/#todo/4
//
// URL is updated to: http://localhost/#todo/4/edit
// but editTodo() function is not invoked even though location we end up is mapped to it.
//
// logs: View todo requested.
```

It is also possible for `Router.navigate()` to trigger the route along with updating the URL fragment by passing the `trigger:true` option.

Note: This usage is discouraged. The recommended usage is the one described above which creates a bookmarkable URL when your application transitions to a specific state.

```javascript
var TodoRouter = Backbone.Router.extend({
  routes: {
    "todo/:id": "viewTodo",
    "todo/:id/edit": "editTodo"
    // ... other routes
  },

  viewTodo: function(id){
    console.log("View todo requested.");
    this.navigate("todo/" + id + '/edit', {trigger: true}); // updates the fragment and triggers the route as well
  },

  editTodo: function(id) {
    console.log("Edit todo opened.");
  }
});

var myTodoRouter = new TodoRouter();

Backbone.history.start();

// Go to: http://localhost/#todo/4
//
// URL is updated to: http://localhost/#todo/4/edit
// and this time editTodo() function is invoked.
//
// logs:
// View todo requested.
// Edit todo opened.
```

A "route" event is also triggered on the router in addition to being fired on Backbone.history.

```javascript
Backbone.history.on('route', onRoute);

// Trigger 'route' event on router instance."
router.on('route', function(name, args) {
  console.log(name === 'routeEvent'); 
});

location.replace('http://example.com#route-event/x');
Backbone.history.checkUrl();
```

## Backbone’s Sync API

We previously discussed how Backbone supports RESTful persistence via its `fetch()` and `create()` methods on Collections and `save()`, and `delete()` methods on Models. Now we are going to take a closer look at Backbone's sync method which underlies these operations.

The Backbone.sync method is an integral part of Backbone.js. It assumes a jQuery-like `$.ajax()` method, so HTTP parameters are organized based on jQuery’s API. Since some legacy servers may not support JSON-formatted requests and HTTP PUT and DELETE operations, Backbone can be configured to emulate these capabilities using the two configuration variables shown below with their default values:

```javascript
Backbone.emulateHTTP = false; // set to true if server cannot handle HTTP PUT or HTTP DELETE
Backbone.emulateJSON = false; // set to true if server cannot handle application/json requests
```

The inline Backbone.emulateHTTP option should be set to true if extended HTTP methods are not supported by the server. The Backbone.emulateJSON option should be set to true if the server does not understand the MIME type for JSON.

```javascript
// Create a new library collection
var Library = Backbone.Collection.extend({
    url : function() { return '/library'; }
});

// Define attributes for our model
var attrs = {
    title  : "The Tempest",
    author : "Bill Shakespeare",
    length : 123
};
  
// Create a new Library instance
var library = new Library;

// Create a new instance of a model within our collection
library.create(attrs, {wait: false});
  
// Update with just emulateHTTP
library.first().save({id: '2-the-tempest', author: 'Tim Shakespeare'}, {
  emulateHTTP: true
});
    
// Check the ajaxSettings being used for our request
console.log(this.ajaxSettings.url === '/library/2-the-tempest'); // true
console.log(this.ajaxSettings.type === 'POST'); // true
console.log(this.ajaxSettings.contentType === 'application/json'); // true

// Parse the data for the request to confirm it is as expected
var data = JSON.parse(this.ajaxSettings.data);
console.log(data.id === '2-the-tempest');  // true
console.log(data.author === 'Tim Shakespeare'); // true
console.log(data.length === 123); // true
```

Similarly, we could just update using `emulateJSON`:

```javascript
library.first().save({id: '2-the-tempest', author: 'Tim Shakespeare'}, {
  emulateJSON: true
});

console.log(this.ajaxSettings.url === '/library/2-the-tempest'); // true
console.log(this.ajaxSettings.type === 'PUT'); // true
console.log(this.ajaxSettings.contentType ==='application/x-www-form-urlencoded'); // true

var data = JSON.parse(this.ajaxSettings.data.model);
console.log(data.id === '2-the-tempest');
console.log(data.author ==='Tim Shakespeare');
console.log(data.length === 123);
```

`Backbone.sync` is called every time Backbone tries to read, save, or delete models. It uses jQuery or Zepto's `$.ajax()` implementations to make these RESTful requests, however this can be overridden as per your needs.

**Overriding Backbone.sync**

The `sync` function may be overridden globally as Backbone.sync, or at a finer-grained level, by adding a sync function to a Backbone collection or to an individual model.

Since all persistence is handled by the Backbone.sync function, an alternative persistence layer can be used by simply overriding Backbone.sync with a function that has the same signature:

```javascript
Backbone.sync = function(method, model, options) {
};
```

The methodMap below is used by the standard sync implementation to map the method parameter to an HTTP operation and illustrates the type of action required by each method argument:

```javascript
var methodMap = {
  'create': 'POST',
  'update': 'PUT',
  'patch':  'PATCH',
  'delete': 'DELETE',
  'read':   'GET'
};
```

If we wanted to replace the standard `sync` implementation with one that simply logged the calls to sync, we could do this:

```javascript
var id_counter = 1;
Backbone.sync = function(method, model) {
  console.log("I've been passed " + method + " with " + JSON.stringify(model));
  if(method === 'create'){ model.set('id', id_counter++); }
};
```

Note that we assign a unique id to any created models.

The Backbone.sync method is intended to be overridden to support other persistence backends. The built-in method is tailored to a certain breed of RESTful JSON APIs - Backbone was originally extracted from a Ruby on Rails application, which uses HTTP methods like PUT in the same way.

The sync method is called with three parameters:

* method: One of create, update, patch, delete, or read
* model: The Backbone model object
* options: May include success and error methods

Implementing a new sync method can use the following pattern:

```javascript
Backbone.sync = function(method, model, options) {

  function success(result) {
    // Handle successful results from MyAPI
    if (options.success) {
      options.success(result);
    }
  }

  function error(result) {
    // Handle error results from MyAPI
    if (options.error) {
      options.error(result);
    }
  }

  options || (options = {});

  switch (method) {
    case 'create':
      return MyAPI.create(model, success, error);

    case 'update':
      return MyAPI.update(model, success, error);

    case 'patch':
      return MyAPI.patch(model, success, error);

    case 'delete':
      return MyAPI.destroy(model, success, error);

    case 'read':
      if (model.attributes[model.idAttribute]) {
        return MyAPI.find(model, success, error);
      } else {
        return MyAPI.findAll(model, success, error);
      }
  }
};
```

This pattern delegates API calls to a new object (MyAPI), which could be a Backbone-style class that supports events. This can be safely tested separately, and potentially used with libraries other than Backbone.

There are quite a few sync implementations out there. The following examples are all available on GitHub:

* Backbone localStorage: persists to the browser's local storage
* Backbone offline: supports working offline
* Backbone Redis: uses Redis key-value store
* backbone-parse: integrates Backbone with Parse.com
* backbone-websql: stores data to WebSQL
* Backbone Caching Sync: uses local storage as cache for other sync implementations


## Dependencies

The official Backbone.js [documentation](http://backbonejs.org/) states:

>Backbone's only hard dependency is either Underscore.js ( >= 1.4.3) or Lo-Dash. For RESTful persistence, history support via Backbone.Router and DOM manipulation with Backbone.View, include json2.js, and either jQuery ( >= 1.7.0) or Zepto.

What this translates to is that if you require working with anything beyond models, you will need to include a DOM manipulation library such as jQuery or Zepto. Underscore is primarily used for its utility methods (which Backbone relies upon heavily) and json2.js for legacy browser JSON support if Backbone.sync is used.

## Summary

In this chapter we have introduced you to the components you will be using to build applications with Backbone: Models, Views, Collections, and Routers. We've also explored the Events mix-in that Backbone uses to enhance all components with publish-subscribe capabilities and seen how it can be used with arbitrary objects. Finally, we saw how Backbone leverages the Underscore.js and jQuery/Zepto APIs to add rich manipulation and persistence features to Backbone Collections and Models.

Backbone has many operations and options beyond those we have covered here and is always evolving, so be sure to visit the official [documentation](http://backbonejs.org/) for more details and the latest features. In the next chapter you will start to get your hands dirty as we walk you through implementation of your first Backbone application.


