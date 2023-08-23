# translate

## 功能

将给定字符串 `source` 中出现在 `from_string` 中的字符替换为对应位置的 `to_string` 中的字符。

如果被替换后的结果字符串的长度超过了 `VARCHAR` 类型的最大长度（1048576），那么结果字符串将会是 `NULL`。

## 语法

```SQL
TRANSLATE( <source>, <from_string>, <to_string> )
```

## 参数说明

- `str`: 支持的数据类型为 `VARCHAR`。要进行字符替换的字符串。对于不在 `from_string` 中的字符，直接在结果字符串中原样输出。

- `from_string`: 支持的数据类型为 `VARCHAR`。每个 `from_string` 中的字符，
  - 如果在 `to_string` 中有对应位置的字符，那么替换为该字符。
  - 如果在 `to_string` 中没有对应位置的字符（`to_string` 的字符长度小于 `from_string`），那么在结果字符串删除这个字符。

- `to_string`：支持的数据类型为 `VARCHAR`。用于替换 `from_string` 中对应位置的字符。如果 `to_string` 的字符长度大于 `from_string`，那么这部分多余的字符被忽略掉，不会有任何影响。

## 返回值说明

返回的数值类型为 `VARCHAR`。

## 示例

```SQL
MySQL > select translate('s1m1a1rrfcks','mf1','to') as test;

+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

MySQL > select translate('测abc试忽略', '测试a忽略', 'CS1') as test;
+-------+
| test  |
+-------+
| C1bcS |
+-------+

MySQL > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

MySQL > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

MySQL > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

MySQL > select translate(concat('b', repeat('a', 1024*1024-3)), 'a', '膨') as test;
+--------+
| test   |
+--------+
| <null> |
+--------+

select length(translate(concat('b', repeat('a', 1024*1024-3)), 'b', '膨')) as test
+---------+
| test    |
+---------+
| 1048576 |
+---------+

```
