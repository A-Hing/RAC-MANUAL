

#####textField输入有变化时，产生next事件

```
[self.usernameTextField.rac_textSignal subscribeNext:^(id x){  
  NSLog(@"%@", x);  
}]; 
```

#####超过3个字符长度的用户名，输出next事件

```

[[self.usernameTextField.rac_textSignal  
  filter:^BOOL(NSString*text){  
    return text.length > 3;  
  }]  
  subscribeNext:^(id x){  
    NSLog(@"%@", x);  
  }]; 
  
  
  或者：
  
RACSignal *usernameSourceSignal =  
    self.usernameTextField.rac_textSignal;  
    
RACSignal *filteredUsername =[usernameSourceSignal  
  filter:^BOOL(id value){  
    NSString*text = value;  
    return text.length > 3;  
  }];  
    
[filteredUsername subscribeNext:^(id x){  
  NSLog(@"%@", x);  
}]; 
  

```

#####UITextField监听键盘的Return键

```
    [[self.verifyCodeView.verifyCodeField rac_signalForControlEvents:UIControlEventEditingDidEndOnExit] subscribeNext:^(id x) {
        NSLog(@"%s", __func__);
    }];
    
```