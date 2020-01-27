# subscribe-for-data

Fast implementation of multiple related models properties fetching & mixing to your model.

##   

 * DB agnostic. 
 * flexible query condition
 * one DB query per subscription.
 * all queries running parallel, starting at same moment, you choose then 
 * No loops, stream based.
 * zero dependencies
 
## Workflow description 

You create `subscription` around your **related model** with `makeSubscription(model, options)`

You can create any count of subscriptions you need.

Then you can to `subscription.add(target)` **target objects** you want to mix in 
properties from **related model** data. 

After you've added all needed **targets** to all `subscriptions` you can anytime run 
`fillSubscriptions()` 

`fillSubscriptions()` assigns data as it goes via **stream** with auto **parallelization** 
if multiple `subscriptions` created. One query per `subscription` is executed.
 
It generates mongo condition. If you return from 
`options.getCondition(target)` *scalar value* then is generated `$in` query. I   
to query your source, 

Mongo query generation is just default behavior, you can alter it as you want.

## Installation

```shell script
npm i subscribe-for-data subscribe-for-data-from-mongoose
```

## Import

```javascript
const mongoosePlugin = require('subscribe-for-data-from-mongoose');
const { 
  makeSubscription, fillSubscriptions 
} = require('subscribe-for-data').use(mongoosePlugin);
```

## Example

By default it works with mongoose. This behavior can be easily overriden by 
setting custom `getStream` option callback.

```javascript
const mongoosePlugin = require('subscribe-for-data-from-mongoose');
const { makeSubscription, fillSubscriptions } = require('subscribe-for-data').use(mongoosePlugin);
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
  await RootModel.find({}).cursor().eachAsync((root) =>  
    [relatedSubscription, anotherRelatedSubscription]
      .forEach(subscription => subscription.add(root)); // subscribed
    roots.push(root);
  });
  await fillSubscriptions(); // 2 DB queries executed in parallel, no loops then
  
  подождите fillSubscriptions(); // 2 DB запросы выполняются параллельно, затем без скобок
  
  console.log(roots[0]);
})();
```

Expected output:

```json
{
  "_id": "000000",
  "position": 42,
  "someFields": [2, 5, 8, 5],
  "otherTypes": [23, 42, 78]
}
```

# API Reference

##

