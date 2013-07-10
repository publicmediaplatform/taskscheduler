taskscheduler
=============

A topic-based task scheduler for Node.js. Allows for pluggable queue implementation. 

For a sample queue plugin that uses Amazon SQS, see: [SQSTask](https://github.com/publicmediaplatform/sqstask)

## Status

A [very] early release. Feel free to: inspect, hack, enjoy and contribute (e.g.: feedback, documentation or bug fixes), 
but use at your own risk in production. 

## Installation

Since they are under active development, TaskScheduler and SQSTask are intentionally not published to npm, yet. 
Until that happens best way to use the modules is to point to Github URLs in your packages.json file, with something
like the following:

```json
{
  "name" : "Example Project"
, "version" : "1.1.0"
, "dependencies": {
    "taskscheduler"      : "git://github.com/publicmediaplatform/taskscheduler.git"
  , "sqstask"            : "git://github.com/publicmediaplatform/sqstask.git"
  }
}
```

## USAGE

Before you can use taskscheduler you have to configure couple of things:

### Setup

```javascript

var AWSConfig = {
      "accessKeyId"     : "..."
    , "secretAccessKey" : "..."
    , "awsAccountId"    : "..."
  };

var util      = require('util')
  , sqstask   = require('lib/sqstask')(AWSConfig)
  , scheduler = require('lib/taskscheduler')(sqstask);
  
```  

### Registering a Handler

```javascript
var publisherHandlerID = scheduler.addTopicHandler("publisher", taskJob, 100);

function taskJob(topic, message, callback) {

  console.dir("Task job fired, with message: " + message);
   
  var err = null;
  
  var random = Math.floor(Math.random() * 5) + 1;
  if (random === 5) {
    err = new Error("something");
    console.log("Error simulated for message: " + message);
  }    

  callback(err);
    
};
```

### De-Registering a Handler

```javascript
//-- You can also de-register a task, if you don't want it running "forever".

setTimeout(function(hID) {
  scheduler.removeTopicHandler(hID);
}, 1000, publisherHandlerID);
```

### Sending messages

```javascript

scheduler.topicEnsureExist(test_topic, function(err) {

  if (!err) {
    sendmessagesAndReadMessages();
  } else {
    console.dir(err);
  }

  
}); // end of topic ensuring.


function  sendmessagesAndReadMessages() {
  for (var i = 0; i<5; i++) {
    scheduler.message( "publisher"
                     , "This is message # " + new Date().getTime()
                     , function(err, result) {
      if (err) {
        util.log("Error sending a message to the queue: " + util.inspect(err.Body.ErrorResponse.Error));
        console.log(err);
      }
    });
  }
}
```
