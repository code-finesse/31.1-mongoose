# 31.1-mongoose

Mongoose is a dependency

it makes it easier for us to interact with MongoDB and NodeJS via javascript

we use MongooseJS to relay all of the requests to and from MongoDB in our NodeJS applications

it also acts like a safeguard against bad data by checking the data against a ***schema***

## Make A Schema

- a schema will allow us to set specific keys in our objects
    - if we have a key of name, we won't be able to insert other keys that don't match like firsName or names
    - helps keep our data mor organized and reduces the chance of errors
    - e.g.

        ```jsx
        const articleSchema = new Schema(
          {
            title: { type: String, required: true, unique: true }, //can say whether we want properties to be required or unique
            author: { type: String, required: true },
            body: String,
            comments: [{ body: String, commentDate: Date }], // can have arrays of objects with specific properties
            publishDate: { type: Date, default: Date.now }, // can set defaults for properties
            hidden: Boolean,
            meta: {
              // can have properties that are objects
              votes: Number,
              favs: Number
            }
          },
          { timestamps: true }
        );
        ```

### Basic Set Up

Same as creating a node project

- Change directories to your `sandbox`.
- Create a new directory: `mkdir intro_to_mongoose`
- `cd intro_to_mongoose`
- `touch app.js`
- `npm init -y` and go through the prompts
- `npm i mongoose`
- `touch tweet.js`
- `code .`

### Set Up Mongoose

set configuration at top of app.js (almost like a boilerplate)

```jsx
// Dependencies
const mongoose = require("mongoose");
const Tweet = require("./tweet.js");

// Global Configuration
// these are the same
// const mongoURI = 'mongodb://localhost:27017/tweets'
const mongoURI = "mongodb://localhost:27017/" + "tweets";
// db is just a shorter variable name
const db = mongoose.connection;

// Connect to Mongo
// Remember about deprecated uses
mongoose.connect(mongoURI,{
  useNewUrlParser: true,
  useCreateIndex: true,
  useUnifiedTopology: true
}, () => {
  console.log("the connection with MongoDB is established");
});

// Connection Error/Success - optional but can be helpful
// Define callback functions for various events
db.on("error", err => console.log(err.message + " is MongoDB not running?"));
db.on("connected", () => console.log("MongoDB connected on: ", mongoURI));
db.on("disconnected", () => console.log("MongoDB disconnected"));

// Automatically close after 5 seconds
// for demonstration purposes to see that you must use `db.close()` in order to regain control of Terminal tab
setTimeout(() => {
  db.close();
}, 5000);
// we can also get out of mongo with control + C
```

## Set Up Schema

in tweet.js (example name)

```jsx
const mongoose = require("mongoose"); // require mongoose
const Schema = mongoose.Schema; // create a shorthand for the mongoose Schema constructor

// create a new Schema
// This will define the shape of the documents in the collection
// https://mongoosejs.com/docs/guide.html
const tweetSchema = new Schema(
  {
    title: String,
    body: String,
    author: String,
    likes: { type: Number, default: 0 },
    sponsored: { type: Boolean, default: false }
  },
  { timestamps: true }
);

// Creating Tweet model : We need to convert our schema into a model-- will be stored in 'tweets' collection.  Mongo does this for you automatically
// Model's are fancy constructors compiled from Schema definitions
// An instance of a model is called a document.
// Models are responsible for creating and reading documents from the underlying MongoDB Database
// from here: https://mongoosejs.com/docs/models.html
const Tweet = mongoose.model("Tweet", tweetSchema);

//make this exportable to be accessed in `app.js`
module.exports = Tweet;
```

## Create a Document with Mongoose

In app.js

Let's make ourselves an object to insert into our database

```jsx
const myFirstTweet = {
  title: "Deep Thoughts",
  body: "Friends, I have been navel-gazing",
  author: "Karolin"
};

// Then we'll use Mongoose to create a Tweet:

Tweet.create(myFirstTweet, (error, tweet) => {
  if (error) {
    //if there is an error console log it
    console.log(error);
  } else {
    // else show us the created tweet
    console.log(tweet);
  }
  // get control of terminal back
  // else just use control c
  db.close();
});

// Let's run this with node app.js inside the terminal
```

multiple can be created with

```jsx
Tweet.insertMany(manyTweets, (error, tweets) => {
  if (error) {
    console.log(error);
  } else {
    console.log(tweets);
  }
  db.close();
});
```

make sure to comment out any creations so that duplicates aren't inserted

## Find Documents with Mongoose

- Mongoose has 4 methods for this
- `find` - generic
- `findById` - finds by ID - great for Show routes!
- `findOne` - limits the search to the first document found
- `[where](http://mongoosejs.com/docs/queries.html)` - allows you to build queries, we won't cover this today

- Let's find all

    ```jsx
    Tweet.find((err, tweets) => { console.log(tweets); db.close();
    });
    ```

- Let's limit the fields returned, the second argument allows us to pass a string with the fields we are interested in:

    ```jsx
    Tweet.find({}, "title body", (err, tweets) => { console.log(tweets); db.close();
    });
    ```

- Let's look for a specific tweet:

    ```jsx
    Tweet.find({ title: "Water" }, (err, tweet) => { console.log(tweet); db.close();
    });
    ```

- We can also use advanced query options. Let's find the tweets that have 20 or more likes

    ```jsx
    Tweet.find({ likes: { $gte: 20 } }, (err, tweets) => { console.log(tweets); db.close();
    });
    ```

## Delete Documents with Mongoose

- `remove()` danger! Will remove all instances
- `findOneAndRemove()` - this seems like a great choice
- `.findByIdAndRemove()`finds by ID - great for delete routes!
- e.g.

    ```jsx
    Tweet.findOneAndRemove({ title: "Deep Thoughts" }, (err, tweet) => {
      if (err) {
        console.log(err);
      } else {
        console.log("This is the deleted tweet:", tweet);
      }
      db.close();
    });
    ```

## Update Documents with Mongoose

Finally, we have a few options for updating

- `update()` - the most generic one
- `findOneAndUpdate()`Let's us find one and update it
- `findByIdAndUpdate()` - Let's us find by ID and update - great for update/put routes!
- e.g.

    If we want to have our updated document returned to us in the callback, we have to set an option of {new: true} as the third argument

    ```jsx
    Tweet.findOneAndUpdate(
      { title: "Vespa" },
      { sponsored: true },
      { new: true },
      (err, tweet) => {
        if (err) {
          console.log(err);
        } else {
          console.log(tweet);
        }
        db.close();
      }
    );
    ```
