

####textField输入有变化时，产生next事件

```
[self.usernameTextField.rac_textSignal subscribeNext:^(id x){  
  NSLog(@"%@", x);  
}]; 
```

####UITextField

```
    [[self.verifyCodeView.verifyCodeField rac_signalForControlEvents:UIControlEventEditingDidEndOnExit] subscribeNext:^(id x) {
        NSLog(@"%s", __func__);
    }];
    
```