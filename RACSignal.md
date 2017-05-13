####RACSignal

信号类，一般表示将来有数据传递，只要有数据改变，信号内部接收到数据，就会马上发出数据。注意是，数据发出，并不是信号类发出。

- 信号类(RACSignal)，只是表示当数据改变时，信号内部会发出数据，它本身不具备发送信号的能力，而是交给内部一个订阅者去发出。

- 默认一个信号都是冷信号，也就是值改变了，也不会触发，只有订阅了这个信号，这个信号才会变为热信号，值改变了才会触发。

- 调用信号RACSignal的subscribeNext就能订阅

####RACSignal使用步骤：

1. 创建信号 + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe
2. 发送信号 - (void)sendNext:(id)value

3. 订阅信号，才会激活信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock

####RACSignal底层实现：

1. 创建子类信号RACDynamicSignal，首先把didSubscribe这个block保存到信号RACDynamicSignal中，但是还不会触发(冷信号)。

2. 当信号被订阅，也就是调用signal的subscribeNext:nextBlock。subscribeNext内部会创建订阅者subscriber，并且把nextBlock保存到订阅者subscriber中，此时为热信号。

3. subscribeNext内部会调用RACDynamicSignal的didSubscribe这个block。通常也就是在RACDynamicSignal的didSubscribe中调用[subscriber sendNext:@1]。

4. sendNext底层其实就是执行订阅者subscriber的nextBlock。

5. 标记信号发送完成或者取消订阅。

6. 执行RACDisposable的disposeBlock中的代码。



####代码

```
    // 1.创建信号
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        // block调用时刻：每当有订阅者订阅信号，就会调用block。
        
        // 2.发送信号
        [subscriber sendNext:@1];
        
        // 如果不在发送，最好发送信号完成，内部会自动调用[RACDisposable disposable]取消订阅信号。
        //        [subscriber sendCompleted];
        
        return [RACDisposable disposableWithBlock:^{
            
            // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。
            // 执行完Block后，当前信号就不在被订阅了。
            
            NSLog(@"信号被销毁");
        }];
    }];
    
    // 3.订阅信号,才会激活信号.
    [siganl subscribeNext:^(id x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"接收到数据:%@",x);
    }];

```

```
2017-05-13 10:51:55.476 DesignPattern[65036:1842467] 接收到数据:1
2017-05-13 10:51:55.476 DesignPattern[65036:1842467] 信号被销毁

```

