---
title: runtime的weak属性现实
date: 2018-05-15 16:31:12
tags: [iOS]
categories: 赖钧
description: 本文主要描述weak引用的存储过程
---

### weak属性赋值时的方法调用过程
~~~ c
@interface TestObject : NSObject

@property(nonatomic,weak) NSObject *weakObj1;
@property(nonatomic,weak) NSObject *weakObj2;
@property(nonatomic,weak) NSObject *weakObj3;
@property(nonatomic,weak) NSObject *weakObj4;
@property(nonatomic,weak) NSObject *weakObj5;

@end

@implementation TestObject

- (void)dealloc
{
    NSLog(@"TestObject %@",NSStringFromSelector(_cmd));
}

@end

~~~

![](/img/laijun/weak_reference/image1.png)
可以看出weak引用赋值的时候方法调用时这样：objc_storeWeak(NSObject.mm)-->storeWeak(NSObject.mm)-->weak_register_no_lock(objc-weak.mm)，那应该weak引用的存储就应该是在weak_register_no_lock这个方法

objc_storeWeak方法

~~~ c
// Template parameters.
enum HaveOld { DontHaveOld = false, DoHaveOld = true };
enum HaveNew { DontHaveNew = false, DoHaveNew = true };

/** 
 * This function stores a new value into a __weak variable. It would
 * be used anywhere a __weak variable is the target of an assignment.
 * 
 * @param location The address of the weak pointer itself
 * @param newObj The new object this weak ptr should now point to
 * 
 * @return \e newObj
 */
id
objc_storeWeak(id *location, id newObj)
{
    return storeWeak<DoHaveOld, DoHaveNew, DoCrashIfDeallocating>
        (location, (objc_object *)newObj);
}
~~~
内部直接调用了storeWeak方法,传递了两个参数，locaiton弱引用指针内存地址本身，newObj为引用的对象

~~~ c
template <HaveOld haveOld, HaveNew haveNew,
          CrashIfDeallocating crashIfDeallocating>
static id 
storeWeak(id *location, objc_object *newObj)
{
    assert(haveOld  ||  haveNew);
    if (!haveNew) assert(newObj == nil);

    Class previouslyInitializedClass = nil;
    id oldObj;
    SideTable *oldTable;
    SideTable *newTable;

    // Acquire locks for old and new values.
    // Order by lock address to prevent lock ordering problems. 
    // Retry if the old value changes underneath us.
 retry:
    if (haveOld) {
        oldObj = *location;
        oldTable = &SideTables()[oldObj];
    } else {
        oldTable = nil;
    }
    if (haveNew) {
        newTable = &SideTables()[newObj]; //注解1
    } else {
        newTable = nil;
    }

    SideTable::lockTwo<haveOld, haveNew>(oldTable, newTable);

    if (haveOld  &&  *location != oldObj) {
        SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);
        goto retry;
    }

    // Prevent a deadlock between the weak reference machinery
    // and the +initialize machinery by ensuring that no 
    // weakly-referenced object has an un-+initialized isa.
    if (haveNew  &&  newObj) {
        Class cls = newObj->getIsa();
        if (cls != previouslyInitializedClass  &&  
            !((objc_class *)cls)->isInitialized()) 
        {
            SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);
            _class_initialize(_class_getNonMetaClass(cls, (id)newObj));

            // If this class is finished with +initialize then we're good.
            // If this class is still running +initialize on this thread 
            // (i.e. +initialize called storeWeak on an instance of itself)
            // then we may proceed but it will appear initializing and 
            // not yet initialized to the check above.
            // Instead set previouslyInitializedClass to recognize it on retry.
            previouslyInitializedClass = cls;

            goto retry;
        }
    }

    // Clean up old value, if any.
    if (haveOld) { 
        weak_unregister_no_lock(&oldTable->weak_table, oldObj, location);
    }

    // Assign new value, if any.
    if (haveNew) {
        //注解2
        newObj = (objc_object *)
            weak_register_no_lock(&newTable->weak_table, (id)newObj, location, 
                                  crashIfDeallocating);
        // weak_register_no_lock returns nil if weak store should be rejected

        // Set is-weakly-referenced bit in refcount table.
        if (newObj  &&  !newObj->isTaggedPointer()) {
            newObj->setWeaklyReferenced_nolock();
        }

        // Do not set *location anywhere else. That would introduce a race.
        *location = (id)newObj;
        //注解2
    }
    else {
        // No new value. The storage is not changed.
    }
    
    SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);

    return (id)newObj;
}
~~~

注解1 newTable = &SideTables()[newObj]; 根据newObj对象地址做了移位求余计算索引，然后通过索引从array数组获取SideTable类型的newTable(这个table内部包含存储弱引用的hash_table)。

~~~ c
class StripedMap {
    ......
    
