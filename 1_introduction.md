# What is Vue?

Vue is a library, the core library is focused on the View layer of the classic MVC model, but along with [supporting libraries](https://github.com/vuejs/awesome-vue#libraries--plugins), one can easily build a single page app in Vue. Vue provides tools for simplifying the development. We will use some of them as we go ahead in this book.

Coming back to the example we used in the installation chapter.

## Loading Vue
`<script src="vue.js"> </script>` 

The simplest way to load Vue is to use the script tag.  It creates the global variable `Vue` which we will be using in the `<script>` tag below the body. 

file: `0basic_example.html`

	<html>
	<head>
	   <script src="vue.js"> </script>
	</head>
	<body>
	   <div id="app">
	      {{ message }}
	   </div>
		<script>
			var app = new Vue({
				el: '#app',
				data: {
					message : 'good morning'
				}
			})
		</script>
	</body>
	</html>

This is a complete Vue app. It looks simple enough, we defined the value to `message` and it got displayed in the <div> tag. But, Vue has done a lot of work under the hood. Everything on the page is reactive, it means that whenever the value of the `message` in `data` changes, it'll be automatically updated in our html.

To verify our claim, open the web inspecter in your browser, on Firefox do a right click on the page and click "inspect element". It shall open the web inspector. Press the escape key and the console would pop up. Set the message to any value you'd like like this, `app.message='oh my Dod, it did change.'`

This proves that the value of the message variable is dynamically bound with what is being displayed in our html.

## Delimiters

Vue uses a templating syntax to render data in our html page. The default syntax is `['{{', '}}']`, but we can change it to whatever syntax we want. The way to do it is 

	var app = new Vue({
		el: '#app',
		data: {
			message : 'good morning'
		},
		delimiters: ['${', '}']
	})


## Obligatory theory

We first created a Vue instance. The syntax is

	var app = new Vue({
		// options
	})

We pass the options object while intializing Vue's root instance for our app. It contains the 

1. `el`: element which the app is going to be bound
1. `data`: the data of our app. Every variable which we are going to use in the template needs to be defined here.
1. `lifecycle callbacks`: There are many of these, we will get to them later. Creating an object in Vue is a process, there are lifecycle callbacks involved like mounted (when the componenet is ready and inserted in the DOM).

Each Vue instance proxies all the properties found in its `data` object.
	
	var data = { a: 1 }
	var vm = new Vue({
	  data: data
	})
	
	vm.a === data.a // -> true
	
	// setting the property also affects original data
	vm.a = 2
	data.a // -> 2
	
	// ... and vice-versa
	data.a = 3
	vm.a // -> 3

It should be noted that only these proxied properties are reactive. If you attach a new property to the instance after it has been created, it will not trigger any view updates. We will discuss the reactivity system in detail later.

In addition to data properties, Vue instances expose a number of useful instance properties and methods. These properties and methods are prefixed with $ to differentiate them from proxied data properties. For example:

	var data = { a: 1 }
	var vm = new Vue({
	  el: '#example',
	  data: data
	})
	
	vm.$data === data // -> true
	vm.$el === document.getElementById('example') // -> true
	// $watch is an instance method
	vm.$watch('a', function (newVal, oldVal) {
	  // this callback will be called when `vm.a` changes
	})
	

## Expressions

So far we’ve only been binding to simple property keys in our templates. But Vue.js actually supports the full power of JavaScript expressions inside all data bindings:

	{{ number + 1 }}

	{{ ok ? 'YES' : 'NO' }}

	{{ message.split('').reverse().join('') }}

	<div v-bind:id="'list-' + id"></div>

These expressions will be evaluated as JavaScript in the data scope of the owner Vue instance. One restriction is that each binding can only contain one single expression, so the following will NOT work:

	<!-- this is a statement, not an expression: -->

	{{ var a = 1 }}

	<!-- flow control won't work either, use ternary expressions -->

	{{ if (ok) { return message } }}

> Template expressions are sandboxed and only have access to a whitelist of globals such as Math and Date. You should not attempt to access user defined globals in template expressions.

## A todo list manager.

Our requirement is simple, we need to add/update/delete items in our todo list.

We start small.

file: `1basic-todo.html`

	<body>
	  <div id="app4">
	      <ol>
	            <li>{{message}}</li>
	      </ol>
	  </div>
	  
	  <script>
	    var app4 = new Vue({
	        el:"#app4",
	        data: {
	            todo: "do something!"
	        }
	    })
	  </script>
	</body>

> In the interest of saving space, we will be skipping the <head> portion. It is the same and it needs to be present for your app to work.


> When we have an example, please create a new html file and save it according to whatever convention you want and open it in a browser to verify what you have learned.

When we open this in a browser, you will see "do something" being displayed thrice. We do not want the same message to be displayed! We want different messages and eventually we want to add our own todos.

Earlier we had one variable `todo`, but since we want to display multiple messages, we need to define an array. Update the data field.

> For variables to be reactive, we need to define them in the data field. For components data is a function and not a variable. We will get to that later.

file: `2basic-todo-2.html`

    data: {
        todos: [
          { text: "do something!"},
          { text: "now do something!"},
          { text: "now do something else!"}
        ]
	}

Doing this bounds the `todos` array to the current `Vue` instance named `app4`.

We now need to change the HTML page as well,

Instead of `<li>{{message}}</li>`, we will have to now display an array. Vue provides us with a `for` construct.

	<li v-for="(todo, index)">{{ todo.text }}</li>

Anything that starts with `v-` is a directive. It is evaluated by Vue. If something is wrong, it will complain in the JS console which we checked few lines back. **Always open the JS console while writing a vue app**

While evaluating the loop, `event` will contain `{text:}` and index would be the position of the `todo` in the array, it starts with 0.

## Getting User Input

We now want to be able to add todo items in our list.

file: `3basic-todo-3.html`

Make the following changes to the HTML in the `<ol>` tag.

	<input placeholder="add title"  v-model="todo.title" />
	<input placeholder="add text"  v-model="todo.text" />
	<button v-on:click="AddTodo(todo)">Add todo</button>

	<ol>
	      <li v-for="(todo, index) in todos">
	          {{ todo.title }} : {{ todo.text }}: {{ todo.assign }}
	      </li>
	</ol>

Make the following changes to the Vue instance, the other part stays the same.

	data: {
	    todo: {title: '', text:''},
	    todos: [
	        {title: 'Lean JS' , text: "What is JS?"},
	        {title:'Learn vue',  text: "Vue has nice docs!"},
	        {title:'Build something',  text: "what to build?"}
	    ]
	},
	methods: {
	  AddTodo: function (item) {
	    alert(item.text + " " + item.title)
	  }
	}

For adding a new item to our todo list, we need to do the following things:

1. Get the input from the <input /> tag
1. Add the input to our `todos`
1. Render them in our ol tag

## v-on
This directive handles events. When we do a v-on:click="AddMethod", it'll run AddMethod when the element is clicked, which is exactly what is happening in our case.

For now, let's just print the user input which we get. Later, it is just one line of code which adds the item to the array.

We need to add a v-model to our input tag, it provides two way data binding. In simple words, when we do this `v-model="todo.title"`, it'll change the `todo` object which we created, also, if we change the todo object, it'll change the value of the input. This is the meaning of the two way data binding. Try it in the devtools to be sure about this.

We need to define a `todo` object to bind it to the user input `todo: {title: '', text:''}`.

At this point of time, we just need to add the todo when the user does some action. This is done by adding the directive `v-on:click`, it'll trigger `AddTodo(todo)` when the user will click the button. The AddTodo is a method which we define in our Vue instance. It is bound to our `app4` instance.

Now, we need to add the todo item to our html. You would be right if you are thinking to change the `alert` line in the AddTodo method to an array append of todos.

	this.todos.push(this.todo)

> It is important to use `this.` to refer to `todos` because scoping is a big issue here and Vue won't be able to find out what todo you are referring to.

We did not touch the HTML because Vue is going to update the HTML based on what we have in the `todos` array. This is the beauty of front end frameworks and the beauty of Vue is that you can get started so easily in it.

## Add feature to assign task

`file: 4basic-todo-4.html`

We now have a new requirement, we need to add the feature to assign tasks to someone.

Since Vue is going to handle everything in the front end, we first need to add `assign` as a part of the `todo` object.

`todo: {title: '', text:'', assign:''},`

Add a new `<input >` with the v-model as `todo.assign`. it'll bind the input field and the todo object's assign field's values.

make the changes to the template and add a new `{{ todo.assign }}`. 

Save and open the page in a browser. There is one catch, not all tasks have an assign field. There is a way to handle this in Vue. It is by using the `v-if` directive. 

## v-if 
v-if directive will show the HTML tag only if there is a value in the object assigned to it. 

## The template tag

In our example, there is another catch, we are rendering the content just by doing `{{}}`. This begs the question as to where to place the `v-if`. This is where the `<template>` tag comes in handy. It is a blank tag which does nothing provided by Vue in this condition.

Make the following change to the HTML

    {{ todo.title }} : {{ todo.text }} 
    <template v-if="todo.assign"> {{ :todo.assign }}</template>

This will render todo.assign only if the task has the value in the object. Save and open the page in a browser and add a new task with keeping the assign field blank and one with assign field as any value. You'll notice that only in the second case does the value display on the html page.

## Polishing

If you notice, once we add the task to our todo list, the input fields still have the same value. We do not want that. We want the input fields to be blank as soon as we add the task to the html.

Add the following line after the `push` of the AddTodo.

	this.todo = {title:'', text:'', assign:''}

This will reset the input elements.

## Using Components

Web components are an amazing concept. They allow us to create custom HTML tags. Now, we can have a `<todo>` tag for our app rather than having to write all the `<li>` stuff directly in our `index.html`, the advantage it offers is that our HTML pages look clean, plus, we can reuse the components in other projects.

When we build a custom element, we can hide a lot of code in our index.html.

Let's name our custom element `<todo>`

We will include this in our index.html as 

	<ol>
	   <todo></todo>
	</ol>

A component should be defined before the Vue instance is created

	Vue.component('todo', {
	  template: '<li> hello world </li>'
	})

You shall see hello world repeated twice. This is because Vue will replace `<todo>` with the `template` field. Components support a lot of options some of which we will see later.

We got started with components, we need to now render the actual tasks.

We will _not_ define the data in the component because we want the UI and the data to be decoupled, hence we will pass the data to the component from the html page and the component would just render the data which is passed to it. Data is passed to a component using the `props` field.

	Vue.component('todo', {
		props:['item'],
		template: '<li>{{item.title}} \
		{{ item.text}} \
		<template v-if="item.assign">\
		{{item.assign}}</template></li>'
	});
	
    <todo v-for="todo in todos" v-bind:item="todo"></todo>

We want to show each element in the array `todos` and want to bind the current element's `item` prop to current 'todo' in the loop.

> Usually Vue templates are long, hence we have to use the '\' delimiter to have multi line templates. 

## v-bind
In HTML elements, we can't use mustaches `{{ }}`, so we use v-bind like this

    <div v-bind:id="dynamicId"></div>

It also works for boolean attributes - the attribute will be removed if the condition evaluates to a falsy value:

	<button v-bind:disabled="someDynamicCondition">Button</button>


### Shorthands

v-bind Shorthand
	
	<!-- full syntax -->
	<a v-bind:href="url"></a>

	<!-- shorthand -->
	<a :href="url"></a>

v-on Shorthand
	
	<!-- full syntax -->
	<a v-on:click="doSomething"></a>

	<!-- shorthand -->
	<a @click="doSomething"></a>
