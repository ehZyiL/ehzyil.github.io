# freemarker基本语法

**取值**

* `${student}`：获取变量 `student` 的值。
* `${student.id}`：获取变量 `student` 中 `id` 属性的值。

**直接输出字符串**

* `${"hello freemarker"}`：直接输出字符串 "hello freemarker"。

**if 判断**

* `<#if student.id == 0>`：如果 `student.id` 等于 0，则执行内部代码块。
* `<#else if student.id == 1>`：如果 `student.id` 等于 1，则执行内部代码块。
* `<#else>`：如果以上条件都不满足，则执行内部代码块。

**比较运算符**

* `==`：判断两个值是否相等。
* `!=`：判断两个值是否不等。
* `>` 或 `gt`：判断左边值是否大于右边值。
* `>=` 或 `gte`：判断左边值是否大于等于右边值。
* `<` 或 `lt`：判断左边值是否小于右边值。
* `<=` 或 `lte`：判断左边值是否小于等于右边值。
* `&&`：逻辑与。
* `||`：逻辑或。

**List 集合（数组）**

* `<#list arrays as item>`：遍历数组 `arrays` 中的每个元素，并将其分配给变量 `item`。
* `${item_index}`：获取当前元素的索引。
* `${item.id}`：获取当前元素的 `id` 属性。

**Map 集合（字典）**

* `<#if map??>`：如果 `map` 变量不为 null，则执行内部代码块。
* `<#list map?keys as key>`：遍历 `map` 中的键，并将其分配给变量 `key`。
* `key:${key!}`：获取当前键。
* `value:${map[key]!}`：获取当前键对应的值。

**日期类型转换**

* `${date?date}`：解析日期。
* `${date?time}`：解析时间。
* `${date?datetime}`：解析日期和时间。
* `${date?string('yyyy/MM/dd HH:mm:ss')}`：使用指定格式自定义日期字符串。

**null 值的处理**

* `!`：指定缺失变量的默认值。
* `??`：判断某个变量是否存在。
* `${val!}`：如果 `val` 为 null，则输出空白。
* `${val!"默认值"}`：如果 `val` 为 null，则指定默认值。

**用 if 判断 null 值**

* `<#if val??>`：如果 `val` 不为 null，则执行内部代码块。
* `<#else>`：如果 `val` 为 null，则执行内部代码块。