# Building an app

We will be building a full app in this chapter by using Vue. You will need the download the code from Github. That is the basic HTML which we have to transform.

It is the HTML for a todo list manager. It is really simple, we have a `<div id="timeline">` where we are listing the tasks. We have a navigation drawer where we have the list of categories and we can select which one we want to select. Each task can either be pending/deleted/completed.

First of all, add the script tag to inclue Vue at the bottom of the body tag.

Currently, the `tasks` id is attached to the body tag, we need to change this since it is not recommended to attach a Vue object to the body tag, we will wrap everything inside the body tag with a span like this. <span id="tasks"></span>

Then, we create a Vue app. Since we are going to write a backend in Go, we need the delimiter to be ${} as it would interfere with Go's templating system. Feel free to skip changing the delimiters if you aren't using Go for your backend.

	var app = new Vue({ 	
		//~ this is the element in the html page where Vue will be anchored
		el: '#tasks',
		delimiters: ['${','}']
	)}

Now we need to understand how many variables we need to create. Start by running the html and seeing what all data is bring displayed. That'll give us a starting point regarding what variables to create. Do make a note that Vue is going to handle all the data portion of our html page, everything should be done via Vue that means the state of the html page would change as the variables which we define in Vue will change.

The first variable we'll create is to store the 'Pending'. We'll call it navigation.

> In this chapter, we will be adding or editing elements of the above declaration of Vue, to save space, we won't be writing the entire block, just copy paste at the relevant portions

Add a new variable,

	data:{
		navigation='Pending',	
	}

At the same time, we edit the 'Pending' from the html page to ${navigation}.

Save and refresh.

the html should look like this

	<p class="navbar-brand" href="/pending">
		${navigation}
	</p>
	

We now directly jump to the `timeline`. The timeline id will be where our actual content rests. It currently consists of multiple blocks of html, each block starts with `<div class="note" id="1">`. We will be using Vue to modify that.

These are the elements inside each task

1. noteHeading
1. noteContent
1. created
1. priority
1. 1 out of 3 is completed (this is when we display a task list in github flavoured markdown)
1. commentslist (there can be more than one comments)
	1. content
	1. author
	1. time

Along with this, we also need to be able to toggle the visibility of our noteContent, there is a button at the top right corner near the note title, when we click that, it should toggle visibility of noteContent. We will be using Vue to do that. This is why, we need a boolean variable to store the state of visibility. that's where `showComment` comes into picture. It's default value is false. When we click it, it should turn to true.

We need to create an object to enclose everything. Add these two elements to the data part of the Vue instance.
	
	task:{ID:"", title:"", content:"", category:'', priority:'', comments:[], showComment:false},
	comment:{content:"", author:"", created:""},
	
But, this is a task variable, we need an array of this variable to store our tasks.
	tasks: [] // stores all the tasks

use this as sample data. Add this element to the data part of our Vue instance.
	
	tasks:[task:{ID:"1", title:"title1", content:"content1", category:'dummy', priority:'1', comments:[comment:{content:"dummy comment", author:"sherlock", created:"2016-12-3"},], showComment:false},task:{ID:"1", title:"title2", content:"content1", category:'dummy', priority:'1', comments:[comment:{content:"dummy comment", author:"sherlock", created:"2016-12-3"},], showComment:false},]

	
> Note that the `task` variable would store the information of one task, that is, we will bind the `task.title` to our input form's title field. Primarily `task` will be used only when we take input. But, the `tasks` variable is the most important variable in our example as it is going to store everything. It is the main variable.

	<span v-for="(task, taskIndex) in tasks">

We cycle in the array in a span tag.

Since we can't use ${} inside a tag like this <div id="${task.ID}"> we have to end up binding the id field like this 
`<div class="note" v-bind:id="task.ID">`

We then turn to changing the elements of the task.

1. "Replace One task" with `${task.title}`
1. "This is a task" with `${task.content}`
1. "1 out of 3 is completed" with `${task.completed}`

Now, comments.

We have more than one comments per task possible, thus, we will be cycling through comments too.

    <div class="commentslist" v-for="(comment, commentIndex) in task.comments">

replace 

1. "tsample comment" with `${comment.content}`
1. "suraj" with `${comment.author}`
1. "26 April" with `${comment.created}`

Now, we move to the bottom of the task,

1. "Priority: 3" with `Priority: ${task.priority}`.
1. "dummy" with `${task.category}`

delete the other note tag. We do not need any more elements because we are going to cycle through the `tasks` object we created in our Vue instance and have Vue handle the html on its own.

Save and refresh the page, you shall see that the tasks are rendering properly.

Now, we will be fine tuning the changes we made and add a few more parameters wherever required.

since it is not necessary that every task's content would be a markdown tasklist, we need to display the '1 out of 3 completed' message only if it exists, thus we wrap it in a template tag and use the `v-if` directive, this means that the template would be rendered only when there is value inside task.completed. 

	<template v-if="task.completed">${task.completed}</template>

A keep observer might note that what if there are no tasks created, what do we show then? As of now, there is nothing displayed on the page, this is bad from the UX perspective because the user does not understand what is happening and if our app is not intuitive then users will hate it.

Let's think about what we will be doing in this scenario. our logic would be simple to handle this case, 

1. if there are tasks, then show them
1. if there are no tasks, show a blank element saying "there are no tasks."

We will be using the `v-if` directive again, since tasks is an array, we need to check the length of it to see if it has values.
we wrap the current task html code inside a span tag `<span v-if="tasks.length">`.

You'll see a commented block of html starting with "this is to be displayed". Wrap that block in a `<span v-else></span>` and uncomment it. It is the message we want to display when there is nothing in the tasks array. Please note that we have to use `tasks.length` because if we use `v-if="tasks"` it is going to be true always as this is an array, thus, if the length of the array is 0, we want to display the "no tasks present" message.

Save and refresh, in the Vue instance, remove everything from the tasks object. This shall trigger the `else` block and we shall see the message.

It is time to start interacting with the page via events. We did our basic job until now to ensure that the backbone for events exist.

## Adding a new task

Locate the html comment <!-- Add note modal -->

We first need to change `<form enctype="multipart/form-data" action="/add/" method="POST">`. It was an app written in pure html and thus it used to call the HTTP endpoint /add/ with the POST request.

We will remove the `action=` and add `v-on:submit.prevent="onSubmit"`. 

The reason why we remove the `action=` is that we do not want the page to be redirected to /add/ as form normally does. Plus, when we click submit, we want to prevent the form from being submitted. 

always remember, `Vue does everything`.

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

	category:{categoryID:'',categoryName:'', taskCount:''}, // data structure to store category
	categories:[{categoryID:'1',categoryName:'dummy', taskCount:'2'},{categoryID:'2',categoryName:'Batman', taskCount:'12'}], // stores all the categories

As we did for the task, we have two objects, category and categories. One will be bound to the input tag when we take category name and another would be the list of categories in our current instance.

	<select name="category" class="dropdown" v-model="task.category">
		<template v-for="category in categories">
			<option v-bind:value="category.categoryName"> ${ category.categoryName } </option>
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
		    this.task = {title:"", content:"", category:'', priority:'', comments:[], showComment:false}
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

What we will have to do here is that assign `this.tasks` to a call to the backend API to load the particular tasks
