taskscheduler
=============

A topic-based task scheduler for Node.js. Allows for pluggable queue implementation. 

For a sample queue plugin using Amazon SQS, see: [SQSTask](https://github.com/publicmediaplatform/sqstask)

## USAGE

### Registering a Handler

```javascript

var AWSConfig = {
      "accessKeyId"     : "..."
    , "secretAccessKey" : "..."
    , "awsAccountId"    : "..."
  };

var util    = require('util')
  , sqstask = require('../sqstask')(AWSConfig)
  , ts      = require('../taskscheduler')(sqstask);

var publisherHandlerID = ts.addTopicHandler("publisher", taskJob, 100);

var taskJob = function(topic, message, callback) {

  console.dir("Task job fired, with message: " + message);
   
  var err = null;
  
  var random = Math.floor(Math.random() * 5) + 1;
  if (random === 5) {
    err = new Error("something");
    console.log("Error simulated for message: " + message);
  }    

  callback(err);
    
};

//-- You can also de-register a task, if you don't want it running "forever".

setTimeout(function(hID) {
  ts.removeTopicHandler(hID);
}, 1000, publisherHandlerID);
```

