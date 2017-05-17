####简介
ReactiveCocoa的灵感来自函数式响应式编程（FRP）。RAC并不采用随时可变的变量，而是用事件流（表现为Signal和SignalProducer）的方式来捕捉值的变化。

####解决的问题

问题：
在我们iOS开发过程中，经常会响应某些事件来处理某些业务逻辑，例如按钮的点击，上下拉刷新，网络请求，属性的变化（通过KVO）或者用户位置的变化（通过CoreLcation）。但是这些事件都用不同的方式来处理，比如action、delegate、KVO、callback等。代码散落各处。
>解决：
RAC统一以上的处理方法。把要处理的事情，和监听的事情的代码放在一起。不需要跳到对应的方法里。减少了类与类之间通信所需要的胶水代码。

####ReactiveCocoa核心方法bind

ReactiveCocoa操作的核心方法是bind（绑定），而且RAC中核心开发方式，也是绑定，之前的开发方式是赋值，而用RAC开发，应该把重心放在绑定，也就是可以在创建一个对象的时候，就绑定好以后想要做的事情，而不是等赋值之后在去做事情。

例如：把数据展示到控件上，之前都是重写控件的setModel方法，用RAC就可以在一开始创建控件的时候，就绑定好数据。


####ReactiveCocoa 主要由以下四大核心组件构成：

* 信号源：RACStream 及其子类；
* 订阅者：RACSubscriber 的实现类及其子类；
* 调度器：RACScheduler 及其子类；
* 清洁工：RACDisposable 及其子类。

其中，信号源又是最核心的部分，其他组件都是围绕它运作的。


####优势

OC处理

1. 通过实现UITextField的代理
在 - (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string; 
2. 方法中获取输入的文字，赋值给username属性和password属性
3. 再判断username和password是否符合要求
4. 再设置按钮的enabled属性


ReactiveCocoa处理

```
RAC(self.viewModel, userName) = self.tfUserName.rac_textSignal;
RAC(self.viewModel, password) = self.tfPassword.rac_textSignal;
RAC(self.btLogin, enabled) = [self.viewModel validSignal];
```

```
//viewModel
- (RACSignal *)validSignal {
    RACSignal *validSignal = [RACSignal combineLatest:@[_userNameSignal, _passwordSignal]
                                               reduce:^id(NSString *userName, NSString *password){
        // 要求用户名和密码大于6位数
        return @(userName.length >= 6 && password.length >= 6);
    }];
    return validSignal;
}

```





