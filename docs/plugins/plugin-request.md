---
translateHelp: true
---

# @umijs/plugin-request


`@umijs/plugin-request` is based on [umi-request](https://github.com/umijs/umi-request) and [@umijs/hooks](https://github.com/umijs/hooks) `useRequest` provides a unified network request and error handling solution.

## How to enable

Enabled by default

## Introduction

Error handling is a problem that all projects will encounter. We have agreed on an interface format specification as follows:

```typescript
interface ErrorInfoStructure {
  success: boolean; // if request is success
  data?: any; // response data
  errorCode?: string; // code for errorType
  errorMessage?: string; // message display to user 
  showType?: number; // error display type： 0 silent; 1 message.warn; 2 message.error; 4 notification; 9 page
  traceId?: string; // Convenient for back-end Troubleshooting: unique request ID
  host?: string; // onvenient for backend Troubleshooting: host of current access server
}
```

If the backend interface specification is not satisfied, you can configure it by configuring `errorConfig.adaptor`. When `success` returns `false` we will follow `showType` and `errorMessage` for unified error notification and throw an exception.

The format of the exception thrown is:

```typescript
interface RequestError extends Error {
  data?: any; // Here is the raw data returned by the backend
  info?: ErrorInfoStructure;
}
```

In addition, you can determine whether it is an error thrown by `success` being `false` by checking whether `Error.name` is `BizError`.

## Configuration

### Build-time configuration

The currently supported build-time configuration is as follows:

```typescript
export default {
  request: {
    dataField: 'data',
  },
};
```

#### dataField

* Type: `string`

`dataField` corresponds to the `data` field in the unified format of the interface. For example, if the unified specification is `{success: boolean, data: any}`, then no configuration is required, so that when you consume through `useRequest`, a default data will be generated. The `formatResult` directly returns the data in `data` for ease of use. If your backend interface does not comply with this specification, you can configure `dataField` yourself. When configured as `''` (empty string), no processing is done.

### Runtime configuration

In `src/app.ts` you can configure some runtime configuration items to achieve some custom requirements. The sample configuration is as follows:

```typescript
import { RequestConfig } from 'umi';

export const request: RequestConfig = {
  timeout: 1000,
  errorConfig: {},
  middlewares: [],
  requestInterceptors: [],
  responseInterceptors: [],
};
```

This configuration returns an object. Except for `errorConfig` and` middlewares`, all other configurations are global configurations that directly pass through [umi-request](https://github.com/umijs/umi-request).

#### errorConfig.adaptor

When the back-end interface does not meet this specification, you need to convert the back-end interface data to this format through this configuration. The following configuration is only used for error handling and does not affect the final data format passed to the page.

```typescript
import { RequestConfig } from 'umi';

export const request: RequestConfig = {
  errorConfig: {
    adaptor: (resData) => {
      return {
        ...resData,
        success: resData.ok,
        errorMessage: resData.message,
      };
    },
  },
};
```

#### errorConfig.errorPage

When `showType` is `9`, it will jump to `/exception` page by default, you can modify the path through this configuration.

#### middlewares

umi-request provides [middleware mechanism](https://github.com/umijs/umi-request#middleware), which was introduced through `request.use (middleware)` before, and can now be passed through `request.middlewares` To configure.

```typescript
export const request = {
  middlewares: [
    async function middlewareA(ctx, next) {
      console.log('A before');
      await next();
      console.log('A after');
    },
    async function middlewareB(ctx, next) {
      console.log('B before');
      await next();
      console.log('B after');
    }
  ]
}
```

#### requestInterceptors

This configuration receives an array, and each item of the array is a request interceptor. Equivalent to `request.interceptors.request.use ()` of umi-request. For details, see [Interceptor Document](https://github.com/umijs/umi-request#interceptor) of umi-request.

#### responseInterceptors

This configuration receives an array, and each item in the array is a response interceptor. Equivalent to `request.interceptors.response.use ()` of umi-request. For details, see [Interceptor Document](https://github.com/umijs/umi-request#interceptor) of umi-request.

## API

### useRequest

The plug-in has built-in [@umijs/use-request](https://hooks.umijs.org/zh-CN/async), you can use the Hook in the component to easily and conveniently consume data. Examples are as follows:

```typescript
import { useRequest } from 'umi';

export default () => {
  const { data, error, loading } = useRequest(() => {
    return services.getUserList('/api/test');
  });
  if (loading) {
    return <div>loading...</div>;
  }
  if (error) {
    return <div>{error.message}</div>;
  }
  return <div>{data.name}</div>;
};
```

For more configuration, you can refer to the documentation of [@umijs/use-request](https://hooks.umijs.org/zh-CN/async). Compared with `@umijs/use-request` itself, `import {useRequest} from 'umi';` There are two differences as follows:

- According to the interface request specification, built-in `formatResult: res => res?.data` allows you to use the data more conveniently. Of course, you can also configure `formatResult` to override the built-in logic.
- Unified error handling logic according to interface error specifications.

You can also check the Zhihu column [《useRequest- Ant Central Request for Hooks》](https://zhuanlan.zhihu.com/p/106796295) to understand useRequest.

### request

With `import {request} from 'umi';` you can use the built-in request method. `request` accepts two parameters, the first parameter is `url` and the second parameter is the requested options. Refer to [umi-request](https://github.com/umijs/umi-request) for the specific format of `options`.

Most of the usage of `request` is equivalent to` umi-request`. One difference is that `options` extends a configuration `skipErrorHandler`. This configuration is `true` which will skip the default error handling and is used in the project part. Special interface.

Examples are as follows:

```typescript
request('/api/user', {
  params: {
    name: 1,
  },
  skipErrorHandler: true,
})
```

### RequestConfig

This is a TypeScript definition, which can help you better configure the runtime configuration.

```typescript
import { RequestConfig } from 'umi';

export const request: RequestConfig = {};
```

### ErrorShowType

`import { ErrorShowType } from 'umi';` This is a TypeScript definition that defines the types of showType supported in the agreed format.

```typescript
export enum ErrorShowType {
  SILENT = 0, // 不提示错误
  WARN_MESSAGE = 1, // 警告信息提示
  ERROR_MESSAGE = 2, // 错误信息提示
  NOTIFICATION = 4, // 通知提示
  REDIRECT = 9, // 页面跳转
}
```
