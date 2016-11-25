# Installation

Technically speaking, you do not have to install Vue. You can just include Vue in a `<script>` tag and use it just like you'd use jQuery or any other JavaScript file.

It is recommended to use a CDN

1. [unpkg](https://unpkg.com/vue/dist/vue.js)
2. [jsdelivr](http://cdn.jsdelivr.net/vue/2.0.5/vue.js)
3. [cloudflare](http://cdnjs.cloudflare.com/ajax/libs/vue/2.0.5/vue.js)

It is not mandatory to use a CDN, you can just host the file to your webserver. but it makes sense only when you are using the app internally and it is not deployed on the Internet. It is always useful to use a CDN, because that way, when the app is deployed, there are chances that the user has the script already cached.

# First example

We will be developing a small app as we understand concepts of Vue. This is a basic app which you are not yet expected to understand, just copy paste it in your machine in a html file.

Create a fixed folder in your machine for this book. 

`mkdir -p  ~/code/intro-to-vue`
`cd ~/code/intro-to-vue`
`touch index.html`

Download the `vue.js` file in your `~/code/intro-to-vue` folder.

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

Save the file and open it in your browser. You can open it by right click -> open with -> firefox/chrome