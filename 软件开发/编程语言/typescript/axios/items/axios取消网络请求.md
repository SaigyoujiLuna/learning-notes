axios使用AbortController进行网络请求取消，通过将controller绑定到对应请求上后，对Controller调用请求方法来取消请求
```
const controller = new AbortController();  
  
axios.get('/foo/bar', {  
signal: controller.signal  
}).then(function(response) {  
//...  
});  
// 取消请求  
controller.abort()
```