## Fetch

`polyfill` 

意思：用于实现浏览器并不支持的原生api的代码。一个Polyfill是抹平新老浏览器和标准原生API之间的差距的一种封装，而不是实现自己的API。

### 1.语法

```Promise<Response> fetch(input[, init]);```

input：一般是请求url

init(可选): 一个配置项对象，包括所有对请求的设置。可选的参数有：

> - `method`: 请求使用的方法，如 `GET、``POST。`
> - `headers`: 请求的头信息，形式为 [`Headers`](https://developer.mozilla.org/zh-CN/docs/Web/API/Headers) 的对象或包含 [`ByteString`](https://developer.mozilla.org/zh-CN/docs/Web/API/ByteString) 值的对象字面量。
> - `body`: 请求的 body 信息：可能是一个 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)、[`BufferSource`](https://developer.mozilla.org/zh-CN/docs/Web/API/BufferSource)、[`FormData`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)、[`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 或者 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 对象。注意 GET 或 HEAD 方法的请求不能包含 body 信息。
> - `mode`: 请求的模式，如 `cors、` `no-cors 或者` `same-origin。`
> - `credentials`: 请求的 credentials，如 `omit、``same-origin 或者` `include`。为了在当前域名内自动发送 cookie ， 必须提供这个选项， 从 Chrome 50 开始， 这个属性也可以接受 [`FederatedCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/FederatedCredential) 实例或是一个 [`PasswordCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/PasswordCredential) 实例。
> - `cache`:  请求的 cache 模式: `default`、 `no-store`、 `reload` 、 `no-cache `、 `force-cache `或者 `only-if-cached` 。
> - `redirect`: 可用的 redirect 模式: `follow` (自动重定向), `error` (如果产生重定向将自动终止并且抛出一个错误）, 或者 `manual` (手动处理重定向). 在Chrome中，Chrome 47之前的默认值是 follow，从 Chrome 47开始是 manual。
> - `referrer`: 一个 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 可以是 `no-referrer、``client`或一个 URL。默认是 `client。`
> - `referrerPolicy`: 指定了HTTP头部referer字段的值。可能为以下值之一： `no-referrer、` `no-referrer-when-downgrade、` `origin、` `origin-when-cross-origin、` `unsafe-url 。`
> - `integrity`: 包括请求的  [subresource integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) 值 （ 例如： `sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=）。`

### 2.使用

#### 发送表单

```js
var form = document.querySelector('form')

fetch('/users', {
  method: 'POST',
  body: new FormData(form)
})
```

#### 发送json格式

```js
fetch('/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Hubot',
    login: 'hubot',
  })
})
```

#### 处理response

```js
fetch('/users.json')
  .then(function(response) {
    return response.json() //转为json对象
  }).then(function(json) {
    console.log('parsed json', json)
  }).catch(function(ex) {
    console.log('parsing failed', ex)
  })
```



### 3.fetch处理状态值

对于4xx或者5xx的错误，fetch不会认为是错误，而会正常解析(resolve)，但是会将 resolve 的返回值的 ok 属性设置为 false。fetch对于网络故障或者请求被拦截，才会reject。

如果你想正确地处理4xx或5xx这类状态值，你可以自己做处理：

```js
function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response
  } else {
    var error = new Error(response.statusText)
    error.response = response
    throw error
  }
}

function parseJSON(response) {
  return response.json()
}

fetch('/users')
  .then(checkStatus)
  .then(parseJSON)
  .then(function(data) {
    console.log('request succeeded with JSON response', data)
  }).catch(function(error) {
    console.log('request failed', error)
  })

```



### 4.fetch 和 cookie

1. 接收cookie

fetch可以接收cookie

2. 发送cookie

fetch支持发送cookie，但是默认只在同一个域有效，即credentials: 'same-origin'

> credentials: 请求的 credentials，如 omit、same-origin 或者 include。为了在当前域名内自动发送 cookie ， 必须提供这个选项

为了能够更好的处理跨域请求，在请求头加入配置credentials: 'include'

```javascript
fetch('https://example.com:1234/users', {
  credentials: 'include'
})
```

