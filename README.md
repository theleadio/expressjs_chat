# NodeJS Chat Application Demo

This is a demo of how to build a chat application using NodeJS. We will be using Socket.IO for the chat functionality and integrate it with ExpressJS, which is an improved version compared to the original one on the official website.

## Start a new project using Express Generator

```bash
express chat --view=ejs
```

We use Embedded JavaScript (ejs) as our view engine. 

p/s: `chat` is the name of the app, you may change it to any name you like


## Install the dependencies and test our website

Run the following command to install necessary dependencies for nodejs and express.

```bash
npm install
```

Let's also install `socket.io` to enable real time communication inside our chat application.

```bash
npm install socket.io --save
```

Run the following command and visit http://localhost:3000 to make sure our website is up and running properly.

```bash
npm run start
```


## Create a route and view for chat

Next, let's configure a route for chat. In order to do that, we need to 

- setup a view
- configure a route


Let's first setup the view by creating a `chat.ejs` in our `views` folder, with the following HTML code.

```html
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```


next, create a route in our main route file


```js
router.get('/chat', function(req, res, next) {
  res.render('chat');
});
```

and test it in our browser with `/chat` path.

p/s: don't forget to restart your node app everytime you made some changes to the code, or ideally, install `nodemon` instead to automatically rerun.


## Configure SocketIO

In order for socket io to work there are two parts we need to take care. 

1. Enable socket at the server side. 
2. Communication with server side socket from front-end.


```js
app.use('/static', express.static('node_modules'));

var http = require('http');
var server = http.createServer(app);
var io = require('socket.io')(server);

io.on('connection', function(socket){
  console.log('a user has connected...');
});
```


don't forget to change this at `app.js`

```js
module.exports = {app: app, server: server};
```

and change line 7 at `www`

```js
var app = require('../app').app;
```

and line 22 at `www`

```js
var server = require('../app').server;
```

at the front end, let's modify our `chat.ejs` and add the following code before `</body>`


```html
    <script src="static/socket.io-client/dist/socket.io.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.js"></script>
    <script>
      $(function () {
        var socket = io();
        $('form').submit(function(e){
          e.preventDefault(); // prevents page reloading
          socket.emit('chat message', $('#m').val());
          $('#m').val('');
          return false;
        });

      });
    </script>
```


## Broadcast message

Now we need to handle the message that we receive. As soon as we received it, we broadcast it back to the original channel. Let's add the following to `io.on('connection')`.

```js
  socket.on('chat message', function(msg){
    console.log('received message: ' + msg);
    io.emit('chat message', msg);
  });
```

Let's add the corresponding code on front-end to handle the event.

```js
	socket.on('chat message', function(msg){
	  $('#messages').append($('<li>').text(msg));
	});
```