* [SubscribeForData](#SubscribeForData) : <code>object</code>
    * [.makeSubscription(source, options)](#SubscribeForData.makeSubscription) ⇒ <code>Object</code>
    * [.fillSubscriptions()](#SubscribeForData.fillSubscriptions) ⇒ <code>Promise</code>
    * [.assignDefaultOptions(mixin)](#SubscribeForData.assignDefaultOptions)

##

* [assignData](#assignData) : <code>function</code>
* [getKey](#getKey) ⇒ <code>\*</code>
* [extractKey](#extractKey) ⇒ <code>\*</code>
* [getCondition](#getCondition) ⇒ <code>\*</code>
* [getDataHandler](#getDataHandler) ⇒ <code>function</code>
* [getAddingMethod](#getAddingMethod) ⇒ <code>function</code>
* [getStream](#getStream) : <code>function</code>

<a name="SubscribeForData"></a>

## SubscribeForData : <code>object</code>
subscribe-for-data


* [SubscribeForData](#SubscribeForData) : <code>object</code>
    * [.makeSubscription(source, options)](#SubscribeForData.makeSubscription) ⇒ <code>Object</code>
    * [.fillSubscriptions()](#SubscribeForData.fillSubscriptions) ⇒ <code>Promise</code>
    * [.assignDefaultOptions(mixin)](#SubscribeForData.assignDefaultOptions)

<a name="SubscribeForData.makeSubscription"></a>

### SubscribeForData.makeSubscription(source, options) ⇒ <code>Object</code>
Creates subscription for related model data

Создаёт подписку для связанных данных модели


| Param | Type | Description |
| --- | --- | --- |
| source | <code>Object</code> | Source model |
| options | <code>Object</code> | Options |
| options.targetField | <code>String</code> | field data to be saved into (optional) |

| options.targetField | <code>String</code> | данные поля для сохранения в (optional) |

| options.baseCondition | <code>Object</code> | Base condition |

| options.baseCondition | <code>Object</code> | Базовое состояние |

| options.defaultValue | <code>\*</code> | Default value for field |

| options.defaultValue | <code>\*</code> | Значение по умолчанию для поля |

 which returns unique key from target model (model.id by default) |
 которая возвращает уникальный ключ из целевой модели

| options.getCondition | [<code>getCondition</code>](#getCondition) | returns condition, using target model (model.id by default) |

| options.getCondition | [<code>getCondition</code>](#getCondition) | возвращает условие, используя целевую модель (model.id by default) |
| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |

возвращает уникальный ключ целевой модели из внешней модели 

| options.isMultiple | <code>Boolean</code> | if one to many relation |

если одна во многих отношениях

| options.useEachAsync | <code>Boolean</code> | only for mongoose cursor |

только для указателя mongoose

| options.parallel | <code>Number</code> | parallel parameter for `eachAsync` if `useEachAsync` is `true` |
| options.foreignField | <code>String</code> | If `getCondition` returns scalar values this field will be used for $in |

Если `getCondition возвращается, то скалярные величины этого поля будут использованы для $in |

| options.sourceField | <code>String</code> | field to use of foreign model |

поле для использования внешней модели

| options.assignData | [<code>assignData</code>](#assignData) | Do model filling by itself, otherwise use `targetField` |

Пусть модель заполнится самостоятельно, в противном случае используйте `targetField` |

| options.getStream | [<code>getStream</code>](#getStream) | returns stream from source and condition (using mongoose model by default) |

возвращает поток от источника и состояния (исаолтзуя модель mongoose по умолчанию)

| options.getDataHandler | [<code>getDataHandler</code>](#getDataHandler) | Get data handler for processing related models |

Устанавливает обработчик данных для обработки связанных моделей

| options.getAddingMethod | [<code>getAddingMethod</code>](#getAddingMethod) | Get `add()` method of future `subscription` |

Установить `add()` метод будущего `subscription` |

<a name="SubscribeForData.fillSubscriptions"></a>

### SubscribeForData.fillSubscriptions() ⇒ <code>Promise</code>
Fill subscribed targets

<a name="SubscribeForData.assignDefaultOptions"></a>

### SubscribeForData.assignDefaultOptions(mixin)
change default options


| Param |
| --- |
| mixin |

<a name="assignData"></a>

## assignData : <code>function</code>
Assigns data from foreign model to target


| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |
| foreign | <code>Object</code> | foreign model |

<a name="getKey"></a>

## getKey ⇒ <code>\*</code>
get unique identifier of target for internal indexing

**Returns**: <code>\*</code> - target identifier

| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |

<a name="extractKey"></a>

## extractKey ⇒ <code>\*</code>
get unique identifier of target from foreign model

**Returns**: <code>\*</code> - target identifier

| Param | Type | Description |
| --- | --- | --- |
| foreign | <code>Object</code> | Foreign model data |

<a name="getCondition"></a>

## getCondition ⇒ <code>\*</code>
get condition

**Returns**: <code>\*</code> - condition, can be scalar or object

| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |

<a name="getDataHandler"></a>

## getDataHandler ⇒ <code>function</code>
get foreign data handler

**Returns**: <code>function</code> - Callback handling data assignment

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | Options |
| options.targets | <code>Object</code> | targets index |
| options.targetField | <code>String</code> | field data to be saved into |

данные поля для сохранения в

| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |

возвращает уникальный ключ целевой модели от внешней модели

| options.isMultiple | <code>Boolean</code> | if one to many relation 

если один во многих отношениях

| options.sourceField | <code>String</code> | field to use of foreign model |

поле для использования внешней модели

| options.assignData | [<code>assignData</code>](#assignData) | Do model filling by itself, otherwise use `targetField` |

Пусть модель заполнится самостоятельно, в противном случае используйте `targetField` |

<a name="getAddingMethod"></a>

## getAddingMethod ⇒ <code>function</code>
get future `subscription.add()` method

запустите будущий `subscription.add()` метод

**Returns**: <code>function</code> - Callback handling data assignment

**Возвращается**: <code>function</code> - Callback обработка данных

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | Options |
| options.targets | <code>Object</code> | targets index |
| options.getKey | [<code>getKey</code>](#getKey) | Callback which returns unique key from target model (model.id by default) |

Callback , который возвращает уникальный ключ от целевой модели (model.id by default) |

| options.getCondition | [<code>getCondition</code>](#getCondition) | returns condition, using target model (model.id by default) |

возвращает условие, использующее целевую модель (model.id by default) |

| options.defaultValue | <code>\*</code> | Default value for field |

Значение по умолчанию для поля

| options.targetField | <code>String</code> | field data to be saved into |

поле данных для сохранения в 

| options.condition | <code>object</code> | DB Query condition, being prepared |
 
DB условие запроса, предварительно подготовленного

| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |

возвращает уникальный ключ целевой модели от внешней модели

| options.foreignField | <code>String</code> | If `getCondition` returns scalar values this field will be used for $in |

возвращает скалярные величины, которые это поле будет использовать для $in

| options.inner | <code>Array</code> | Internal array for condition storing |

внутренний массив для хранения условий

<a name="getStream"></a>

## getStream : <code>function</code>
get stream from model using condition


| Param | Description |
| --- | --- |
| source | Source model |
| condition | Query condition |
