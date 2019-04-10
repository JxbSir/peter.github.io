---
layout: post
title: iOS之CoreText文本绘制
date: 2019-04-09 20:00:00
tags: iOS
---

### 为什么要使用CoreText绘制文本
> 一般情况下，我们都会使用UILabel来布局文本。当我们使用少量的UILabel时，肉眼并不能明显的看到卡顿，但是当一个屏幕内出现大量UILabel时，就会明显感觉到卡顿了，这是为什么呢？
> 
> 因为，UILabel这些UIKit中的文本控件的排版与绘制都是在主线程进行的，当出现大量文本时，CPU的压力会非常之大，所以就会出现卡顿问题。
> 
> 严重时可能FPS会降到50以下，当全部使用CoreText绘制时，FPS可以达到58或更高（这里的case是界面上只有文本，不考虑存在其他控件）
> 
> 所以此时就需要CoreText来进行异步绘制了，它的性能也是非常高的。


### CoreText优缺点
> CoreText是Mac OS和iOS系统中处理文本的low-level API, 不管是使用OC还是swift, 实际我们使用CoreText都还是间接或直接使用C语言在写代码。CoreText是iOS和Mac OS中文本处理的根基, TextKit和WebKit都是构建于其上。

##### 优点
- 性能高，绘制时间少
- 异步渲染，减少CPU压力
- 更底层，直接与CoreGraphics打交道
- 扩展性高，各种排版都可以实现

##### 缺点
- 因为API比较底层，使用复杂，学习成本相对高


### CoreText框架
> 坐标系：CoreText的坐标与UIKit的坐标是不一样的，使用的时候需要换算
![](http://xbqn.nbshk.cn/20190410105922_WslVBp_Screenshot.jpeg)
-
> 基础框架：
> ![](http://xbqn.nbshk.cn/20190410110304_rTDRhy_Screenshot.jpeg) 
>
- CTFrameSetter：相当于一个工作，来生产CTFrame，一个界面上可以有多个CTFrame
- CTFrame：可以视为一个画布，范围是由CGPath来决定，其后的绘制是只在这个范围内绘制
- CTLine：一个CTFrame由多个CTLine组成，一行就是一个CTLine
- CTRun：一个CTLine是由一个或多个CTRun组成，可以理解为一个块，当一个CTLine中包含了多个不同属性时，比如字体、颜色、Attachment等，都会通过CTRun将CTLine分隔开
- CTRunDelegate：其实看名字大致了解这个类的用处，就是为CTRun提供数据源（width，ascent，descent）等

### CoreText绘制核心代码
设置画布大小

```
CGRect boxRect = CGRectMake(0, 0, boxSize.width, boxSize.height);
CGMutablePathRef path = CGPathCreateMutable();
CGPathAddRect(path, NULL, CGRectMake(0, 0, boxSize.width, boxSize.height));
```

生成CTFrame

```
CTFramesetterRef ctSetter = CTFramesetterCreateWithAttributedString((CFTypeRef)string);
CTFrameRef ctFrame = CTFramesetterCreateFrame(ctSetter, CFRangeMake(0, string.length), path, nil);
```

通过CTFrame获取CTLine

```
CFArrayRef ctLines = CTFrameGetLines(ctFrame);
NSUInteger lineCount = CFArrayGetCount(ctLines);
//i = index
CTLineRef ctLine = CFArrayGetValueAtIndex(ctLines, i);

//获取此line的range
CFRange range = CTLineGetStringRange(_line);
NSRange _range = NSMakeRange(range.location, range.length);
```

然后获取每个CTLine的原始坐标

```
CGPoint *lineOrigins = malloc(lineCount * sizeof(CGPoint));
if (lineOrigins == NULL) {
    //goto finish
}
CTFrameGetLineOrigins(ctFrame, CFRangeMake(0, lineCount), lineOrigins);

// 获取CTLine的width、ascent、descent、leading
_width = CTLineGetTypographicBounds(ctline, &_ascent, &_descent, &_leading);

```

获取当前CTLine的所有CTRun

```
CFArrayRef runs = CTLineGetGlyphRuns(self.line);
NSUInteger runCount = CFArrayGetCount(runs);
//i = index
CTRunRef run = CFArrayGetValueAtIndex(runs, i);
//获取当前run的所有属性，所有存在attachment，可以做相应的attachment绘制
NSDictionary *attrs = (id)CTRunGetAttributes(run);
```

将CTLine画到画布上

```
CGContextSetTextPosition(context, point.x, self.size.height + line.descent - point.y - size.height);
CTLineDraw(ctline, context);
```

若有attachment，需要提前通过CTRunDelegateRef占位

```
//占位
CTRunDelegateRef delegateRef = delegate.CTRunDelegate;
                [attr addAttribute:(id)kCTRunDelegateAttributeName value:(__bridge id)delegateRef range:attachRange];
if (delegate) CFRelease(delegateRef);
                
//将cgImage画上
CGContextSaveGState(context);
CGContextTranslateCTM(context, 0, CGRectGetMaxY(rect) + CGRectGetMinY(rect));
CGContextScaleCTM(context, 1, -1);
CGContextDrawImage(context, rect, ref);
CGContextRestoreGState(context);
```

#### OK，通过以上简单的介绍，应该可以满足基本日常，当然，上面的核心代码只是摘录，实际实现的时候里面还有很多细节流程，这边就不赘述了。