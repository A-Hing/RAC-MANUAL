#####RACMulticastConnection

用于当一个信号，被多次订阅时，为了保证创建信号时，避免多次调用创建信号中的block，造成副作用，可以使用这个类处理。

需求：假设在一个信号中发送请求，每次订阅一次都会发送请求，这样就会导致多次请求。 解决：使用RACMulticastConnection就能解决.

```
// 发送请求，用一个信号内包装，不管有多少个订阅者，只想要发送一次请求
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
   // didSubscribe（block）中的代码都统称为副作用（Side Effects）。
   // 发送请求
   NSLog(@"发送请求");
   [subscriber sendNext:@1];
   return nil;
}]; 
// 订阅信号
[signal subscribeNext:^(id x) {
   NSLog(@"接收数据：%@",x);
}];
[signal subscribeNext:^(id x) {
   NSLog(@"接收数据：%@",x);
}];


```

```
2016-03-20 13:02:59.130 RACDemo[45388:5236769] 发送请求
2016-03-20 13:02:59.134 RACDemo[45388:5236769] 接收数据：1
2016-03-20 13:02:59.134 RACDemo[45388:5236769] 发送请求
2016-03-20 13:02:59.135 RACDemo[45388:5236769] 接收数据：1

```

运行结果，会执行两遍发送请求，也就是每次订阅都会发送一次请求。


```

```
// RACMulticastConnection:解决重复请求问题
// 1.创建信号
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
   NSLog(@"发送请求");
   [subscriber sendNext:@1];
   return nil;
}];
// 2.创建连接
RACMulticastConnection *connect = [signal publish];
// 3.订阅信号，
// 注意：订阅信号，也不能激活信号，只是保存订阅者到数组，必须通过连接,当调用连接，就会一次性调用所有订阅者的sendNext:
[connect.signal subscribeNext:^(id x) {
   NSLog(@"订阅者一信号");
}];
[connect.signal subscribeNext:^(id x) {
   NSLog(@"订阅者二信号");
}];
// 4.连接,激活信号
[connect connect];

```

```
2016-03-20 13:05:13.803 RACDemo[45398:5237395] 发送请求
2016-03-20 13:05:13.804 RACDemo[45398:5237395] 订阅者一信号
2016-03-20 13:05:13.804 RACDemo[45398:5237395] 订阅者二信号


```
