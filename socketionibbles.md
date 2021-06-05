# Socket.io Nibbles

As my first node.js and first socket.io project, I made a simple multiplayer version of Nibbles (aka Snake).  This works in a similar way to http://slither.io (but is much less sophisticiated)

This page serves as documentation of what I learned.

socket.io is a javascript library that provides an implementation of the WebSocket protocol.  The WebSocket protocol allows us to establish a persistent connection between a web client and web server.  Using this we can rapidly send messages between the machines, without the overhead of establishing new connections.

## Project structure

I bootstrapped a react.js project using create-react-app.  In the root of the project I added an additional file, server.js, which would become the code for the node.js server.

I installed the following packages using npm install:

* express
* socket.io

I found that both the react code and the node.js code can use the packages installed in the node_modules folder.

## Server code

socket.io requires us to start the connection by providing a http server.  We can use the built in 'http' module, or we can use Express, a more powerful web application framework.  I decided to use express.

The code to start up express and socket.io on the server looks like this: 

```javascript
var express = require('express');
var app = express();
var http = require('http').Server(app);
var io = require('socket.io')(http);
const path = require('path');

// serve the build folder as static files.
app.use(express.static(path.join(__dirname, 'build')));

// listen for connections
http.listen(8000, function(){
    console.log('Server started');
});

io.on('connection', function(client) {  
    console.log('Client connected');

    client.on('join', snakeId => {
        // code to handle this message from the client
    });

});

```

As you can see, the code does several things:

* it tells express to serve the "build" folder as static files.  this is where our react.js build output will go.  This means the server can serve our compiled client code to the web browser.
* it starts a http server listening on port 8000
* it listens for socket.io connections being created
* it handles a socket.io message from the client, "join"

Then I implemented some simple snake game code on the server.  I set snakes at initial positions on the screen when they send the "join" message to join them game.

Then I created a timer using setInterval which moves all players in the direction that they're currently facing, implements collision detection, and sends the new positions of all snakes out to all clients using io.sockets.emit()

Finally I listened for a changeDirection message from the client, which changes the direction that their snake is travelling.

The [socket.io documentation](https://socket.io/docs/emit-cheatsheet/) describes several interesting ways to send out messages.  I used two methods, one that sends messages to a single client (client.emit) and another to send messages to all clients (io.sockets.emit)

## Client code

The client component constructor establishes a socket.io connection to the server.  I didn't add any error handling if the server isn't there.  That ought to be done really.

The client listens for key presses and simply sends the join and changeDirection messages to the server when the keys are pressed.  That  code essentially looks like this:

```javascript
import openSocket from 'socket.io-client';

class App extends React.Component {

    constructor() {
        super();
        this.socket = openSocket('http://localhost:8000');
    }

    onKeyDown(e) {
        switch (e.keyCode) {
            case 37:
                this.socket.emit('changeDirection', { snakeId: this.state.snakeId, direction:  LEFT });
                break;
            case 38:
                this.socket.emit('changeDirection', { snakeId: this.state.snakeId, direction: UP } );
                break;
            case 39:
                this.socket.emit('changeDirection', { snakeId: this.state.snakeId, direction: RIGHT } );
                break;
            case 40:
                this.socket.emit('changeDirection', { snakeId: this.state.snakeId, direction: DOWN } );
                break;
            case 74: // J = join
                if (this.state.alive == false) {
                    let newSnakeId = Math.random();
                    this.socket.emit('join', newSnakeId);
                    this.setState({snakeId: newSnakeId});
                }
                break;
            default:
                break;
        }
    }
}
```

As you can see I'm also using a random ID to distinguish which of the snakes the messages are for.  There's probably better (more secure..) ways to do that, but this sufficed.

In the constructor I also listen for messages from the server.  When the server sends the snakes new positions, I simply write this into the React state.  This causes a re-render, which will render my canvas, including the new positions of the snakes.

```javascript
// receive snake positions
this.socket.on('snakes', data => {
    this.setState({
        snakes: data
    });
});
```

I think this is probably not the right way to implement a real game, as the rendering of each screen refresh is being driven by the receipt of messages from the server.  For smoother gameplay in low-ping situations I think we'd have to build a solution where the screen re-renders based on a local timer.  Then we might figure out a method to deal with lag (i.e the client is rendering a prediction of what happens, because it hasn't yet retrieved the truth from the server)  I think that's how Rocket League (and perhaps FPS games) work.  But that's out of scope here.

