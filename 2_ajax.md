# Loading data with Ajax

Ajax is short for Async Javascript and XML. It is used to develop dynamic apps which do not require us to refresh the page to update information. Vue does not support AJAX with it's own library, but we are free to use existing libraries like axios/loadash/vue-resource. We will be using vue-resource in this book.

The source code is available on Github. You can clone the repo and delete the logic part and start fresh [go-vue-events](http://github.com/thewhitetulip/go-vue-events).

In the first chapter we saw how to use `v-if`, `v-on`, `<template>` and `v-bind`. The UI which we have to build is simple, we want to build a simple event manager which will interact with the backend.

There are two parts to our HTML page, the event list which will render our `events` array and another is the input tags which will enable the user to add new events. The rendered list will also allow the user to delete events.

We will not be focusing on the HTML part, as we have spent one chapter discussing how to write the front end code. What we will focus on in this chapter would be the AJAX part.

Just a brief overview of the HTML:

`<event-item v-for="(event,index) in events" :event="event" :index="index"></event-item>`

This will be our custom component which we define in app.js file. We include this file _after_ including Vue.js and Vue-resource. If we include this file before either of those two, then there would be errors since the script depends on both of them.

We start with a simple Vue instance.


	var app = new Vue({
	  el: '#events',
	
	  data: {
	    event: { title: '', detail: '', date: '' },
	    events: []  
	  },
	  
	  delimiters: ['${', '}'],
	  
	
	  mounted: function () {
	    this.fetchEvents();
	  },
	
	  methods: {
	
	    fetchEvents: function () {
	        var events = [
	        {title:"title one", detail:'detail1', date:"2016-1-1"},
	        {title:"title two", detail:'detail2', date:"2016-1-21"},
	        {title:"title three", detail:'detail2', date:"2016-2-21"}
	        ];
	        Vue.set(this.$data, 'events', events);
	    },
	
	    addEvent: function () {
	      console.log("add function")
	    },
	    
	    deleteEvent: function(index) {
	      console.log("deleting "+ index)
	    }
	});
	
## Callbacks

In the first chapter we saw that when we define a Vue instance, among the options is something called `callbacks`. Mounted is one such callback. After the element has been created and inserted into the DOM, it is called mounted. Immediately after the element is mounted, which means our app is being used, we want to fetch our events and assign the array we fetch to the array we defined in the data portion of our app.

We use Vue.set to set the data (events) field of the Vue instance to the events variable we will fetch from the AJAX request. As of now, we will simulate the AJAX request to ensure that the app works, once we are sure it works, then we can just replace this with an AJAX call.

It is advisable to not start creating components at the start, according to the startup mantra "Do things which don't scale". Once we have our code ready, we can easily swap elements by custom elements.

While writing apps which work using AJAX, we send the HTTP request to our backend API like `delete`, if we get a success response, then we remove the element from the JS array. We do the same with other requests like `add`, we send the data to our API, if API sends back status as success, we add the element to our Javascript array otherwise we show an error message.

## Methods
Reading the documentation of vue-resource, we see that this is the syntax to send an http request

	fetchEvents: function () {
        var events = [];
        this.$http.get('/api/events/')
        .then(response => response.json())
        .then(result => {
           Vue.set(this.$data, 'events', result);
            console.log("success in getting events")  
        })
        .catch(err => {
            console.log(err);
        });
    }
    
    addEvent: function () {
      if (this.event.title.trim()) {
        // this.events.push(this.event);
        // this.event = { title: '', detail: '', date: '' };
        this.$http.post('/api/events/', this.event,{emulateJSON: true})
          .then(response => response)
          .then( result => {
            this.events.push(this.event);
            console.log('Event added!');
            this.event = { title: '', detail: '', date: '' };
          }).catch( err => {
            console.log(err);
          });
      }
    },
    
    deleteEvent: function (index) {
      if (confirm('Really want to delete?')) {
        console.log(index);
        this.$http.delete('/api/events/' + index)
          .then(response => response)
          .then( result =>{
              console.log(result);
              app.events.splice(index, 1);
            }).catch( err => {
              console.log(err);
              alert("unable to delete")
            });
      }
    }


The general trend is, send the HTTP request, take the response, convert it to json, if it throws an error (like when the API is not accessible) then log the error, otherwise do some action push the new element to the array or remove the element from the array.

We can now use web components to represent our list. We do the following:

	Vue.component('event-item', {
	  props: ['event', 'index'],
	  template:'\
	  <span>\
	        <h4 class="list-group-item-heading"><i class="glyphicon glyphicon-bullhorn"></i>  {{event.title }}</h4>\
	        <h5><i class="glyphicon glyphicon-calendar" v-if="event.date"></i> {{ event.date }}</h5>\
	        <p class="list-group-item-text" v-if="event.detail">{{ event.detail }}</p>\
	        <button class="btn btn-xs btn-danger" v-on:click="deleteEvent(index)">Delete</button> </span>',
	  methods: {  
	     deleteEvent: function (index) {
	      if (confirm('Really want to delete?')) {
	        console.log(index);
	        this.$http.delete('/api/events/' + index)
	          .then(response => response)
	          .then( result =>{
	              console.log(result);
	              this.events.splice(index, 1);
	            }).catch( err => {
	              console.log(err);
	              alert("unable to delete")
	            });
	      }
	    }
	  }
	});
	
This is fine, but there is one problem, there is no way for Vue to know where the `events` array exists which we are trying to splice. This is because the array was created in the Vue instance and thus can't be accessed by using this. We need to use `app.todos` to access it.

## The API
This example code does HTTP calls. It expects an API to be present. If you know how to build an API, you would need to build the following things:

1. DELETE /api/events/{id} : This will delete the event from the DB
1. GET /api/events: will fetch all events
1. POST /api/events: This will add the new event

You can use any language to build the backend, if you want to see how to build a backend in Go, please refer to my [Go book](https://github.com/thewhitetulip/web-dev-golang-anti-textbook/), you can read the server code in `server.go`.

## Pitfalls
GET and DELETE requests are simple, they are basically just hitting the API without any other data. POST is a bit complicated. This is why in the line `this.$http.post('/api/events/', this.event,{emulateJSON: true})`, we are using `{emulateJSON:true}`, bad things happen if we don't use this. It is because the backend API expects to talk in JSON and the default POST doesn't use JSON to talk, it talks like this title=sometitle&content=thiscontent. When we give the `emulateJSON:true`, we tell Vue to send it as JSON.
