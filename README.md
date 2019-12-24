# subscribe-for-data

Tool, that fills objects (models?) with related data. 

 * DB agnostic. 
 * one DB query per subscription.
 * all queries running parallel, starting at same moment, you choose then 
 * No data loops inside, stream based.
 * zero dependencies
 
It generates mongo condition to query your source, but this is just default 
behavior

## Installation

```shell script
npm i subscribe-for-data
```

## Example

By default it works with mongoose. This behavior can be easily overriden by 
setting custom `getStream` option callback.

```javascript
const { makeSubscription, fillSubscriptions } = require('subscribe-for-data');
const RootModel = require('./MainModel');
const RelatedModel = require('./RelatedModel');
const AnotherRelatedModel = require('./AnotherRelatedModel');

(async () => {
  const relatedSubscription = makeSubscription(RelatedModel, { // single field attach
    targetField: 'position', // key of property to be created on roots
    foreignField: 'root_id', // field to build condition against
    sourceField: 'position'
  });
  const anotherRelatedSubscription = makeSubscription(AnotherRelatedModel, { // something completely different
    getCondition({ mysteriousTimestamp, type }) {
      return { type, updatedAt: { $gt: mysteriousTimestamp} };
    }, 
    assignData(root, { someField, otherType }) {
      (root.someFields = root.someField || []).push(someField);
      (root.otherTypes = root.otherTypes || new Set()).add(otherType);
    },
  });
  const roots = [];
  await RootModel.find({}).cursor().eachAsync((root) => {
    [relatedSubscription, anotherRelatedSubscription]
      .forEach(subscription => subscription.add(root)); // subscribed
    roots.push(root);
  });
  await fillSubscriptions(); // 2 DB queries executed in parallel, no loops then
  console.log(roots[0]);
})();
```

```json
{
  "_id": "000000",
  "position": 42,
  "someFields": [2, 5, 8, 5],
  "otherTypes": [23, 42, 78]
}
```
