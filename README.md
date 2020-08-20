<h1 align="center">Welcome to request-cache 👋</h1>
<p>
  <a href="https://www.npmjs.com/package/request-cache" target="_blank">
    <img alt="Version" src="https://img.shields.io/npm/v/request-cache.svg">
  </a>
  <a href="#" target="_blank">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" />
  </a>
</p>

## Intro

在本地开发的时候，有时候后端的接口非常慢，我们刷新页面调试的时候浪费在等待接口上的时间太长。但很多时候其实我们只是想要一份可用的数据来调试 UI 界面，并不是很关心数据的实时性。

这个库就是用来解决上述问题的，`request-dev-cache` 会在 fetch 获取到结果的时候，基于你自己定义的生成 key 的策略（默认是使用 `url + JSON.stringify(body)`）来把结果保存到本地存储中，下次再请求相同的 key 的数据就可以不再经过后端，这在本地开发调试的时候有时非常有效。

使用 [localforage](https://github.com/localForage/localForage) 这个库作为底层的存储，它默认采用 `IndexedDB` 进行本地存储。

## Install

```sh
npm install request-dev-cache -S
npm install localforage -S
```

## Usage

### Options

```ts
interface CacheFetchOptions {
  /**
   * 生成缓存key的策略，默认策略是直接拼接 url + stringify(body)
   */
  generateKey?: (url: RequestInfo, body: object) => string;
  /**
   * 传入 url 和 fetch 选项 判断是否需要缓存
   */
  shouldHandleRequest?(url: RequestInfo, requestInit?: RequestInit): boolean;
  /**
   * 传入 response 响应对象 判断是否需要缓存
   */
  shouldCacheResult?(response: Response): Promise<boolean>;
}
```

```js
import { startCache } from 'request-cache';
import { omit } from 'lodash'

startCache({
  generateKey(url, body) {
    // 由于默认情况下 拼接 key 的时候会把 body 里的所有属性都带上 有些变化特别频繁但是我们又不是很关心的 key （比如当前时间戳）就会导致缓存失效
    return `${url}-${JSON.stringify(omit(body, ['currentTime']))}`;
  },
  shouldHandleRequest(url: string) {
    // 可以规定 /api 开头的请求才缓存
    return url.startsWith('/api');
  },
  shouldCacheResult: async response => {
    // 过滤掉不想缓存的结果
    const result = await response.json();
    return (
      response.headers.get('content-type') === 'application/json' &&
      response.status === 200
    );
  },
});
```

### API

#### window.cleanAllRequestDevCaches

在控制台调用，清除所有的缓存。

#### window.cleanRequestDevCache

在控制台调用，根据输入的值模糊查询存储的 key 清除匹配到的所有缓存。

## Run tests

```sh
npm run test
```

## Author

👤 **ssh**

- Website: https://ssh-blog.now.sh
- Github: [@sl1673495](https://github.com/sl1673495)

## Show your support

Give a ⭐️ if this project helped you!

---

_This README was generated with ❤️ by [readme-md-generator](https://github.com/kefranabg/readme-md-generator)_
