---
title: 实现根据名称的首字母进行定位
date: 2018-06-05 13:15:12
tags: [xhtml+lua]
categories: 匡利金
description: 实现根据名称的首字母进行定位
---
## 页面实现代码

```
<div class="zmmddiv" border="0">
    <div name="div_val" class="pgcsta" border="0">
        <div class="zmdiv,szmlb" border="0">
            <label class="zmlab">首字母</label>
        </div>
        <div class="zmdiv" onclick="func_dw('1','A')" delay="1" border="0">
            <label class="zmlab">A</label>
        </div>
        <div class="zmdiv" onclick="func_dw('2','B')" delay="1" border="0">
            <label class="zmlab">A</label>
        </div>
        <div class="zmdiv" onclick="func_dw('3','C')" delay="1" border="0">
            <label class="zmlab">A</label>
        </div>
        ......
    </div>
</div>
```

## 脚本实现代码

```
function func_dw(ky,yk)
    dw_num=ky; 
    dw=yk;
    local divVal = document:getElementsByName("div_val");
    local y_top = tostring(dw);
    local y_value="0px";
    y_value = ert("@"..y_top):css("top");
    if y_value == nil then
        y_value="0px"
    end;
    local x_value="0";
    emp:divScrollTo(divVal[1],x_value,y_value);
    dw="A";
    dw_num=0;
end
```
