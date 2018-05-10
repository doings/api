# Api Workshop

Angular client -> https://bit.ly/2KQQ9cy

## 0. Install nodemon
---------------------

npm i -g nodemon

## 1. Create package.json
-------------------------

```javascript
{
  "name": "doings-api",
  "version": "1.0.0",
  "description": "doings app simple api",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcrypt": "^2.0.1",
    "body-parser": "^1.18.2",
    "cors": "^2.8.4",
    "express": "^4.16.3",
    "jwt-simple": "^0.5.1",
    "mongoose": "^5.0.17",
    "passport": "^0.4.0",
    "passport-jwt": "^4.0.0",
    "sha1": "^1.1.1",
    "validator": "^10.1.0"
  }
}
```


## 2. Create express - index.js
-------------------------------

```javascript
const express = require('express')
const app = express()

app.listen(3000, function () {
  console.log('doings-api listening on port 3000!')
})
```


## 3. Create service
--------------------

Create service file
-------------------
touch movement.js

```javascript
module.exports = function(app) {

  app.route('/movement')
    .get(function (req, res) {
      res.send('Get Movement!')
    })
    .post(function (req, res) {
      res.send('Post Movement!')
    })
    .put(function (req, res) {
      res.send('Put Movement!')
    })
    .delete(function (req, res) {
      res.send('Delete Movement!')
    })
}
```

Include service file in index.js
--------------------------------

```javascript
require('./movement.js')(app);
```


## 4. Connect mongo (without mongo...)
--------------------------------------

```javascript
const mongoose = require('mongoose')

mongoose.connect('mongodb://localhost:27017/doings_workshop');
var db = mongoose.connection;

db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('Connected to MongoDB');

  require('./movement.js')(app);

  app.listen(3000, function () {
    console.log('doings-api listening on port 3000!')
  })
});
```


## 5. Install mongo
-------------------

https://docs.mongodb.com/manual/administration/install-community/

## 6. Install bodyparser middleware
-----------------------------------

```javascript
const bodyParser = require('body-parser');

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: true
}));
```


## 7. Install CORS middleware
-----------------------------

```javascript
const cors = require('cors')

app.use(cors({
  origin: '*',
  credentials: true
}));
```


## 8. Let's read some input data
--------------------------------

```javascript
app.route('/movement')
  .get(function (req, res) {
    res.send('Get Movements')
  })
  .post(function (req, res) {
    res.send('Post Movement: ' + JSON.stringify(req.body))
  })
  .put(function (req, res) {
    res.send('Put Movement: ' + JSON.stringify(req.body))
  })

app.route('/movement/:movement_uuid')
  .delete(function (req, res) {
    res.send('Delete Movement: ' + JSON.stringify(req.params.movement_uuid))
  })
```

## 9. Create a mongoose model
-----------------------------

touch movement.model.js

```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var movementSchema = new Schema({
  movement_uuid: {
    type: String,
    unique: true,
    required: true
  },
  concept: String,
  // balance - income - outcome
  type: String,
  date: Date,
  amount: Number,
  created_at: Date,
  updated_at: Date
});
movementSchema.index({
  movement_uuid: 1,
  type: 1
});

movementSchema.pre('save', function(next) {
  var movement = this;
  const currentDate = new Date();

  this.updated_at = currentDate;
  this.amount = movement.amount ? movement.amount : 0;

  if (!this.created_at)
    this.created_at = currentDate;

  next();
});

module.exports = mongoose.model('Movement', movementSchema);
```


## 10. Save movement
--------------------

```javascript
var mongoose = require('mongoose');
var sha1 = require('sha1');

var MovementModel = require('./movement.model');

.post(function (req, res) {

  if (!req.body.concept) {
    res.json({
      success: false,
      msg: 'Please insert the movement concept.'
    });
  } else {

    // create uuid
    var uuid = sha1(Date.now() + req.body.amount + req.body.concept)

    var newMovement = new MovementModel(req.body);
    newMovement.movement_uuid = uuid;
    newMovement.save(function(err) {
      if (err) {
        return res.json({
          success: false,
          msg: 'Uuid already exists. Try again.'
        });
      }
      newMovement._id = undefined;
      newMovement.__v = undefined;
      res.json({
        success: true,
        movement: newMovement
      });
    });
  }
})
```

## 11. Get movements
--------------------

```javascript
.get(function (req, res) {

  MovementModel.find({}, {
      _id: 0,
      __v: 0
    },
    function(err, movements) {
      if (err) throw err;
      if (!movements) {
        res.json({
          success: false,
          msg: 'Movements not found.'
        });
      } else {
        res.json({
          success: true,
          movements: movements
        });
      }
    });
})
```


## 12. Update movement
----------------------

```javascript
.put(function (req, res) {

  MovementModel
    .findOne({
      movement_uuid: req.body.movement_uuid
    }, {},
    function(err, movement) {
      if (err || !movement) {
        res.json({
          success: false,
          msg: 'Movement not found.'
        });
      } else {
        movement.concept = req.body.concept,
        movement.type = req.body.type,
        movement.date = req.body.date,
        movement.amount = req.body.amount
        movement.save(function(err, data) {
          res.json({
            success: true,
            movement: movement
          });
        });
      }
    });
})
```


## 13. Delete movement
----------------------

```javascript
.delete(function (req, res) {

  MovementModel.findOneAndRemove({
    movement_uuid: req.params.movement_uuid
  }, {},
  function(err, deleted) {
    res.json({
      success: true,
      deleted: deleted
    });
  })

})
```

