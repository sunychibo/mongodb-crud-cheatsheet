## MongoDB CRUD Cheat Sheet

### Database structure
<pre>
C:/
   |
   |- mongodb (distributive)
   |- mongodb-data (local database folder) 
</pre>   

### CMD Command
`/mongodb/bin/mongod.exe --dbpath=/mongodb-data`

### Configuring and database connecting
#### Custom ID's generating 
```javascript
const { ObjectID } = require('mongodb');
// ObjectID - let us generating our own ID's
// mongodb - native mongodb driver for node.js
const id = new ObjectID(); // generating a new ID
console.log(id); // 5da394e96da6533a0c512fe7 (contain a timestamp)
console.log(id.id.length);
console.log(id.toHexString());
console.log(id.getTimestamp()); // 2019-10-13T21:23:03.000Z
```
#### Database connection
```javascript
const { MongoClient, ObjectID } = require('mongodb');
// const connectionURL = 'mongodb://localhost:27017'; may cause some problems so the next is better
const connectionURL = 'mongodb://127.0.0.1:27017';
const databaseName = 'task-manager';

// 2nd argument - options object (useNewUrlParser: true because previous was deprecated)
// 3rd argument - callback is gonna get called then we actually connected to the database
// connecting to the database is not asynchronous operation
MongoClient.connect(connectionURL, { useNewUrlParser: true }, (error, client) => {
  if (error) {
    return console.log('Unable to connect to the database');
  }
  const db = client.db(databaseName); // send back a database reference
  ... // CRUD OPERATION
});
```
### CRUD operations
#### CREATE
- CREATE ONE - `insertOne()`
```javascript
db.collection('users').insertOne({
  _id: id, // for most cases we don't need to provide our own id, mongo do it well automatically
  name: 'Ali',
  age: 21
}, (error, result) => {
  if (error) {
    return console.log('Unable to insert user');
  }
  console.log(result.ops); // array of documents within operation
});
```
- CREATE MANY - `insertMany()`
```javascript
db.collection('users').insertMany([
  {
    name: 'Jan',
    age: 28
  }, {
    name: 'Gunther',
    age: 25
  }
], (error, result) => {
  if (error) {
    return console.log('Unable to insert user');
  }
  console.log(result.ops);
})

db.collection('tasks').insertMany([
  {
    description: 'Buy milk',
    completed: false
  }, {
    description: 'Wash car',
    completed: true
  }, {
    description: 'Feed the cat',
    completed: true
  }
], (error, result) => {
  if (error) {
    return console.log('Unable to insert tasks');
  }
  console.log(result.ops);
})
```
#### READ
- READ ONE - `findOne()`
```javascript
db.collection('users').findOne({ name: 'Jan' }, (error, user) => {
  if (error) {
    return console.log('Unable to fetch');
  }
  console.log(user); // { _id: 5da3883755126720346edb8e, name: 'Jan', age: 28 } || null (if not find)
})

// For searching by id we need format id (by new ObjectID) to the native binary format which uses for store ID's
db.collection('users').findOne({ _id: new ObjectID('5da396463df02129445d3404') }, (error, user) => {
  if (error) {
    return console.log('Unable to fetch');
  }
  console.log(user); // { _id: 5da396463df02129445d3404, name: 'Ali', age: 21 }
})
```
- READ MANY - `find()`
```javascript
// method find returns cursor (pointer to the data) of matching documents instead data
// method find don't provide a callback function as an argument
// for fetching data we can use toArray method which provides a callback
db.collection('users').find({ age: 28 }).toArray((error, users) => {
  if (error) {
    return console.log('Unable to fetch');
  }
  console.table(users);
});

// calculate a count of matching
db.collection('users').find({ age: 28 }).count((error, count) => {
  if (error) {
    return console.log('Unable to fetch');
  }
  console.log(count);
});
```
#### UPDATE
- UPDATE ONE - `updateOne()`
```javascript
// update method was deprecated
// We want to update document with an id 5da396463df02129445d3404
// updateOne method returns a promise if no callback passed
const updatePromise = db.collection('users').updateOne({
  _id: new ObjectID('5da396463df02129445d3404') // filter object
}, { // which fields to update object
  $set: { // it's only impacting the fields we've specify
    name: 'Mike'
  }
})

updatePromise.then((result) => {
  console.log(result.result); // { n: 1, nModified: 0, ok: 1 }
}).catch((error) => {
  console.log(error);
});

// Shorthand version
db.collection('users').updateOne({
  _id: new ObjectID('5da396463df02129445d3404') // filter object
}, { // which fields to update object
  $set: { // it's only impacting the fields we've specify
    name: 'Mike'
  }
}).then((result) => {
  console.log(result.result); // { n: 1, nModified: 0, ok: 1 }
}).catch((error) => {
  console.log(error);
});
```
- UPDATE MANY - `updateMany()`
```javascript
// We want to find all document with completed: true and set it to false
db.collection('tasks').updateMany({
  completed: true // filter object
}, { // which fields to update object
  $set: { // it's only impacting the fields we've specify
    completed: false
  }
}).then((result) => {
  console.log(result.result); // { n: 2, nModified: 2, ok: 1 }
}).catch((error) => {
  console.log(error);
});
```
#### DELETE
- DELETE ONE - `deleteOne()`
```javascript
db.collection('users').deleteOne({ // filter object
  age: 28
}).then((result) => {
  console.log(result); // { n: 1, ok: 1 }
}).catch((error) => {
  console.log(error);
});
```
- DELETE MANY - `deleteMany()`
```javascript
db.collection('users').deleteMany({ // filter object
  age: 29
}).then((result) => {
  console.log(result); // { n: 3, ok: 1 }
}).catch((error) => {
  console.log(error);
});
```
## Mongoose CRUD Cheat Sheet
```javascript
const mongoose = require('mongoose');
const validator = require('validator');
const connectionURL = 'mongodb://127.0.0.1:27017';
const databaseName = 'task-manager-api';
// Mongoose uses the Mongo D.B. module behind the scenes so it's provides the abilities of mongodb library
// 1st argument - db connection URL
// 2nd argument - options object
mongoose.connect(`${connectionURL}/${databaseName}`, {
  useNewUrlParser: true,
  useCreateIndex: true // to make sure what all indexes will be created
});

// Define the model we're working with
// 1st argument - string name for your model - it converts it to lower case and it pluralized it and
// then uses that as the collection name: 'User' --> 'users'
// 2nd argument - object of properties definitions (determine what fields we gonna have)
const User = mongoose.model('User', {
  name: {
    type: String,
    required: true, // make the field required then create
    trim: true      // delete spaces from the start and end of the input string
  },
  password: {
    type: String,
    required: true,
    minlength: 7,
    trim: true,
    validate(value) {
      if (validator.contains(value, 'password')) {
        throw new Error('Password cannot contain "password"');
      }
    }
  },
  email: {
    type: String,
    required: true,
    trim: true,
    lowercase: true, // format input string to lowercase
    validate(value) { // for validation we can use popular npm module 'validator'
      if (!validator.isEmail(value)) {
        throw new Error('Email is not valid');
      }
    }
  },
  age: {
    type: Number,
    default: 0, // set the default value if we do not provide this field  
    validate(value) {
      if (value < 0) {
        throw new Error('Age must be a positive number');
      }
    }
  }
});

// // Then we have our model defined we can create instances of that model
const newUser = new User({
  name: 'Leila',
  email: 'leila@GMAIL. Com ',
  password: 'purplePotato123'
})

// // And to add new document to the database we need to use .save() method on our new instance
// // method save() returns us a promise
newUser.save().then((result) => {
  console.log(result); // { _id: 5da4bade0fe2690e184ad7b2, name: 'Leila', age: 23, __v: 0 }
}).catch((error) => {
  console.log('Error!', error); // Error! { ValidationError: User validation failed: email: Email is not valid
});

const Task = mongoose.model('Task', {
  description: {
    type: String,
    trim: true,
    required: true
  },
  completed: {
    default: false,
    type: Boolean
  }
});

const newTask = new Task({
  description: 'Watch TV'
});

newTask.save().then((result) => {
  console.log(result);
}).catch((error) => {
  console.log(error);
});
```

### Promises
```javascript
  //========= Common Asynchronous Callback =========/
  const doWorkCallback = (callback) => {
    setTimeout(() => {
      // console.log('This is my error', undefined);
      console.log(undefined, [1, 4, 7]);
    }, 2000)
  };

  doWorkCallback((error, result) => {
    if (error) {
      return console.log(error);
    }
    console.log(result);
  });

  //=================== Promise ===================/
  const doWorkPromise = new Promise((resolve, reject) => {
    setTimeout(() => {
      // reject('Things went wrong!');
      resolve([1, 4, 7]);
    }, 2000)
  });

  doWorkPromise.then((result) => {
    console.log('Success!', result);
  }).catch((error) => {
    console.log('Error', error);
  });

  //
  //                              fulfilled (resolved)
  //                           /
  // Promise    -- pending -->
  //                           \
  //                              rejected
  //
```
