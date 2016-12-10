# Core Concepts

In the last chapter, we learned how to build an app using Vue. There are always two aspects to take care about when we start writing apps. The first one is the logic of the app or the architecture and another one is how to accomodate that architecture using the syntax which our stdlib or framework provides. It is the same in our case as well, we learned how to write an app using Vue in the earlier chapter and we built the core architecture too, which included creating the necessary variables to display content on our web page. The only thing and perhaps the most important thing which we missed was to interaction with the server.

We need to design a few things before we can start hooking our front end with the backend. There are two aspects of interaction with the server

1. Fetch everything when the web page is loaded.
1. Fetch as per user interaction.

Both methods are fine, but the catch is that when our data is huge, the app becomes slow due to the large data fetch. That is why we end up choosing the second option to fetch data incrementally.

We have the following data flow:

1. Load all pending tasks
	1. If the user interacts with the tasks, do the changes to the Vue data store _after_ sending the changes back to the server.
1. If the user clicks on category
	1. Go back to the server and fetch todos related to the category
		1. If the user adds/deletes tasks, add them to the Vue data store _after_ sending it back to the server.
		1. If user clicks on category name/pending => fetch them again (the webserver is our single source of truth)
1. If the user clicks on pending/deleted/completed
	1. Go back to the server and fetch related tasks
		
This means that in our architecture, we only go back to the server when we need to fetch data, modifying data is done in place after sending the modified data back to the server and if the server responds back with a 200 status code which means everything went fine on the server.
