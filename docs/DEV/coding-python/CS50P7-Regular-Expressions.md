# CS50P: 7. Regular Expressions

## Regular Expressions / Regexes

### 判断用户是否输入邮箱地址📮 

```python
email = input("What's your email? ").strip()
username, domain = email.split('@')
if username and domain.endswith(".edu"):
    print("Valid")
else:
    print("Invalid")
```

`if '@' in email` 判断字符串中是否有某字符

### re

[python's library: re](https://docs.python.org/3/library/re.html)

#### re.search( , , )

```python
re.search(pattern, string, flags=0)
```

在 `string` 中查找一个 `pattern` 

### patterns

#### 基础符号

![截屏2024-07-21 15.23.01](./image/%E6%88%AA%E5%B1%8F2024-07-21%2015.23.01.png){width=70%}

`.` 表示任意 character

`.*` 表示 `(空)` 或 `.` 或 `..` 等等

例如：

```python
if re.search(".*@.*", email):
```

finite state machine 运行机制：start $\Rightarrow$ `.` 匹配 `c-h-a-s-e` 循环 $\Rightarrow$ 匹配 `@` $\Rightarrow$ 匹配 `163.com` $\Rightarrow$ 双线圈，结束，表达式有效

![截屏2024-07-21 15.37.45](./image/%E6%88%AA%E5%B1%8F2024-07-21%2015.37.45.png){width=70%}

#### raw string

```python
if re.search(r".+@.+\.edu", email):
```

如果想表达 `.`（真实存在于文本中的），需要 `r` 和 `\` (tell python not treat . as a special sign)

注意：输入 `chase@buaa.edu.` 仍会返回 `valid` ，因为没有要求以 `.edu` 结尾

#### 进阶符号

![截屏2024-07-21 16.05.31](./image/%E6%88%AA%E5%B1%8F2024-07-21%2016.05.31.png){width=70%}

```python
if re.search(r"^.+@.+\.edu$", email):
```

![截屏2024-07-21 16.05.41](./image/%E6%88%AA%E5%B1%8F2024-07-21%2016.05.41.png){width=70%}

`[abcd]` 表示只匹配 abcd 这几个字母，`[^a]` 表示当前序列不能有 a

```python
if re.search(r"^[^@]+@[^@]+\.edu$", email):
```

缩小范围：

```python
if re.search(r"^[a-zA-Z0-9_]+@[a-zA-Z0-9_]+\.edu$", email):
```

`[]` 中间不要加空格，逗号等

![截屏2024-07-21 16.05.49](./image/%E6%88%AA%E5%B1%8F2024-07-21%2016.05.49.png){width=70%}

`\w` 相当于 `[a-zA-Z0-9_]`

![截屏2024-07-21 16.33.12](./image/%E6%88%AA%E5%B1%8F2024-07-21%2016.33.12.png){width=70%}

例如：`(edu|org|gov|com|net)` 

## Case Sensitivity

### flags

```bash
re.IGNORECASE
re.MULTILINE
re.DOTALL
```

`re.DOTALL` ：`.` 可以匹配换行

使用：`if re.search(r"^\w+@\w+\.edu$", email, re.IGNORECASE):`

### ..?(two dots)

@subdomain.domain.tld ? (tld = top level domain)

```python
if re.search(r"^\w+@(\w+\.)?\w+\.edu$", email, re.IGNORECASE):
```

`(\w+\.)?` 表示 `name.` 可以出现0次或1次

### validate an email address

最终版

```bash
^[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$
```

## re--Plus

[python: re](https://docs.python.org/3/library/re.html)

### re.match( , , )

```python
re.match(pattern, string, flags=0)
```

自动从开头匹配

### re.fullmatch( , , )

```python
re.fullmatch(pattern, string, flags=0)
```

自动匹配开头至结尾

## Clean Up User Input

Q: user输入 `Malan, David` ，要求返回 `hello, David Malan` 

```python
import re
name = input("What's your name? ").strip()
matches = re.search(r"^(.+), *(.+)$", name)
if matches:
    last, first = matches.groups()
    name = f"{first} {last}"
print(f"hello, {name}")
```

- regular expressions 需要引进 re 库
- re.search( , , ) 会返回很多信息
- 括起来的部分会被 captured ，用 `.groups()` 可以得到

line 5~6 可替换为：

```python
name = matches.group(2) + " " + matches.group(1)
```

- 下标从1起

line 3~4 ：

```python
if matches := re.search(r"^(.+), *(.+)$", name):
```

`:=` (walrus operator)assign sth. from right to left & ask boolean question

同时获得 capture 括号的值，并询问 boolean question

## Extract User Input

Q: prompt users for the url of their Twitter profile, and get their username

思路：https://twitter.com/xxx 中的 xxx 就是 username

问题：

- 输入 `www.twitter.com/`
- `xxx/`
- `https` or `http`
- 其他无关输入 `My username is ...`

### re.sub( , , , )

```python
re.sub(pattern, repl, string, count=0, flags=0)
```

pattern: 想替换的部分

repl: 替换成repl

返回新字符串

```python
url = input("URL: ").strip()
username = re.sub(r"^(https?://)?(www\.)?twitter\.com/", "", url)
```

### re.search( , , )

```python
if matches := re.search(r"^(https?://)?(www\.)?twitter\.com/([a-z0-9_]+)", url, re.IGNORECASE):
    print(f"Username:", matches.group(3))	# attention
```

进一步：

```python
if matches := re.search(r"^(?:https?://)?(?:www\.)?twitter\.com/([a-z0-9_]+)", url, re.IGNORECASE):
    print(f"Username:", matches.group(3))
```

`(?:xxx)`

- yes: use `()` to group

- no: capture

> You will not be happy if you try to write out a whole complicated regular expression all at once. Just take these baby steps and make sure it works. You add one more feature make sure it works and hopefully by the end, because you’ve done each of those steps one at a time, the whole thing will make sense to you.  --David Malan