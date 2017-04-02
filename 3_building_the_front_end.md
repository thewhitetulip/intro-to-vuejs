# Building an app

> This chapter makes two assumptions, first is that you know enough JS to understand the code displayed below and that you have written a web app before, if you haven't written a web app, you can read [Write webapps in Go without using a framework](https://github.com/thewhitetulip/web-dev-golang-anti-textbook/) to learn how to write one in Go or any other book if you prefer another language.

We will be building a full app in this chapter by using Vue. You will need to download the code from Github. That is the basic HTML which we have to transform using Vue.

We should first understand the need of using Vue. Without using a framework like Vue, we can either have a pure HTML app which requires the pages to be reloaded for every transaction which happens at the backend, or we can use jQuery to handle AJAX requests which would change the state of our webapp whenever we send a request to the webserver. The jQuery way works for small applications, but as the application becomes large, it is a bit difficult to maintain. Thus, Vue.js was created, it provides us a minimalist framework to build a single page app along with some supporting libraries.

In the code repository, open the folder named chapter3, it contains two folders, initial and final. In the `initial` folder, there is a file called `tasks.html` which is the HTML for a simple todo list manager. `final` contains the code which we will write through this chapter. I recommend not reading it until you have finished this chapter.

The structure of the html page is simple, `timeline` block where we are listing the tasks, a navigation drawer which has a list of clickable categories, the menu bar will display what type of tasks are being displayed.

Each task can either be pending/deleted/completed. One category can have multiple tasks, category names should be unique.

First of all, add the script tag to include Vue at the bottom of the body tag.

Currently, the `tasks` id is attached to the body tag, we need to change this since it is not recommended to attach a Vue object to the body tag, we will wrap everything inside the body tag with a span like this. `<span id="tasks"></span>`

Then, we create a Vue app. Since we are going to write a backend in Go, we need the delimiter to be `${}` as it would interfere with Go's templating system. Feel free to skip changing the delimiters if you aren't using Go for your backend.

	var app = new Vue({ 	
		// this is the element in the html page where Vue will be anchored
		el: '#tasks',
		delimiters: ['${','}']
	)}

As we said above, we will be managing the state of our html via vue, this means understanding what data elements we create in our Vue app. These "data elements" would be simple variables inside the `data` block. 

### Running the html

For running the html, you have to use a webserver, otherwise the static files won't be found. When we do not use webservers for rendering static files, they have an absolute path like `./scripts/jquery.js` but with a webserver in hand, the path is `/scripts/js`, without a webserver to render `/script/`, our web page would not render properly. 

### Server to use 

