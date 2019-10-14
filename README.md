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

```javascript
// const mongodb = require('mongodb'); // native mongodb driver for node.js
// const MongoClient = mongodb.MongoClient;
// const ObjectID = mongodb.ObjectID; let us generating our own ID's
const { MongoClient, ObjectID } = require('mongodb');
const id = new ObjectID(); // generating a new ID
console.log(id); // 5da394e96da6533a0c512fe7 (contain a timestamp)
console.log(id.id.length);
console.log(id.toHexString());
console.log(id.getTimestamp()); // 2019-10-13T21:23:03.000Z

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
  //============================================= CREATE =============================================//
  //-------------------------------------- INSERT ONE DOCUMENT --------------------------------------//
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

  //-------------------------------------- INSERT FEW DOCUMENTS --------------------------------------//
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
  //============================================= /CREATE =============================================//

  //==================================== READ (Querying Database) ====================================//
  //--------------------------------------- FIND ONE DOCUMENT ---------------------------------------//
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
  //--------------------------------------- FIND FEW DOCUMENTs --------------------------------------//
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
  //============================================= /READ =============================================//

  //============================================= UPDATE =============================================//
  //------------------------------------------- UPDATE ONE -------------------------------------------//
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

  // // Shorthand version
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
  //------------------------------------------- UPDATE MANY -------------------------------------------//
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
  //============================================= /UPDATE =============================================//

  //============================================= DELETE =============================================//
  //------------------------------------------- DELETE ONE ------------------------------------------//
  db.collection('users').deleteOne({ // filter object
    age: 28
  }).then((result) => {
    console.log(result); // { n: 1, ok: 1 }
  }).catch((error) => {
    console.log(error);
  });
  //------------------------------------------- DELETE MANY ------------------------------------------//
  db.collection('users').deleteMany({ // filter object
    age: 29
  }).then((result) => {
    console.log(result); // { n: 3, ok: 1 }
  }).catch((error) => {
    console.log(error);
  });
  //============================================= /DELETE =============================================//
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