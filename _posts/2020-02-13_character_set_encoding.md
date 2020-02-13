---
title: 理解字符集编码及Java内存编码运行机制
date: 2020-02-13 22:15:32
tags: encoding
permalink: character-set-encoding
keywords: 字符集编码, java内存
rId: MB-20021301
---

## 常见编码详悉

### Unicode和UTF-8的关系

Unicode又称万国码，它将全世界大部分语言的常用字符都纳入了编码范围。Unicode和UTF-8的关系简单的来说可以理解为：

* Unicode是字符集
* UTF-8是编码规则

字符集：为每一个“字符”分配一个唯一的数值（ID，或称码点、码位），即给汉字、英文等各种字符分配一个不重复的数值

编码规则：将“码点”转换为字节序列的规则（数值转化为字节值的过程）



Unicode字符集为每一个字符分配一个码点，例如：“中”的码位是20013，记作U+4E2D（20013的16进制是4E2D）

UTF-8是一套以8位为一个编码单位的可变长编码，会将一个码点编码为1到4个字节（广义上的UTF-8），编码规则如下：

```
  U+0000 ~   U+007F: 0XXXXXXX
  U+0080 ~   U+07FF: 110XXXXX 10XXXXXX
  U+0800 ~   U+FFFF: 1110XXXX 10XXXXXX 10XXXXXX
 U+10000 ~ U+10FFFF: 11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

根据上表可知，“中”字的码点U+4E2D属于第三行范围：

```184
       4    E    2    D
‭    0100 1110 0010 1101‬    二进制值
------------------------
    0100   111000   101101 二进制值重新对齐
1110XXXX 10XXXXXX 10XXXXXX 模板
11100100 10111000 10101101 代入模板后值
     228      184      173 字节值(byte，负值处理为正值了+256)
