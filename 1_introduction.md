# Interactive web applications
The Internet has come a long way since the first website http://cern.ch was created. Earlier, it used to be a platform for sharing research between scientists, which was later adapted for building one of the most tranformative means of communication and a platform which opened the world up to a large range of possibilities.

With the large scale possibilities, the user expectations have sky rocketed, back in the early 2000s even Google's webpages were having barely any interactivity, when they started using auto-complete it was a _big_ deal. But now, users just expect auto complete and a host of other features which make the webapp interactive, what we mean by interactive is that the user should not need to refresh the page for updating the content. The content should be updated automatically.

There are two aspects to this interactivity,

1. Websockets: This comes into picture when you want to keep _all_ the open tabs in sync. Let's say the same user has logged into your webapp using three different browsers, you'd use websockets to keep all those pages updated. 
2. AJAX: When you login to a todo manager, it doesn't refresh the page, but just updates the state of the website depending on the action you took (i.e. login), it will then show you the pending tasks.

# What is Vue?

Vue is a library for writing such applications. Vue is technically a library focused on the View layer of the classic MVC model, along with [supporting libraries](https://github.com/vuejs/awesome-vue#libraries--plugins), one can easily build a interactive webapp. Vue provides tools for simplifying the development but you do not have to use any of those to get started, speaking from experience, I've always found it better to start without using any tool "vanilla" and later, when we realize _why_ the tools were created, then we can start using them.

Let's continute the example in the end of the installation chapter.

## Loading Vue
For using Vue, we first have to load it. Since it is a JS library, all we have to do to use Vue is to load it using a script tag, like we include jQuery and other libraries. `<script src="vue.js"> </script>`.  

Do note that Vue loads _after_ the html page has loaded (this is the reason if it takes a lot of time to fetch data or do computation, the user would see your `{{code}}` because in that case it'd take Vue some time to initialize the state of the app and render the data.

That's why we place it at the end of the body tag when including the file as a script tag. If we do not include it at the bottom of the page, then it'd be loaded _before_ the html is loaded and it might not work properly. When we include the file in the script tag, a global variable by the name of `Vue` is created. We use this variable to create the new app or components.

file: `chapter1/0basic_example.html`

	<html>
	<head>
	</head>
	<body>
	   <div id="app">
	      {{ message }}
	   </div>
	   	<script src="vue.js"> </script>
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

This is the simplest possible Vue app. We are rendering the variable `message` in the `<div>` tag. If you have had previous experience in template rendering then it might sound trivial, since all rendering engines do this work. But here, things are different. Vue has done a lot of work under the hood. It hasn't _just_ rendered `message`, the variable is "reactive". If the value of `message` changes, then the html would be automatically updated by Vue. The advantage of this feature is that wheneve the value of `message` variable changes, it'd get reflected in our HTML _without_ us having to render the variable again. Vue does all the heavy lifting. All we are left with is creation of variables and maintaining the state of the app _correctly_ using the variables.

To verify our claim, open the web inspecter in your browser, on Firefox do a right click on the page and click "inspect element". It shall open the web inspector. Press the escape key and the console would pop up. Set the message to any value you'd like like this, `app.message='oh my Dod, it did change.'`

You'd see that the html was changed. During runtime, Vue uses a diffing algorithm to make optimal changes to the DOM. It uses shadow DOM to render html, it is a feature which would come into all browsers eventually.

> Note: Firefox has a developer edition (https://www.mozilla.org/en-US/firefox/developer) of it's browser which is tailored for web developers

## Syntax

Since Vue works on data, it uses a special syntax to render the data, by default it is `{{ }}`, but we can change it to whatever syntax we want. The way to do it is 

	var app = new Vue({
		el: '#app',
		data: {
			message : 'good morning'
		},
		delimiters: ['${', '}']
		# this makes our syntax as ${}
	})

Change of rendering syntax is necessary while using Vue with languages which also use `{{ }}` for template rendering like [Go](http://golang.org)

## Obligatory theory

We first created a Vue instance. The syntax is

	var app = new Vue({
		// options
	})

We pass the options object while intializing Vue's root instance for our app. 

It contains the 

1. `el`: element which the app is going to be bound. The Vue app exists **only** in this tag.
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

In addition to data properties, Vue instances expose a number of useful instance properties and methods. These properties and methods are prefixed with $ to differentiate them from proxied data properties. 

For example:

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

So far we’ve only been binding to simple property keys in our templates. But Vue actually supports the full power of JavaScript expressions inside all data bindings:

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

	  <div id="app4">
	      <ol>
	            <li>{{todo}}</li>
		    <li>{{todo}}</li>
		    <li>{{todo}}</li>
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


####Note 
In the interest of saving space, we will showing just snippets. It is the same and it needs to be present for your app to work.


**Tip**: When we have an example, please create a new html file and save it according to whatever convention you want and open it in a browser to verify what you have learned.

When we open this in a browser, you will see "do something" being displayed thrice. We do not want the same message to be displayed! We want different messages and eventually we want to add our own todos.

####Note 
For variables to be reactive, we need to define them in the data field. For components data is a function and not a variable. We will get to that later.

file: `2basic-todo-2.html`

Earlier we had one variable `todo`, but since we want to display multiple messages, we need to define an array. Update the data field by the following:

    data: {
        todos: [
          { text: "do something!"},
          { text: "now do something!"},
          { text: "now do something else!"}
        ]
	}

Doing this bounds the `todos` array to the current `Vue` instance named `app4`. This creates the array of tasks which will be showed on the html page. For that to happen, we need to change the HTML page as well.

Instead of `<li>{{todo}}</li>`, we will have to now display an array. Vue provides us with a `for` construct.

### v-for
v-for has two syntaxes, `v-for="item in items"` or `v-for="(item, index)" in items`.
The first one is just a loop over arrays, the second one loops over the array _and_ has the index of each element during the for loop itself.

We are going to need the index for our delete function, hence we use the second iteration of `v-for`.

	<li v-for="(todo, index)">{{ todo.text }}</li>

Anything that starts with `v-` is a directive. It is evaluated by Vue. If something is wrong, it will complain in the JS console which we checked few lines back. 

> **Always open the JS console while writing a vue app**

When the loop evaluates, `todo` will take one value at a time till it reaches th end of todos, the index will do the same about the position of the value in the `todos` array.

## Getting User Input

We now want to be able to add todo items in our list.

file: `3basic-todo-3.html`

For adding a new element to the existing html, we need to use a new directive called `v-model`.

### v-model
v-model is used to double bind the value of an input tag to a Javascript variable. We define the variable in the `data` field of our Vue instance. Suppose we are having the input tag which takes the title of the task, we write `v-model="todo.title"` in the input tag. This will sync the input tag's data _and_ the value of the `todo.title` variable.

We first need to define the `todo` variable for this double binding to work, please note that we can't do double data binding on our todos array. It will be the data store, and `todo` is going to be like our RAM.

### Methods

Methods are functions which are going to be valid for that Vue instance. Currently we require the method to add a todo. For starters it should not be doing everything, we are going incrementally. Let's first ensure that we get the user data correctly, once we have the data in our `todo` variable, then we can do anything with it.

We add the definition of `todo` and the methods as stated below:

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

There are two changes which need to be done to the html part of the page. First is where we take input and second is where we display todos.

	<input placeholder="add title" v-model="todo.title" />
	<input placeholder="add text" v-model="todo.text" />
	<button v-on:click="AddTodo(todo)">Add todo</button>

	<ol>
	      <li v-for="(todo, index) in todos">
	          {{ todo.title }} : {{ todo.text }}: {{ todo.assign }}
	      </li>
	</ol>

For adding a new item to our todo list, we need to do the following things:

1. Get the input from the <input /> tag (this is done automatically by two way data binding using v-model)
1. Add the input to our `todos` 
1. Render them in our ol tag

## v-on
This directive handles events. When we do a v-on:click="AddMethod", it'll run AddMethod when the element is clicked.

We are nearly done in our app. We have taken input from the user, we render the array correctly. We have the event handling mechanism ready. Just the missing link is to do something with the data which we are practically dumping with sending just an alert. We need to call our method which will add todo in our todos array. We do that by event handling i.e. the `v-on` directive. The `AddTodo` method is bound to our Vue instance, it won't be available outside the `<div id="app">` tag.

In our `AddTodo` method we change the alert line to	`this.todos.push(this.todo)`. It will add the `todo` to our `todos` array.

> It is important to use `this.` to refer to `todos` because scoping is a big issue here and Vue won't be able to find out what todo you are referring to.

## No HTML change
If you were paying attention, then you would have noted that we didn't manually change the HTML. We just changed the rendering and made changes to the JS portion. Vue will handle everything by itself.

## Add feature to assign task

`file: 4basic-todo-4.html`

We now have a new requirement, we need to add the feature to assign tasks to someone.

We first need to add `assign` as a part of the `todo` object `todo: {title: '', text:'', assign:''},`

Then we need to add a new `<input >` with the v-model as `todo.assign`. It'll do two way binding of the input field and `todo.assign`.

Lastly, we need to display the new variable while rendering. Just adding a new `{{ todo.assign }}` would be enough. 

Save and open the page in a browser. There is one catch, not all tasks have an assign field. There is a way to handle this in Vue. It is by using the `v-if` directive. 

## v-if 
v-if directive will show the HTML tag only if there is a value in the object assigned to it. 

## v-else-if
This is a new tag in Vue2.1, it allows a proper if-else statement which we are so familiar with from other languages to be used in Vue templates.

## The template tag

In our example, there is another catch, we are rendering the content just by doing `{{}}`. This begs the question as to where to place the `v-if`. Because `v-if` is to be used in a html tag. The `<template>` tag was created for this purpose only. It is a blank tag which can be wrapped around a group of elements if we want to conditional render them.

Make the following change to the HTML

    {{ todo.title }} : {{ todo.text }} 
    <template v-if="todo.assign"> {{ :todo.assign }}</template>

This will render `todo.assign` only if the task has the value in the object. 

Save and open the page in a browser and add a new task with keeping the assign field blank and one with assign field as any value. You'll notice that only in the second case does the value display on the html page.

## Polishing

If you notice, once we add the task to our todo list, the input fields still have the same value. We do not want that. We want the input fields to be blank as soon as we add the task to the html.

Add the following line after the `push` of the AddTodo.

	this.todo = {title:'', text:'', assign:''}

This will reset the input elements. This plays an important role apart from resetting the input tags. If we do not reset `this.todo`, then, we are binding each new element which we add in. Try doing it yourself once and then you'll understand the real issue.

## Using Components

For decades we have been using reusable components in our other programming projects. But since the web wasn't really imagined beyond document sharing between DARPA scientists, there wasn't much of a structure to it. The companies like Netscape which created some of the basic things and Xerox which invented TCP/IP later established the standards. One such missing standard was web components.

It is an amazing concept. Allow us to create custom HTML tags, which we can easily reuse in other projects.

Now, we can have a `<todo>` tag for our app rather than having to write all the `<li>` stuff directly in our `index.html`, the advantage it offers is that our HTML pages look clean, plus, we can reuse the components in other projects.

When we build a custom element, we can hide a lot of code in our index.html.

We will include this in our index.html just as we use normal HTML tags like follows:

	<ol>
	   <todo></todo>
	</ol>

A component should be defined before the Vue instance is created

	Vue.component('todo', {
	  template: '<li> hello world </li>'
	})

You shall see hello world repeated twice. This is because Vue will replace `<todo>` with the `template` field. Components support a lot of options some of which we will see later.

We got started with components, we need to now render the actual tasks.

Vue components do allow us to define data field. But we will _not_ define the data in the component because we want the UI and the data to be decoupled, hence we will pass the data to the component from the html page and the component would just render the data which is passed to it. Data is passed to a component using the `props` field.

This way, we can fetch data by AJAX calls and render them in the component. Plus we can take this component and easily add to another project as we are just going to render external data in this component.

The syntax to define a component is 

	Vue.component('name',{
		// options
	})

The name should always be in kebab case, smaller case alphabets separated by a -, like <my-element> and **not** <MyElement>. Vue does not restrict us from using CamelCase, but we should not use it just to stay compliant with HTML standards.

Our todo has the following definition:

	Vue.component('todo', {
		props: ['item'],
		template: '<li>{{item.title}} \
		{{ item.text}} \
		<template v-if="item.assign">\
		{{item.assign}}</template></li>'
	});
	
    <todo v-for="todo in todos" v-bind:item="todo"></todo>

We want to show each element in the array `todos` and want to bind the current element's `item` prop to current 'todo' in the loop.

**Note:** Usually Vue templates are long, hence we have to use the '\' delimiter to have multi line templates. 

> If you want to check a sample app, check [go-vue-events](http://github.com/thewhitetulip/go-vue-events), the code uses a component based model to delete an event using a HTTP request. Backend can be written in any language, this project's backend is written in Go. We will be refering to it in a future chapter. But it is a good idea to jump into reading other projects while learning a language/framework so that you would develop your logic.


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
