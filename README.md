# sequelize-better-transparent-cache

Better version of sequelize-transparent-cache.

* Raw Instance Support
* Count Support

Simple to use and universal cache layer for Sequelize.

* Abstract: does not depends on underlying database, or cache specific
* Transparent: objects returned from cache are regular Sequelize instances with all your methods
* Explicit: all calls to cache comes through `cache()` method
* Lightweight: zero additional dependencies

## Installation

Install sequelize-transparent-cache itself:

```npm install --save sequelize-better-transparent-cache```

Find and install appropriate adaptor for your cache system, see "Available adaptors" section below.
In this example we will use [memcached](https://www.npmjs.com/package/memcached)

```npm install --save sequelize-transparent-cache-memcached```

## Example usage

```javascript
onst Memcached = require('memcached')
const memcached = new Memcached('localhost:12345')
 
const MemcachedAdaptor = require('sequelize-transparent-cache-memcached')
const memcachedAdaptor = new MemcachedAdaptor({
  client: memcached,
  namespace: 'datas',
  lifetime: 15
});

const sequelizeCache = require('sequelize-better-transparent-cache')
const { withCache } = sequelizeCache(redisAdaptor)

const Sequelize = require('sequelize')
const sequelize = new Sequelize('database', 'user', 'password', {
  dialect: 'mysql',
  host: 'localhost',
  port: 3306
})

// Register and wrap your models:
// withCache() will add cache() methods to all models and instances in sequelize v4
const User = withCache(sequelize.import('./models/user'))

await sequelize.sync()

// Cache result of arbitrary query - requires cache key
await User.cache('active-users').findAll({
  where: {
    status: 'ACTIVE'
  }
})

// Create user in db and in cache
await User.cache().create({
  id: 1,
  name: 'Emre'
})

// Load user from cache
const user = await User.cache().findByPk(1);

// Update in db and cache
await user.cache().update({
  name: 'memrearal'
})

```

## Methods

Object returned by `cache()` call contains wrappers for **limited subset** of sequelize model or instance methods.

Instance:

* [`save()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#instance-method-save)
* [`update()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-update)
* [`destroy()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#instance-method-destroy)
* [`reload()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#instance-method-reload)

Model:
* Automatic cache methods - does not require cache key: `cache()`
  * [`create()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-create)
  * [`count()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-count)
  * [`findByPk()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-findByPk)
  * [`upsert()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-upsert)
  * [`insertOrUpdate()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-upsert)
* Manual cache methods - require cache key: `cache(key)`
  * [`findAll()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-findAll)
  * [`findOne()`](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-findOne)
  * `clear()` - remove data associated with key from cache

In addition, both objects will contain `client()` method to get cache adaptor.

## Available adaptors

* [memcached](https://www.npmjs.com/package/sequelize-transparent-cache-memcached)
* [memcache-plus](https://www.npmjs.com/package/sequelize-transparent-cache-memcache-plus)
* [ioredis](https://www.npmjs.com/package/sequelize-transparent-cache-ioredis)
* [variable](https://www.npmjs.com/package/sequelize-transparent-cache-variable)

You can easy write your own adaptor. Each adaptor must implement 3 methods:

* `get(path: Array<string>): Promise<object>`
* `set(path: Array<string>, value: object): Promise<void>`
* `del(path: Array<string>): Promise<void>`

Checkout existed adaptors for reference implementation.