    #if TARGET_OS_EMBEDDED
    enum { StripeCount = 8 };
#else
    enum { StripeCount = 64 };
#endif

    ......
    
    PaddedT array[StripeCount];

    static unsigned int indexForPointer(const void *p) {
        uintptr_t addr = reinterpret_cast<uintptr_t>(p);
        return ((addr >> 4) ^ (addr >> 9)) % StripeCount;
    }

 public:
    T& operator[] (const void *p) { //c++语法,重载[]操作符,所以&SideTables()[newObj]调用会进入此方法
        return array[indexForPointer(p)].value; 
    }
    ......    
}
~~~
SideTable数据结构

~~~ c
struct SideTable {
    spinlock_t slock;
    RefcountMap refcnts;
    weak_table_t weak_table;

    SideTable() {
        memset(&weak_table, 0, sizeof(weak_table));
    }

    ~SideTable() {
        _objc_fatal("Do not delete SideTable.");
    }

    void lock() { slock.lock(); }
    void unlock() { slock.unlock(); }
    void forceReset() { slock.forceReset(); }

    // Address-ordered lock discipline for a pair of side tables.

    template<HaveOld, HaveNew>
    static void lockTwo(SideTable *lock1, SideTable *lock2);
    template<HaveOld, HaveNew>
    static void unlockTwo(SideTable *lock1, SideTable *lock2);
};
~~~
SideTable主要有三个组成部分,
1、weak_table_t: 保存weak 引用的全局 hash 表
2、RefcountMap : 引用计数的 hash 表 
3、slock: 保证原子操作的自旋锁 
接着再来看weak_table_t的结构

~~~ c
/**
 * The global weak references table. Stores object ids as keys,
 * and weak_entry_t structs as their values.
 */
struct weak_table_t {
    weak_entry_t *weak_entries;
    size_t    num_entries;
    uintptr_t mask;
    uintptr_t max_hash_displacement;
};
~~~
从注释可以看出，这个就是存储weak引用的表，ids(对象地址作为key),weak_entry_t结构做为values。
再来看weak_entry_t结构

~~~ c
/**
 * The internal structure stored in the weak references table. 
 * It maintains and stores
 * a hash set of weak references pointing to an object.
 * If out_of_line_ness != REFERRERS_OUT_OF_LINE then the set
 * is instead a small inline array.
 */
#define WEAK_INLINE_COUNT 4

// out_of_line_ness field overlaps with the low two bits of inline_referrers[1].
// inline_referrers[1] is a DisguisedPtr of a pointer-aligned address.
// The low two bits of a pointer-aligned DisguisedPtr will always be 0b00
// (disguised nil or 0x80..00) or 0b11 (any other address).
// Therefore out_of_line_ness == 0b10 is used to mark the out-of-line state.
#define REFERRERS_OUT_OF_LINE 2

    struct weak_entry_t {
    DisguisedPtr<objc_object> referent;
    union {
        struct {
            weak_referrer_t *referrers;
            uintptr_t        out_of_line_ness : 2;
            uintptr_t        num_refs : PTR_MINUS_2;
            uintptr_t        mask;
            uintptr_t        max_hash_displacement;
        };
        struct {
            // out_of_line_ness field is low bits of inline_referrers[1]
            weak_referrer_t  inline_referrers[WEAK_INLINE_COUNT];
        };
    };

    bool out_of_line() {
        return (out_of_line_ness == REFERRERS_OUT_OF_LINE);
    }

    weak_entry_t& operator=(const weak_entry_t& other) {
        memcpy(this, &other, sizeof(other));
        return *this;
    }

    weak_entry_t(objc_object *newReferent, objc_object **newReferrer)
        : referent(newReferent)//weak_entry_t构造函数
    {
        inline_referrers[0] = newReferrer;
        for (int i = 1; i < WEAK_INLINE_COUNT; i++) {
            inline_referrers[i] = nil;
        }
    }
};
~~~
可以看出weak_entry_t主要有两个成员referent和一个有两个结构组成的union。referent时DisguisedPtr类型，用来接收经过伪装(强转为unsigned long类型,防止内存查看工具查看)的弱引用对象的地址。而referrers或者inline_referrers用来存储弱引用指针,从WEAK_INLINE_COUNT和REFERRERS_OUT_OF_LINE宏注解可知，当weak引用数量少于等于4个时候时存储在inline_referrers数组，大于就存储在referrers，后面会看到原理。

感觉有点跑远了，接着回到storeWeak函数的注解2处代码

~~~ C
if (haveNew) {
        //注解2
        newObj = (objc_object *)
            weak_register_no_lock(&newTable->weak_table, (id)newObj, location, 
                                  crashIfDeallocating);
        // weak_register_no_lock returns nil if weak store should be rejected

        // Set is-weakly-referenced bit in refcount table.
        if (newObj  &&  !newObj->isTaggedPointer()) {
            newObj->setWeaklyReferenced_nolock();
        }

        // Do not set *location anywhere else. That would introduce a race.
        *location = (id)newObj;
        //注解2
    }