#### Canvas

I used the additional package [react-canvas-component](https://www.npmjs.com/package/react-canvas-component) which provided a wrapper around the HTML canvas element from React.  I think this package isn't contributing much so we probably don't need it.  I notice the very nice code provided for [Reacteroids](https://github.com/chriz001/Reacteroids/) has a way of grabbing the canvas context using a ref.  That's what I'd prefer to do next time.

In React's render() method I render the react-canvas-component and bind a local function called drawCanvas.

```xml
<Canvas
    id="canvas"
    tabIndex="1"
    draw={this.drawCanvas.bind(this)}
    width={800}
    height={510}
    onKeyDown={this.onKeyDown.bind(this)}
/>
```

The drawCanvas function renders the game screen, drawing walls and the snakes.

```javascript
drawCanvas({ctx, time}) {
    const {width, height} = ctx.canvas;

    ctx.save();

    // draw background
    ctx.fillStyle = 'blue';
    ctx.fillRect(0, 0, width, height);

    // draw walls
    ctx.fillStyle = 'red';
    ctx.fillRect(0, 0, 800, 10); // top
    ctx.fillRect(0, 500, 800, 10); // bottom
    ctx.fillRect(0, 0, 10, 500); // left
    ctx.fillRect(790, 0, 10, 500); // right

    // draw snakes
    this.state.snakes.forEach( snake => {
        ctx.fillStyle = snake.colour;
        snake.segments.forEach( segment => {
            ctx.fillRect(segment.x * 10, segment.y * 10, 10, 10);
        });
    });

    // draw food
    ctx.fillStyle = 'lightgreen';
    ctx.fillRect(this.state.food.x * 10, this.state.food.y * 10, 10, 10);

    ctx.restore()
}
```

## Hosting node.js on nearlyfreespeech

I wanted to host my node.js code on a public server.  That means I need a server that will let me listen on my websocket port.  (e.g. a virtual private server)

I've used nearlyfreespeech for cheap hosting for years, and was pleased to see that they actually provide a way for us to run custom servers.  

I read on the nearlyfreespeech blog that they will let you run any kind of service (e.g a database) but your external connections are limited to HTTP.  The traffic gets proxied through their Squid HTTP proxy before reaching a server that you can open ports on.

This helpful [guide](http://www.mopsled.com/2015/run-nodejs-on-nearlyfreespeechnet/) gave me the information about how to set this up.  

The basic steps I took were:

* Using the nearlyfreespeech admin control panel, create a new site specifying the "custom" domain type.
* upload server.js and the react build folder to the /home/protected folder on the new site.  I did my code uploads by using sshfs to mount the filesystem, but you could also use SCP or FTP.
* in an SSH session on the server I ran npm install to restore the packages.  The custom server already had node and npm available.
* add a run.sh file to the /node/protected folder.  I also chmod this to allow execution, but I'm not sure if that is required here.  The code for the script is simply:

```text
node /home/protected/server.js
```

* in the the nearlyfreespeech admin control panel, add a daemon and set the command line to /home/protected/run.sh.  I set "run daemon as" to "web", which runs the service as a public web user.
* in the the nearlyfreespeech admin control panel, add a proxy and set protocol=http, document root=/, targetport=8000 (which is where my server.js is set to listen), and enabled "bypass apache entirely".  This then appears to proxy the connections through a Squid which is listening on port 80 to the server listening on port 8000.  The WebSocket connections appear to make it through Squid without issue.
* in the the nearlyfreespeech admin control panel, start the daemon.

Luckily the log of any daemon output is available so you can resolve any issues.  You can find the log in /home/logs/.  

Initially I encountered "permission denied" errors when starting the daemon.  I probably fixed that by altering the user/group or permission attributes on the files.  The permissions that ended up working look like this:

```text
drwxrwx---   4 261502  web         7 Feb 11 09:33 .
drwxr-xr-x   8 root    261502      8 Feb 10 09:57 ..
drwxrwxr-x   3 261502  261502      8 Feb 11 09:34 build
drwxrwxr-x  82 261502  261502     82 Feb 10 10:14 node_modules
-rw-rw-r--   1 261502  261502  21391 Feb 10 10:14 package-lock.json
-rwxrwxr-x   1 261502  261502     32 Feb 10 10:21 run.sh
-rw-rw-r--   1 261502  261502   6900 Feb 11 09:33 server.js
```


