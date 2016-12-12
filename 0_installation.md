# Installation

Technically speaking, you do not have to install Vue. You can just include Vue in a `<script>` tag and use it just like you'd use jQuery or any other JavaScript file.

It is recommended to use a CDN

1. [unpkg](https://unpkg.com/vue/dist/vue.js)
2. [jsdelivr](https://cdn.jsdelivr.net/vue/2.1.3/vue.js)
3. [cloudflare](https://cdnjs.cloudflare.com/ajax/libs/vue/2.1.3/vue.js)

> Note: Please do not use Github as your CDN. vue.js is hosted on Github too, but Github is _not_ a CDN.

It is not mandatory to use a CDN, you can just host the file to your webserver. but it makes sense only when you are using the app internally and it is not deployed on the Internet. It is always useful to use a CDN, because that way, when the app is deployed, there are chances that the user has the script already cached.

## First example

We will be developing a small app as we understand concepts of Vue. This is a basic app which you are not yet expected to understand, just copy paste it in your machine in a html file.

Create a fixed folder in your machine for this book. 

```bash
mkdir -p  ~/code/intro-to-vue`
cd ~/code/intro-to-vue`
touch index.html
```

Download the `vue.js` file in your `~/code/intro-to-vue` folder.

```html
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
```

Save the file and open it in your browser. You can open it by right click -> open with -> Firefox/Chrome

## Another way to use Vue

This book is focused for those who are new to front end development in general. A working knowledge of Javascript is presumed, if you don't have that working knowledge, at least you should be able to follow up with the Javascript by doing internet searches and all.

If you are comfortable with using build systems with Node.js, you can use multiple tools like webpack and browserify. Unfortunately, I do not know much of either of those, this book aims to teach the absolute basics of building a Vue app. The future chapters may contain a chapter on how to use webpack, but as of now it is not planned. Please refer to the [official docs](https://vuejs.org) for more details on the advance usage of the framework.