~~~

调用weak_register_no_lock方法，传入weak_table，引用对象newObj,和弱引用指针地址location。方法内部实现引用对象和弱引用指针的存储。如果成功就在refcount table设置一个标志位，最后弱引用指针location指向来newObj。

weak_register_no_lock函数实现

~~~ c
/** 
 * Registers a new (object, weak pointer) pair. Creates a new weak
 * object entry if it does not exist.
 * 
 * @param weak_table The global weak table.
 * @param referent The object pointed to by the weak reference.
 * @param referrer The weak pointer address.
 */
id 
weak_register_no_lock(weak_table_t *weak_table, id referent_id, 
                      id *referrer_id, bool crashIfDeallocating)
{
    objc_object *referent = (objc_object *)referent_id;
    objc_object **referrer = (objc_object **)referrer_id;

    if (!referent  ||  referent->isTaggedPointer()) return referent_id;

    // ensure that the referenced object is viable
    bool deallocating;
    if (!referent->ISA()->hasCustomRR()) {
        deallocating = referent->rootIsDeallocating();
    }
    else {
        BOOL (*allowsWeakReference)(objc_object *, SEL) = 
            (BOOL(*)(objc_object *, SEL))
            object_getMethodImplementation((id)referent, 
                                           SEL_allowsWeakReference);
        if ((IMP)allowsWeakReference == _objc_msgForward) {
            return nil;
        }
        deallocating =
            ! (*allowsWeakReference)(referent, SEL_allowsWeakReference);
    }

    if (deallocating) {
        if (crashIfDeallocating) {
            _objc_fatal("Cannot form weak reference to instance (%p) of "
                        "class %s. It is possible that this object was "
                        "over-released, or is in the process of deallocation.",
                        (void*)referent, object_getClassName((id)referent));
        } else {
            return nil;
        }
    }

    // now remember it and where it is being stored
    weak_entry_t *entry;
    if ((entry = weak_entry_for_referent(weak_table, referent))) {
        append_referrer(entry, referrer);
    } 
    else {
        weak_entry_t new_entry(referent, referrer);//创建新entry
        weak_grow_maybe(weak_table);//当表空间使用率达到3/4则扩充空间
        weak_entry_insert(weak_table, &new_entry);//插入新的entry
    }

    // Do not set *referrer. objc_storeWeak() requires that the 
    // value not change.

    return referent_id;
}
~~~
方法内部调用对引用对象进行了空和存活判断，然后调用weak_entry_for_referent函数进行弱表的entry(对象地址和弱引用指针存储的结构)查找，如果找到就把软引用添加进entry，否则就创建一个entry并加入到weak_table。

weak_entry_for_referent函数实现

~~~ c
/** 
 * Return the weak reference table entry for the given referent. 
 * If there is no entry for referent, return NULL. 
 * Performs a lookup.
 *
 * @param weak_table 
 * @param referent The object. Must not be nil.
 * 
 * @return The table of weak referrers to this object. 
 */
static weak_entry_t *
weak_entry_for_referent(weak_table_t *weak_table, objc_object *referent)
{
    assert(referent);

    weak_entry_t *weak_entries = weak_table->weak_entries;

    if (!weak_entries) return nil;

    size_t begin = hash_pointer(referent) & weak_table->mask;
    size_t index = begin;
    size_t hash_displacement = 0;
    while (weak_table->weak_entries[index].referent != referent) {
        index = (index+1) & weak_table->mask;
        if (index == begin) bad_weak_table(weak_table->weak_entries);
        hash_displacement++;
        if (hash_displacement > weak_table->max_hash_displacement) {
            return nil;
        }
    }
    
    return &weak_table->weak_entries[index];
}
~~~
首先对weak_entries数组判空，然后进行对象指针的hash计算并与weak_table的mask进行&操作，从获取数组遍历开始的index下标。后面通过while循环解决hash碰撞。

已存在entry添加弱引用函数append_referrer实现

~~~ c
/** 
 * Add the given referrer to set of weak pointers in this entry.
 * Does not perform duplicate checking (b/c weak pointers are never
 * added to a set twice). 
 *
 * @param entry The entry holding the set of weak pointers. 
 * @param new_referrer The new weak pointer to be added.
 */