```



我们可以用java编写程序实现unicode值（16进制表示）和字符的转换，如下所示

```
public class UnicodeCoder {
	/** 字符串转为unicode值 */
	public static String encode(String str) {
		StringBuilder sb = new StringBuilder();
		char[] charArray = str.toCharArray();
		for (int i = 0; i < charArray.length; i++) {
			char c = charArray[i];
			sb.append(charToUnicodeString(c));
		}
		return sb.toString();
	}
	/** unicode值转为字符串 */
	public static String decode(String str) throws UnsupportDecodeException {
		StringBuilder sb = new StringBuilder();
		char[] charArray = str.toCharArray();
		for (int i = 0; i < charArray.length; i++) {
			char c = charArray[i];
			if (c == '\\') {
				if (i == charArray.length - 1) {
					throw new UnsupportDecodeException();
				}
				if (charArray[i + 1] == '\\') {
					sb.append("\\");
					i++;
				} else if (charArray[i + 1] == 'u') {
					if (i >= charArray.length - 5 || !isHexChar(charArray[i + 2])
							 || !isHexChar(charArray[i + 3]) || !isHexChar(charArray[i + 4])
							 || !isHexChar(charArray[i + 5])) {
						throw new UnsupportDecodeException();
					}
					String hexInt = "" + charArray[i + 2] + charArray[i + 3] + charArray[i + 4] + charArray[i + 5];
					char v = (char) Integer.valueOf(hexInt, 16).intValue();
					sb.append(v);
					i += 5;
				} else {
				}
			} else {
				sb.append(c);
			}
		}
		return sb.toString();
	}
	private static boolean isHexChar(char c) {
		if (c >= '0' && c <= '9') return true;
		if (c >= 'a' && c <= 'f') return true;
		if (c >= 'A' && c <= 'F') return true;
		return false;
	}
	private static String charToUnicodeString(char c) {
		if (c < 0x100) {
			if (c == '\\') {
				return "\\\\";
			} else {
				return c + "";
			}
		}
		String hex = Integer.toHexString(c);
		if (c >= 0x1000) {
			return "\\u" + hex;
		} else {
			return "\\u0" + hex;
		}
	}
	private static class UnsupportDecodeException extends Exception {
		private static final long serialVersionUID = 10000L;
		public UnsupportDecodeException() {
		}
	}
	public static void main(String[] args) throws UnsupportDecodeException {
        System.out.println(encode("中")); // \u4e2d
		System.out.println("\u4e2d"); // 不需要转换，jvm自动处理
		System.out.println(decode("\\u4e2d")); // 中
	}
}
```

### UTF-8和UTF-8MB4关系

Unicode字符集，实际上有两个系列，分为2个字节和4个字节的两种，2个字节的Unicode字符集称为`UCS-2`（通常说的Unicode大都是`UCS-2`），4个字节的Unicode字符集又称为`UCS-4`，`UCS-4`是`UCS-2`的超集。`UCS-2`可容纳65536个码位，`UCS-4`目前可用码位分为了17个平面（每个平面可容纳65536个码位，即一共1114112个码位，第1个平面被成为基本多语言平面，即`UCS-2`的字符集，其他16个平面统称为辅助平面），它为现有的所有文字和符号以及将来可能出现的字符都指定（或预留）了一个唯一的数字编码。

**UTF-8MB3**

通常所说的UTF-8（即狭义上的UTF-8），实际上是UTF-8MB3，一个字符占1-3个字节（只有编码规则的前三行），它是以`UCS-2`为字符集的编码规则。

**UTF-8MB4**

除了UTF-8MB3以外，还存在UTF-8MB4编码，一个字符占1-4个字节，可以表示超过65536个字符（没能达到1114112个，仅能表示20多万个），因此有能力表示`UCS-4`中的部分字符。



### UTF-8、UTF-16和GBK、GB2312的区别

UFT-8、UTF-16、GBK、GB2312都能编码汉字，但在汉字的支持数量上、以及字符集上有所差异。

**GB2312**

GB2312是由国内制定，在Ascii码的基础上扩充汉字编码制定的字符集与编码规则，共收录了6763个常用汉字，英文占一个字节，汉字占两个字节。

**GBK**

GBK编码，扩展了GB2312编码收录的字符数量，并沿用了GB2312的码位（是GB2312编码的超集），收录了21003个汉字，英文占一个字节，汉字占两个字节。向下兼容GB2312编码。

**GB18030**

GB18030编码，在GBK的基础上再次扩充了汉字字符数量，并增加了少数民族字符，收录了70244个汉字。与GB2312-1980完全向后兼容，与GBK基本兼容，并支持Unicode的所有码位。编码包含三种长度：单子节ASCII，双字节的GBK（略带扩展）、以及用于填补所有Unicode码位的四字节UTF区块（Unicode码位数1114112少于GB18030的161668个码位）。

**BIG5**

BIG5编码是台湾制定的基于ASCII扩充的中文编码规则，但其与GB2312字符集码位不是一个体系。



**UTF-8**

就像GBK编码一样，各国都对ASCII进行扩展定义了自己的编码，不利于国际间文件交换。Unicode编码是在这种背景下，国际标准组织将世界各国语言都纳入了编码体系，形成的一个字符集。前面说过Unicode有`UCS-2`和`UCS-4`两种，`UCS-2`汉字编码范围在U+4E00-9FA5，包含20928个汉字，容纳中日韩（CJK）统一编码的汉字共27484个，而`UCS-4`的第1个平面即是`UCS-2`的码位，所以包含的汉字数量必然大于`UCS-2`（具体值没找到资料）。

UTF-8编码，以`UCS-2`字符集为基础的编码（UTF-8MB4除外，UTF-8MB4字符集已经超过`UCS-2`，包含部分`UCS-4`字符）。

**UTF-16**

UTF-16与UTF-8一样，是众多UTF(UCS Transfer Format)标准中的一个，都是以`UCS-4`为字符集的编码规则，有2个字节和4个字节两种长度，2个字节的用于编码“基本多语言平面中的字符”（即`UCS-2`部分），4个字符编码其余辅助平面的字符。



UTF-8和UTF-16算是一个字符集体系的编码，GB系列算一个字符集体系的编码。



## Java中的内存编码

Java内存唯一使用的编码是`Unicode`（`UCS-2`字符集，注意是内存编码，不是运行时与用户交互的编码）。平时调用的`String.getBytes()`方法，实际上就是将内存中字符的码点按指定编码规则组织为bytes数组，同理，`new String(bytes)`方法，即是将bytes数组按指定编码规则翻译成码点存在内存中。


### 前端输入的emoji字符由UTF-8MB4编码，传到后端Java是如何处理的呢？

Java采取的策略是将其转为两个Utf8mb3字符。具体转换过程如下：

1. **首先读取字节，如以😭表情为例，读取到如下字节（负数已处理为正数）**
```
240       159       152       173
```
二进制表示为
```
bytes[0]  bytes[1]  bytes[2]  bytes[3]
‭11110000‬  ‭10011111‬  ‭10011000‬  ‭10101101‬
```
由于值是以utf8mb4编码的
```
11110xxx  10xxxxxx  10xxxxxx  10xxxxxx    二进制值模板
     000    011111    011000‬    101101‬    抽离模板后得到码点值的二进制表示
```
码点计算公式为
```
codePoint = ((bytes[0] & 0x7) << 18) + ((bytes[1] & 0x3f) << 12) + (bytes[2] & 0x3f) << 6) + (bytes[3] & 0x3f)
```

2. **将utf8mb4字符拆成为两个字符**
步骤如下
* 先将码点值减掉(2^16)，并去掉第1位
```
val = codePoint - (1 << 16)
val = val & 0xFFFFF;
     000    001111    011000    101101        计算后的二进制值
      00    001111    011000    101101        去掉第1位
