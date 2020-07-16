# BlockCircularReference
关于Block循环引用的几种情况代码示例

```objc
//
//  ViewControllerB.m
//  BlockTest
//
//  Created by HayChao on 2020/7/8.
//  Copyright © 2020 HayChao. All rights reserved.
//

#import "ViewControllerB.h"
@interface ViewControllerB ()
@property (nonatomic, copy) void(^TestBlock)(void);
@property (nonatomic, copy) void(^TestBlockNew)(ViewControllerB *vc);
@property (nonatomic, strong) NSString *name;

@end

@implementation ViewControllerB

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor yellowColor];
    //输出
    [self blockOne];
    
//    [self blockTwo];
//    [self blockThree];
//    [self blockFour];
}
- (void)blockOne
{
    self.name = @"这是文字";
    //循环引用产生原因：self->block->self
    //为解决循环引用的问题用__weak若引用self，但是此方法依然不完善，详情见blockTwo
    __weak typeof(self) weakSelf = self;
    self.TestBlock = ^{
       NSLog(@"%@",weakSelf.name);
    };
    self.TestBlock();
       
}

- (void)blockTwo
{
    self.name = @"这是文字";
   __weak typeof(self) weakSelf = self;
   self.TestBlock = ^{
      __strong typeof(self) strongSelf = weakSelf;
       //当延迟2秒进行self的属性操作后，发现输出为null
       //产生原因当weakSelf.name被调用时，weakSelf已被释放，故在此需要对weakSelf进行一次__strong
       dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

           NSLog(@"%@",strongSelf.name);
       });
      
   };
   self.TestBlock();
}

- (void)blockThree
{
    self.name = @"这是文字";
    __block ViewControllerB *vc = self;
      self.TestBlock = ^{
          //延迟两秒执行
          dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
              NSLog(@"%@",vc.name);
              //为了防止Vc没有被释放，在此手动释放vc
              vc = nil;
          });
         
      };
      self.TestBlock();
}

//最优雅的方法
- (void)blockFour
{
    self.name = @"这是文字";
    self.TestBlockNew = ^(ViewControllerB *vc) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

            NSLog(@"%@",vc.name);
        });
    };
      self.TestBlockNew(self);
}



- (void)dealloc
{
    NSLog(@"dealloc来了");
}


- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{

    [self dismissViewControllerAnimated:YES completion:^{
        
    }];
}

@end

```
（更多iOS开发干货，欢迎关注  [微博@3W_狮兄 ](http://weibo.com/hanjunzhao/) ）

----------
Posted by  [微博@3W_狮兄 ](http://weibo.com/hanjunzhao/))  
原创文章，版权声明：自由转载-非商用-非衍生-保持署名 |
