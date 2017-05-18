####Button

rac_signalForControlEvents
```
[[self.button rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {

        NSLog(@"按钮被点击了");
    }];
    
//或者:
self.button.rac_command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
    NSLog(@"RAC检测按钮点击2");
    return [RACSignal empty]; //返回空信号
}];    
    
```

####点击事件的信号和登录流程连接起来

```
//点击事件的信号和登录流程连接起来

[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
  flattenMap:^id(id x){
      return[self signInSignal];
  }]
 subscribeNext:^(NSNumber*signedIn){
     BOOL success =[signedIn boolValue];
     self.signInFailureText.hidden = success;
     if(success){
         [self performSegueWithIdentifier:@"signInSuccess" sender:self];
     }  
 }];

//用信号量封装一个方法

- (RACSignal *)signInSignal {
    
    return [RACSignal createSignal:^RACDisposable *(id subscriber){
        
        //登录方法
        [self.signInService
         signInWithUsername:self.usernameTextField.text
         password:self.passwordTextField.text
         complete:^(BOOL success){
             [subscriber sendNext:@(success)];
             [subscriber sendCompleted];
         }];
        return nil;
    }];
}

```