```
* 然后下面二进制值为模板依次填充x的位置
```
11101101  1010xxxx  10xxxxxx    11101101  1011xxxx  10xxxxxx    固定转化模板（后面解释）
11101101  10100000  10111101    11101101  10111000  10101101    填充后二进制值
```
* 再计算新的码点，此时是两个三字节utf8，所以模板如下
```
1110xxxx  10xxxxxx  10xxxxxx    1110xxxx  10xxxxxx  10xxxxxx    三字节格式二进制值模板
    1101    100000    111101        1101    111000    101101    抽离模板后值
```
* 整理二进制位，得到两个新的unicode码点：D83D, DE2D
```
11011000  00111101              11011110  00101101
D83D                            DE2D
```

上面的转换步骤稍加整理后，可以得到公式：

```
新码点的值计算：
val = ((bytes[0] & 0x7) << 18) + ((bytes[1] & 0x63) << 12) + (bytes[2] & 0x63) << 6) + (bytes[3] & 0x63) - (1 << 16)
val = val & 0xFFFFF
// 110110xxxxxxxxxx 110111xxxxxxxxxx    来自固定转化模板的值
// 0b110110 = 54
// 54 << 10 = 0xD800
codePointN[0] = (val >>> 10) + 0xD800
// 0b110111 = 55
// 55 << 10 = 0xDC00
codePointN[1] = (val & 0x3FF) + 0xDC00
```



**拆分字符时使用的固定转化模板为什么是这个？**

**其一** 3字节utf8，固定了一些位

```
1110xxxx  10xxxxxx  10xxxxxx    1110xxxx  10xxxxxx  10xxxxxx
```

**其二** 新码点不能与utf8mb3冲突，所以选用为utf16永久保留不映射的码点区间(`0xD800-0xDBFF`和`0xDC00-0xDFFF`)作为新值所在的区间（两个区间的区别只是与大端和小端的规定有关）
因此又可以固定剩余几位（`D8`二进制为‭`11011000`‬，`DB`二进制为‭`11011011`‬，`DC`二进制为`‭11011100`‬，`DF`二进制为`‭11011111`‬）

```
11101101  1010xxxx  10xxxxxx    11101101  1011xxxx  10xxxxxx
```

**关于码点值减掉(2^16)**

参见JDK中的`java.lang.Character#highSurrogate`方法，这是高位计算，相应的，地位计算方法见`java.lang.Character#lowSurrogate`方法，调用点见`sun.nio.cs.UTF_8.Decoder#decode`方法



### 关于内存编码处理的测试代码
```
	...
	public static void main(String[] args) throws IOException, UnsupportDecodeException {
		byte[] bytes = new byte[10];
		int read = System.in.read(bytes);
		System.out.println("读取byte如下 --> ");
		for (int i = 0; i < read; i++) { // 最后一个打印10为回车键
			System.out.println(bytes[i] & 0xff);
		}
		System.out.println("读取内容如下 --> ");
		System.out.println(new String(bytes, 0, read));
		System.out.println("-----------------");
		int val = ((bytes[0] & 0x7) << 18) + ((bytes[1] & 0x3f) << 12) + ((bytes[2] & 0x3f) << 6) + (bytes[3] & 0x3f) - (1 << 16);
		val = val & 0xFFFFF;
		int high = (val >>> 10) + 0xD800;
		int low = (val & 0x3FF) + 0xDC00;
		System.out.println(Integer.toHexString((high & 0xffff) >> 8));
		System.out.println(Integer.toHexString(high & 0xff));
		System.out.println(Integer.toHexString((low & 0xffff) >> 8));
		System.out.println(Integer.toHexString(low & 0xff));
		System.out.println("-----------------");

		System.out.println();
		System.out.println("重复输入一次，验证unicode编码");
		Scanner scanner = new Scanner(System.in);
		if (scanner.hasNext()) {
			String next = scanner.next();
			System.out.println();
			System.out.println(encode(next));
		}
	}
```



## MySQL中的编码

了解了码点之后以及Java编码转化的本质后，应该能够明白MySQL中，客户端与服务器端编码不同，首先字符集就已经不同，存入数据库之后乱码也就不难理解了。

在MySQL数据库中，新建数据库的时候会要求选择字符集和排序规则，这里的字符集就是本文所说的编码规则，而排序规则，则是在SQL查询中对文本排序时的排序依据（可以类比为Java中`java.util.Comparator`的实现），比如：是按ascii码排序、还是按汉字拼音首字母排序、或者按德语字母顺序排序、或者排序时大小写不同是否看成同一个字符。

以常用的`utf8mb4_general_ci`、`utf8mb4_unicode_ci`和`utf8mb4_bin`为例：`utf8mb4_unicode_ci`针对各语言做了比较复杂的处理，在各语言中的排序更加准确（主要是在德语、法语等中有影响，对中文、英文没影响），排序速度也比更慢；`utf8mb4_general_ci`大小写不敏感，相比`utf8mb4_unicode_ci`排序速度也更快，`utf8mb4_bin`大小写敏感。




