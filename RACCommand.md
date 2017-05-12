RAC中用于处理事件的类，可以把事件如何处理,事件中的数据如何传递，包装到这个类中，他可以很方便的监控事件的执行过程。

* 使用场景:监听按钮点击，网络请求。

实例

```
- (void)ah_RACCommand {
    // 1.创建命令类
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        
        // 调用execute，开始执行initWithSignalBlock
        NSLog(@"1、执行命令 %@", input);
        
        /*
         block有什么作用:描述下如何处理事件，网络请求:
             RACCommand必须返回信号，处理事件的时候，肯定会有数据产生，产生的数据就通过返回的信号发出。
             注意：数据传递完，最好调用sendCompleted，这时命令才执行完毕。
         */
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            
            //发送处理事件的信号，当信号被订阅的时候才会调用
            [subscriber sendNext:@"信号发出的内容"];
            [subscriber sendCompleted];
            
            return nil;
        }];
    }];
    
    // 强引用命令，不要被销毁，否则接收不到数据
    _command = command;
    
    // 3.如果想要订阅接收信号源的信号内容，必须保证命令类不会被销毁
    [command.executionSignals subscribeNext:^(id x) {
        // x -> 信号
        [x subscribeNext:^(id x) {
            
            NSLog(@"3、subscribeNext=%@",x);
        }];
    }];
    // 4.执行命令,开始调用signalBlock
    [command execute:@1];
    
    // RAC高级用法
    // switchToLatest:用于signal of signals，获取signal of signals发出的最新信号,也就是可以直接拿到RACCommand中的信号
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {
        
        NSLog(@"switchToLatest = %@",x);
    }];
    
    /*
         5.监听命令是否执行完毕，默认会来一次，可以直接跳过，skip表示跳过第一次信号。
         如果不跳过，在initWithSignalBlock之后会执行一次“执行完成”
     */
    [[command.executing skip:1] subscribeNext:^(id x) {
        if ([x boolValue] == YES) {
            // 正在执行
            NSLog(@"2、skip：正在执行");
            
        }else{
            // 执行完成
            NSLog(@"4、skip：执行完成");
        }
    }];
}
```