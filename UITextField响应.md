
####UITextField

```
    [[self.verifyCodeView.verifyCodeField rac_signalForControlEvents:UIControlEventEditingDidEndOnExit] subscribeNext:^(id x) {
        NSLog(@"%s", __func__);
    }];
    
```