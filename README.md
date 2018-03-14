# axios-extensions

[![npm version](https://img.shields.io/npm/v/axios-extensions.svg?style=flat-square)](https://www.npmjs.com/package/axios-extensions)
[![coverage](https://img.shields.io/codecov/c/github/kuitos/axios-extensions.svg?style=flat-square)](https://codecov.io/gh/kuitos/axios-extensions)
[![npm downloads](https://img.shields.io/npm/dt/axios-extensions.svg?style=flat-square)](https://www.npmjs.com/package/axios-extensions)
[![Build Status](https://img.shields.io/travis/kuitos/axios-extensions.svg?style=flat-square)](https://travis-ci.org/kuitos/axios-extensions)

## Install
```bash
npm i axios-extensions -S
```
or
```bash
yarn add axios-extensions
```

## Usage

```javascript
import axios from 'axios';
import { cacheAdapterEnhancer, throttleAdapterEnhancer } from 'axios-extensions';

// enhance the original axios adapter with throttle and cache enhancer 
const http = axios.create({
	baseURL: '/',
	headers: { 'Cache-Control': 'no-cache' },
	adapter: throttleAdapterEnhancer(cacheAdapterEnhancer(axios.defaults.adapter, true).adapter)
});
```

### Enable Logging

It is highly recommended to enable the request logging recorder in development environment(it is disabled in default).

#### browser (webpack)
```js
new webpack.DefinePlugin({
  'process.env.LOGGER_LEVEL': JSON.stringify('info')
})
```
#### node
```json
// package.json
"scripts": {
	"start": "cross-env LOGGER_LEVEL=info node server.js"
}
```

## API

### cacheAdapterEnhancer(adapter, cacheEnabledByDefault = false, enableCacheFlag = 'cache', cacheAge = FIVE_MINUTES) : enhancedAdapter
makes axios cacheable

#### basic usage

```javascript
import axios from 'axios';
import { cacheAdapterEnhancer } from 'axios-extensions';

const http = axios.create({
	baseURL: '/',
	headers: { 'Cache-Control': 'no-cache' },
	// cache will be enabled by default
	adapter: cacheAdapterEnhancer(axios.defaults.adapter, true).adapter
});

http.get('/users'); // make real http request
http.get('/users'); // use the response from the cache of previous request, without real http request made
http.get('/users', { cache: false }); // disable cache manually and the the real http request invoked   

```

#### custom cache flag

```javascript
const http = axios.create({
	baseURL: '/',
	headers: { 'Cache-Control': 'no-cache' },
	// disable the default cache and set the cache flag
	adapter: cacheAdapterEnhancer(axios.defaults.adapter, false, 'useCache').adapter
});

http.get('/users', ); // default cache was disabled and then the real http request invoked 
http.get('/users', { useCache: true }); // make the request cacheable(real http request made due to first request invoke)
http.get('/users', { useCache: true }); // use the response cache from previous request
```

#### explicitly manipulate the cache ([LRUCache](https://github.com/isaacs/node-lru-cache))

```javascript
const cacheAdapter = cacheAdapterEnhancer(axios.defaults.adapter, true)
const http = axios.create({
	baseURL: '/',
	headers: { 'Cache-Control': 'no-cache' },
	// disable the default cache and set the cache flag
	adapter:cacheAdapter.adapter
});

http.get('/users'); // response is cached
http.get('/users'); // cached response returned
cachedAdapter.cache.reset(); // explicitly clear the cache
http.get('/users'); // request is to made again

```

### throttleAdapterEnhancer(adapter, threshold = 1000, cacheCapacity = 10) : enhancedAdapter
throttle requests most once per threshold milliseconds

```js
import axios from 'axios';
import { throttleAdapterEnhancer } from 'axios-extensions';

const http = axios.create({
	baseURL: '/',
	headers: { 'Cache-Control': 'no-cache' },
	adapter: throttleAdapterEnhancer(axios.defaults.adapter, 2 * 1000)
});

http.get('/users'); // make real http request
http.get('/users'); // responsed from the cache
http.get('/users'); // responsed from the cache

setTimeout(() => {
	http.get('/users'); // after 2s, the real request makes again
}, 2 * 1000);
```
