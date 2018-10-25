---
title: XLForm实现
时间: 2018-10-25
tags: ios
categories: 常志杰
description: 本文主要介绍XLForm的使用
---
## **一、XLForm介绍**

XLForm是创建动态表格视图比较灵活的第三方库，可以快速实现表单的功能。

XLForm提供了很多的规范，可以动态的实现控制UI的展示，及时进行UI的刷新实现。

XLForm是非常轻量级的框架，在自定义表单实现的时候，非常快速便捷，可以去除许多多余的代码，使代码简单易懂，更加快捷方便的使用。

## **二、XLForm表单的创建**


1、导入头文件： #import "XLForm.h"。创建一个新的文件，继承XLFormViewController，代码如下，然后我们就可以实现表单的创建了。


~~~
#import <UIKit/UIKit.h>
#import "XLForm.h"

@interface ViewController : XLFormViewController

@end
~~~

2、创建表单，我们要实现 initView 方法

我们使用三个类定义表单

- XLFormDescriptor
- XLFormSectionDescriptor
- XLFormRowDescriptor

XLFormDescriptor是表的实例，这个实例可以包含一个或多个的分组。

XLFormSectionDescriptor是分组的实例，这个实例每一个可以包含一行或者多行的实现。

XLFormRowDescriptor是行的实例，可以实现多行和自定义的实现。

大家注意到，这个结构类似于UITableView的结构（表 --> 分组 --> 行），生产的表单结构，可以自定义灵活实现。

下面我们看看 initView 方法，创建表单的实现

~~~

- (void)initView{
    XLFormDescriptor *form; //创建表
    XLFormSectionDescriptor *section; //创建section
    XLFormRowDescriptor *row; // 创建row
    
    form = [XLFormDescriptor formDescriptorWithTitle:@"add event"];
    section = [XLFormSectionDescriptor  formSectionWithTitle:@"sectionOne"];
    [form addFormSection:section];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"one" rowType:XLFormRowDescriptorTypeName title:@"编码实现:"];
    [row.cellConfigAtConfigure setObject:@"title" forKey:@"textField.placeholder"];
    [section addFormRow:row];
    
    
    
    section = [XLFormSectionDescriptor formSectionWithTitle:@"sectionTwo"];
    [form addFormSection:section];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"iphone" rowType:XLFormRowDescriptorTypeNumber title:@"电话号码:"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"image" rowType:XLFormRowDescriptorTypeImage title:@"图片:"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"picker" rowType:XLFormRowDescriptorTypeDateTimeInline title:@"picker:"];
    row.value = [NSDate new];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"Info" rowType:XLFormRowDescriptorTypeInfo title:@"Info:"];
    row.value = @"hahahah";
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"check"  rowType:XLFormRowDescriptorTypeBooleanCheck title:@"check:"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"switch"  rowType:XLFormRowDescriptorTypeBooleanSwitch title:@"switch:"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"Button"  rowType:XLFormRowDescriptorTypeButton title:@"Button:"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"Slider"  rowType:XLFormRowDescriptorTypeSlider title:@"Slider:"];
    row.value = @(30);
    [row.cellConfigAtConfigure setObject:@(100) forKey:@"slider.maximumValue"];
    [row.cellConfigAtConfigure setObject:@(10) forKey:@"slider.minimumValue"];
    [row.cellConfigAtConfigure setObject:@(4) forKey:@"steps"];
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"AlertView"  rowType:XLFormRowDescriptorTypeSelectorAlertView title:@"AlertView:"];
    row.value = @"aaaaa";
    [section addFormRow:row];
    
    row = [XLFormRowDescriptor formRowDescriptorWithTag:@"Popover"  rowType:XLFormRowDescriptorTypeSelectorSegmentedControl title:@"Popover:"];
    [section addFormRow:row];
    
    self.form = form;
}

~~~

每一个section的分组可以添加不同类型的行row，进行展示。XLForm与UITableView的区别就是不需要关心NSIndexPath，UITableViewDelegate，UITableViewDataSource或其他复杂性。

### **行的定义类型**

 **输入行**


输入行允许用户输入文本值。我们通常是使用 UITextField 和 UITextView 。
这里XLForm给我们提供了很多类型的实现，我们只需要进行调用实现。

~~~
static NSString *const XLFormRowDescriptorTypeText = @"text";

static NSString *const XLFormRowDescriptorTypeName = @"name";

static NSString *const XLFormRowDescriptorTypeURL = @"url";

static NSString *const XLFormRowDescriptorTypeEmail = @"email";

static NSString *const XLFormRowDescriptorTypePassword = @"password";

static NSString *const XLFormRowDescriptorTypeNumber = @"number";

static NSString *const XLFormRowDescriptorTypePhone = @"phone";

static NSString *const XLFormRowDescriptorTypeTwitter = @"twitter";

static NSString *const XLFormRowDescriptorTypeAccount = @"account";

static NSString *const XLFormRowDescriptorTypeInteger = @"integer";

static NSString *const XLFormRowDescriptorTypeDecimal = @"decimal";

static NSString *const XLFormRowDescriptorTypeTextView = @"textView";
~~~

**行row的选择的实现**
XLForm可以支持实现这样的类型。

~~~
static NSString *const XLFormRowDescriptorTypeSelectorPush = @"selectorPush";