You are free to use any webserver you would like, `python2 -m SimpleHTTPServer`, `python3 -m http.server` or [the f server](http://github.com/thewhitetulip/f). I wrote f, it is a 0 configuration server which takes only the port number as an argument, `go get github.com/thewhitetulip/f` would install it, `f 9090` would run a server on port 9090, it is faster than the python alternative.

Running the server on port `9090` in the `initial` folder and navigate to `http://localhost:9090/tasks.html` and take a look at the page. You will need to start understanding how to manage the data architecture, for that, you'll need to look at what data is being displayed. That'll give us a starting point regarding what variables to create. Do take note that Vue is going to handle all the data portion of our html page, everything should be done via Vue, that means the state of the html page will change as the variables which we define in Vue will change.

The first variable we'll create is to store the 'Pending'. We'll call it navigation. (It is a bad variable name, you can change it to anything you want).

> To save space, we won't be writing the entire Vue block here so just copy paste the snippets at relevant positions.

Add a new variable,

	data:{
		navigation='Pending',	
	}

At the same time, we edit the 'Pending' from the html page to `${navigation}`.

Save and refresh.

the html should look like this

	<a class="navbar-brand" >
		${navigation}
	</a>
	
We now directly jump to the `timeline`. Timeline block contains all our tasks. It currently has multiple blocks of html, each block  `<div class="note" id="1"></div>` corresponds to one `task`. 

We will be using Vue to render the tasks from a JSON which we eventually fetch from the server using RESTful APIs.

These are the elements inside each task

1. noteHeading
1. noteContent
1. created
1. priority
1. 1 out of 3 is completed (this is when we display a task list in github flavoured markdown)
1. commentslist (there can be more than one comment)
	1. content
	1. author
	1. time

Along with this, we also need to be able to toggle the visibility of our noteContent, there is a button at the top right corner near the note title, when we click that, it should toggle visibility of noteContent. We will be using Vue to do that. This is why, we need a boolean variable to store the state of visibility. That's where `showComment` comes into the picture. It's default value is false. When we click it, it should turn to true.

> For those who are coming from a jQuery background, it would be difficult at first to stop thinking in the jQuey way, but it is important to start thinking in the "Vue" way of doing things.

Since Vue is going to render elements, we need to create an object to store the data which we want to render.

Add these two elements to the data part of the Vue instance. These are the individual `task` and `comment` instances. We also have the `tasks` array which stores all the data attributes associated with each task.
	
	task:{ID:"", title:"", content:"", category:'', priority:'', comments:[], showComment:false},
	comment:{content:"", author:"", created:""},
	tasks: [] // stores all the tasks

Before we hook this up with a RESTful API, we would need to render some data, thus, we use sample data to get things working. Once we confirm that our front end works correctly, we can hook up the server. 

Modify the `tasks` element.
	
	tasks:[
	task:{ID:"1", title:"title1", content:"content1", category:'dummy', priority:'1', 
	comments:[comment:{content:"dummy comment", author:"sherlock", created:"2016-12-3"},],
	showComment:false},task:{ID:"1", title:"title2", content:"content1", category:'dummy',
	priority:'1', comments:[comment:{content:"dummy comment", author:"sherlock", 
	created:"2016-12-3"},], showComment:false},]

We will be binding input tags to elements inside the `task` variable. For instance, the input tag which takes the title of the task would be bound to `task.title`, other input tags are bound to corresponding attributes to our `task` object.

`tasks` is the main variable which will control what is currently getting displayed in our page.

We cycle in the array in a span tag. `<span v-for="(task, taskIndex) in tasks">`

Since we can't use `${}` inside a tag like  `<div id="${task.ID}">` we have to bind the id field like this `<div class="note" v-bind:id="task.ID">`

We then turn to changing the elements of the task.

1. "Replace One task" with `${task.title}`
1. "This is a task" with `${task.content}`
1. "1 out of 3 is completed" with `${task.completed}`

Next stop, comments.

We have more than one comments per task possible, thus, we will be cycling through comments too.

    <div class="commentslist" v-for="(comment, commentIndex) in task.comments">

replace 

1. "tsample comment" with `${comment.content}`
1. "suraj" with `${comment.author}`
1. "26 April" with `${comment.created}`

Now, we move to the bottom of the task,

1. "Priority: 3" with `Priority: ${task.priority}`.
1. "dummy" with `${task.category}`

Delete the other note tag. If you notice, we will have one block per task, when we generate html using our server, we will need to create a template and render the data according to task list which we get from the server, but we do not need to do all that when we use Vue, the framework takes care of the html, we just need to tell it what we want.

Save and refresh the page, you shall see that the tasks are rendering properly.

Now, we will be fine tuning the changes we made and add a few more parameters wherever required.

Since it is not necessary that every task's content would be a markdown tasklist, we need to display the '1 out of 3 completed' message only if it exists, thus we wrap it in a template tag and use the `v-if` directive, this means that the template would be rendered only when there is value inside task.completed. 

	<template v-if="task.completed">${task.completed}</template>

A keep observer might note that what we have not handled the case if there are no tasks. We show a blank page in that case. This is bad from the UX perspective because the user does not understand what is happening and if our app is not intuitive then users will hate it.

Let's think about what we will be doing in this scenario. our logic would be simple to handle this case, 

1. if there are tasks, then show them
1. if there are no tasks, show a blank element saying "there are no tasks."

We will be using the `v-if` directive again, since tasks is an array, we need to check the length of it to see if it has values.
we wrap the current task html code inside a span tag `<span v-if="tasks.length">`.

You'll see a commented block of html starting with "this is to be displayed". Wrap that block in a `<span v-else></span>` and uncomment it. It is the message we want to display when there is nothing in the tasks array. Please note that we have to use `tasks.length` because if we use `v-if="tasks"` it is going to be true always as this is an array, thus, if the length of the array is 0, we want to display the "no tasks present" message.

Save and refresh, in the Vue instance, remove everything from the tasks object. This shall trigger the `else` block and we shall see the message.

It is time to start interacting with the page via events. We did our basic job until now to ensure that the backbone for events exist.

## Adding a new task

Locate the html comment `<!-- Add note modal -->`

We first need to change `<form enctype="multipart/form-data" action="/add/" method="POST">`. It was an app written in pure html and thus it used to call the HTTP endpoint /add/ with the POST request.

We will remove the `action=` and add `v-on:submit.prevent="onSubmit"`. 

The reason why we remove the `action=` is that we do not want the page to be redirected to /add/ as form normally does. Plus, when we click submit, we want to prevent the form from being submitted. 

Always remember, `Vue does everything relatd to html`.

our form tag looks like this `<form enctype="multipart/form-data" method="POST" v-on:submit.prevent="onSubmit">`

We will not bind our input variables with our `task` object. This is why we created the `task` object.

"title" to `v-model="task.title"` 
"content" to `v-model="task.content"`
"priority" to `v-model="task.priority"`
"category" to `v-model="task.category"`

The priority tag should look like this

	<input type="radio" name="priority" value="3" v-model="task.priority" /> High
	
select tag
	
	<select name="category" class="dropdown" v-model="task.category">

We need categories to be rendered, thus we will create a variable `categories`

	category:{categoryID:'',categoryName:'', taskCount:''}, 
	categories:[
	{categoryID:'1',categoryName:'dummy', taskCount:'2'},
	{categoryID:'2',categoryName:'Batman', taskCount:'12'}], 

As we did for the task, we have two objects, category and categories. One will be bound to the input tag when we take category name and another would be the list of categories in our current instance.

	<select name="category" class="dropdown" v-model="task.category">
		<template v-for="category in categories">
			<option v-bind:value="category.categoryName">
			${ category.categoryName } </option>
		</template>
	</select>
	

This will show the list of categories.

`<button type="button" class="btn btn-primary" v-on:click="addTask(task)">Submit</button>`

Only one aspect is left, to actually add a note. We will add the `v-on:click` to our Submit button. Please note that we aren't handling the forms like regular forms are handled with web apps. We will be calling a function addTask and passing it the `task` variable. 

The reason behind this is that we are binding each input element, every input tag is bound to an element _inside_ the task object, thus the task object is the content of the entire task which we should add. We pass the object to the method.

Save and refresh, and click on the + button at the right bottom corner.

Add a new `methods` element to our app

	var app = new Vue({ 
	....
	methods: {
		addTask: function (item) {
		    this.tasks.push(this.task);
		    this.task = {title:"", content:"", category:'', 
		    priority:'', comments:[], showComment:false}
		    $('#addNoteModal').modal('hide');
		  }
	}

We define a function with an argument item, we can access the `tasks` and `task` which we declare in our data field.

	this.tasks.push(this.task) 
	
This will add the elements to the task. The task would then be appended to the array. But if you save and refresh, you'll notice that the input tag would still be having the values. This is because of the two way data binding. We need the replace `this.task` with a blank object. If we do not do that, then all the new elements would still be doubly bound to our input tag.


Save and refresh, it should now append tasks to our task list.

The next part would be performing actions to our tasks. If you see the task, there are multiple icons to do stuff

1. edit
2. complete
3. delete

We need to write three functions to do the respecive tasks.

Let's start with the simplest, delete.

	add `@click="deleteTask(taskIndex)` to the delete icon's tag.
	
Now we will write the delete method:

	deleteTask: function(index){
	  this.tasks.splice(index, 1);
	}

We just pass the index of the task to this method and it'll splice that index from the array.


	add `@click="complete(taskIndex)"` to the complete icon's tag.

At this point of time, our app will have the follow this data architecture:

1. tasks : array will store the tasks which are currently displayed
1. completed: array to store all completed tasks

when we want to display the completed tasks, we will assign the `completed` array to `tasks`.

#### Completed

	  // this will mark the task as completed
	  complete: function(index){
		  this.completedTasks.push(this.tasks[index]);
		  this.tasks.splice(index, 1);
	  }

`complete` will add the task to the completed array and delete it from the existing tasks.

#### Edit

The last part of the task related activities is edit. during edit, we will be assigning the task's value to the input tag. Then we would 


## Categories

The next important aspect of our app is categories. The user can add/edit/delete categories. They can also display tasks related to a particular category by clicking on the category name in the sidebar.

First, we will render the categories. In the `<!-- SIDEBAR -->` block. This is kept as exercise to the readers, we have done exactly the same above in the add task modal.

We then have the task of displaying tasks of the particular category. The `@click="taskByCategory(category)` needs to be added to each category.

	taskByCategory: function(category){
	  this.selectedCategoryName = category.categoryName;
	  this.selectedTaskTypeName = ''
	  this.tasks = [];
	},

What we will have to do here is that assign `this.tasks` to a call to the backend API to load the particular tasks.
