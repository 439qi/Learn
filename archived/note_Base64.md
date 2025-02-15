<!-- 2023.10.13 created -->
# Base64 Encode

## 用途

由于历史原因，部分系统不支持非ASCII码传输  
最高位为1的字节被视为传输错误，会被置1或丢弃，因而需要将其他编码的字符映射到ASCII码进行传输，接收后再反向映射回原先编码  
Base64即为其中一种映射规则

## 定义

将任意符号映射到a-z，A-Z，0-9，+，/共64个字符集

> 实际上还有一个=

|索引|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32|33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49|50|51|52|53|54|55|56|57|58|59|60|61|62|63|
|-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |-- |
|字符|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z|a|b|c|d|e|f|g|h|i|j|k|l|m|n|o|p|q|r|s|t|u|v|w|x|y|z|0|1|2|3|4|5|6|7|8|9|+|/|

## 转换流程

将要转换的数据转换为二进制序列，每6个bit分为一组，并将高2位补0，重新组成一个Byte，即原本每3个Byte转换为4个Byte，并对每一Byte查表映射到Base64字符集
`00xxxxxx 00xxxxxx 00xxxxxx 00xxxxxx`

对于原二进制序列非3Byte整数倍的情况

1. 2Byte
   转换后第三个字符仅有4bit，低2位和高2位补0，最后一个字符为=
   `00xxxxxx 00xxxxxx 00xxxx00 =`
2. 1Byte
   转换后第二个字符仅有2bit，低4位和高2位补0，最后两个字符为=
   `00xxxxxx 00xx0000 = =`
