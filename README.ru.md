# subscribe-for-data

Fast implementation of multiple related models properties fetching & mixing to your model.

Быстрая реализация нескольких связанных извлечений свойств моделей и смешивание к вашей модели.

##   

 * DB agnostic. 
 * flexible query condition
 * one DB query per subscription.
 * all queries running parallel, starting at same moment, you choose then 
 * No loops, stream based.
 * zero dependencies
 
 * DB агностика.
 * гибкое условие запроса
 * один DB запрос на подписку. 
 * все запросы идут параллельно, начинаясь в одно и то же время, которое вы выьерете потом
 * Нет петель, основанных на потоке.
 * нулевая зависимость
 
## Workflow description 

## Workflow описание

You create `subscription` around your **related model** with `makeSubscription(model, options)`

Вы создаёте `subscription` вокруг вашей **родственной модели** с `makeSubscription(model, options)` 

You can create any count of subscriptions you need.
Вы можете создать любое колличество подписок, которое вам необходимо.

Then you can to `subscription.add(target)` **target objects** you want to mix in 
properties from **related model** data. 

Затем вы можете `subscription.add(target)` **целевые объекты**, которые вы хотите смешать в 
свойства из **родственной модели** данных.

After you've added all needed **targets** to all `subscriptions` you can anytime run 
`fillSubscriptions()` 

После того, как вы добавили всё, что необходимо, **цели** ко всем `subscriptions` вы можете в любое время
перейти в `fillSubscriptions()` 

`fillSubscriptions()` assigns data as it goes via **stream** with auto **parallelization** 
if multiple `subscriptions` created. One query per `subscription` is executed.
 
`fillSubscriptions()` назначит данные, так как это идёт через **поток** с авто **распараллеливанием**
если умножить созданные `subscriptions`. Один запрос на `subscription` выполнен.
 
It generates mongo condition. If you return from 
`options.getCondition(target)` *scalar value* then is generated `$in` query. I   
to query your source, 

Он производит монго состоятие. Если вы вернётесь из `options.getCondition(target)`
*скалярное значение* затем генерируется в `$in` запрос.
чтобы запросить ваш источник,

Mongo query generation is just default behavior, you can alter it as you want.

Генерация монго запросов происходит по умолчанию, вы можете изменить это так, как захотите.


## Installation

## Установка

```shell script
npm i subscribe-for-data subscribe-for-data-from-mongoose
```

## Import

##  Импорт 


```javascript
const mongoosePlugin = require('subscribe-for-data-from-mongoose');
const { 
  makeSubscription, fillSubscriptions 
} = require('subscribe-for-data').use(mongoosePlugin);
```

## Example

## Пример

By default it works with mongoose. This behavior can be easily overriden by 
setting custom `getStream` option callback.

