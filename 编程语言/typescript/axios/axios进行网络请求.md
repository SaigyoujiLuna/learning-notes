引入axios依赖后直接调用以下方法即可：

```typescript
axios.request(config)

axios.get(url[, config])

axios.delete(url[, config])

axios.head(url[, config])

axios.options(url[, config])

axios.post(url[, data[, config]])

axios.put(url[, data[, config]])

axios.patch(url[, data[, config]])

axios.postForm(url[, data[, config]])

axios.putForm(url[, data[, config]])

axios.patchForm(url[, data[, config]])
```

config对象定义链接:[Request Config](https://axios-http.com/docs/req_config)

这些网络请求方法返回值都是Promise<AxiosResponse<T>>类型

AxiosResponse对象定义:[Response Schema](https://axios-http.com/docs/res_schema)

