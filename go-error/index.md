# Go错误处理


## panic

1. 在程序启动的时候，如果有强依赖的服务出现故障时 panic 退出

2. 在程序启动的时候，如果发现有配置明显不符合要求， 可以 panic 退出（防御编程）

3. 其他情况下只要不是不可恢复的程序错误，都不应该直接 panic 应该返回 error

```Go
// panic 恢复，需要在报错之前声明
defer func(){
    if err := recover(); err != nil {
        log.Printf("panic: %+v", err)
    }
}()
```

## error

1. 我们在应用程序中使用 github.com/pkg/errors 处理应用错误，**注意在公共库当中，我们一般不使用这个**

2. error 应该是函数的最后一个返回值，当 error 不为 nil 时，函数的其他返回值是不可用的状态，不应该对其他返回值做任何期待

3. 错误处理的时候应该先判断错误（使用卫语句）， if err != nil 出现错误及时返回，使代码是一条流畅的直线，避免过多的嵌套

```Go
func foo() error {
    err := A()
    if err != nil {
        return err
    }

    // ... 其他逻辑
    return nil
}
```

4. 在应用程序中出现错误需要自定义错误时，使用 errors.New() 或者 errors.Errorf() 返回错误

5. 如果是调用本应用程序的其他函数出现错误，请直接返回，如果需要携带信息，请使用 errors.WithMessage

```Go
errors.WithMessage(err, "其他附加信息")
```

6. 如果是调用其他库（标准库、企业公共库、开源第三方库等）出现错误时，请使用 errors.Wrap 添加堆栈信息
   1. 不要每个地方都是用 errors.Wrap 只需要在错误第一次出现时进行 errors.Wrap 即可
   2. 根据场景进行判断是否需要将其他库的原始错误吞掉
   3. 编写基础库一般不用 errors.Wrap 避免堆栈信息重复

```Go
errors.Wrap(err, "其他附加信息")
```

7. 禁止每个出错的地方都打日志，只需要在进程的最开始（最外层）的地方使用 %+v 进行统一打印

```Go
log.Errorf("%+v", err)
```

8. 错误判断使用 errors.Is 进行比较

```Go
func foo() error {
    err := A()
    if errors.Is(err, io.EOF){
    	return nil
    }

    // 其他逻辑
    return nil
}
```

9.  错误类型判断，使用 errors.As 进行赋值

```Go
func foo() error {
    err := A()

    var errA errorA
    if errors.As(err, &errA){
    	// ...
    }

    // 其他逻辑
    return nil
}
```

10. 获取原始错误，使用 errors.Cause(err error)

11. 处理错误的时候，需要释放已分配的资源，使用 defer 进行清理，例如文件句柄

12. 对于业务错误，推荐在一个统一的地方创建一个错误字典，错误字典里面应该包含错误的 code，并且在日志中作为独立字段打印，方便做业务告警的判断，错误必须有清晰的错误文档

## 错误码设计

纯数字表示，不同部位代表不同的服务，不同的模块。错误代码示例：100101

1. 10: 服务。
2. 01: 某个服务下的某个模块。
3. 01: 模块下的错误码序号，每个模块可以注册 100 个错误。

注意：如果每个模块的错误码超过 100 个，要么说明这个模块太大了，建议拆分；要么说
明错误码设计得不合理，共享性差，需要重新设计。

## 参考

https://lailin.xyz/post/go-training-03.html

https://github.com/pkg/errors

