####简介
RAC中用于处理事件的类，可以把事件如何处理,事件中的数据如何传递，包装到这个类中，他可以很方便的监控事件的执行过程。

* 使用场景:监听按钮点击，网络请求。
* 使用步骤:
    1. 创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock
    2. 在signalBlock中，创建RACSignal，并且作为signalBlock的返回值
    3. 执行命令 - (RACSignal *)execute:(id)input


* RACCommand使用步骤:

1. 创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock
2. 在signalBlock中，创建RACSignal，并且作为signalBlock的返回值
3. 执行命令 - (RACSignal *)execute:(id)input

* RACCommand使用注意:
signalBlock必须要返回一个信号，不能传nil。
如果不想要传递信号，直接创建空的信号[RACSignal empty]。
RACCommand中信号如果数据传递完，必须调用[subscriber sendCompleted]，这时命令才会执行完毕，否则永远处于执行中。
RACCommand需要被强引用，否则接收不到RACCommand中的信号，因此RACCommand中的信号是延迟发送的。

* RACCommand设计思想：内部signalBlock为什么要返回一个信号，这个信号有什么用。
在RAC开发中，通常会把网络请求封装到RACCommand，直接执行某个RACCommand就能发送请求。
当RACCommand内部请求到数据的时候，需要把请求的数据传递给外界，这时候就需要通过``signalBlock返回的信号传递了。

* 如何拿到RACCommand中返回信号发出的数据。
RACCommand有个执行信号源executionSignals，这个是``signal of signals(信号的信号),意思是信号发出的数据是信号，不是普通的类型。
订阅``executionSignals就能拿到RACCommand中返回的信号，然后订阅signalBlock返回的信号，就能获取发出的值。

#####实例
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