static void append_referrer(weak_entry_t *entry, objc_object **new_referrer)
{
    if (! entry->out_of_line()) {
    // entry的out_line_ness不等于REFERRERS_OUT_OF_LINE 
        // Try to insert inline.
        for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
            if (entry->inline_referrers[i] == nil) {
                entry->inline_referrers[i] = new_referrer;
                return;
            }
        }

        // Couldn't insert inline. Allocate out of line.
        weak_referrer_t *new_referrers = (weak_referrer_t *)
            calloc(WEAK_INLINE_COUNT, sizeof(weak_referrer_t));
        // This constructed table is invalid, but grow_refs_and_insert
        // will fix it and rehash it.
        for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
            new_referrers[i] = entry->inline_referrers[i];
        }
        entry->referrers = new_referrers;
        entry->num_refs = WEAK_INLINE_COUNT;
        entry->out_of_line_ness = REFERRERS_OUT_OF_LINE;
        entry->mask = WEAK_INLINE_COUNT-1;
        entry->max_hash_displacement = 0;
    }

    assert(entry->out_of_line());

    if (entry->num_refs >= TABLE_SIZE(entry) * 3/4) {
        return grow_refs_and_insert(entry, new_referrer);
    }
    size_t begin = w_hash_pointer(new_referrer) & (entry->mask);
    size_t index = begin;
    size_t hash_displacement = 0;
    while (entry->referrers[index] != nil) {
        hash_displacement++;
        index = (index+1) & entry->mask;
        if (index == begin) bad_weak_table(entry);
    }
    if (hash_displacement > entry->max_hash_displacement) {
        entry->max_hash_displacement = hash_displacement;
    }
    weak_referrer_t &ref = entry->referrers[index];
    ref = new_referrer;
    entry->num_refs++;
}
~~~
1，当entry的out_line_ness不等于REFERRERS_OUT_OF_LINE时，循环大小为4的inline_referrers数组，如果数组未满就插入弱引用指针，并跳出函数。如果数组已经满了（元素个数大于4），则分配内存创建entry的另一个成员referrers，并把inline_referrers数组内的元素转移到referrers里面，并设置out_line_ness=REFERRERS_OUT_OF_LINE。
2，当out_line_ness等于REFERRERS_OUT_OF_LINE时，如果存储弱引用的表空间达到3/4时则进行表空间扩充。然后对弱引用指针地址进行hash并和entry->mask进行&运算获得referrers数组遍历的起始下标index，然后通过while循环解决hash冲突,最后通过伪装进weak_referrer_t类型的变量ref并存储进referrers数组的index位置。

再来看看weak_referrer_t结构

~~~ c
typedef DisguisedPtr<objc_object *> weak_referrer_t;
template <typename T>
class DisguisedPtr {
    uintptr_t value;

    static uintptr_t disguise(T* ptr) { //伪装方法
        return -(uintptr_t)ptr;
    }

    static T* undisguise(uintptr_t val) {
        return (T*)-val;
    }

    ......
};
~~~
其实weak_referrer_t就是DisguisedPtr，DisguisedPtr就是一个对指针地址伪装的一个类，已经防止内存查看工具直接查看。伪装方法disguise的实现其实就是地址强转为-(uintptr_t)类型。

接着我们再回到weak_entry_for_referent函数实现，当weak_table不存在entry的时候，则创建一个entry，增加表空间大小(如果需要),最后把entry插入到weak_table中。

weak_entry_insert函数实现

~~~ c
/** 
 * Add new_entry to the object's table of weak references.
 * Does not check whether the referent is already in the table.
 */
static void weak_entry_insert(weak_table_t *weak_table, weak_entry_t *new_entry)
{
    weak_entry_t *weak_entries = weak_table->weak_entries;
    assert(weak_entries != nil);

    size_t begin = hash_pointer(new_entry->referent) & (weak_table->mask);
    size_t index = begin;
    size_t hash_displacement = 0;
    while (weak_entries[index].referent != nil) {
        index = (index+1) & weak_table->mask;
        if (index == begin) bad_weak_table(weak_entries);
        hash_displacement++;
    }

    weak_entries[index] = *new_entry;
    weak_table->num_entries++;

    if (hash_displacement > weak_table->max_hash_displacement) {
        weak_table->max_hash_displacement = hash_displacement;
    }
}
~~~
函数实现也同通过对引用对象指针进行hash后与weak_table->mask做&
运算计算出weak_entries数组下标，通过while循环解决hash冲突，最后把entry存储在下标为index的weak_entries数组空间。

### weak引用的存储过程总结：
引用对象地址通过移位异或求余等操作计算出下标index，通过index取到一个SideTable类型的table(这个table内部包含存储弱引用的weak_table),弱引用和引用对象组成weak_entry_t类型的结构存储在weak_table里面weak_entry_t结构的weak_entries数组，数组存放的下标通过对引用对象地址进行hash运算后得出。当一个引用对象对应多个弱引用时，多个弱引用存储在weak_entry_t结构里面的referrers或inline_referrers。


[runtime源码下载连接](https://opensource.apple.com/source/objc4/)

