### 向服务器端发送请求,接收响应

- 同步与异步
  - 同步:服务端处理客户端的请求时,客户端等待服务端的响应
  - 异步:客户端在等待服务端响应期间可执行其他操作

- 请求方法:

  ```js
  axios({
  	method: 'GET',
  	url: '请求路径'
  	data: '数据'
  }).then((result.data) => {
  	console.log(result.data);
  }).catch((err) => {
      aleter(err);
  })
  ```

  then():成功回调函数,在接收到响应后执行

  catch():失败回调函数,未接收到响应后执行

- 请求方法别名:

  ```js
  axios.get(url,data,config);
  ```

- 修饰符async与await

  - async:声明一个异步方法
  - await:等待异步任务执行