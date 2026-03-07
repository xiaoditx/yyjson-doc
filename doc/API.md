API
===

@note 翻译最后更新时间：2026/2/27 13:01<br>翻译质量反馈：[yyjson文档中文翻译](https://xiaoditx.github.io/moments/2026-3/07-2115/)

本文档包含了yyjson库所有API的用法和示例。


# API设计

## API前缀

所有公共函数和结构体都使用`yyjson_`作为前缀，所有常量包括宏定义都使用`YYJSON_`作为前缀。

## 可变/不可变数据的API

本库有两种数据结构：不可变（immutable）数据和可变（mutable）数据。

|                 | 不可变     | 可变            |
|-----------------|------------|----------------|
| 文档（Document） | yyjson_doc | yyjson_mut_doc |
| 值（Value）      | yyjson_val | yyjson_mut_val |

当读取JSON时，yyjson返回不可变文档和值。<br/>
当构建JSON时，yyjson创建可变文档和值。<br/>
文档管理所有JSON值与字符串的内存。<br/>

对于大多数不可变API，你可以在`yyjson_`后面加一个`mut`使之变为可变版本，例如：
```c
char *yyjson_write(yyjson_doc *doc, ...);
char *yyjson_mut_write(yyjson_mut_doc *doc, ...);

bool yyjson_is_str(yyjson_val *val);
bool yyjson_mut_is_str(yyjson_mut_val *val);
```

库中也提供了一些可以将值在可变与不可变之间相互转换的函数：<br/>

```c
// doc -> mut_doc
yyjson_mut_doc *yyjson_doc_mut_copy(yyjson_doc *doc, ...);
// val -> mut_val
yyjson_mut_val *yyjson_val_mut_copy(yyjson_val *val, ...);

// mut_doc -> doc
yyjson_doc *yyjson_mut_doc_imut_copy(yyjson_mut_doc *doc, ...);
// mut_val -> val
yyjson_doc *yyjson_mut_val_imut_copy(yyjson_mut_val *val, ...);
```

## 字符串API
本库支持带或不带空终止符（'\0'）的字符串。<br/>
当你想要使用无空终止符的字符串或者当你清楚的知道字符串长度的时候，你可以使用结尾为字母`n`的函数，例如：
```c
// 需要空终止符的版本
bool yyjson_equals_str(yyjson_val *val, const char *str);
// 空终止符是可选的版本
bool yyjson_equals_strn(yyjson_val *val, const char *str, size_t len);
```

为了获得更好的性能，yyjson在创建JSON时会将字符串视为常量。当然，如果你的字符串可能被修改，需要使用带有`cpy`字样的函数以将字符串复制进文档，例如：
```c
// 仅引用，需要空终止符
yyjson_mut_val *yyjson_mut_str(yyjson_mut_doc *doc, const char *str);
// 仅引用，空终止符可选
yyjson_mut_val *yyjson_mut_strn(yyjson_mut_doc *doc, const char *str, size_t len);

// 复制，需要空终止符
yyjson_mut_val *yyjson_mut_strcpy(yyjson_mut_doc *doc, const char *str);
// 复制，空终止符可选
yyjson_mut_val *yyjson_mut_strncpy(yyjson_mut_doc *doc, const char *str, size_t len);
```



---------------

# 读取JSON
本库提供了五个读JSON的函数。<br/>
每个函数接受一个UTF-8编码的数据或文件作为输入。<br/>
如果成功，函数返回一个文档，失败则返回`NULL`。

## 从字符串读取JSON
`dat`参数接受一个UTF-8字符串，不需要空终止符。<br/>
`len`参数是`dat`中字符串的字节长度<br/>
`flg`是读取器标志，不需要时可传入 0，详见[读取器标志](#读取器标志)。<br/>
加入输入是不合法的，函数会返回`NULL`。

```c
yyjson_doc *yyjson_read(const char *dat, 
                        size_t len, 
                        yyjson_read_flag flg);
```
示例代码：

```c
const char *str = "[1,2,3,4]";
yyjson_doc *doc = yyjson_read(str, strlen(str), 0);
if (doc) {...}
yyjson_doc_free(doc);
```

## 从文件读JSON

`path`参数接受一个JSON文件的路径。要求使用系统原生编码的带空终止符的字符串。<br/>
`flg`参数是读取器标志，不需要时可传入 0，详见[读取器标志](#读取器标志)。<br/>
`alc`参数用于指定内存分配器，如果不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`err`参数是一个接受错误信息的指针，如果不需要请传入NULL。<br/>
假如输入是不合法的，函数会返回`NULL`。

```c
yyjson_doc *yyjson_read_file(const char *path,
                             yyjson_read_flag flg,
                             const yyjson_alc *alc,
                             yyjson_read_err *err);
```

示例代码：

```c
yyjson_doc *doc = yyjson_read_file("/tmp/test.json", 0, NULL, NULL);
if (doc) {...}
yyjson_doc_free(doc);
```

## 从文件指针读取JSON

`fp`参数接受一个文件指针。yyjson会从文件的当前位置到结尾读取数据。<br/>
`flg`参数是读取器标志，不需要时可传入 0，详见[读取器标志](#读取器标志)。<br/>
`alc`参数用于指定内存分配器，如果不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`err`参数是一个接受错误信息的指针，如果不需要请传入NULL。<br/>
假如输入是不合法的，函数会返回`NULL`。

```c
yyjson_doc *yyjson_read_fp(FILE *fp,
                           yyjson_read_flag flg,
                           const yyjson_alc *alc,
                           yyjson_read_err *err);
```

示例代码：

```c
FILE *fp = fdopen(fd, "rb"); // fd为POSIX文件描述符
yyjson_doc *doc = yyjson_read_fp(fp, 0, NULL, NULL);
if (fp) fclose(fp);
if (doc) {...}
yyjson_doc_free(doc);
```

## 带选项读取JSON
`dat`参数接受一个UTF-8字符串。
假如你没有在`flg`参数处使用`YYJSON_READ_INSITU`标志则可以传入一个字符串常量。<br/>
`len`参数是`dat`中字符串的字节长度<br/>
`flg`参数是读取器标志，不需要时可传入 0，详见[读取器标志](#读取器标志)。<br/>
`alc`参数用于指定内存分配器，如果不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`err`参数是一个接受错误信息的指针，如果不需要请传入NULL。<br/>
假如输入是不合法的，函数会返回`NULL`。

```c
yyjson_doc *yyjson_read_opts(char *dat, 
                             size_t len, 
                             yyjson_read_flag flg,
                             const yyjson_alc *alc, 
                             yyjson_read_err *err);
```

示例代码：

```c
const char *dat = your_file.bytes;
size_t len = your_file.size;

yyjson_read_flag flg = YYJSON_READ_ALLOW_COMMENTS | YYJSON_READ_ALLOW_INF_AND_NAN;
yyjson_doc *doc = yyjson_read_opts((char *)dat, len, flg, NULL, NULL);

if (doc) {...}

yyjson_doc_free(doc);
```

## 增量读取JSON

读取一个很大的JSON文档可能会短暂的冻结程序，假如这是不可接受的，可以选择使用增量读取。

仅当文档较大且程序需要及时响应时，才推荐使用增量读取；此方法相较于`yyjson_read()`和`yyjson_read_opts()`会略慢一些。

注意：增量读取仅支持标准的JSON语法。
非标准的功能标志（比如注释, 尾随逗号）会被忽略。

下面是增量读取大体积JSON文档的流程：

1. 调用`yyjson_incr_new()`来创建一个增量阅读状态（state）对象。
2. 反复调用`yyjson_incr_read()`。
3. 调用`yyjson_incr_free()`以释放状态对象。

### 为增量阅读创建状态对象

`buf`参数接收一个不需要空结束符的UTF-8字符串。
假如你没有在`flg`参数处使用`YYJSON_READ_INSITU`标志则可以传入一个字符串常量。<br/>
`buf_len`参数是`buf`中字符串的字节长度。
`flg`参数是读取器标志，不需要时可传入 0，详见[读取器标志](#读取器标志)。<br/>
`alc`参数用于指定内存分配器，如果不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>

正常情况下这个函数会返回一个新的状态，但如果`flg`参数是无效的或者内存分配器发生了故障，函数返回NULL

```c
yyjson_incr_state *yyjson_incr_new(char *buf, size_t buf_len, yyjson_read_flag flg, const yyjson_alc *alc);
```

### 执行增量读取

执行最多`len`字节的增量读取。

增量读取的`state`参数由`yyjson_incr_new()`创建。<br/>
`len`参数表示从JSON数据开头开始计数的要读取的最大字节数。<br/>
`err`参数是一个接受错误信息的指针，本参数不可空。<br/>

当读取完成时，本函数返回一个文档对象，否则返回NULL。
假如`err->code`被设置为了`YYJSON_READ_ERROR_MORE`，意味着语法解析没有完成。
此时，需要将`len`增加几千字节（KB），随后重新调用本函数。
持续增加`len`直到`len == buf_len`（达到输入缓冲区的总长度）或者直到返回一个不是`YYJSON_READ_ERROR_MORE`的错误。

注意： 增量很小时语法分析效率不高。
建议单次增量几千字节或几兆字节（MB）

译者注：其实`YYJSON_READ_ERROR_MORE`不算错误，实际上已经正确读取了数据，只是没读完需要更多数据罢了，不知道你们能不能理解这个，反正我一开始没绕过来...

```c
yyjson_doc *yyjson_incr_read(yyjson_incr_state *state, size_t len, yyjson_read_err *err);
```

### 释放用于增量读取的状态

释放由`yyjson_incr_new()`创建的`state`.

```c
void yyjson_incr_free(yyjson_incr_state *state);
```

### 示例代码

```c
const char *dat = your_file.bytes;
size_t len = your_file.size;

yyjson_read_flag flg = YYJSON_READ_NOFLAG;
yyjson_incr_state *state = yyjson_incr_new(dat, len, flg, NULL);
yyjson_doc *doc;
yyjson_read_err err;
size_t read_so_far = 0;
do {
    read_so_far += 100000;
    if (read_so_far > len)
        read_so_far = len;
    doc = yyjson_incr_read(state, read_so_far, &err);
    if (err.code != YYJSON_READ_ERROR_MORE)
        break;
} while (read_so_far < len);
yyjson_incr_free(state);

if (doc != NULL) { ... }

yyjson_doc_free(doc);
```

## 读取器错误处理

当JSON读取失败且需要错误信息时，可向调用的`yyjson_read_xxx()`函数传递一个`yyjson_read_err`指针来接收详细的错误信息。 

示例代码：
```c
char *dat = ...;
size_t dat_len = ...;
yyjson_read_err err;
yyjson_doc *doc = yyjson_read_opts(dat, dat_len, 0, NULL, &err);

if (!doc) {
    printf("读取错误： %s，错误码:%u，位置：第%lu字节\n", 
            err.msg, err.code, err.pos);
    // 输出结果:
    // 读取错误: trailing comma is not allowed，错误码：7，位置：第40字节
	//           ↑  翻译： 不允许尾随逗号
}

yyjson_doc_free(doc);
```

错误信息中的`pos`标明了错误发生在的字节位置。假如你需要错误的行号和列号，可以使用`yyjson_locate_pos()`函数。需要注意的是行（`line`）与列（`column`）是从1开始的而字符（`character`）是从0开始的。上述所有值都是基于Unicode字符计算的，以便更好的兼容绝大多数的文本编辑器

示例代码：
```c
char *dat = ...;
size_t dat_len = ...;
yyjson_read_err err = ...;

size_t line, col, chr;
if (yyjson_locate_pos(dat, dat_len, err.pos, &line, &col, &chr)) {
    printf("错误发生在第 %lu 行 第 %lu 列，字符索引：%lu\n",
           line, col, chr);
    // 输出结果：
    // 错误发生在第 3 行 第 5 列，字符索引：32
}
```

## 读取器标志
本库提供了一组JSON读取器标志。<br/>

你可以单独使用一个标志，或者使用按位或操作符`|`将多个标志结合在一起。<br/>

非标准的标志（比如`YYJSON_READ_JSON5`，支持JSON 5 语法）在读取标准JSON输入时不会影响性能

### **YYJSON_READ_NOFLAG = 0**

这是JSON读取器的默认标志（遵循RFC-8259或ECMA-404）：

- 将正整数读入`uint64_t`。
- 将负整数读入`int64_t`。
- 将浮点数正确四舍五入后读入`double`。
- 对于超出`uint64_t`或`int64_t`范围的整数读入`double`。
- 当double型数值为无穷大时报错。
- 当字符串包含非法的UTF-8字符或BOM时报错。
- 遇到尾随逗号、注释、`Inf`和`NaN`字面值时报错。

### **YYJSON_READ_INSITU**
就地读取数据（不拷贝）。<br/>

这个选项允许读取器修改并使用输入数据来存储字符串值，这样可以略微提升读取速度，但调用者必须确保在文档被释放之前输入数据一直有效。输入数据必须至少填充`YYJSON_PADDING_SIZE`字节的额外空间。例如：`[1,2]`应写为`[1,2]\0\0\0\0`，输入长度应为5（填充字节不计入）。

示例代码：

```c
size_t dat_len = ...;
char *buf = malloc(dat_len + YYJSON_PADDING_SIZE); // 创建一个大于(len + 4)的缓冲区
read_from_socket(buf, ...);
memset(buf + file_size, 0, YYJSON_PADDING_SIZE); // 在数据后设置四字节的填充

yyjson_doc *doc = yyjson_read_opts(buf, dat_len, YYJSON_READ_INSITU, NULL, NULL);
if (doc) {...}
yyjson_doc_free(doc);
free(buf); // 输入数据需要在文档释放后释放
```

### **YYJSON_READ_STOP_WHEN_DONE**
当到达JSON文档的末尾是就停止解析，即使后面有附加内容也不报错。<br/>

这个选项对于解析大体积的JSON的一小部分是很有用的，像是[NDJSON](https://en.wikipedia.org/wiki/JSON_streaming)这样。（译者注：国内打不开维基百科，请自行寻找镜像替代）<br/>

示例代码：

```c
// 一个文件里存放了多个JSON，比如：
// [1,2,3] [4,5,6] {"a":"b"}

size_t file_size = ...;
char *dat = malloc(file_size + 4);
your_read_file(dat, file);
memset(dat + file_size, 0, 4); // 加入额外填充
    
char *hdr = dat;
char *end = dat + file_size;
yyjson_read_flag flg = YYJSON_READ_INSITU | YYJSON_READ_STOP_WHEN_DONE;

while (true) {
    yyjson_doc *doc = yyjson_read_opts(hdr, end - hdr, flg, NULL, NULL);
    if (!doc) break;
    your_doc_process(doc);
    hdr += yyjson_doc_get_read_size(doc); // 移动到下一位置
    yyjson_doc_free(doc);
}
free(dat);
```

### **YYJSON_READ_ALLOW_TRAILING_COMMAS**
允许在对象或数组尾部出现单个尾随逗号（非标准），例如：

```
{
    "a": 1,
    "b": 2,
}

[
    "a",
    "b",
]
```

### **YYJSON_READ_ALLOW_COMMENTS**
允许C风格的单行和多行注释（非标准），例如：

```
{
    "name": "玉墨", // 单行注释
    "id": /* 多行注释 */ 114514
}
```

### **YYJSON_READ_ALLOW_INF_AND_NAN**
允许NaN和Inf以及其他不区分大小写的字面量，例如：

```
{
    "large": 123e999,
    "nan1": NaN,
    "nan2": nan,
    "inf1": Inf,
    "inf2": -Infinity
}
```

### **YYJSON_READ_NUMBER_AS_RAW**
将所有数字以原始字符串读入而不做解析处理。

如果你想自己解析数字，这个标志就十分有用。
你可以使用下面这些函数来提取原始字符串：
```c
bool yyjson_is_raw(yyjson_val *val);
const char *yyjson_get_raw(yyjson_val *val);
size_t yyjson_get_len(yyjson_val *val)
```

### **YYJSON_READ_BIGNUM_AS_RAW**
将大数以原始字符串读入。

如果你想自己解析大数（比如你有个高精模拟什么的）这个标志就很有用
大数包括不能被`int64_t`和`uint64_t`表示的整数以及不能被有限的`double`表示的浮点数

需要注意的是这个标志会被`YYJSON_READ_NUMBER_AS_RAW`标志覆盖

### **YYJSON_READ_ALLOW_INVALID_UNICODE**
解析字符串时允许读取无效的Unicode（非标准），
例如：
```
"\x80xyz"
"\xF0\x81\x81\x81"
```
这个标志允许无效的字符出现在字符串值中，但是仍然会对无效的转义序列报错。标志不会影响正确编码字符串的性能。

***警告***: 使用这个选项时，请小心JSON内的字符串可能会包含的错误编码，你需要谨慎的处理这些字符串以避免安全问题。

### **YYJSON_READ_ALLOW_BOM**
允许带BOM的UTF-8并在解析前跳过BOM（非标准）。

### **YYJSON_READ_ALLOW_EXT_NUMBER**
允许扩展的数字格式（非标准）：
- 十六进制数字，比如`0x7B`。
- 带有前导或尾随小数点的数字，比如`.123`、`123.`。
- 带有前导加号的数字，比如`+123`。

### **YYJSON_READ_ALLOW_EXT_ESCAPE**
在字符串中允许扩展的转义序列（非标准）
- 一些额外的转义：`\a`、`\e`、`\v`、``\'``、`\?`、`\0`。
- 十六进制转义：形如`\xNN`，比如`\x7B`。
- 续行符: 反斜线后跟随行终止符序列。
- 未知的转义：假如反斜线跟随了一个不支持的字符，
    则反斜线将被删除，跟随的字符原被样保留。
    当然，`\1`-`\9`仍然会触发错误。

### **YYJSON_READ_ALLOW_EXT_WHITESPACE**
允许扩展的空白符（非标准）
- 垂直制表符`\v`和换页符`\f`。
- 行分隔符`\u2028`和段落分隔符`\u2029`。
- 不间断空格`\xA0`。
- 字节顺序标记：`\uFEFF`。
- 其他处于Unicode Zs（空格、分隔符）分类下的字符。

### **YYJSON_READ_ALLOW_SINGLE_QUOTED_STR**
允许使用被单引号括起来的字符串（非标准），比如``'ab'``。

### **YYJSON_READ_ALLOW_UNQUOTED_KEY**
允许对象键不带引号（非标准），例如`{a:1,b:2}`。该选项扩展了 ECMAScript 的标识符名称规则，允许使用任何码点高于`U+007F`的非空白字符。

译者注：这里的意思就是Unicode内超出ASCII范围（高于`U+007F`）的所有非空白字符都可以直接用作对象键，无需加引号；而ASCII范围内的部分符号必须由引号包裹比如`-`，这些符号往往是在JSON中发挥作用的特殊符号

### **YYJSON_READ_JSON5**
允许JSON5格式，有关JSON5请见：https://json5.org。
Allow JSON5 format, see: https://json5.org.

这个标志在支持所有JSON5功能的基础上还额外附加了一些扩展功能：
- 相较于JSON5而言能接收更多的转义序列（比如`\a`、`\e`）。
- 无引号的键不被ECMAScript标识符名称规则所限制。
- 允许大小写不敏感的`NaN`、`Inf`和`Infinity`字面量。

例如：
```json
{
    /* JSON5示例 */
    id: 123,
    name: 'Yumo-sama',
    color_argb: 0x0d000721,
    min: .001,
    max: Inf,
    data: '\x00\xAA\xFF',
}
```

---------------
# 写出JSON
本库提供了五组用于写出JSON的函数。<br/>
每个函数都接受一个JSON文档对象或根值对象（译者注：`yyjson_doc`或`yyjson_val`）作为输入，并返回一个UTF-8字符串或文件。

## 将JSON写入字符串
`doc/val`参数接收一个JSON文档对象或根值对象，假如你传递的是NULL，你得到的返回值也是NULL。<br/>
`flg`参数是写出器标志，如果不需要就传入0，详见[写出器标志](#写出器标志)。<br/>
`len`是一个用来接收输出文本长度的指针（空结束符不算在长度内），如不需要，请传递NULL。
。<br/>
正常情况下函数返回一个新的JSON字符串；如果有错误发生，则会返回NULL。<br/>
字符串以UTF-8编码，尾部带有空结束符。<br/>
在你不需要再使用这个字符串时，需要使用`free()`或`alc->free()`来释放它。

```c
// doc -> str
char *yyjson_write(const yyjson_doc *doc, yyjson_write_flag flg, size_t *len);
// mut_doc -> str
char *yyjson_mut_write(const yyjson_mut_doc *doc, yyjson_write_flag flg, size_t *len);
// val -> str
char *yyjson_val_write(const yyjson_val *val, yyjson_write_flag flg, size_t *len);
// mut_val -> str
char *yyjson_mut_val_write(const yyjson_mut_val *val, yyjson_write_flag flg, size_t *len);
```

示例代码1：

```c
yyjson_doc *doc = yyjson_read("[1,2,3]", 7, 0);
char *json = yyjson_write(doc, YYJSON_WRITE_PRETTY, NULL);
printf("%s\n", json);
free(json);
```

示例代码2：
```c
yyjson_mut_doc *doc = yyjson_mut_doc_new(NULL);
yyjson_mut_val *arr = yyjson_mut_arr(doc);
yyjson_mut_doc_set_root(doc, arr);
yyjson_mut_arr_add_int(doc, arr, 1);
yyjson_mut_arr_add_int(doc, arr, 2);
yyjson_mut_arr_add_int(doc, arr, 3);
    
char *json = yyjson_mut_write(doc, YYJSON_WRITE_PRETTY, NULL);
printf("%s\n", json);
free(json);
```

## 将JSON写入文件
`path`参数指定输出的JSON文件路径，其应当是一个使用系统原生编码的以空终止符结尾的字符串。假如传递的路径是无效的，你将收到一条报错；假如文件是非空的，原本的内容会被覆盖掉。<br/>
`doc/val`参数是JSON文档对象或根值对象，传入NULL则会触发报错。<br/>
`flg`参数是写出器标志，如不需要请传入0，详见[写出器标志](#写出器标志)。<br/>
`alc`参数用于指定内存分配器，如不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`err`参数用于指定一个接受错误信息的指针，如果不需要请传入NULL。<br/>
函数在成功时返回true，发生错误时返回假。<br/>

```c
// doc -> file
bool yyjson_write_file(const char *path, const yyjson_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// mut_doc -> file
bool yyjson_mut_write_file(const char *path, const yyjson_mut_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// val -> file
bool yyjson_val_write_file(const char *path, const yyjson_val *val, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// mut_val -> file
bool yyjson_mut_val_write_file(const char *path, const yyjson_mut_val *val, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
```

示例代码：

```c
yyjson_doc *doc = yyjson_read_file("/tmp/test.json", 0, NULL, NULL);
bool suc = yyjson_write_file("tmp/test.json", doc, YYJSON_WRITE_PRETTY, NULL, NULL);
if (suc) printf("完成");
```

## 将JSON写入文件指针
`fp`参数是用于输出的文件指针，数据将被写到文件的当前位置。<br/>
`doc/val`参数是JSON文档对象或者根值对象，传递NULL将触发错误。<br/>
`flg`参数是写出器标志，如不需要请传入0，详见[写出器标志](#写出器标志)。<br/>
`alc`参数指定内存分配器，如不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`err`参数用于指定一个接受错误信息的指针，如果不需要请传入NULL。<br/>
函数在成功时返回true，发生错误时返回假。<br/>

```c
// doc -> file
bool yyjson_write_fp(FILE *fp, const yyjson_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// mut_doc -> file
bool yyjson_mut_write_fp(FILE *fp, const yyjson_mut_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// val -> file
bool yyjson_val_write_fp(FILE *fp, const yyjson_val *val, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
// mut_val -> file
bool yyjson_mut_val_write_fp(FILE *fp, const yyjson_mut_val *val, yyjson_write_flag flg, const yyjson_alc *alc, yyjson_write_err *err);
```

示例代码：

```c
FILE *fp = fdopen(fd, "wb"); // fd是POSIX文件描述符
bool suc = yyjson_write_fp(fp, doc, YYJSON_WRITE_PRETTY, NULL, NULL);
if (fp) fclose(fp);
if (suc) printf("完成");
```

## 将JSON写入缓冲区
`buf`参数指定输出缓冲区，传递NULL时函数将返回0。<br/>
`buf_len`参数指定输出缓冲区的长度，假如长度过小，函数会失败并返回0。<br/>
`doc/val`参数接收一个JSON文档对象或根值对象，传入NULL则会触发报错。<br/>
`flg`参数是写出器标志，如不需要请传入0，详见[写出器标志](#写出器标志)。<br/>
`err`参数用于指定一个接受错误信息的指针，如果不需要请传入NULL。<br/>
函数正常会返回写出内容的字节长度（包含空终止符），失败时则会返回0。<br/>

该函数不会分配内存，但缓冲区必须大于最终的JSON大小，以为临时操作留出空间。

每个值在写入时都要临时占据一块额外的空间，这块空间将被后面的值重用：
 - 数字： `40`字节
 - 字符串： `16 + (str_len * 6)`字节
 - 其他值： `16`字节
 - 嵌套深度： `16 * max_json_depth`字节

```c
// doc -> buffer
size_t yyjson_write_buf(char *buf, size_t buf_len, const yyjson_doc *doc, yyjson_write_flag flg, yyjson_write_err *err);
// mut_doc -> buffer
size_t yyjson_mut_write_buf(char *buf, size_t buf_len, const yyjson_mut_doc *doc, yyjson_write_flag flg, yyjson_write_err *err);
// val -> buffer
size_t yyjson_val_write_buf(char *buf, size_t buf_len, const yyjson_val *val, yyjson_write_flag flg, yyjson_write_err *err);
// mut_val -> buffer
size_t yyjson_mut_val_write_buf(char *buf, size_t buf_len, const yyjson_mut_val *val, yyjson_write_flag flg, yyjson_write_err *err);
```

示例代码：

```c
char buf[512];
size_t len = yyjson_write_buf(buf, sizeof(buf), doc, YYJSON_WRITE_PRETTY, NULL);
if (len > 0) printf("完成，输出内容：\n%s\n", buf);
```


## 带选项的写出JSON
`doc/val`参数接收一个JSON文档对象或根值对象，假如你传递的是NULL，你得到的返回值也是NULL。<br/>
`flg`参数是写出器标志，如果不需要就传入0，详见[写出器标志](#写出器标志)。<br/>
`alc`参数用于指定内存分配器，如不需要请传入NULL，详见[内存分配器](#内存分配器)。<br/>
`len`是一个用来接收输出文本长度的指针（空结束符不算在长度内），如不需要，请传递NULL。
。<br/>
`err`参数用于指定一个接受错误信息的指针，如果不需要请传入NULL。<br/>

正常情况下函数返回一个新的JSON字符串；如果有错误发生，则会返回NULL。<br/>
字符串以UTF-8编码，尾部带有空结束符。<br/>
在你不需要再使用这个字符串时，需要使用`free()`或`alc->free()`来释放它。

```c
char *yyjson_write_opts(const yyjson_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, size_t *len, yyjson_write_err *err);

char *yyjson_mut_write_opts(const yyjson_mut_doc *doc, yyjson_write_flag flg, const yyjson_alc *alc, size_t *len, yyjson_write_err *err);

char *yyjson_val_write_opts(const yyjson_val *val, yyjson_write_flag flg, const yyjson_alc *alc, size_t *len, yyjson_write_err *err);

char *yyjson_mut_val_write_opts(const yyjson_mut_val *val, yyjson_write_flag flg, const yyjson_alc *alc, size_t *len, yyjson_write_err *err);
```

示例代码：

```c
yyjson_doc *doc = ...;

// 在栈内存上初始化内存分配器
char buf[64 * 1024];
yyjson_alc alc;
yyjson_alc_pool_init(&alc, buf, sizeof(buf));

// 写出
size_t len;
yyjson_write_err err;
char *json = yyjson_write_opts(doc, YYJSON_WRITE_PRETTY | YYJSON_WRITE_ESCAPE_UNICODE, &alc, &len, &err);

// 获取结果
if (json) {
    printf("成功： %lu\n%s\n", len, json);
} else {
    printf("错误：%u msg:%s\n", err.code, err.msg);
}
alc.free(alc.ctx, json);
```


## 写出器标志
本库提供了一组JSON写出器标志。<br/>
你可以单独使用一个标志，或者使用按位或操作符`|`将多个标志结合在一起。

### **YYJSON_WRITE_NOFLAG = 0**
本标志是JSON写出器的默认标志：

- 用压缩格式写出JSON（去除不必要的缩进）
- 处理数字时遇到`inf`或`nan`将报错。
- 处理字符串时遇到无效的Unicode字符将报错。
- 不转义Unicode字符或斜杠。

### **YYJSON_WRITE_PRETTY**
用带有四个空格缩进的更漂亮的格式输出JSON。

### **YYJSON_WRITE_PRETTY_TWO_SPACES**
用带有两个空格缩进的更漂亮的格式输出JSON。
本标志将会覆盖`YYJSON_WRITE_PRETTY`标志。

### **YYJSON_WRITE_ESCAPE_UNICODE**
将Unicode转义为`\uXXXX`格式，保证输出内容只有ASCII字符，例如：

```json
["Alizée, 😊, 小狄同学呀"]
["Aliz\\u00E9e, \\uD83D\\uDE0A, \\u5C0F\\u72C4\\u540C\\u5B66\\u5440"]
```

### **YYJSON_WRITE_ESCAPE_SLASHES**
将正斜杠`/`转义为`\/`，例如：

```json
["https://github.com"]
["https:\/\/github.com"]
```

### **YYJSON_WRITE_ALLOW_INF_AND_NAN**
将inf（无穷大）/nan（非数值）的数字写出为`Infinity`和`NaN`字面值而不是报错。<br/>

注意这个输出**不是**JSON的标准写法且别的JSON库可能无法解析。

示例输出：

```js
{"not_a_number":NaN,"large_number":Infinity}
```

### **YYJSON_WRITE_INF_AND_NAN_AS_NULL**
将inf（无穷大）/nan（非数值）的数字写出为`null`字面量而不是报错。<br/>
本标志将覆盖`YYJSON_WRITE_ALLOW_INF_AND_NAN`标志。

输出示例：

```js
{"not_a_number":null,"large_number":null}
```

### **YYJSON_WRITE_ALLOW_INVALID_UNICODE**
在编码字符串值时允许无效的Unicode。

字符串中无效的字符将会被原样逐字节复制。如果同时还设置了`YYJSON_WRITE_ESCAPE_UNICODE`标志，则无效字符将被转义为`\uFFFD`（替换字符）。

本标志在字符串编码正确时不会影响性能。

### **YYJSON_WRITE_NEWLINE_AT_END**
在JSON的末尾加上一个换行符`\n`。
这对文本编辑器或NDJSON更加友好。

### **YYJSON_WRITE_FP_TO_FLOAT**
使用单精度（float）写出浮点数。
这将会把`double`在序列化前转换为`float`。
用这种方法产生的输出更短但是可能会失去一些精度。
当`YYJSON_WRITE_FP_TO_FIXED(prec)`标志被同时使用时，本标志将失效。

### **YYJSON_WRITE_FP_TO_FIXED(prec)**
使用定点表示法写入浮点数。
这类似于 ECMAScript 的`Number.prototype.toFixed(prec)`，但会移除末尾的零。`prec`的取值范围为 1 到 15。
这样产生的输出更短，但可能会损失一些精度。


---------------
# 访问JSON文档

## JSON文档

你可有用下面的这些函数来访问JSON文档对象的内容：
```c
// 获取JSON文档对象的根值
yyjson_val *yyjson_doc_get_root(yyjson_doc *doc);

// 获取解析JSON时读取的字节数。
// 例如 对于"[1,2,3]"，返回7
size_t yyjson_doc_get_read_size(yyjson_doc *doc);

// 获取JSON文档对象内各种值的总数。
// 例如 对于"[1,2,3]"，返回4（一个数组，三个数字）。
size_t yyjson_doc_get_val_count(yyjson_doc *doc);
```

文档对象持有其内部所有值和字符串的内存。当你不再需要它时，应释放该文档对象以回收所有内存：
```c
// 释放文档对象；假如传入NULL将不会做任何事情。
void yyjson_doc_free(yyjson_doc *doc);
```

## JSON值对象

每一个JSON值对象都有一个类型和子类型，如下表所示：

| 类型             | 子类型                | 解释                     |
| ---------------- | -------------------- | ----------------------- |
| YYJSON_TYPE_NONE |                      | 无效值                   |
| YYJSON_TYPE_RAW  |                      | 原始字符串               |
| YYJSON_TYPE_NULL |                      | `null`字面量             |
| YYJSON_TYPE_BOOL | YYJSON_SUBTYPE_FALSE | `false`字面量            |
| YYJSON_TYPE_BOOL | YYJSON_SUBTYPE_TRUE  | `true`字面量             |
| YYJSON_TYPE_NUM  | YYJSON_SUBTYPE_UINT  | `uint64_t`数字           |
| YYJSON_TYPE_NUM  | YYJSON_SUBTYPE_SINT  | `int64_t`数字            |
| YYJSON_TYPE_NUM  | YYJSON_SUBTYPE_REAL  | `double`数字             |
| YYJSON_TYPE_STR  |                      | 字符串值                 |
| YYJSON_TYPE_STR  | YYJSON_SUBTYPE_NOESC | 字符串值，无转义          |
| YYJSON_TYPE_ARR  |                      | 数组值                   |
| YYJSON_TYPE_OBJ  |                      | 对象值                   |

- `YYJSON_TYPE_NONE`表示无效值，它不会在JSON文档解析成功时出现。
- `YYJSON_TYPE_RAW`只在使用了`YYJSON_READ_XXX_AS_RAW`这类标志时才会出现。
- `YYJSON_SUBTYPE_NOESC`用于优化无需转义的字符串的写出速度。此子类型仅在内部使用，用户无需处理。

下面的函数被用来确定JSON值对象的类型：

```c
// 返回JSON值对象的类型以及子类型。
// 输入时NULL时返回0 。
yyjson_type yyjson_get_type(yyjson_val *val);
yyjson_subtype yyjson_get_subtype(yyjson_val *val);

// 返回值对象的标签，详见`数据结构`文档。
uint8_t yyjson_get_tag(yyjson_val *val);

// 返回类型描述，比如：  
// "null", "string", "array", "object", "true", "false",
// "uint", "sint", "real", "unknown"
const char *yyjson_get_type_desc(yyjson_val *val);

// 当JSON值对象是指定类型时返回true。
// 当输入为NULL或者不是指定类型时返回false。
bool yyjson_is_null(yyjson_val *val);  // 空（null）
bool yyjson_is_true(yyjson_val *val);  // 真（true）
bool yyjson_is_false(yyjson_val *val); // 假（false）
bool yyjson_is_bool(yyjson_val *val);  // 布尔（true/false）
bool yyjson_is_uint(yyjson_val *val);  // 无符号整数（uint64_t）
bool yyjson_is_sint(yyjson_val *val);  // 有符号整数（int64_t）
bool yyjson_is_int(yyjson_val *val);   // 整数（uint64_t/int64_t）
bool yyjson_is_real(yyjson_val *val);  // 小数（double）
bool yyjson_is_num(yyjson_val *val);   // 数字（uint64_t/int64_t/double）
bool yyjson_is_str(yyjson_val *val);   // 字符串（string）
bool yyjson_is_arr(yyjson_val *val);   // 数组（array）
bool yyjson_is_obj(yyjson_val *val);   // 对象（object）
bool yyjson_is_ctn(yyjson_val *val);   // 数组/对象（array/object）
bool yyjson_is_raw(yyjson_val *val);   // 原始字符串（raw string）
```

使用下面的函数可以获取JSON值对象的内容。

```c
// 返回原始字符串，当`val`不是原始字符串时返回NULL。
const char *yyjson_get_raw(yyjson_val *val);

// 返回布尔值，当`val`不是布尔值时返回false。（译者注：我感觉这个设计有点奇葩）
bool yyjson_get_bool(yyjson_val *val);

// 返回uint64_t值，假如`val`不是无符号整数则返回0。
uint64_t yyjson_get_uint(yyjson_val *val);

// 返回int64_t值，假如`val`不是有符号整数则返回0。
int64_t yyjson_get_sint(yyjson_val *val);

// 返回整数值，如果`val`不是有/无符号整数则返回0。
int yyjson_get_int(yyjson_val *val);

// 返回double值，如果`val`不是浮点数则返回0。
double yyjson_get_real(yyjson_val *val);

// 返回（类型转换过的）double值，如果`val`不是整数或浮点数则返回0。
double yyjson_get_num(yyjson_val *val);

// 返回字符串值，如果`val`不是字符串值则返回NULL。
const char *yyjson_get_str(yyjson_val *val);

// 返回内容的长度（字符串的字节长度，数组成员数，
// 对象大小），如果传入的值对象不包含有长度的数据则返回0。
size_t yyjson_get_len(yyjson_val *val);

// 判断传入的值和传入的字符串是否相同。
// 当输入是NULL或`val`不是字符串时，将返回false。
bool yyjson_equals_str(yyjson_val *val, const char *str);
bool yyjson_equals_strn(yyjson_val *val, const char *str, size_t len);
```


下面的函数可以用来修改JSON值对象的内容。<br/>

警告：对于不可变文档，这些函数将会打破“不可变”的约定，你需要谨慎的使用这组API（例如：保证涉及的文档只能在单线程中访问）。

```c
// 将值对象设置成新的类型和内容。
// 当传入NULL、`val`参数是对象或数组时，函数都将返回false。
// 译者注：有点麻烦，要是C支持重载就好了
bool yyjson_set_raw(yyjson_val *val, const char *raw, size_t len);
bool yyjson_set_null(yyjson_val *val);
bool yyjson_set_bool(yyjson_val *val, bool num);
bool yyjson_set_uint(yyjson_val *val, uint64_t num);
bool yyjson_set_sint(yyjson_val *val, int64_t num);
bool yyjson_set_int(yyjson_val *val, int64_t num);
bool yyjson_set_float(yyjson_val *val, float num);
bool yyjson_set_double(yyjson_val *val, double num);
bool yyjson_set_real(yyjson_val *val, double num);

// 字符串不会被复制，需要由调用者持有。
bool yyjson_set_str(yyjson_val *val, const char *str);
bool yyjson_set_strn(yyjson_val *val, const char *str, size_t len);
```


## JSON数组

下面的这些函数可以用来访问JSON数组。<br/>

注意，通过索引访问数组元素可能需要线性搜索时间。（译者注：耗时与数组大小呈线性关系，时间复杂度O(n)。）所以如果你想要遍历整个数组，更推荐使用迭代器API。

```c
// 返回数组的元素数量。
// 当输入不是数组时返回0。
size_t yyjson_arr_size(yyjson_val *arr);

// 返回指定位置的元素（线性搜索时间）。
// 当输入的值不是数组或者索引超出范围时返回NULL。
yyjson_val *yyjson_arr_get(yyjson_val *arr, size_t idx);

// 返回数组的第一个元素（固定时间）。
// 当数组是空的或者输入的值不是数组时返回NULL。
yyjson_val *yyjson_arr_get_first(yyjson_val *arr);

// 返回数组的最后一个元素（线性搜索时间）
// 当数组是空的或者输入的值不是数组时返回NULL。
yyjson_val *yyjson_arr_get_last(yyjson_val *arr);
```

## JSON数组迭代器
遍历数组有下面两种方法：<br/>

示例代码1（迭代器API）：
```c
yyjson_val *arr; // 欲遍历的数组

yyjson_val *val;
yyjson_arr_iter iter = yyjson_arr_iter_with(arr);
while ((val = yyjson_arr_iter_next(&iter))) {
    your_func(val);
}
```

示例代码2 （foreach宏）：
```c
yyjson_val *arr; // 欲遍历的数组

size_t idx, max;
yyjson_val *val;
yyjson_arr_foreach(arr, idx, max, val) {
    your_func(idx, val);
}
```
<br/>

还有一个可变版本API，用来遍历可变数组：<br/>

示例代码1（可变迭代器API）：
```c
yyjson_mut_val *arr; // 欲遍历的数组

yyjson_mut_val *val;
yyjson_mut_arr_iter iter = yyjson_mut_arr_iter_with(arr);
while ((val = yyjson_mut_arr_iter_next(&iter))) {
    if (your_val_is_unused(val)) {
        // 你可以在迭代器中删除当前值
        yyjson_mut_arr_iter_remove(&iter); 
    }
}
```

示例代码2 （可变的foreach宏）：
```c
yyjson_mut_val *arr; // 欲遍历的数组

size_t idx, max;
yyjson_mut_val *val;
yyjson_mut_arr_foreach(arr, idx, max, val) {
    your_func(idx, val);
}
```


## JSON对象
下面的函数可以用来访问JSON对象：<br/>

注意，按对象键访问元素可能需要线性搜索时间。因此，如果需要遍历一个对象，建议使用迭代器API。


```c
// 返回对象中键值对的数量。
// 如果输入不是对象，则返回0。
size_t yyjson_obj_size(yyjson_val *obj);

// 返回指定键所映射的值。
// 如果对象中不包含该键的映射，则返回NULL。
yyjson_val *yyjson_obj_get(yyjson_val *obj, const char *key);
yyjson_val *yyjson_obj_getn(yyjson_val *obj, const char *key, size_t key_len);

// 如果对象键的顺序在编译时已知，
// 你可以使用此方法避免搜索整个对象。
// 例如：{ "x":1, "y":2, "z":3 }
yyjson_val *obj = ...;
yyjson_obj_iter iter = yyjson_obj_iter_with(obj);

yyjson_val *x = yyjson_obj_iter_get(&iter, "x");
yyjson_val *z = yyjson_obj_iter_get(&iter, "z");
```

## JSON对象迭代器
有两种遍历对象的方法：<br/>

示例代码1（迭代器API）：
```c
yyjson_val *obj; // 欲遍历的对象

yyjson_val *key, *val;
yyjson_obj_iter iter = yyjson_obj_iter_with(obj);
while ((key = yyjson_obj_iter_next(&iter))) {
    val = yyjson_obj_iter_get_val(key);
    your_func(key, val);
}
```

示例代码2（foreach宏）：
```c
yyjson_val *obj; // 这是你的对象（想啥呢，不是那个对象）

size_t idx, max;
yyjson_val *key, *val;
yyjson_obj_foreach(obj, idx, max, key, val) {
    your_func(key, val);
}
```
<br/>

还有一个可变版本的API用于遍历可变对象：<br/>

示例代码1（可变迭代器API）：
```c
yyjson_mut_val *obj; // 欲遍历的对象

yyjson_mut_val *key, *val;
yyjson_mut_obj_iter iter = yyjson_mut_obj_iter_with(obj);
while ((key = yyjson_mut_obj_iter_next(&iter))) {
    val = yyjson_mut_obj_iter_get_val(key);
    if (your_key_is_unused(key)) {
        // 你可以在迭代过程中删除当前的键值对
        yyjson_mut_obj_iter_remove(&iter);
    }
}
```

示例代码2（可变foreach宏）：
```c
yyjson_mut_val *obj; // 欲遍历的对象

size_t idx, max;
yyjson_val *key, *val;
yyjson_obj_foreach(obj, idx, max, key, val) {
    your_func(key, val);
}
```

---------------
# 创建JSON文档
`yyjson_mut_doc`及相关API用于构建JSON文档。<br/>

请注意，`yyjson_mut_doc`使用**内存池**来保存所有字符串和值。内存池只能作为一个整体被创建、扩展或释放。因此，`yyjson_mut_doc`更适合一次性写入，而不是对现有文档进行修改。<br/>

JSON对象和数组由链表组成，因此每个`yyjson_mut_val`只能被添加到一个对象或数组中。

示例代码：

```c
// 构建以下JSON：
//     {
//        "page": 123,
//        "names": [ "Harry", "Ron", "Hermione" ]
//     }

// 创建一个可变文档。
yyjson_mut_doc *doc = yyjson_mut_doc_new(NULL);

// 创建一个对象，该值的内存由doc持有。
yyjson_mut_val *root = yyjson_mut_obj(doc);

// 创建键和值，添加到根对象中。
yyjson_mut_val *key = yyjson_mut_str(doc, "page");
yyjson_mut_val *num = yyjson_mut_int(doc, 123);
yyjson_mut_obj_add(root, key, num);

// 创建3个字符串值，添加到数组对象中。
yyjson_mut_val *names = yyjson_mut_arr(doc);
yyjson_mut_val *name1 = yyjson_mut_str(doc, "Harry");
yyjson_mut_val *name2 = yyjson_mut_str(doc, "Ron");
yyjson_mut_val *name3 = yyjson_mut_str(doc, "Hermione");
yyjson_mut_arr_append(names, name1);
yyjson_mut_arr_append(names, name2);
yyjson_mut_arr_append(names, name3);
yyjson_mut_obj_add(root, yyjson_mut_str(doc, "names"), names);

// ❌ 错误！该值已被添加到另一个容器中。
yyjson_mut_obj_add(root, key, name1);

// 设置文档的根值。
yyjson_mut_doc_set_root(doc, root);

// 写入JSON字符串
const char *json = yyjson_mut_write(doc, 0, NULL);

// 释放doc及其创建的所有值的内存。
yyjson_mut_doc_free(doc);
```


## 可变文档

以下函数用于创建、修改、复制和销毁JSON文档。<br/>

```c
// 创建并返回一个新的可变JSON文档。
// 错误时返回NULL（例如内存分配失败）。
// 如果`alc`为NULL，将使用默认分配器。
yyjson_mut_doc *yyjson_mut_doc_new(yyjson_alc *alc);

// 删除JSON文档，释放该doc及其创建的所有值的内存
void yyjson_mut_doc_free(yyjson_mut_doc *doc);

// 设置内部内存池大小（字符串长度和值数量）。
// 可用于为下一次字符串和值创建预留内存。
bool yyjson_mut_doc_set_str_pool_size(yyjson_mut_doc *doc, size_t len);
bool yyjson_mut_doc_set_val_pool_size(yyjson_mut_doc *doc, size_t count);

// 获取或设置此JSON文档的根值。
yyjson_mut_val *yyjson_mut_doc_get_root(yyjson_mut_doc *doc);
void yyjson_mut_doc_set_root(yyjson_mut_doc *doc, yyjson_mut_val *root);

// 从输入复制并返回新的可变文档/值。
// 错误时返回NULL（例如内存分配失败）。

// doc -> mut_doc
yyjson_mut_doc *yyjson_doc_mut_copy(yyjson_doc *doc, const yyjson_alc *alc);
// val -> mut_val
yyjson_mut_val *yyjson_val_mut_copy(yyjson_mut_doc *doc,  yyjson_val *val);
// mut_doc -> mut_doc
yyjson_mut_doc *yyjson_mut_doc_mut_copy(yyjson_mut_doc *doc, const yyjson_alc *alc);
// mut_val -> mut_val
yyjson_mut_val *yyjson_mut_val_mut_copy(yyjson_mut_doc *doc, yyjson_mut_val *val);
// mut_doc -> doc
yyjson_doc *yyjson_mut_doc_imut_copy(yyjson_mut_doc *doc, yyjson_alc *alc);
// mut_val -> doc
yyjson_doc *yyjson_mut_val_imut_copy(yyjson_mut_val *val, yyjson_alc *alc);
```

## JSON值创建
以下函数用于创建可变JSON值，值的内存由文档持有。<br/>

```c
// 创建并返回一个新值，错误时返回NULL。
yyjson_mut_val *yyjson_mut_null(yyjson_mut_doc *doc);
yyjson_mut_val *yyjson_mut_true(yyjson_mut_doc *doc);
yyjson_mut_val *yyjson_mut_false(yyjson_mut_doc *doc);
yyjson_mut_val *yyjson_mut_bool(yyjson_mut_doc *doc, bool val);
yyjson_mut_val *yyjson_mut_uint(yyjson_mut_doc *doc, uint64_t num);
yyjson_mut_val *yyjson_mut_sint(yyjson_mut_doc *doc, int64_t num);
yyjson_mut_val *yyjson_mut_int(yyjson_mut_doc *doc, int64_t num);
yyjson_mut_val *yyjson_mut_float(yyjson_mut_doc *doc, float num);
yyjson_mut_val *yyjson_mut_double(yyjson_mut_doc *doc, double num);
yyjson_mut_val *yyjson_mut_real(yyjson_mut_doc *doc, double num);

// 创建一个字符串值，输入字符串不会被复制。
yyjson_mut_val *yyjson_mut_str(yyjson_mut_doc *doc, const char *str);
yyjson_mut_val *yyjson_mut_strn(yyjson_mut_doc *doc, const char *str, size_t len);

// 创建一个字符串值，输入字符串会被复制并由文档持有。
yyjson_mut_val *yyjson_mut_strcpy(yyjson_mut_doc *doc, const char *str);
yyjson_mut_val *yyjson_mut_strncpy(yyjson_mut_doc *doc, const char *str, size_t len);
```


## JSON数组创建
以下函数用于创建可变JSON数组。<br/>

```c
// 创建并返回一个空的可变数组，错误时返回NULL。
yyjson_mut_val *yyjson_mut_arr(yyjson_mut_doc *doc);

// 使用C数组创建并返回一个可变数组。
yyjson_mut_val *yyjson_mut_arr_with_bool(yyjson_mut_doc *doc, bool *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_sint(yyjson_mut_doc *doc, int64_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_uint(yyjson_mut_doc *doc, uint64_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_real(yyjson_mut_doc *doc, double *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_sint8(yyjson_mut_doc *doc, int8_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_sint16(yyjson_mut_doc *doc, int16_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_sint32(yyjson_mut_doc *doc, int32_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_sint64(yyjson_mut_doc *doc, int64_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_uint8(yyjson_mut_doc *doc, uint8_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_uint16(yyjson_mut_doc *doc, uint16_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_uint32(yyjson_mut_doc *doc, uint32_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_uint64(yyjson_mut_doc *doc, uint64_t *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_float(yyjson_mut_doc *doc, float *vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_double(yyjson_mut_doc *doc, double *vals, size_t count);
// 示例代码：
int vals[3] = {-1, 0, 1};
yyjson_mut_val *arr = yyjson_mut_arr_with_sint32(doc, vals, 3);

// 使用字符串创建并返回一个可变数组，
// 字符串应以UTF-8编码。
yyjson_mut_val *yyjson_mut_arr_with_str(yyjson_mut_doc *doc, const char **vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_strn(yyjson_mut_doc *doc, const char **vals, const size_t *lens, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_strcpy(yyjson_mut_doc *doc, const char **vals, size_t count);
yyjson_mut_val *yyjson_mut_arr_with_strncpy(yyjson_mut_doc *doc, const char **vals, const size_t *lens, size_t count);
// 示例代码：
const char strs[3] = {"Jan", "Feb", "Mar"};
yyjson_mut_val *arr = yyjson_mut_arr_with_str(doc, strs, 3);
```

## JSON数组修改

以下函数用于修改JSON数组的内容。<br/>

```c
// 在数组的指定索引处插入一个值。
// 错误时返回false（例如越界）。
// 注意此函数需要线性搜索时间。
bool yyjson_mut_arr_insert(yyjson_mut_val *arr, yyjson_mut_val *val, size_t idx);

// 在数组末尾插入一个值，错误时返回false。
bool yyjson_mut_arr_append(yyjson_mut_val *arr, yyjson_mut_val *val);

// 在数组头部插入一个值，错误时返回false。
bool yyjson_mut_arr_prepend(yyjson_mut_val *arr, yyjson_mut_val *val);

// 替换指定索引处的值并返回旧值，错误时返回NULL。
// 注意此函数需要线性搜索时间。
yyjson_mut_val *yyjson_mut_arr_replace(yyjson_mut_val *arr, size_t idx, yyjson_mut_val *val);

// 移除并返回指定索引处的值，错误时返回NULL。
// 注意此函数需要线性搜索时间。
yyjson_mut_val *yyjson_mut_arr_remove(yyjson_mut_val *arr, size_t idx);

// 移除并返回数组中的第一个值，错误时返回NULL。
yyjson_mut_val *yyjson_mut_arr_remove_first(yyjson_mut_val *arr);

// 移除并返回数组中的最后一个值，错误时返回NULL。
yyjson_mut_val *yyjson_mut_arr_remove_last(yyjson_mut_val *arr);

// 移除数组中指定范围内的所有值。
// 注意此函数需要线性搜索时间。
bool yyjson_mut_arr_remove_range(yyjson_mut_val *arr, size_t idx, size_t len);

// 移除数组中的所有值。
bool yyjson_mut_arr_clear(yyjson_mut_val *arr);

// 便捷API：
// 在数组末尾添加一个值，错误时返回false。
bool yyjson_mut_arr_add_val(yyjson_mut_val *arr, yyjson_mut_val *val);
bool yyjson_mut_arr_add_null(yyjson_mut_doc *doc, yyjson_mut_val *arr);
bool yyjson_mut_arr_add_true(yyjson_mut_doc *doc, yyjson_mut_val *arr);
bool yyjson_mut_arr_add_false(yyjson_mut_doc *doc, yyjson_mut_val *arr);
bool yyjson_mut_arr_add_bool(yyjson_mut_doc *doc, yyjson_mut_val *arr, bool val);
bool yyjson_mut_arr_add_uint(yyjson_mut_doc *doc, yyjson_mut_val *arr, uint64_t num);
bool yyjson_mut_arr_add_sint(yyjson_mut_doc *doc, yyjson_mut_val *arr, int64_t num);
bool yyjson_mut_arr_add_int(yyjson_mut_doc *doc, yyjson_mut_val *arr, int64_t num);
bool yyjson_mut_arr_add_float(yyjson_mut_doc *doc, yyjson_mut_val *arr, float num);
bool yyjson_mut_arr_add_double(yyjson_mut_doc *doc, yyjson_mut_val *arr, double num);
bool yyjson_mut_arr_add_real(yyjson_mut_doc *doc, yyjson_mut_val *arr, double num);
bool yyjson_mut_arr_add_str(yyjson_mut_doc *doc, yyjson_mut_val *arr, const char *str);
bool yyjson_mut_arr_add_strn(yyjson_mut_doc *doc, yyjson_mut_val *arr, const char *str, size_t len);
bool yyjson_mut_arr_add_strcpy(yyjson_mut_doc *doc, yyjson_mut_val *arr, const char *str);
bool yyjson_mut_arr_add_strncpy(yyjson_mut_doc *doc, yyjson_mut_val *arr, const char *str, size_t len);

// 便捷API：
// 在数组末尾创建并添加一个新数组。
// 返回新数组，错误时返回NULL。
yyjson_mut_val *yyjson_mut_arr_add_arr(yyjson_mut_doc *doc, yyjson_mut_val *arr);

// 便捷API：
// 在数组末尾创建并添加一个新对象。
// 返回新对象，错误时返回NULL。
yyjson_mut_val *yyjson_mut_arr_add_obj(yyjson_mut_doc *doc, yyjson_mut_val *arr);
```

## JSON对象创建
以下函数用于创建可变JSON对象。<br/>

```c
// 创建并返回一个可变对象，错误时返回NULL。
yyjson_mut_val *yyjson_mut_obj(yyjson_mut_doc *doc);

// 使用键和值创建并返回一个可变对象，
// 错误时返回NULL。键和值不会被复制。
// 字符串应以UTF-8编码且带有空终止符。
yyjson_mut_val *yyjson_mut_obj_with_str(yyjson_mut_doc *doc,
                                        const char **keys,
                                        const char **vals,
                                        size_t count);
// 示例代码：
const char keys[] = {"name", "type", "id"};
const char *vals[] = {"Harry", "student", "123456"};
yyjson_mut_obj_with_str(doc, keys, vals, 3);

// 使用键值对创建并返回一个可变对象，
// 错误时返回NULL。键和值不会被复制。
// 字符串应以UTF-8编码且带有空终止符。
yyjson_mut_val *yyjson_mut_obj_with_kv(yyjson_mut_doc *doc,
                                       const char **kv_pairs,
                                       size_t pair_count);
// 示例代码：
const char *pairs[] = {"name", "Harry", "type", "student", "id", "123456"};
yyjson_mut_obj_with_kv(doc, pairs, 3);
```

## JSON对象修改
以下函数用于修改JSON对象的内容。<br/>

```c
// 在对象末尾添加一个键值对。
// 键必须是字符串值。
// 此函数允许对象中存在重复的键。
bool yyjson_mut_obj_add(yyjson_mut_val *obj, yyjson_mut_val *key,yyjson_mut_val *val);

// 向对象添加一个键值对。
// 键必须是字符串值。
// 此函数可能会在添加前移除给定键的所有键值对。
// 注意此函数需要线性搜索时间。
bool yyjson_mut_obj_put(yyjson_mut_val *obj, yyjson_mut_val *key, yyjson_mut_val *val);

// 从对象中移除指定键的键值对。
// 注意此函数需要线性搜索时间。
bool yyjson_mut_obj_remove(yyjson_mut_val *obj, yyjson_mut_val *key);

// 移除对象中的所有键值对。
bool yyjson_mut_obj_clear(yyjson_mut_val *obj);

// 便捷API：
// 在对象末尾添加一个键值对。键不会被复制。
// 注意这些函数允许对象中存在重复的键。
bool yyjson_mut_obj_add_null(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key);
bool yyjson_mut_obj_add_true(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key);
bool yyjson_mut_obj_add_false(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key);
bool yyjson_mut_obj_add_bool(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, bool val);
bool yyjson_mut_obj_add_uint(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, uint64_t val);
bool yyjson_mut_obj_add_sint(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, int64_t val);
bool yyjson_mut_obj_add_int(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, int64_t val);
bool yyjson_mut_obj_add_float(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, float val);
bool yyjson_mut_obj_add_double(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, double val);
bool yyjson_mut_obj_add_real(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, double val);
bool yyjson_mut_obj_add_str(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, const char *val);
bool yyjson_mut_obj_add_strn(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, const char *val, size_t len);
bool yyjson_mut_obj_add_strcpy(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, const char *val);
bool yyjson_mut_obj_add_strncpy(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, const char *val, size_t len);
yyjson_mut_val *yyjson_mut_obj_add_arr(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *_key);
yyjson_mut_val *yyjson_mut_obj_add_obj(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *_key);
                              
// 便捷API：
// 移除指定键的所有键值对。
// 注意此函数需要线性搜索时间。
bool yyjson_mut_obj_remove_str(yyjson_mut_val *obj, const char *key);
bool yyjson_mut_obj_remove_strn(yyjson_mut_val *obj, const char *key, size_t len);

// 便捷API：
// 将所有匹配的键替换为新键。
// 如果至少有一个键被重命名则返回true。
// 此函数需要线性搜索时间。
yyjson_api_inline bool yyjson_mut_obj_rename_key(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, const char *new_key);
yyjson_api_inline bool yyjson_mut_obj_rename_keyn(yyjson_mut_doc *doc, yyjson_mut_val *obj, const char *key, size_t len, const char *new_key, size_t new_len);
```


---------------
# JSON Pointer和Patch

## JSON Pointer
本库支持使用`JSON Pointer`（[RFC 6901](https://tools.ietf.org/html/rfc6901)）查询JSON值。

```c
// `JSON pointer`是一个以空终止符结尾的字符串。
yyjson_val *yyjson_ptr_get(yyjson_val *val, const char *ptr);
yyjson_val *yyjson_doc_ptr_get(yyjson_doc *doc, const char *ptr);
yyjson_mut_val *yyjson_mut_ptr_get(yyjson_mut_val *val, const char *ptr);
yyjson_mut_val *yyjson_mut_doc_ptr_get(yyjson_mut_doc *doc, const char *ptr);

// 带字符串长度的`JSON pointer`，允许内部包含NUL（Unicode U+0000）字符。
yyjson_val *yyjson_ptr_getn(yyjson_val *val, const char *ptr, size_t len);
yyjson_val *yyjson_doc_ptr_getn(yyjson_doc *doc, const char *ptr, size_t len);
yyjson_mut_val *yyjson_mut_ptr_getn(yyjson_mut_val *val, const char *ptr, size_t len);
yyjson_mut_val *yyjson_mut_doc_ptr_getn(yyjson_mut_doc *doc, const char *ptr, size_t len);

// 带字符串长度、上下文和错误信息的`JSON pointer`。
yyjson_val *yyjson_ptr_getx(yyjson_val *val, const char *ptr, size_t len, yyjson_ptr_err *err);
yyjson_val *yyjson_doc_ptr_getx(yyjson_doc *doc, const char *ptr, size_t len, yyjson_ptr_err *err);
yyjson_mut_val *yyjson_mut_ptr_getx(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
yyjson_mut_val *yyjson_mut_doc_ptr_getx(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
```

例如，给定以下JSON文档：
```json
{
    "size" : 3,
    "users" : [
        {"id": 1, "name": "Harry"},
        {"id": 2, "name": "Ron"},
        {"id": 3, "name": "Hermione"}
    ]
}
```
以下JSON字符串将解析为对应的值：

|指针|匹配的值|
|:--|:--|
| `""` | `整个文档` |
| `"/size"`| `3` |
| `"/users/0"` | `{"id": 1, "name": "Harry"}` |
| `"/users/1/name"` | `"Ron"` |
| `"/no_match"` | NULL |
| `"no_slash"` | NULL |
| `"/"` | NULL（匹配空键：root[""]） |

```c
yyjson_doc *doc = ...;
yyjson_val *val = yyjson_doc_ptr_get(doc, "/users/1/name");
printf("%s\n", yyjson_get_str(val)); // Ron

yyjson_ptr_err err;
yyjson_val *val2 = yyjson_doc_ptr_getx(doc, "/", 1, &err);
if (!val2) printf("错误 %d: %s\n", err.code, err.msg); // 错误 3: 无法解析
```

本库还支持使用`JSON Pointer`修改JSON值。
```c
// 添加或插入一个新值。
bool yyjson_mut_ptr_add(yyjson_mut_val *val, const char *ptr, yyjson_mut_val *new_val, yyjson_mut_doc *doc);
bool yyjson_mut_ptr_addn(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_mut_doc *doc);
bool yyjson_mut_ptr_addx(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_mut_doc *doc, bool create_parent, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
                                           
bool yyjson_mut_doc_ptr_add(yyjson_mut_doc *doc, const char *ptr, yyjson_mut_val *new_val);
bool yyjson_mut_doc_ptr_addn(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val);
bool yyjson_mut_doc_ptr_addx(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val, bool create_parent, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);

// 设置一个新值（不存在则添加，存在则替换）。
bool yyjson_mut_ptr_set(yyjson_mut_val *val, const char *ptr, yyjson_mut_val *new_val, yyjson_mut_doc *doc);
bool yyjson_mut_ptr_setn(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_mut_doc *doc);
bool yyjson_mut_ptr_setx(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_mut_doc *doc, bool create_parent, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
                                             
bool yyjson_mut_doc_ptr_set(yyjson_mut_doc *doc, const char *ptr, yyjson_mut_val *new_val);
bool yyjson_mut_doc_ptr_setn(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val);
bool yyjson_mut_doc_ptr_setx(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val, bool create_parent, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);

// 替换一个现有值。
yyjson_mut_val *yyjson_mut_ptr_replace(yyjson_mut_val *val, const char *ptr, yyjson_mut_val *new_val);
yyjson_mut_val *yyjson_mut_ptr_replacen(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val);
yyjson_mut_val *yyjson_mut_ptr_replacex(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
    
yyjson_mut_val *yyjson_mut_doc_ptr_replace(yyjson_mut_doc *doc, const char *ptr, yyjson_mut_val *new_val);
yyjson_mut_val *yyjson_mut_doc_ptr_replacen(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val);
yyjson_mut_val *yyjson_mut_doc_ptr_replacex(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_mut_val *new_val, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);

// 移除一个现有值。
yyjson_mut_val *yyjson_mut_ptr_remove(yyjson_mut_val *val, const char *ptr);
yyjson_mut_val *yyjson_mut_ptr_removen(yyjson_mut_val *val, const char *ptr, size_t len);
yyjson_mut_val *yyjson_mut_ptr_removex(yyjson_mut_val *val, const char *ptr, size_t len, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);

yyjson_mut_val *yyjson_mut_doc_ptr_remove(yyjson_mut_doc *doc, const char *ptr);
yyjson_mut_val *yyjson_mut_doc_ptr_removen(yyjson_mut_doc *doc, const char *ptr, size_t len);
yyjson_mut_val *yyjson_mut_doc_ptr_removex(yyjson_mut_doc *doc, const char *ptr, size_t len, yyjson_ptr_ctx *ctx, yyjson_ptr_err *err);
```

例如：
```c
yyjson_mut_doc *doc = ...;
// doc: {"a":0,"b":[1,2,3]}

yyjson_mut_doc_ptr_set(doc, "/a", yyjson_mut_int(doc, 9));
// 现在: {"a":9,"b":[1,2,3]}

yyjson_mut_doc_ptr_add(doc, "/b/-", yyjson_mut_int(doc, 4));
// 现在: {"a":9,"b":[1,2,3,4]}

yyjson_mut_doc_ptr_remove(doc, "/b");
// 现在: {"a":9}
```

所有以`x`结尾的上述函数可用于获取结果上下文`ctx`和错误信息`err`。例如：
```c
// doc: {"a":0,"b":[null,2,3]}
yyjson_mut_doc *doc = ...;

// 获取错误码和错误信息
yyjson_ptr_err err;
yyjson_mut_doc_ptr_setx(doc, "/b/99", 4, yyjson_mut_int(doc, 99), true, NULL, &err);
if (err.code) printf("错误: %s\n", err.msg); // 错误: 无法解析

// 获取目标值的上下文
// 无需重新解析JSON Pointer即可执行某些操作
yyjson_mut_val *val = yyjson_mut_doc_ptr_getx(doc, "/b/0", 4, &ctx, &err);
if (yyjson_mut_is_null(val)) yyjson_ptr_ctx_remove(&ctx);
// 现在: {"a":0,"b":[2,3]}
```



## JSON Patch
本库支持JSON Patch（RFC 6902）。
规范和示例：<https://tools.ietf.org/html/rfc6902>
```c
// 创建并返回一个打过补丁的JSON值。
// 如果补丁无法应用则返回NULL。
yyjson_mut_val *yyjson_patch(yyjson_mut_doc *doc,
                             yyjson_val *orig,
                             yyjson_val *patch,
                             yyjson_patch_err *err);

yyjson_mut_val *yyjson_mut_patch(yyjson_mut_doc *doc,
                                 yyjson_mut_val *orig,
                                 yyjson_mut_val *patch,
                                 yyjson_patch_err *err);
```


## JSON Merge Patch
本库支持JSON Merge Patch（RFC 7386）。
规范和示例：<https://tools.ietf.org/html/rfc7386>
```c
// 创建并返回一个合并补丁后的JSON值。
// 如果补丁无法应用则返回NULL。
yyjson_mut_val *yyjson_merge_patch(yyjson_mut_doc *doc,
                                   yyjson_val *orig,
                                   yyjson_val *patch);

yyjson_mut_val *yyjson_mut_merge_patch(yyjson_mut_doc *doc,
                                       yyjson_mut_val *orig,
                                       yyjson_mut_val *patch);
```


---------------
# 数字处理

## 数字读取器
本库内置高性能数字读取器，<br/>
默认情况下，它将按以下规则读取数字：<br/>

* 正整数被读取为`uint64_t`。如果发生溢出，则转换为`double`。
* 负整数被读取为`int64_t`。如果发生溢出，则转换为`double`。
* 浮点数被正确四舍五入后读取为`double`。
* 如果`double`数值溢出（达到无穷大），则报告错误。
* 如果数字不符合[JSON](https://www.json.org)标准，则报告错误。

有3个标志可用于调整数字解析策略：

- `YYJSON_READ_ALLOW_INF_AND_NAN`：将nan/inf数字或字面量读取为`double`（非标准）。
- `YYJSON_READ_NUMBER_AS_RAW`：将所有数字作为原始字符串读取而不进行解析。
- `YYJSON_READ_BIGNUM_AS_RAW`：将大数（溢出或无穷大）作为原始字符串读取而不进行解析。

详见`读取器标志`章节。

## 数字写出器
本库内置高性能数字写出器，<br/>
默认情况下，它将按以下规则写出数字：<br/>

* 正整数不带符号写出。
* 负整数带负号写出。
* 浮点数使用[ECMAScript格式](https://www.ecma-international.org/ecma-262/11.0/index.html#sec-numeric-types-number-tostring)写出，但有以下修改：
    * 如果数字是`Infinity`或`NaN`，则报告错误。
    * 保留`-0.0`的负号以保持输入信息。
    * 移除指数部分的正号。
* 浮点数写出器将生成最短的、正确四舍五入的十进制表示。

有几个标志可用于调整数字写出策略：

- `YYJSON_WRITE_ALLOW_INF_AND_NAN`：将inf/nan数字写为`Infinity`和`NaN`字面量而不报错（非标准）。
- `YYJSON_WRITE_INF_AND_NAN_AS_NULL`：将inf/nan数字写为`null`字面量。
- `YYJSON_WRITE_FP_TO_FLOAT`：将实数写为`float`而非`double`。
- `YYJSON_WRITE_FP_TO_FIXED(prec)`：使用定点表示法写出实数。

详见`写出器标志`章节。

还有一些辅助函数可用于控制单个值的输出格式：
- `yyjson_set_fp_to_float(yyjson_val *val, bool flt)`和`yyjson_mut_set_fp_to_float(yyjson_mut_val *val, bool flt)`以`float`或`double`精度写出此实数。
- `yyjson_set_fp_to_fixed(yyjson_val *val, int prec)`和`yyjson_mut_set_fp_to_fixed(yyjson_mut_val *val, int prec)`使用定点表示法写出此实数，prec应在1到15的范围内。

## 数字转换函数

还有两个工具函数可直接访问库内部的数字转换逻辑。
它们用于独立使用，通常不分配内存。
```c
// 从字符串解析数字
const char *yyjson_read_number(const char *dat,
                               yyjson_val *val,
                               yyjson_read_flag flg,
                               const yyjson_alc *alc,
                               yyjson_read_err *err);
// 将数字写入字符串
char *yyjson_write_number(const yyjson_val *val, char *buf);
```



# 文本处理

## 字符编码
默认情况下，本库支持不带BOM的UTF-8编码，如[RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259#section-8.1)所规定：

> 在不属于封闭生态系统的系统之间交换的JSON文本必须使用UTF-8编码。
> 实现不得在网络传输的JSON文本开头添加字节顺序标记（U+FEFF）。

本库默认对输入字符串执行严格的UTF-8编码验证。如果遇到无效字符，将报告错误。

要允许BOM，请使用`YYJSON_READ_ALLOW_BOM`或`YYJSON_READ_ALLOW_EXT_WHITESPACE`标志。

要允许无效的Unicode编码，请使用`YYJSON_READ_ALLOW_INVALID_UNICODE`和`YYJSON_WRITE_ALLOW_INVALID_UNICODE`标志。**注意：** 启用这些标志可能会导致yyjson生成包含无效字符的值，这些值可能被其他代码处理并引入安全风险。

要将字符串标记为在JSON写出时不需要转义，请使用`yyjson_set_str_noesc(yyjson_val *val, bool noesc)`或`yyjson_mut_set_str_noesc(yyjson_mut_val *val, bool noesc)`。这可以提高字符串写出性能并保留原始字符串字节。

## NUL字符
本库支持字符串中的`NUL`字符（也称为`空终止符`，或Unicode `U+0000`，ASCII `\0`）。

读取JSON时，`\u0000`将被转义为`NUL`字符。如果字符串包含`NUL`字符，使用`strlen()`获取的长度将不准确，你应该使用`yyjson_get_len()`获取实际长度。

构建JSON时，输入字符串默认被视为以空终止符结尾。如果需要传入包含`NUL`字符的字符串，应该使用带`n`后缀的API并提供字符串的实际长度。

例如：
```c
// 以空终止符结尾的字符串
yyjson_mut_str(doc, str);
yyjson_obj_get(obj, str);

// 任意字符串，无论是否带有空终止符
yyjson_mut_strn(doc, str, len);
yyjson_obj_getn(obj, str, len);

// C++字符串
std::string sstr = ...;
yyjson_obj_getn(obj, sstr.data(), sstr.length());
```



# 内存分配器
本库不直接调用libc的内存分配函数（malloc/realloc/free）。相反，当需要内存分配时，yyjson的API接受一个名为`alc`的参数，允许调用者传入分配器。如果`alc`为NULL，yyjson将使用默认内存分配器，这是libc函数的简单封装。

使用自定义内存分配器可以让你更好地控制内存分配，以下是几个示例：


## 多个JSON共用单一分配器
如果你需要逐个解析多个小型JSON，可以使用单一分配器来避免多次内存分配。

示例代码：
```c
// 单个JSON的最大数据大小
size_t max_json_size = 64 * 1024;
// 计算单个JSON的最大内存使用量
size_t buf_size = yyjson_read_max_memory_usage(max_json_size, 0);
// 为分配器创建缓冲区
void *buf = malloc(buf_size);
// 使用缓冲区设置分配器
yyjson_alc alc;
yyjson_alc_pool_init(&alc, buf, buf_size);

// 使用一个分配器读取多个JSON
for(int i = 0, i < your_json_file_count; i++) {
    const char *your_json_file_path = ...;
    yyjson_doc *doc = yyjson_read_file(your_json_file_path, 0, &alc, NULL);
    ...
    yyjson_doc_free(doc);
}

// 释放缓冲区
free(buf);
```

如果你不确定处理JSON所需的内存量，可以使用动态分配器。
```c
// 创建动态分配器
yyjson_alc *alc = yyjson_alc_dyn_new();

// 使用一个分配器读取多个JSON
for(int i = 0, i < your_json_file_count; i++) {
    const char *your_json_file_path = ...;
    yyjson_doc *doc = yyjson_read_file(your_json_file_path, 0, alc, NULL);
    ...
    yyjson_doc_free(doc);
}

// 释放分配器
yyjson_alc_dyn_free(alc);
```



## 栈内存分配器
如果JSON足够小，可以使用栈内存来读取或写入。

示例代码：
```c
char buf[128 * 1024]; // 栈缓冲区
yyjson_alc alc;
yyjson_alc_pool_init(&alc, buf, sizeof(buf));

yyjson_doc *doc = yyjson_read_opts(dat, len, 0, &alc, NULL);
...
yyjson_doc_free(doc); // 这是可选的，因为内存位于栈上
```

## 使用第三方分配器库
你可以为yyjson使用第三方高性能内存分配器，例如[jemalloc](https://github.com/jemalloc/jemalloc)、[tcmalloc](https://github.com/google/tcmalloc)、[mimalloc](https://github.com/microsoft/mimalloc)。你也可以参考以下代码实现自己的分配器。

示例代码：
```c
// 使用 https://github.com/microsoft/mimalloc

#include <mimalloc.h>

// 等同于 malloc(size)
static void *priv_malloc(void *ctx, size_t size) {
    return mi_malloc(size);
}

// 等同于 realloc(ptr, size)
// `old_size` 是原来分配的内存大小
static void *priv_realloc(void *ctx, void *ptr, size_t old_size, size_t size) {
    return mi_realloc(ptr, size);
}

// 等同于 free(ptr)
static void priv_free(void *ctx, void *ptr) {
    mi_free(ptr);
}

// 分配器对象
static const yyjson_alc PRIV_ALC = {
    priv_malloc,
    priv_realloc,
    priv_free,
    NULL // `ctx` 将传递给上述函数
};

// 使用自定义分配器读取
yyjson_doc *doc = yyjson_doc_read_opts(dat, len, 0, &PRIV_ALC, NULL);
...
yyjson_doc_free(doc);

// 使用自定义分配器写出
yyjson_alc *alc = &PRIV_ALC;
char *json = yyjson_doc_write(doc, 0, alc, NULL, NULL);
...
alc->free(alc->ctx, json);

```



# 栈内存使用
本库中的大多数函数都使用固定大小的栈内存。这包括JSON读写函数以及JSON Pointer处理函数。

然而，有少数函数使用了递归，如果嵌套层级过深可能会导致栈溢出。这些函数在头文件中会用以下警告标记：
> @warning
> 本函数使用了递归，如果对象层级过深可能会导致栈溢出。



# 空值检查
本库的公共API会对每个输入参数执行`空值检查`以防止程序崩溃。

例如，在读取JSON时，你就不需要亲自对每个值进行空值检查或类型检查：
```c
yyjson_doc *doc = yyjson_read(NULL, 0, 0); // doc为NULL
yyjson_val *val = yyjson_doc_get_root(doc); // val为NULL
const char *str = yyjson_get_str(val); // str为NULL
if (!str) printf("err!");
yyjson_doc_free(doc); // 什么都不做
```

然而，如果你十分确定某个值非空且为符合预期的类型，可以使用带有`unsafe`前缀的API来跳过空值检查。

例如，在遍历数组或对象时，值和键必定非空：
```c
size_t idx, max;
yyjson_val *key, *val;
yyjson_obj_foreach(obj, idx, max, key, val) {
    // 这是一个合法的JSON，所以键必定是合法的字符串
    if (unsafe_yyjson_equals_str(key, "id") &&
        unsafe_yyjson_is_uint(val) &&
        unsafe_yyjson_get_uint(val) == 1234) {
        ...
    }
}
```



# 线程安全
本库不使用全局变量。因此如果你能确保输入函数的实参是线程安全的，那么函数调用就也是线程安全的。<br/>

一般来说，`yyjson_doc`和`yyjson_val`是不可变的且线程安全的，而`yyjson_mut_doc`和`yyjson_mut_val`是可变的且非线程安全的。



# 区域设置独立性
本库设计上是区域设置无关的。（译者注：库的运行效果和你所处的国家或地区无关）

然而，有一些情况你需要注意避免：

1. 当你在代码中使用了libc的`setlocale()`函数来更改区域设置。
2. 当你的运行环境不遵循IEEE 754浮点数标准（例如某些IBM大型机），或者你在构建时显式设置了`YYJSON_DISABLE_FAST_FP_CONV`，在这种情况下yyjson将使用`strtod()`来解析浮点数。

如果**同时**满足这两个条件，请避免在某个线程正在解析JSON时调用`setlocale()`。否则当解析到JSON中的浮点数时可能会返回错误。
