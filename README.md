      __ _          _                    
     / _(_)        | |                   
    | |_ _ _ __ ___| |__   ___  ___  ___ 
    |  _| | '__/ _ \ '_ \ / _ \/ __|/ _ \
    | | | | | |  __/ | | | (_) \__ \  __/
    |_| |_|_|  \___|_| |_|\___/|___/\___|
    
    Build Realtime web applications in Ruby

# What is Firehose?

Firehose is both a Rack application and JavasSript library that makes building scalable real-time web applications possible.

# How is it different from socket.io?

socket.io attempts to store connection state per node instance. Firehose makes no attempt to store connection state.

Also, socket.io attempts to abstract a low-latency full-duplex port. Firehose assumes that its impossible to simulate this in older web browsers that don't support WebSockets. As such, Firehose focuses on low-latency server-to-client connections and encourages the use of HTTP transports for client-to-server communications.

Finally, firehose attempts to solve data consistency issues and authentication by encourage the use of proxying to the web application.

# Getting Started

First, you'll need to install and run RabbitMQ.

```
apt-get install rabbitmq    # Install on Ubuntu
brew install rabbitmq       # Install on Mac Homebrew
```

## The Consumer

The consumer is the web server that your client connects to for real-time updates. Create a config.ru file with the following:

```ruby
require 'rubygems'
require 'firehose'

run Firehose::Rack::App.new
```

Now run the config.ru file in a server that supports async Rack callbacks (like thin or rainbows)

```ruby
thin -R config.ru -p 4000 start
```

## The Publisher

Lets test the producer! Open two terminal windows. In one window, curl the consumer server:

```sh
curl "http://localhost:4000/"
```

Then run the following script in another terminal:

```ruby
require 'rubygems'
require 'firehose'

Firehose::Publisher.new.publish('hi there!').to('/greetings')
```

## JavaScript Client

Then in your browser create a new Firehose Client object as such:

```javascript
new Firehose.Client()
  .url({
    websocket: 'ws://localhost:5100',
    longpoll:  'http://localhost:5100'
  })
  .params({
    cid: '024023948234'
  })
  .options({
    timeout: 5000
  })
  .message(function(msg){
    console.log(msg); // Fires when a message is received from the server.
  })
  .connected(function(){
    console.log('Howdy friend!');
  })
  .disconnected(function(){
    console.log('Bu bye');
  })
  .connect()
```