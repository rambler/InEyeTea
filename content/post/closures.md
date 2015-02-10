+++
date = "2013-12-23T10:52:13-06:00"
title = "Closures in Node.js with Async.js"

+++

## Closures in Node.js with Async.js

I recently had a chance to use closures as part of solving a real world problem. The elegance of the approach helped to improve my attitude about working in JavaScript. Don’t get me wrong – Node.js’s asynchronous approach is a thing of beauty. But there’s a lot of ugliness in JavaScript as well and this is the first thing besides Node.js itself that struck me as particularly beautiful.

The problem is this: I have a list of questions, each of which has an associated policy. For a particular user, I need to check the policy to see if it’s appropriate to ask the question for this user.

So, policy evaluation is per user and per question. However, my API offers the ability to get all relevant questions for a particular user. So every time, I need to evaluate policy for a set of questions. Each policy evaluation may involve an async call to the data store. The policy evaluation function for a particular user and question returns a boolean, like this:

```js
var evalUserQuestionPolicy = function(uid, question, cb) {
```

To minimize impact on the data store, I want to do these calls in series, a functionality provided by the javascript Async.js library through async.filterSeries. However, the iterator argument to filterSeries takes a single argument (i.e. the question). I could work around this by making the arr argument an array of objects with both a user and a question but that’s inefficient and inelegant.

A much more elegant solution is to use closures to produce just the iterator function I want: one for a particular user.

```javascript
var policyEval = function(user) {
    return function(question, cb) {
        return evalUserQuestionPolicy(user, question, cb);
    }
}
```

Now, I can call this function to produce the iterator when I need it:

```js
async.filterSeries(questions,
                   policyEval(req.params.uid),
                   function(results) {
```