По умолчанию это работает с мангуста. Этот режим может быть легко отменён 
установкой пользовательской  `getStream` настройки отозвать.

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
  await RootModel.find({}).cursor().eachAsync((root) => {
    [relatedSubscription, anotherRelatedSubscription]
      .forEach(subscription => subscription.add(root)); // subscribed
    roots.push(root);
  });
  await fillSubscriptions(); // 2 DB queries executed in parallel, no loops then
  console.log(roots[0]);
})();
```

Expected output:

Ожидаемый результат:

```json
{
  "_id": "000000",
  "position": 42,
  "someFields": [2, 5, 8, 5],
  "otherTypes": [23, 42, 78]
}
```

# API Reference

# API Ссылка

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


| Param | Type | Description |
| --- | --- | --- |
| source | <code>Object</code> | Source model |
| options | <code>Object</code> | Options |
| options.targetField | <code>String</code> | field data to be saved into (optional) |
| options.baseCondition | <code>Object</code> | Base condition |
| options.defaultValue | <code>\*</code> | Default value for field |
| options.getKey | [<code>getKey</code>](#getKey) | Callback which returns unique key from target model (model.id by default) |
| options.getCondition | [<code>getCondition</code>](#getCondition) | returns condition, using target model (model.id by default) |
| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |
| options.isMultiple | <code>Boolean</code> | if one to many relation |
| options.useEachAsync | <code>Boolean</code> | only for mongoose cursor |
| options.parallel | <code>Number</code> | parallel parameter for `eachAsync` if `useEachAsync` is `true` |
| options.foreignField | <code>String</code> | If `getCondition` returns scalar values this field will be used for $in |
| options.sourceField | <code>String</code> | field to use of foreign model |
| options.assignData | [<code>assignData</code>](#assignData) | Do model filling by itself, otherwise use `targetField` |
| options.getStream | [<code>getStream</code>](#getStream) | returns stream from source and condition (using mongoose model by default) |
| options.getDataHandler | [<code>getDataHandler</code>](#getDataHandler) | Get data handler for processing related models |
| options.getAddingMethod | [<code>getAddingMethod</code>](#getAddingMethod) | Get `add()` method of future `subscription` |

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

Назначает данные от иностранной модели к цели


| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |
| foreign | <code>Object</code> | foreign model |

<a name="getKey"></a>

## getKey ⇒ <code>\*</code>
get unique identifier of target for internal indexing

даёт уникальный идентификатор цели для внутренней индексации

**Returns**: <code>\*</code> - target identifier

**возвращает**: <code>\*</code> - целевой идентификатор

| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |

<a name="extractKey"></a>

## extractKey ⇒ <code>\*</code>
get unique identifier of target from foreign model

даёт уникальный идентификатор цели из внешней модели

**Returns**: <code>\*</code> - target identifier

**возвращает**: <code>\*</code> - целевой идентификатор

| Param | Type | Description |
| --- | --- | --- |
| foreign | <code>Object</code> | Foreign model data |

<a name="getCondition"></a>

## getCondition ⇒ <code>\*</code>
get condition

получить условие

**Returns**: <code>\*</code> - condition, can be scalar or object

**Возвращает**: <code>\*</code> - условие, которое может быть скалярным или обЪектом

| Param | Type | Description |
| --- | --- | --- |
| target | <code>Object</code> | your target model |

<a name="getDataHandler"></a>

## getDataHandler ⇒ <code>function</code>
get foreign data handler

получить обработчик внешних данных

**Returns**: <code>function</code> - Callback handling data assignment

**Возвращает**: <code>function</code> - Обратный вызов назначенных данных

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | Options |
| options.targets | <code>Object</code> | targets index |
| options.targetField | <code>String</code> | field data to be saved into |
| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |
| options.isMultiple | <code>Boolean</code> | if one to many relation |
| options.sourceField | <code>String</code> | field to use of foreign model |
| options.assignData | [<code>assignData</code>](#assignData) | Do model filling by itself, otherwise use `targetField` |

<a name="getAddingMethod"></a>

## getAddingMethod ⇒ <code>function</code>
get future `subscription.add()` method

**Returns**: <code>function</code> - Callback handling data assignment

**Возвращает**: <code>function</code> - Обратный вызов назначенных данных

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | Options |
| options.targets | <code>Object</code> | targets index |
| options.getKey | [<code>getKey</code>](#getKey) | Callback which returns unique key from target model (model.id by default) |
| options.getCondition | [<code>getCondition</code>](#getCondition) | returns condition, using target model (model.id by default) |
| options.defaultValue | <code>\*</code> | Default value for field |
| options.targetField | <code>String</code> | field data to be saved into |
| options.condition | <code>object</code> | DB Query condition, being prepared |
| options.extractKey | [<code>extractKey</code>](#extractKey) | returns unique key of target model from foreign model |
| options.foreignField | <code>String</code> | If `getCondition` returns scalar values this field will be used for $in |
| options.inner | <code>Array</code> | Internal array for condition storing |

<a name="getStream"></a>

## getStream : <code>function</code>
get stream from model using condition

получить поток из модели, используя условие


| Param | Description |
| --- | --- |
| source | Source model |
| condition | Query condition |
