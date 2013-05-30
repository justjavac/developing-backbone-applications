# Unit Testing

单元测试的定义就是把整个应用中小片可测试的代码从代码库中隔离，然后检验它的行为是否跟期望的一致。

一个被称为'好'测试的应用，很明显功能上应该有分开的单元测试，以验证不同情况下它的正确性。所有测试应该在功能完成之前介入。这可以让开发者修改一块代码的时候通过单元测试确认他的修改是否会引发问题，建立代码质量的信心。

一个最基本的单元测试例子，开发者想要断言传入指定的value給一个sum函数其返回结果是否正确。与我们这本书有关的例子就是，我们想要断言一个用户添加一个新的Todo项到列表中，是否添加了一个指定类型的Model到Todos Collection。

构建现代的web应用时，通常认为最好的实践方式是在开发过程中引入自动的单元测试。这里我们关注下使用Jasmine的方案，当然也有很多其它选择值得考虑，包括QUnit。

# Jasmine

## 行为-驱动开发

这一节，我们会讲述如何使用一个流行的测试框架[Jasmine](http://pivotal.github.com/jasmine/)来测试Backbone应用，这个框架来自Pivotal Labs。

Jasmine自称是一个用于测试JavaScript代码的行为-驱动开发(Behavior-Driven Development，BDD)的框架。在开始使用这个框架之前，我们先来弄清楚下什么是[BDD](http://en.wikipedia.org/wiki/Behavior_Driven_Development)。

BDD是一种第二代测试方法，由[Dan North](http://dannorth.net/introducing-bdd/) (BDD方面的权威)首次定义，它是试图测试软件的行为。称之为第二代是因为其想法来自于领域驱动设计(Domain driven design，DDD)和精益软件开发，通过在敏捷过程中回答许多令人困惑的问题来帮助团队产出高质量软件。这类问题通常包含相关文档和测试。

如果阅读过一本关于BDD的书籍，有可能它会被描述成'由外及内的、基于拉(pull)的(outside-in and pull-based)'。原因就是它从精益生产借鉴了pull特性， 通过a) 注重系统的预期输出(outputs),b) 确保这些输出被达到，这2两点有效的确保开发出正确的软件方案。

BDD提出在一个项目中通常有多元利益体并且系统不是只有一个单一的无形用户。这些不同的群体，将会以不同的形式影响所编写的软件，而且软件系统的质量对于他们的意义他们有不同的理解。所以，要明白对于这个软件谁会给你带来价值以及软件会给他们带来什么价值，这点非常重要。

最后，BDD依赖于自动化。一旦定义好你期望的质量，你的团队可能就会定期的检查正在做的功能是否与他们期望的一致。为了促进效率，这个过程需要自动完成。BDDIn order to facilitate this efficiently, the process has to be automated. BDD严重依赖于自动规格测试，而Jasmine正好是一个做这件事的工具。

BDD可以帮助开发者和非技术的利益相关者做到：


* 更好的理解和提出解决问题的模式
* 把测试用例以非开发人员也能读懂的方式解释清楚
* 着眼于最小化编写的技术代码到业务表达语言之间的转换。

这就意味着开发者要把Jasmine单元测试给项目的利益相关者做展示，然后他们在观念上要理解代码的用途。

开发者经常跟另外一种测试方法[TDD](http://en.wikipedia.org/wiki/Test-driven_development) (test-driven development)一样来实施BDD。TDD背后的主要观点：

* 编写单元测试描述你的代码想要支持的功能
* 看着这些测试失败(因为要支持这些功能的代码写好)
* 编写代码让测试通过
* 清理，重复和重构

这一章我们将会用这两种方式(BDD和TDD)来为Backbone应用编写单元测试。

***提示*** 我看到很多开发者仍然是在完成编码之后才编写测试来做验证。虽然这也还不错，不过容易调入陷入，它只能测试到你现在代码所支持的行为，而不一定完整包含我们原本设计需要支持的功能。


##Suites, Specs & Spies

使用Jasmine时，要编写suites(套件)和specs(specifications，规格说明)。Suites主要描述场景，specs描述在这些场景l里要做些什么。

每个spec就是一个一个JavaScript函数，调用```it()```来描述，传入一个描述字符串和一个function。描述语要描述出指定单元代码的展现结果，牢记BDD的观念，表述应该有意义。下面是一个简单的例子：

```javascript
it('should be incrementing in value', function(){
    var counter = 0;
    counter++;
});
```

就其本身而言，一个spec如果没有设置行为代码的期望结果就没有什么用处了。使用```expect()```函数和[expectation matcher](https://github.com/pivotal/jasmine/wiki/Matchers) (比如```toEqual()```, ```toBeTruthy()```, ```toContain()```)来定义期望结果。示例：

```javascript
it('should be incrementing in value', function(){
    var counter = 0;
    counter++;
    expect(counter).toEqual(1);
});
```

上面代码中对```counter```的期望值要等于1。这种代码读起来非常简单(凭直觉就可以理解了，无需任何解释)。

一组Specs就构成了suites，通过Jasmine的```describe()```函数定义，传入一个描述字符串和一个函数。suite的名称或者描述通常是需要测试的组件或者模块。

Jasmine会把它作为给出报告时运行specs的分组名称。下面是一个简单的示例：

```javascript
describe('Stats', function(){
    it('can increment a number', function(){
        ...
    });

    it('can subtract a number', function(){
        ...
    });
});
```

Suites共享一个函数域，所以可以在describe函数内声明变量， specs里的函数也可以访问到：

```javascript
describe('Stats', function(){
    var counter = 1;

    it('can increment a number', function(){
        // the counter was = 1
        counter = counter + 1;
        expect(counter).toEqual(2);
    });

    it('can subtract a number', function(){
        // the counter was = 2
        counter = counter - 1;
        expect(counter).toEqual(1);
    });
});
```

***提示：*** Suites是按其定义的顺序执行，如果你要看整个应用测试报告的某个特定部分的测试结果，这可能非常有用。

Jasmine同样支持**spies(监视)** ——在单元测试中一种模仿，监视，和伪造行为的方法。Spies会替换它们监视的函数，可以模仿我们想要伪造的行为。

在下面这个例子中，我们用一个虚拟的Todo function监视```setComplete```方法，测试传入的参数是否符合期望。

```javascript
var Todo = function(){
};

Todo.prototype.setComplete = function (arg){
    return arg;
}

describe('a simple spy', function(){
    it('should spy on an instance method of a Todo', function(){
        var myTodo = new Todo();
        spyOn(myTodo, 'setComplete');
        myTodo.setComplete('foo bar');

        expect(myTodo.setComplete).toHaveBeenCalledWith('foo bar');

        var myTodo2 = new Todo();
        spyOn(myTodo2, 'setComplete');

        expect(myTodo2.setComplete).not.toHaveBeenCalled();

    });
});
```

更有可能会用到spies的地方是测试[asynchronous(异步)](http://en.wikipedia.org/wiki/Asynchronous_communication)行为，比如AJAX请求。Jasmine支持：

* 使用spies模仿AJAX请求来编写测试。可以在AJAX请求之前和请求之后运行测试代码。也可以伪造服务器端的响应，这种类型的测试好处就是更快，不需要等待实际的服务器调用。
* 异步测试部需要依赖spies

第一种测试，可以伪造AJAX请求，验证请求的URL是否正确以及执行回调，如果有的话。

```javascript
it("the callback should be executed on success", function () {
    spyOn($, "ajax").andCallFake(function(options) {
        options.success();
    });

    var callback = jasmine.createSpy();
    getTodo(15, callback);

    expect($.ajax.mostRecentCall.args[0]["url"]).toEqual("/todos/15");
    expect(callback).toHaveBeenCalled();
});

function getTodo(id, callback) {
    $.ajax({
        type: "GET",
        url: "/todos/" + id,
        dataType: "json",
        success: callback
    });
}
```

```andCallFake()```，```toHaveBeenCalled()```是匹配方法，所有Spy可用的匹配方法可以看Jasmine [wiki](https://github.com/pivotal/jasmine/wiki/Spies)。

第二种测试(异步测试)，我们可以用Jasmine支持的下面这三个方法对前面的例子做些改进：

* runs(function) - 立即运行一个代码块
* waits(timeout) - 下一个代码块执行前等待一段时间
* waitsFor(function, optional message, optional timeout)——暂停specs知道某些工作完成。Jasmine会等待这里提供的函数返回true然后再继续执行下一块代码。


```javascript
it("should make an actual AJAX request to a server", function () {

    var callback = jasmine.createSpy();
    getTodo(16, callback);

    waitsFor(function() {
        return callback.callCount > 0;
    });

    runs(function() {
        expect(callback).toHaveBeenCalled();
    });
});

function getTodo(id, callback) {
    $.ajax({
        type: "GET",
        url: "todos.json",
        dataType: "json",
        success: callback
    });
}
```

***提示：*** 当在单元测试中创建真实的服务器端请求时，会极大的拖慢测试运行的速度(有很多因素，包括服务器延迟)。同时也引入了外部依赖，原本可以(也应该要)最小化你的单元测试，所以强烈推荐你选择spies，避免使用真实的服务器端调用。

## beforeEach() and afterEach()

Jasmine同样支持在每个测试之前(```beforeEach()```)或者之后(```afterEach```)执行特定的代码。这对强制为一致的条件非常有用(比如重置specs引入的变量)。下面这个例子中，```beforeEach()```中创建一个specs用于测试属性的Todo model。

```javascript
beforeEach(function(){
   this.todo = new Backbone.Model({
      text: 'Buy some more groceries',
      done: false
   });
});

it('should contain a text value if not the default value', function(){
   expect(this.todo.get('text')).toEqual('Buy some more groceries');
});
```

每个```describe()```中都可嵌套自己的```beforeEach()```和```afterEach()```方法，支持对应suite相关的setup和teardown方法。




`beforeEach()` and `afterEach()` can be used together to write tests verifying that our Backbone routes are being correctly triggered when we navigate to the URL. We can start with the `index` action:

```javascript
describe('Todo routes', function(){

   beforeEach(function(){

        // Create a new router
        this.router = new App.TodoRouter();

        // Create a new spy
        this.routerSpy = jasmine.spy();

        // Begin monitoring hashchange events
        try{
            Backbone.history.start({
                silent:true,
                pushState: true
            });
        }catch(e){
           // ...
        }

        // Navigate to a URL
        this.router.navigate('/js/spec/SpecRunner.html');
   }); 

   afterEach(function(){

        // Navigate back to the URL
        this.router.navigate('/js/spec/SpecRunner.html');

        // Disable Backbone.history temporarily.
        // Note that this is not really useful in real apps but is
        // good for testing routers
        Backbone.history.stop();
   });

   it('should call the index route correctly', function(){
        this.router.bind('route:index', this.routerSpy, this);
        this.router.navigate('', {trigger: true});

        // If everything in our beforeEach() and afterEach()
        // calls have been correctly executed, the following
        // should now pass.
        expect(this.routerSpy).toHaveBeenCalledOnce();
        expect(this.routerSpy).toHaveBeenCalledWith();
   });

});
```

The actual TodoRouter for that would make the above test pass looks like:

```javascript
var App = App || {};
App.TodoRouter = Backbone.Router.extend({
    routes:{
        '': 'index'
    },
    index: function(){
        //...
    }
});
```

## Shared scope

Let's imagine we have a Suite where we wish to check for the existence of a new Todo item instance. This could be done by duplicating the spec as follows:

```javascript
describe("Todo tests", function(){
   
   // Spec
   it("Should be defined when we create it", function(){
        // A Todo item we are testing
        var todo = new Todo("Get the milk", "Tuesday");
        expect(todo).toBeDefined();
   }); 

   it("Should have the correct title", function(){
        // Where we introduce code duplication
        var todo = new Todo("Get the milk", "Tuesday");
        expect(todo.title).toBe("Get the milk");
   });

});
```

As you can see, we've introduced duplication that should ideally be refactored into something cleaner. We can do this using Jasmine's Suite (Shared) Functional Scope.

All of the specs within the same Suite share the same functional scope, meaning that variables declared within the Suite itself are available to all of the Specs in that suite. This gives us a way to work around our duplication problem by moving the creation of our Todo objects into the common functional scope:

```javascript
describe("Todo tests", function(){
    
    // The instance of Todo, the object we wish to test
    // is now in the shared functional scope
    var todo = new Todo("Get the milk", "Tuesday");

    // Spec
    it("should be correctly defined", function(){
        expect(todo).toBeDefined();
    });

    it("should have the correct title", function(){
        expect(todo.title).toBe("Get the milk");
    });

});
```

前面你可能注意到```beforeEach()```调用中我们定义了一个变量```this.todo```，然后在```afterEach()```也可以继续使用它。这要归功于Jasmine的共享函数域。共享域可以让所有块(包括```runs()```)访问的```this```的属性都是相同的，除了声明的变量之外(```var```声明的变量)。

##获取安装

现在我们来分析下基本原理，先下载Jasmine并且做好编写测试前的准备。

官方独立版本可以从这里[下载](https://github.com/pivotal/jasmine/downloads)。

下载到的包中还有一个SpecRunner.html文件。 Jasmine代码仓库可以从用git从这里获取https://github.com/pivotal/jasmine.git。

我们来看下SpecRunner.html文件(下面示例代码可能相对于新版本的Jasmine较老了)：

首先引入Jasmine和必要用于report的css：


	<link rel="stylesheet" type="text/css" href="lib/jasmine-1.1.0.rc1/jasmine.css"/>
	<script type="text/javascript" src="lib/jasmine-1.1.0.rc1/jasmine.js"></script>
	<script type="text/javascript" src="lib/jasmine-1.1.0.rc1/jasmine-html.js"></script>


然后，引入一些测试：


	<script type="text/javascript" src="spec/SpecHelper.js"></script>
	<script type="text/javascript" src="spec/PlayerSpec.js"></script>


最后是需要被测试的代码：


	<script type="text/javascript" src="src/Player.js"></script>
	<script type="text/javascript" src="src/Song.js"></script>


***提示：*** SpecRunner后面的代码是负责运行测试。 这里不做分析，鼓励你看看PlayerSpec.js和SpecHelper.js的代码。这是个如何做一组测试的很好的例子。

##TDD With Backbone

当使用Backbone开发应用时，需要测试个别的模块代码同时也要测试models, views, collections和routers。采用TDD测试方法，我们来看下测试Backbone [Todo](https://github.com/addyosmani/todomvc/tree/gh-pages/architecture-examples/backbone)这个示例项目中Backbone组件的一些specs。这一节我们将使用一个由Larry Myers修改版本的Backbone Koans项目， 在`practicals\jasmine-koans`目录下。

## Models

Backbone models的复杂程度完全依赖于应用要实现的功能。下面例子中，我们将测试默认值，属性，状态改变，和验证规则。

首先，使用```describe()```创建suite：

```javascript
describe('Tests for Todo', function() {
```

Model的属性理想上应该有默认值。可以确保创建实例时未指定值的话可以使用默认值替代。这里的意思主要是与models交互是可以避免一些意外的行为。

下面这个spec，创建一个Todo没有传入任何属性，然后检查下```text```属性的值是什么。因为没有设置任何值，所以我们期望的值是返回```""```。

```javascript
it('Can be created with default values for its attributes.', function() {
    var todo = new Todo();
    expect(todo.get('text')).toBe("");
});
```

如果你在编写model前测试这个spec的话，会引发失败。这个spec需要传入一个```text```属性的默认值。示例：

```javascript

window.Todo = Backbone.Model.extend({

    defaults: {
      text: "",
      done:  false,
      order: 0
    }

```

接下来，我们测试下model在初始化之后会把属性值设为传入的值。另外也测试下其它几个属性的默认值是否是我们期望的。

```javascript
it('Will set passed attributes on the model instance when created.', function() {
    var todo = new Todo({ text: 'Get oil change for car.' });

    // what are the values expected here for each of the
    // attributes in our Todo?

    expect(todo.get('text')).toBe("Get oil change for car.");
    expect(todo.get('done')).toBe(false);
    expect(todo.get('order')).toBe(0);
});
```
Backbone models支持model.change()事件，当model的状态改变时触发。下面这个例子中，通过设置Todo model属性值来改变它的'state(状态)'，状态改变的原因非常值得测试，因为应用中可能有状态依赖的事件，比如当model被修改时想要显示一个确认视图。

```javascript
it('Fires a custom event when the state changes.', function() {

    var spy = jasmine.createSpy('-change event callback-');

    var todo = new Todo();

    // how do we monitor changes of state?
    todo.on('change', spy);

    // what would you need to do to force a change of state?
    todo.set({ text: 'Get oil change for car.' });

    expect(spy).toHaveBeenCalled();
});
```

通常在model中引入验证逻辑来确保来自用户的输入(或者其它模块)是有效的'(valid)'。Todo app可能会验证text输入框输入进来的内容没有粗鲁的单词。 同样的，
当保存Todo项```done```状态时，需要验证传入的值是true/false， 而不是字符串。

在下面的spec中，我们用了一些是验证失败的值，让model.validate()触发"error"事件。检验一下传入无效值是是否会真的触发失败。

使用Jasmine内置```createSpy()```方法创建一个errorCallback spy，就可以检测error事件了：

```javascript
it('Can contain custom validation rules, and will trigger an error event on failed validation.', function() {

    var errorCallback = jasmine.createSpy('-error event callback-');

    var todo = new Todo();

    todo.on('error', errorCallback);

    // What would you need to set on the todo properties to
    // cause validation to fail?

    todo.set({done:'a non-integer value'});

    var errorArgs = errorCallback.mostRecentCall.args;

    expect(errorArgs).toBeDefined();
    expect(errorArgs[0]).toBe(todo);
    expect(errorArgs[1]).toBe('Todo.done must be a boolean value.');
});

```
要让上面这段失败测试代码支持验证非常简单。这个model中，我们重写validate()方法(像Backbone文档推荐的那样)，检查model有'done'属性并且给它传入值时是一个合法的布尔值。

```javascript
validate: function(attrs) {
    if (attrs.hasOwnProperty('done') && !_.isBoolean(attrs.done)) {
        return 'Todo.done must be a boolean value.';
    }
}
```

这个完整的Todo model代码如下：

```javascript
var NAUGHTY_WORDS = /crap|poop|hell|frogs/gi;

function sanitize(str) {
    return str.replace(NAUGHTY_WORDS, 'rainbows');
}

window.Todo = Backbone.Model.extend({

    defaults: {
      text: '',
      done:  false,
      order: 0
    },

    initialize: function() {
        this.set({text: sanitize(this.get('text'))}, {silent: true});
    },

    validate: function(attrs) {
        if (attrs.hasOwnProperty('done') && !_.isBoolean(attrs.done)) {
            return 'Todo.done must be a boolean value.';
        }
    },

    toggle: function() {
        this.save({done: !this.get("done")});
    }

});
```


## Collections

现在我们需要定义specs来测试Backbone Todo model的collection(一个TodoList)。Collections处理列表的排序，过滤等。

测试collections时可能会有下面这些明确的specs：

* 可以添加新的Todo model对象或者对象的数组。
* 属性测试，确保类似collection的base URL是我们期望的值。
* 有意的添加状态```done:true```的todo项， 然后检查collection认为已完成项的数量和未完成项的数量。

这一节我们只会讲到前2个问题，第三个问题作为读者的扩着练习。

测试Todo models可通过一个对象或者数组来添加相对简单。首先，初始化一个TodoList collection，确保其长度为0。然后，添加新的Todos，对象和数组两种情况都添加一次，然后检查collection的length属性是否符合期望值：

```javascript
describe('Tests for TodoList', function() {

    it('Can add Model instances as objects and arrays.', function() {
        var todos = new TodoList();

        expect(todos.length).toBe(0);

        todos.add({ text: 'Clean the kitchen' });

        // how many todos have been added so far?
        expect(todos.length).toBe(1);

        todos.add([
            { text: 'Do the laundry', done: true },
            { text: 'Go to the gym'}
        ]);

        // how many are there in total now?
        expect(todos.length).toBe(3);
    });
...
```

跟model的属性一样，测试collections的属性也非常简单。下面是一个简单的测试collection.url的spec例子：

```javascript
it('Can have a url property to define the basic url structure for all contained models.', function() {
        var todos = new TodoList();

        // what has been specified as the url base in our model?
        expect(todos.url).toBe('/todos/');
});

```

对于第三个spec，collection会实现``done()```和```remaining()```方法，分别过滤已完成Todo项和未完成项。编写一个spec，创建一个collection，添加一个```done```状态为为```true```的model，2个```done```状态为```false```的model。然后测试调用```done()```和```remaining()```方法返回的结果的length，看是否正常。

TodoList collection实现代码可以像下面这样：


```javascript
 window.TodoList = Backbone.Collection.extend({

        model: Todo,

        url: '/todos/',

        done: function() {
            return this.filter(function(todo) { return todo.get('done'); });
        },

        remaining: function() {
            return this.without.apply(this, this.done());
        },

        nextOrder: function() {
            if (!this.length) {
                return 1;
            }

            return this.last().get('order') + 1;
        },

        comparator: function(todo) {
            return todo.get('order');
        }

    });
```


## Views

在开始测试Backbone views前，先简短的来看一个编写Jasmine specs的jQuery plugin。

**The Jasmine jQuery Plugin**

Todo application使用jQuery来做DOM操作，有一个[jasmine-jquery](https://github.com/velesin/jasmine-jquery) 插件可以帮助简化BDD测试view创建的元素。


这个插件提供了很多额外的Jasmine [matchers](https://github.com/pivotal/jasmine/wiki/Matchers) 以帮助测试jQuery包装的sets：

* ```toBe(jQuerySelector)```示例 ： ```expect($('<div id="some-id"></div>')).toBe('div#some-id')```
* ```toBeChecked()``` 示例：```expect($('<input type="checkbox" checked="checked"/>')).toBeChecked()```
* ```toBeSelected()``` 示例 ```expect($('<option selected="selected"></option>')).toBeSelected()```

更多可以参看 [这里](https://github.com/velesin/jasmine-jquery)。它支持的完整matchers可以在项目主页上找到。它跟标准的Jasmine matchers类似，上面的matchers可以加.not前缀反过来使用(比如```expect(x).not.toBe(y)```)：

```javascript
expect($('<div>I am an example</div>')).not.toHaveText(/other/)
```

jasmine-jquery同时包含一个固定装置模型(fixtures model)，可以加载任意HTML内容到时候。可以像下面这样使用：

在一个外部文件中包含一段HTML：

some.fixture.html:
```<div id="sample-fixture">some HTML content</div>```

然后，实际测试时想下面这样载入：

```javascript
loadFixtures('some.fixture.html')
$('some-fixture').myTestedPlugin();
expect($('#some-fixture')).to<the rest of your matcher would go here>
```

jasmine-jquery插件默认会从一个特定目录加载fixtures：spec/javascripts/fixtures。如果想配置这个路径的话在初始化设置中```jasmine.getFixtures().fixturesPath = 'your custom path'```。

最后，jasmine-jquery包含对jQuery事件spying的支持，而且不需要什么额外的工作。可使用```spyOnEvent()```和```assert(eventName).toHaveBeenTriggered(selector)```方法来完成。下面是一个示例：

```javascript
spyOnEvent($('#el'), 'click');
$('#el').click();
expect('click').toHaveBeenTriggeredOn($('#el'));
```


### View测试

这一小节我们从三个维度来看下编写Backbone Views的specs：初始化，view渲染和模板生成。后两个跟通常的测试差不多，不过我会简短的说下为什么对views的初始化编写specs也是有好处的。

#### 初始化

最基本的，为Backbone views写的specs需要验证view被正确的绑定到指定的DOM元素，被有效的数据model支持。考虑这样做的原因是，如果这些specs失败的话会导致后面一些更复杂的测试也失败，而且这些specs写起来比较简单，可以提供一个整体的价值。

为确保一致的测试配置条件，使用```beforeEach()```追加一个空的```UL``` (#todoList)到DOM并且用一个空的Todo model初始化一个TodoView实例。在```afterEach()```中移除前面的#todoList  ```UL```和view实例。

```javascript
describe('Tests for TodoView', function() {

    beforeEach(function() {
        $('body').append('<ul id="todoList"></ul>');
        this.todoView = new TodoView({ model: new Todo() });
    });


    afterEach(function() {
        this.todoView.remove();
        $('#todoList').remove();
    });

...
```

第一个spec就是检查我们创建的TodoView使用了正确的```tagName```(元素或者className)。目的就是确保创建时正确的绑定到DOM元素。

Backbone views通常一旦初始化会创建一些空的DOM元素，不过这些元素不会附加到可见的DOM中， 目的是在不影响性能和渲染的情况下让他们构建出来。

```javascript
it('Should be tied to a DOM element when created, based off the property provided.', function() {
    //what html element tag name represents this view?
    expect(todoView.el.tagName.toLowerCase()).toBe('li');
});
```

如果TodoView还没编写好的话，specs就会失败。通过指定```tagName```创建一个Backbone.View。

```javascript
var todoView = Backbone.View.extend({
    tagName:  "li"
});
```

也可以通过测试className来替代```tagName```，可以使用更高级的jasmine-jquery matcher ```toHaveClass()```来完成。

```
it('Should have a class of "todos"'), function(){
   expect(this.view.$el).toHaveClass('todos');
});
```

```toHaveClass()``` matcher对jQuery操作，而且如果没有使用这个插件的话就会引发异常(如果没有使用jasmine-jquery插件也可通过获取el.className来判断)。

你可能注意到```beforeEach()```中我们传入里一个新创建的Todo给view。Views应该基于一个有数据的model实例。因为它对view的功能非常重要，我们可以写一个spec来确保model是一定义(使用```toBeDefined()``` matcher) 并且测试model有默认属性，而且是我们期望的值。

```javascript
it('Is backed by a model instance, which provides the data.', function() {

    expect(todoView.model).toBeDefined();

    // what's the value for Todo.get('done') here?
    expect(todoView.model.get('done')).toBe(false); //or toBeFalsy()
});
```


#### View渲染


接下来看下给view渲染编写specs。特别是，我们想测试下TodoView实际被渲染出来的元素是否符合期望。

对于小的额applications，有些BDD新手认为视觉上确认view的渲染可以替代view的单元测试。实际上，开发的应用可能变成多视个view时，通常从开端就劲量让这个过程自动化。同样也有aspects来验证屏幕上所看到的渲染效果。

我们编写两个spec来测试view。第一个测试检车view的```render()```方法正确的返回view实例，可以用于链式调用。第二个测试检查生成的HTML是基于TodoView相关联的mode实例的属性所期望的结果。

不同于前面我们写的specs，这一节我们会大量使用```beforeEach()```彰显如何使用嵌套的suites，以及确保specs的条件一致。第一个TodoView的spec，将创建一个简单的model (基于Todo)，然后用这个model初始化一个TodoView。

```javascript
describe("TodoView", function() {

  beforeEach(function() {
    this.model = new Backbone.Model({
      text: "My Todo",
      order: 1,
      done: false
    });
    this.view = new TodoView({model:this.model});
  });

  describe("Rendering", function() {

    it("returns the view object", function() {
      expect(this.view.render()).toEqual(this.view);
    });

    it("produces the correct HTML", function() {
      this.view.render();

      //let's use jasmine-jquery's toContain() to avoid
      //testing for the complete content of a todo's markup
      expect(this.view.el.innerHTML)
        .toContain('<label class="todo-content">My Todo</label>');
    });

  });

});
```

这些specs一旦运行，只有第二个('produces the correct HTML')失败。第一个spec ('returns the view object')，测试```render()```返回的TodoView实例， 可以通过因为这是Backbone的默认行为。我们并没有重写```render()```方法。

**提示：** 为了维护可读性，这一节中所有的模板例子都将使用下面这个最小化版本的Todo view模板。需要使用时可以返回来查看：

	<div class="todo <%= done ? 'done' : '' %>">
	        <div class="display">
	          <input class="check" type="checkbox" <%= done ? 'checked="checked"' : '' %> />
	          <label class="todo-content"><%= text %></label>
	          <span class="todo-destroy"></span>
	        </div>
	        <div class="edit">
	          <input class="todo-input" type="text" value="<%= content %>" />
	        </div>
	</div>



第二个spec失败会有下面这段提示：

```Expected '' to contain '<label class="todo-content">My Todo</label>'.```

原因是render()的默认行为不创建任何标签。我们来编写一个替代的render()来解决它：

```javascript
render: function() {
  var template = '<label class="todo-content"><%= text %></label>';
  var output = template
    .replace("<%= text %>", this.model.get('text'));
  this.$el.html(output);
  return this;
}
```

上面指定了一行模板字符串，然后用model对应的属性值替换"<% %>"区域内的内容。同时也返回TodoView实例，所以第一个spec也可以通过。像这样在spec中使用HTML字符串来进行测试有非常大的缺点。即便是模板微小的变化(一个tab或者空格符)就会导致spec失败，即便渲染结果是一样的。实际应用中模板也会更复杂，将耗费更多的时间去维护。更好的测试渲染输出结果的方法是使用jQuery来选择和检查。

基于这个思想，我们来重写这个spec，使用jasmine-jquery提供的自定义matchers：


```javascript
describe("Template", function() {

  beforeEach(function() {
    this.view.render();
  });

  it("has the correct text content", function() {
    expect(this.view.$('.todo-content'))
      .toHaveText('My Todo');
  });

});
```

讨论单元测试不提到fixtures是不可能的。Fixtures通常包含单元测试当需要时(可以是本地或者从外部文件)载入的测试数据(比如HTML)。 一直以来我们都是把jQuery的期望建立在view的el属性上。大部分情况这是有效的，不过，有时我们需要把标签渲染到document。再specs中处理这个问题的最理想方式就是使用fixtures (jasmine-jquery插件带给我们的另外一个特性)。

使用fixtures重写上面这个spec：


```javascript
describe("TodoView", function() {

  beforeEach(function() {
    ...
    setFixtures('<ul class="todos"></ul>');
  });

  ...

  describe("Template", function() {

    beforeEach(function() {
      $('.todos').append(this.view.render().el);
    });

    it("has the correct text content", function() {
      expect($('.todos').find('.todo-content'))
        .toHaveText('My Todo');
    });

  });

});
```

在上面这个spec中，把渲染的todo元素append到fixture。然后对这个fixture设置期望，当view对应到一个已经存在的DOM元素之后的一些desirable。有必要提供fixture和测试当view初始化的时候```el```属性是否指向正确的元素。


#### 使用模板系统进行渲染


当一个用户设置一个Tood项为完成(done)是，我们期望给他一点视觉上的反馈(比如文本上加一条横线)以区分其余的的项。可以通过给这个项附加一个class来实现。下面开始编写测试：


```javascript
describe('When a todo is done', function() {

  beforeEach(function() {
    this.model.set({done: true}, {silent: true});
    $('.todos').append(this.view.render().el);
  });

  it('has a done class', function() {
    expect($('.todos .todo-content:first-child'))
      .toHaveClass('done');
  });

});
```

这样会失败，并引发下面消息：

```Expected '<label class="todo-content">My Todo</label>'
to have class 'done'.
```


可以在render()方法中解决：


```javascript
render: function() {
  var template = '<label class="todo-content">' +
    '<%= text %></label>';
  var output = template
    .replace('<%= text %>', this.model.get('text'));
  this.$el.html(output);
  if (this.model.get('done')) {
    this.$('.todo-content').addClass('done');
  }
  return this;
}
```

不过，很快它就会变得不那么方便了。随着模板中逻辑的增加，它就会变得越复杂。我们可以通过使用模板库轻易的解决这个问题。有许多模板库也能很好的与测试方案一起配合的很好，比如Jasmine。

JavaScript模板系统(比如Handlebars, Mustache 以及Underscore的Micro-templating)在模板字符串中支持条件逻辑。这就意味着我们可以一行字符串内使用if/else/三元表单时，可以构建更强大的模板。

在我们的案例中，这里我们选择Underscore内置的Microtemplating，不需要添加额外额文件，而且对现有specs也不需要做太多的修改。

假设模板定义在ID `myTemplate`的script标签中:

```
<script type="text/template" id="myTemplate">
    <div class="todo <%= done ? 'done' : '' %>">
            <div class="display">
              <input class="check" type="checkbox" <%= done ? 'checked="checked"' : '' %> />
              <label class="todo-content"><%= text %></label>
              <span class="todo-destroy"></span>
            </div>
            <div class="edit">
              <input class="todo-input" type="text" value="<%= content %>" />
            </div>
    </div>
</script>
```

TodoView可以使用Underscore模板修改成下面这样:

```javascript
var TodoView = Backbone.View.extend({

  tagName: 'li',
  template: _.template($('#myTemplate').html()),

  initialize: function(options) {
    // ...
  },

  render: function() {
    this.$el.html(this.template(this.model.toJSON()));
    return this;
  },

  ...

});
```

So, what's going on here? We're first defining our template in a script tag with a custom script type (e.g., type="text/template"). As this isn't a script type any browser understands, it's simply ignored, however referencing the script by an id attribute allows the template to be kept separate to other parts of the page.

In our view, we're the using the Underscore `_.template()` method to compile our template into a function that we can easily pass model data to later on. In the line `this.model.toJSON()` we are simply returning a copy of the model's attributes for JSON stringification to the `template` method, creating a block of HTML that can now be appended to the DOM.

Note: Ideally all of your template logic should exist outside of your specs, either in individual template files or embedded using script tags within your SpecRunner. This is generally more maintainable.

If you are working with much smaller templates and are not doing this, there is however a useful trick that can be applied to automatically create or extend templates in the Jasmine shared functional scope for each test. 

By creating a new directory (say, 'templates') in the 'spec' folder and including a new script file with the following contents into SpecRunner.html, we can manually add custom attributes representing smaller templates we wish to use:

```javascript
beforeEach(function() {
  this.templates = _.extend(this.templates || {}, {
    todo: '<label class="todo-content">' +
            '<%= text %>' +
          '</label>'
  });
});
```

To finish this off, we simply update our existing spec to reference the template when instantiating the TodoView:

```javascript
describe('TodoView', function() {

  beforeEach(function() {
    ...
    this.view = new TodoView({
      model: this.model,
      template: this.templates.todo
    });
  });

  ...

});
```

The existing specs we've looked at would continue to pass using this approach, leaving us free to adjust the template with some additional conditional logic for Todos with a status of 'done':

```javascript
beforeEach(function() {
  this.templates = _.extend(this.templates || {}, {
    todo: '<label class="todo-content <%= done ? 'done' : '' %>"' +
            '<%= text %>' +
          '</label>'
  });
});
```

This will now also pass without any issues, however as mentioned, this last approach probably only makes sense if you're working with smaller, highly dynamic templates. 


## 总结

现在我们已经讨论如何为Backbone.js应用中的models, views和collections编写Jasmine测试。虽然测试路由(routing)有时是可取的，有些开发者认为它可以由第三方工具更好的完成比如Selenium，所以请记住这点。

## 练习

A作为练习，推荐大家尝试下Jasmine Koans，在`practicals\jasmine-joans`目录下，然后尝试fix一些里面有意提供的失败的测试。这是一种很好的了解Jasmine specs 和 suites工作原理和学习Backbone技巧的方式。


## 扩展阅读
* [Testing Backbone Apps With SinonJS](http://tinnedfruit.com/2011/04/26/testing-backbone-apps-with-jasmine-sinon-3.html) by James Newbry
* [Jasmine + Backbone Revisited](http://japhr.blogspot.com/2011/11/jasmine-backbonejs-revisited.html)
* [Backbone, PhantomJS and Jasmine](http://japhr.blogspot.com/2011/12/phantomjs-and-backbonejs-and-requirejs.html)
