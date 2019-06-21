---
layout: post
title: Objective-C中伪指针Tagged Pointer
date: 2019-06-21 12:00:00
tags: iOS
---

# Tagged Pointer
> 什么是TagPointer？
> 
> 
 - Tagged Pointer专门用来存储小的对象，例如NSNumber和NSDate
 - Tagged Pointer指针的值不再是地址了，而是真正的值。所以，实际上它不再是一个对象了，它只是一个披着对象皮的普通变量而已。所以，它的内存并不存储在堆中，也不需要malloc和free
 - 在内存读取上有着3倍的效率，创建时比以前快106倍
>
> Apple为要引入Tagged Pointer呢？
> > 我们知道NSInterge占用的字节在32位系统是4字节，到了64位系统就是8字节，当我们在存放小对象时导致了在64位系统可能出现成倍的浪费，从而导致内存翻倍，同时会对分配内存、维护引用计数、管理生命周期都增加了负担。
> 
> 一般我们在存放NsNumber和NSDate这一类变量的时候本身占用的内存大小常常不需要8个字节。4字节带符号的整数可以达到2^31=2147483648，99.999%的情况都能满足了。
> 
> ##### 苹果预留了环境变量`OBJC_DISABLE_TAGGED_POINTERS`，通过设置该变量的布尔值，可以将Tagged Pointer技术的启用与关闭的决定权交给开发者！如果禁用Tagged Pointer，只需设置环境变量`OBJC_DISABLE_TAGGED_POINTERS为YES`即可！
> 
> ### 在64位下面，Tagged Pointer的内存图是这样的：
> 
> ![](http://xbqn.nbshk.cn/20190620131955_pRRr0Y_Screenshot.jpeg)
> 

# Tagged Pointer源码
> 经过上面的解释想必对Tagged Pointer已经有了一定的了解，接下来我们来看下它源码，源码在objc/runtime/objc-internal.h中，[下载地址](https://opensource.apple.com/tarballs/objc4/)
> 
```
///生成TaggedPointer
static inline void * _Nonnull
_objc_makeTaggedPointer(objc_tag_index_t tag, uintptr_t value)
{
    if (tag <= OBJC_TAG_Last60BitPayload) {
        uintptr_t result =
            (_OBJC_TAG_MASK | 
             ((uintptr_t)tag << _OBJC_TAG_INDEX_SHIFT) | 
             ((value << _OBJC_TAG_PAYLOAD_RSHIFT) >> _OBJC_TAG_PAYLOAD_LSHIFT));
        return _objc_encodeTaggedPointer(result);
    } else {
        uintptr_t result =
            (_OBJC_TAG_EXT_MASK |
             ((uintptr_t)(tag - OBJC_TAG_First52BitPayload) << _OBJC_TAG_EXT_INDEX_SHIFT) |
             ((value << _OBJC_TAG_EXT_PAYLOAD_RSHIFT) >> _OBJC_TAG_EXT_PAYLOAD_LSHIFT));
        return _objc_encodeTaggedPointer(result);
    }
}
///TaggedPointer编码函数
static inline void * _Nonnull
_objc_encodeTaggedPointer(uintptr_t ptr)
{
    return (void *)(objc_debug_taggedpointer_obfuscator ^ ptr);
}
///TaggedPointer解码函数
static inline uintptr_t
_objc_decodeTaggedPointer(const void * _Nullable ptr)
{
    return (uintptr_t)ptr ^ objc_debug_taggedpointer_obfuscator;
}
```
> 通过以上源码可以看到生成TaggedPointer时最终是通过`_objc_encodeTaggedPointer `函数对真正的指针地址和`objc_debug_taggedpointer_obfuscator `记性异或操作获得伪指针。
> 
> 我们来打印下真伪指针：
> 
```
NSNumber *number1 = @1;
NSNumber *number2 = @2;
NSNumber *number3 = @3;
NSNumber *numberLager = @(555555555555555555);
NSLog(@"number1 pointer is %p", number1);
NSLog(@"number2 pointer is %p", number2);
NSLog(@"number3 pointer is %p", number3);
NSLog(@"numberLager pointer is %p", numberLager);
```
> number1/number2/number3都是小数据，所以肯定会使用Tagged Pointer，但是numberLager因为是大数据，通过Tagger Pointer存放不下（而且也没有必要使用Tagger Pointer），所以numberLager是正在的指针地址
> 
```
number1 pointer is 0x9237a2e097f0e96b  (伪指针)
number2 pointer is 0x9237a2e097f0e95b  (伪指针)
number3 pointer is 0x9237a2e097f0e94b  (伪指针)
numberLager pointer is 0x600001984200  (真指针，0x6堆区，0x7栈区)
```

### 为什么NSNumber、NSDate、NSString会自动转伪指针呢？其他的为什么不会呢？
> 这个就要扯到Tagged Pointer的注册了`_objc_registerTaggedPointerClass`
> 
> #### `_objc_registerTaggedPointerClass`
> 
```
/***********************************************************************
* _objc_registerTaggedPointerClass
* Set the class to use for the given tagged pointer index.
* Aborts if the tag is out of range, or if the tag is already 
* used by some other class.
**********************************************************************/
void
_objc_registerTaggedPointerClass(objc_tag_index_t tag, Class cls)
{
	 //OBJC_DISABLE_TAGGED_POINTERS就是判断开发者有没有关闭Tagged Pointer
    if (objc_debug_taggedpointer_mask == 0) {
        _objc_fatal("tagged pointers are disabled");
    }
    Class *slot = classSlotForTagIndex(tag);
    if (!slot) {
        _objc_fatal("tag index %u is invalid", (unsigned int)tag);
    }
    Class oldCls = *slot;
    if (cls  &&  oldCls  &&  cls != oldCls) {
        _objc_fatal("tag index %u used for two different classes "
                    "(was %p %s, now %p %s)", tag, 
                    oldCls, oldCls->nameForLogging(), 
                    cls, cls->nameForLogging());
    }
    *slot = cls;
    // Store a placeholder class in the basic tag slot that is 
    // reserved for the extended tag space, if it isn't set already.
    // Do this lazily when the first extended tag is registered so 
    // that old debuggers characterize bogus pointers correctly more often.
    if (tag < OBJC_TAG_First60BitPayload || tag > OBJC_TAG_Last60BitPayload) {
        Class *extSlot = classSlotForBasicTagIndex(OBJC_TAG_RESERVED_7);
        if (*extSlot == nil) {
            extern objc_class OBJC_CLASS_$___NSUnrecognizedTaggedPointer;
            *extSlot = (Class)&OBJC_CLASS_$___NSUnrecognizedTaggedPointer;
        }
    }
}
```
>
> - 数组`objc_tag_classes`：存储苹果定义的几个基础类；
> - 数组`objc_tag_ext_classes`：存储苹果预留的扩展类；
> 
```
#if SUPPORT_TAGGED_POINTERS //支持 Tagged Pointer
extern "C" {
    extern Class objc_debug_taggedpointer_classes[_OBJC_TAG_SLOT_COUNT*2];
    extern Class objc_debug_taggedpointer_ext_classes[_OBJC_TAG_EXT_SLOT_COUNT];
}
#define objc_tag_classes objc_debug_taggedpointer_classes
#define objc_tag_ext_classes objc_debug_taggedpointer_ext_classes
#endif
```
>
> #### classSlotForTagIndex
> > 该函数的主要功能就是根据指定索引 tag 获取类指针：
> > 
> >  
- 当索引tag为基础类的索引时，去数组`objc_tag_classes`中取数据；
- 当索引tag为扩展类的索引时，去数组`objc_tag_ext_classes`中取数据；
- 当索引tag无效时，返回一个 nil；
>
```
// Returns a pointer to the class's storage in the tagged class arrays, 
// or nil if the tag is out of range.
static Class *
classSlotForTagIndex(objc_tag_index_t tag)
{
    if (tag >= OBJC_TAG_First60BitPayload && tag <= OBJC_TAG_Last60BitPayload) {
        return classSlotForBasicTagIndex(tag);
    }
    if (tag >= OBJC_TAG_First52BitPayload && tag <= OBJC_TAG_Last52BitPayload) {
        int index = tag - OBJC_TAG_First52BitPayload;
        uintptr_t tagObfuscator = ((objc_debug_taggedpointer_obfuscator
                                    >> _OBJC_TAG_EXT_INDEX_SHIFT)
                                   & _OBJC_TAG_EXT_INDEX_MASK);
        return &objc_tag_ext_classes[index ^ tagObfuscator];
    }
    return nil;
}
```
>
> #### classSlotForBasicTagIndex
> > classSlotForBasicTagIndex() 函数的主要功能就是根据指定索引 tag 从数组`objc_tag_classes`中获取类指针；该函数要求索引tag是个有效的索引！
>
```
// Returns a pointer to the class's storage in the tagged class arrays.
// Assumes the tag is a valid basic tag.
static Class *
classSlotForBasicTagIndex(objc_tag_index_t tag)
{
    uintptr_t tagObfuscator = ((objc_debug_taggedpointer_obfuscator
                                >> _OBJC_TAG_INDEX_SHIFT)
                               & _OBJC_TAG_INDEX_MASK);
    uintptr_t obfuscatedTag = tag ^ tagObfuscator;
    // Array index in objc_tag_classes includes the tagged bit itself
#if SUPPORT_MSB_TAGGED_POINTERS //高位优先
    return &objc_tag_classes[0x8 | obfuscatedTag];
#else
    return &objc_tag_classes[(obfuscatedTag << 1) | 1];
#endif
}
```
> 
> ### 这边来通过文字描述下这个注册的流程
> 
- 1.判断`objc_debug_taggedpointer_mask`是否为0，判断是否开启Tagger Pointer，如果是禁用，直接回终止启动；
- 2.根据tag去取出类指针，若传递的是一个无效的tag，没有取到正确的类指针，还是终止启动；
- 3.尝试通过`Class oldCls = *slot`去获取该指针指向的类，若要注册的类与该处获取的类不是同一个，那么也是终止启动；
- 4.通过`*slot = cls`将入参cls类赋值给指针slot指向的位置
- 5.若注册的不是基础类型，则`OBJC_TAG_RESERVED_7`存储占位类`OBJC_CLASS_$___NSUnrecognizedTaggedPointer`

> ### 上面2个函数的参数都是`objc_tag_index_t `，这个是何方神圣呢？
> 
```
#if __has_feature(objc_fixed_enum)  ||  __cplusplus >= 201103L
enum objc_tag_index_t : uint16_t
#else
typedef uint16_t objc_tag_index_t;
enum
#endif
{
    // 60-bit payloads
    OBJC_TAG_NSAtom            = 0, 
    OBJC_TAG_1                 = 1, 
    OBJC_TAG_NSString          = 2, 
    OBJC_TAG_NSNumber          = 3, 
    OBJC_TAG_NSIndexPath       = 4, 
    OBJC_TAG_NSManagedObjectID = 5, 
    OBJC_TAG_NSDate            = 6,
    // 60-bit reserved
    OBJC_TAG_RESERVED_7        = 7, //60-bit保留位
    // 52-bit payloads
    OBJC_TAG_Photos_1          = 8,
    OBJC_TAG_Photos_2          = 9,
    OBJC_TAG_Photos_3          = 10,
    OBJC_TAG_Photos_4          = 11,
    OBJC_TAG_XPC_1             = 12,
    OBJC_TAG_XPC_2             = 13,
    OBJC_TAG_XPC_3             = 14,
    OBJC_TAG_XPC_4             = 15,
    OBJC_TAG_First60BitPayload = 0, 
    OBJC_TAG_Last60BitPayload  = 6, 
    OBJC_TAG_First52BitPayload = 8, 
    OBJC_TAG_Last52BitPayload  = 263, 
    OBJC_TAG_RESERVED_264      = 264 ////52-bit保留位
};
#if __has_feature(objc_fixed_enum)  &&  !defined(__cplusplus)
typedef enum objc_tag_index_t objc_tag_index_t;
#endif
```
> 其实就是一个枚举，存储在Tagged Pointer的特殊标记的部位，比如NSNumber是3，NSString是2，NDate是6，等等
> 
> 那么这个注册动作是在什么时候调用的呢？打个断点看下堆栈信息：
> 
```
(lldb)  thread backtrace
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000100382213 libobjc.A.dylib`::_objc_registerTaggedPointerClass(tag=OBJC_TAG_NSNumber, cls=__NSCFNumber) at objc-runtime-new.mm:6668
    frame #1: 0x00007fff433a0b5b CoreFoundation`__CFNumberGetTypeID_block_invoke + 65
    frame #2: 0x0000000100d087c3 libdispatch.dylib`_dispatch_client_callout + 8
    frame #3: 0x0000000100d0a48b libdispatch.dylib`_dispatch_once_callout + 87
    frame #4: 0x00007fff433a0b17 CoreFoundation`CFNumberGetTypeID + 39
    frame #5: 0x00007fff433a0103 CoreFoundation`__CFInitialize + 715
    frame #6: 0x0000000100020a68 dyld`ImageLoaderMachO::doImageInit(ImageLoader::LinkContext const&) + 316
    frame #7: 0x0000000100020ebb dyld`ImageLoaderMachO::doInitialization(ImageLoader::LinkContext const&) + 29
    frame #8: 0x000000010001c0da dyld`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 358
    frame #9: 0x000000010001c06d dyld`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 249
    frame #10: 0x000000010001c06d dyld`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 249
    frame #11: 0x000000010001c06d dyld`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 249
    frame #12: 0x000000010001b254 dyld`ImageLoader::processInitializers(ImageLoader::LinkContext const&, unsigned int, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 134
    frame #13: 0x000000010001b2e8 dyld`ImageLoader::runInitializers(ImageLoader::LinkContext const&, ImageLoader::InitializerTimingList&) + 74
    frame #14: 0x000000010000a756 dyld`dyld::initializeMainExecutable() + 169
    frame #15: 0x000000010000f78f dyld`dyld::_main(macho_header const*, unsigned long, int, char const**, char const**, char const**, unsigned long*) + 6237
    frame #16: 0x00000001000094f6 dyld`dyldbootstrap::start(macho_header const*, int, char const**, long, macho_header const*, unsigned long*) + 1154
    frame #17: 0x0000000100009036 dyld`_dyld_start + 54
```
> 可以分析看到，是在`_dyld_start`阶段注册的
> 

### 那么它是如何判断一个指针是否是伪指针的呢？
> 
```
static inline bool 
_objc_isTaggedPointer(const void * _Nullable ptr)
{
    return ((uintptr_t)ptr & _OBJC_TAG_MASK) == _OBJC_TAG_MASK;
}
```
> 
> 就是和`_OBJC_TAG_MASK `进行了&运算，之前生成伪指针的时候可以看到也是和`_OBJC_TAG_MASK `做运算的。简单的说就是，将一个指针地址和`_OBJC_TAG_MASK`量做&运算：判断该指针的最高位(iOS)或者最低位(MacOS)为 1，那么这个指针就是 Tagged Pointer。
> 
> 在iOS系统中，是遵循MSB规则（高位优先），所以`_OBJC_TAG_MASK `值就是`(1UL<<63)` = `0x8000000000000000`，它的二进制就是`1000000000000000000000000000000000000000000000000000000000000000`，看上面的`_objc_isTaggedPointer`，只要指针的最高位是1，结果肯定是True，简单说就是只要大于等于0x8开头就都是指针。
> 
> 我们反过来再看下上面之前输出的指针地址：`0x9237a2e097f0e96b`，0x9 = 1001，最左边是1，所以它就是一个伪指针。
> 

### 那么当它是一个伪指针时，如何找出它的tag类型呢？
>
```
static inline objc_tag_index_t 
_objc_getTaggedPointerTag(const void * _Nullable ptr) 
{
    // assert(_objc_isTaggedPointer(ptr));
    uintptr_t value = _objc_decodeTaggedPointer(ptr); //解码
    uintptr_t basicTag = (value >> _OBJC_TAG_INDEX_SHIFT) & _OBJC_TAG_INDEX_MASK;
    uintptr_t extTag =   (value >> _OBJC_TAG_EXT_INDEX_SHIFT) & _OBJC_TAG_EXT_INDEX_MASK;
    if (basicTag == _OBJC_TAG_INDEX_MASK) {
        return (objc_tag_index_t)(extTag + OBJC_TAG_First52BitPayload);
    } else {
        return (objc_tag_index_t)basicTag;
    }
}
```