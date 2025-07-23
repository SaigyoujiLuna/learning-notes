axios网络请求的返回值是个Promise，**对于无响应，或是返回码非2xx的请求，axios回传为Reject**，catch到后：

* 若存在response，则是返回码错误
* 若存在request，则是无响应
* 若两者都没有，则是请求都未被发起，可能是URL错误，或者断网等情况。

```typescript
axios.get('/user/12345')  
.catch(function (error) {  
if (error.response) {  
// 请求成功发出且服务器也响应了状态码，但状态代码超出了 2xx 的范围  
console.log(error.response.data);  
console.log(error.response.status);  
console.log(error.response.headers);  
} else if (error.request) {  
// 请求已经成功发起，但没有收到响应  
// `error.request` 在浏览器中是 XMLHttpRequest 的实例，  
// 而在node.js中是 http.ClientRequest 的实例  
console.log(error.request);  
} else {  
// 发送请求时出了点问题  
console.log('Error', error.message);  
}  
console.log(error.config);  
});
```