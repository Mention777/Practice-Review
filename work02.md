##网络day01
####理论部分
1.GET请求和POST请求总结:


>**GET**:在请求URL后面以?的形式跟上发给服务器的参数，多个参数之间用&隔开

>例:http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON 
- "协议头+主机地址+端口号+接口名称+?+参数1&参数&参数3"

>使用场景:当只需要进行简单索数据,建议使用GET

*********


>**POST**:发给服务器的参数全部放在**请求体**中,理论上,POST传递的数据量没有限制（具体还得看服务器的处理能力）

>例:http://120.25.226.186:32812/login
- "协议头+主机地址+端口号+接口名称"

>使用场景:

>1.需要传递大量的数据(数据上传)只能使用POST

>2.GET的安全性比POST差,故当需要传递安全程度需要高的数据时,使用POST

>3.如果需要增加\删除\修改数据时,建议使用POST

********

2.发送GET请求的步骤

>1)确定请求路径("后面跟参数")

>2)创建请求对象(默认就是GET请求)

>3)使用NSURLConnection发送请求

>4)解析服务器返回的数据

3.发送POST请求的步骤
>1)确定请求路径(后面不跟参数)

>2)创建"可变的"请求对象

>3)修改请求对象的请求方法为POST

>4)设置请求体(把参数转换为二进制数据设置)

>5)使用NSURLConnection发送请求

>6)解析服务器返回的数据

**********

4.http请求通信过程

>（1）请求:【包括请求头+请求体·非必选】
	
>（2）响应:【响应头+响应体】

>（3）通信过程:
				
>a.发送请求的时候把请求头和请求体（请求体是非必须的）包装成一个请求对象

>b.服务器端对请求进行响应，在响应信息中包含响应头和响应体，响应信息是对服务器端的描述，具体的信息放在响应体中传递给客户端




####代码部分:
######一.NSURLConnection的实现
- 1.NSURLConnection发送GET请求(同步请求)
        
```objc

    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
    //2.创建请求对象
    NSURLRequest *request = [[NSURLRequest alloc]initWithURL:url];
    
    //3.使用NSURLConnection来发送网络请求(同步请求-阻塞)
    NSURLResponse *response = nil;
    NSError *error = nil;//通常定义NSError不使用alloc init的方式
    /*
     第一个参数:sendSynchronousRequest-->请求对象
     第二个参数:returningResponse-->响应头信息 传地址
     第三个参数:error-->错误信息
     返回值:响应体信息(解析)
     */
    NSData *data =  [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    
    NSLog(@"%@---%@",response,error);
    
    //4.解析服务器返回的数据
    NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
```

- 2.NSURLConnection发送GET请求(异步请求)

```objc

    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
    //2.创建请求对象
    NSURLRequest *request = [[NSURLRequest alloc]initWithURL:url];
    
    //3.发送异步请求
    /*
     第一个参数:请求对象
     第二个参数:队列(决定代码块在哪个线程中调用)
     第三个参数:completionHandler 网络请求完成(成功|失败)之后的回调
              NSURLResponse:响应头
              data:响应体
              connectionError:错误信息
     */
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
       
        //先判断请求是否成功
        if (connectionError) {
            NSLog(@"请求失败---%@",connectionError);
            return ;
        }
        
        //4.解析数据
        NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
        NSLog(@"%@",response);
    }];
    
    NSLog(@"--end---");//因为是异步的关系,导致这句话到时可能优先打印

```

- 3.NSURLConnection发送GET请求(代理方法)

```objc

    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
    
    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    
    //3.设置代理,发送网络请求
    //第一种设置代理的方法
    [[NSURLConnection alloc]initWithRequest:request delegate:self];
    
    //第二种设置代理的方法
    //startImmediately是否马上发送网络请求 YES(马上发送)
    NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
    //若为NO,则需要调用发送请求的方法
    //发送网络请求
    [connect start];
    
    
//并实现下列代理方法:

//接收到服务器的响应
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response

//当接收到服务器返回的数据的时候调用(可能会被调用多次)
//P.S.定义一个成员属性保存已经获取获取的Data,并将获得的所有data拼接在一起
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data

//当请求完成的时候调用
//输出响应体,使用[[NSString alloc]initWithData:self.resultData encoding:NSUTF8StringEncoding]方式输出
-(void)connectionDidFinishLoading:(NSURLConnection *)connection

//当请求失败或者是出现错误的时候调用
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
```
- 4.NSURLConnection发送POST请求

```objc
  //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
    
    //2.创建可变的请求对象(需要使用可变类型,因为默认不可变请求方法为GET)
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    
    //3.**修改请求方法POST(大写)**
    request.HTTPMethod = @"POST";
    
    //设置请求对象
    //请求超时 --- 设置一个请求的时间限制,如果超过这个时间那么就认为请求失败
    request.timeoutInterval = 15;
    //设置请求头
    [request setValue:@"iOS 10.1" forHTTPHeaderField:@"User-Agent"];
    
    //4.设置请求体
    //username=520it&pwd=520it&type=JSON
    request.HTTPBody = [@"username=520it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
    
    //5.发送网络请求+接收服务器返回的数据
    [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc]init] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) 
    
    /*
   也可使用 NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) 
   
   这样可以不用创建请求对象
    */
    
        if(connectionError)
        {
            NSLog(@"请求失败---%@",connectionError);
            return ;
        }
        //6.解析是服务器返回的数据
        NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
    }];

```

######二.NSURLSession的实现
- 1.NSURLSession发送GET请求

```objc

//1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
    
    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    
    //3.获得一个会话对象
    NSURLSession *session = [NSURLSession sharedSession];
    
    //4.创建请求任务
    /*
     第一个参数:请求对象
     第二个参数:completionHandler 完成后执行的回调
            data:响应体
            response:响应头
            error:错误信息
     */
    //!!!注意:默认情况下completionHandler在子线程中调用
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
       
    //6.解析服务器返回的数据
    NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    
    //5.**执行task**需要自己手动开启
    [dataTask resume];


```

- 2.NSURLSession发送POST请求

```objc

//1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
    
    //2.创建可变的请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //2.1 修改请求方法
    request.HTTPMethod = @"POST";
    //2.2 设置请求体
    request.HTTPBody = [@"username=520&pwd=520&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
    
    //3.创建会话对象
    NSURLSession *session = [NSURLSession sharedSession];
    
    //4.创建task
   NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
       
       //6.解析数据
       NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    
    //5.执行task
    [dataTask resume];
}
```

- 3.NSURLSession发送请求(代理方法)
  
```objc
//1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];
    //2.创建会话对象
    /*
     第一个参数:配置选项
     第二个参数:谁成为代理
     第三个参数:队列(线程) 决定的是代理方法在哪个线程中调用
     */
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    
    //3.创建task
    NSURLSessionDataTask *dataTask =  [session dataTaskWithURL:url];
    //4.执行task
    [dataTask resume];

//并实现下列代理方法:

//当接收到服务器响应的时候调用
//默认情况下是不会接收服务器返回数据的,如果需要接收应该主动告诉系统(需要在completionHandler中告知系统)
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler

/*
NSURLSessionResponseCancel = 0,     取消(默认的做法)
NSURLSessionResponseAllow = 1,      允许(接收数据)
*/

//当接收到服务器返回数据的时候调用(会调用多次)
//解决方式和NSURLConnection的代理方式一致
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
```