axios提供了一个默认的实例供使用（人话：单例）
通过axios.default即可访问
```typescript
axios.defaults.baseURL = 'https://api.example.com'; axios.defaults.headers.common['Authorization'] = AUTH_TOKEN; axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
//...
```
同时，也可以另外创建一个实例出来
```typescript
// Set config defaults when creating the instance 
const instance = axios.create({ baseURL: 'https://api.example.com' }); 

// Alter defaults after instance has been created 
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```