static NSString *const XLFormRowDescriptorTypeSelectorActionSheet = @"selectorActionSheet";

static NSString *const XLFormRowDescriptorTypeSelectorAlertView = @"selectorAlertView";

static NSString *const XLFormRowDescriptorTypeSelectorLeftRight = @"selectorLeftRight";

static NSString *const XLFormRowDescriptorTypeSelectorPickerView = @"selectorPickerView";

static NSString *const XLFormRowDescriptorTypeSelectorPickerViewInline = @"selectorPickerViewInline";

static NSString *const XLFormRowDescriptorTypeSelectorSegmentedControl = @"selectorSegmentedControl";

static NSString *const XLFormRowDescriptorTypeMultipleSelector = @"multipleSelector";
~~~

**XLForm同样也有控制分组头部的实现**

控制分组的 head 和 foot 的高度实现

~~~
#pragma mark - TableView协议
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section{
     return 20；
}
 
- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section{
     return 20；
}
~~~

实现分组 head 和 foot 的自定义视图 view 的实现

~~~
- ( UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section{
    return [UIView new];
}

- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section{
    return [UIView new];
}
~~~

### 获取每行的数据

**我们可以给每个特定的行 row ，给他写入需要的标识数据**

~~~
 // 自定义cell
    row = [self setupRowData:@{@"tag" : @"XZ", @"type": FormRowDescriptorTypeChoose, @"title" : @"选择"} withXLFormSection:section];
    row.value =  @[@{@"value" : @"XZ-1",
                     @"select": @"0",
                     @"name" : @"选项1"}.mutableCopy,

                   @{@"value" : @"XZ-2",
                     @"select": @"0",
                     @"name" : @"选项2"}.mutableCopy,

                   @{@"value" : @"XZ-3",
                     @"select": @"0",
                     @"name" : @"选项3"}.mutableCopy,

                   @{@"value" : @"XZ-4",
                     @"select": @"0",
                     @"name" : @"选项4"}.mutableCopy,

                   @{@"value" : @"XZ-5",
                     @"select": @"0",
                     @"name" : @"选项5"}.mutableCopy,
                   ];

    //PickerView 三级选择 (自定义)
    row = [self setupRowData:@{@"tag" : @"SJXZ", @"type": CFormRowDescriptorTypeThirdType, @"title" : @"三级选择"} withXLFormSection:section];
    row.selectorOptions = @[
                            @[[XLFormOptionsObject formOptionsObjectWithValue:@"XZ1-1" displayText:@"一级1"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ1-2" displayText:@"一级2"],
                              ],
                            @[[XLFormOptionsObject formOptionsObjectWithValue:@"XZ2-1" displayText:@"二级1"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ2-2" displayText:@"二级2"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ2-3" displayText:@"二级3"],
                              ],
                            @[[XLFormOptionsObject formOptionsObjectWithValue:@"XZ3-1" displayText:@"三级1"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ3-2" displayText:@"三级2"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ3-3" displayText:@"三级3"],
                              [XLFormOptionsObject formOptionsObjectWithValue:@"XZ3-4" displayText:@"三级4"],
                              ]
                            ];

    row.value = @[
                  [XLFormOptionsObject formOptionsObjectWithValue:@"XZ1-1" displayText:@"一级1"],
                  [XLFormOptionsObject formOptionsObjectWithValue:@"XZ2-2" displayText:@"二级2"],
                  [XLFormOptionsObject formOptionsObjectWithValue:@"XZ3-3" displayText:@"三级3"],
                  ].mutableCopy;

~~~

**表单属性设置的实现**

~~~
#pragma mark - 表单属性设置
- (XLFormRowDescriptor *)setupRowData:(NSDictionary *)selectorDic withXLFormSection:(XLFormSectionDescriptor *)section{
    
    // Tag : row的标签, 用于查找区分row
    // Type : row的类型, 框架提供一些, 也支持自定义
    // Title : row的标题
    XLFormRowDescriptor *row = [XLFormRowDescriptor formRowDescriptorWithTag:selectorDic[@"tag"] rowType:selectorDic[@"type"] title:selectorDic[@"title"]];
    [section addFormRow:row];
    // 设置一些属性
    [row.cellConfig setObject:[UIFont systemFontOfSize:14] forKey:@"textLabel.font"];
    [row.cellConfig setObject:[UIColor blackColor] forKey:@"textLabel.textColor"];
    [row.cellConfig setObject:[UIFont systemFontOfSize:14] forKey:@"detailTextLabel.font"];
    [row.cellConfig setObject:[UIColor lightGrayColor] forKey:@"detailTextLabel.textColor"];
    [row.cellConfig setObject:@"" forKey:@""];
    return row;
}

~~~



**获取每行的数据，可以使用 formValues 属性，formValues 就是定义的一个字典属性，会直接对输入的对象进行存储，我们也可以出入 key ，所需要的属性就取出来。下面看代码实现**

~~~
- (void)clickSend {
    NSDictionary *dict = [self formValues];
    NSLog(@"dict ==== %@", dict);
    NSLog(@"%@", [dict[@"LX"] formValue]);
}
~~~



参考资料:[https://www.jianshu.com/p/c4f23acadf6c](https://www.jianshu.com/p/c4f23acadf6c)
