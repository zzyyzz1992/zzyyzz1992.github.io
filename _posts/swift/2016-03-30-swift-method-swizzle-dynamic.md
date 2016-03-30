---
layout: post
title: Swift 中的方法混淆中的 dynamic 修饰符
category: Swift
tags: Swift
description: Swift 中的方法混淆中的 dynamic 修饰符
---

# 思考
 1、extension 中的 func 能不能不加 dynamic？

 2、加不加 dynamic 对交换方法有什么影响？

``` swift
let c = TestSwizzling()
print(c.methodOne())  //2
print(c.methodThree())  //1

class TestSwizzling : NSObject {
    dynamic func methodOne()->Int{
      return 1
    }
    func methodThree() -> Int{
      return 3
    }   
}

extension TestSwizzling {
//在 Objective-C 中,我们在 load() 方法进行 swizzling。但Swift不允许使用这个方法。
  override class func initialize()
  {
    struct Static
    {
        static var token: dispatch_once_t = 0;
    }

    // 只执行一次
    dispatch_once(&Static.token)
    {
        let originalSelector = #selector(TestSwizzling.methodOne);
        let swizzledSelector = #selector(TestSwizzling.methodThree);

        let originalMethod = class_getInstanceMethod(self, originalSelector);
        let swizzledMethod = class_getInstanceMethod(self, swizzledSelector);

        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
  }

  dynamic func methodTwo()->Int{
    // swizzling 后, 该方法就不会递归调用
    print(#function)
    return self.methodTwo()+1
  }
}
```

# 思考成果

  经过我的实验，我推测

  1、extension 中不管加不加 dynamic 都有 dynamic 的效果。

  2、在类本身的定义中如果没加 dynamic 是不能被混淆成其他方法的，但是其他的 dynamic 方法可以混淆成这个没加 dynamic 修饰的方法。

###### 环境
Xcode 7.3

###### 参考来源

1、[如何在 Swift 中高效地使用 Method Swizzling](http://swift.gg/2016/03/29/effective-method-swizzling-with-swift/)
