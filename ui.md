# UI

```Objc
NSURL *url = [NSURL URLWithString:urlStr];
    
    //2.创建请求对象
    NSURLRequest *request = [[NSURLRequest alloc]initWithURL:url];
    
    //3.发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
       
        if (connectionError) {
            NSLog(@"请求失败--%@",connectionError);
            return ;
        }

```

