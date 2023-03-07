# UTF8与UTF8MB4

## 内容

官方手册 https://dev.mysql.com/doc/refman/5.6/en/charset-unicode-utf8mb4.html 的说明：

>The character set named utf8 uses a maximum of three bytes per character and contains only BMP characters. The utf8mb4 character set uses a maximum of four bytes per character supports supplementary characters:
>
>- For a BMP character, utf8 and utf8mb4 have identical storage characteristics: same code values, same encoding, same length.
>- **For a supplementary character, utf8 cannot store the character at all, whereas utf8mb4 requires four bytes to store it.** Because utf8 cannot store the character at all, you have no supplementary characters in utf8 columns and need not worry about converting characters or losing data when upgrading utf8 data from older versions of MySQL.

即：三字节内容，MYSQL版UTF8与UTF8MB4完全一致，但是UTF8完全不能存储“拓展集”而UTF8MB4就可以做到。

## 资料

[mysql使用utf8mb4经验吐血总结_胡庚申的博客-CSDN博客_mysql utf8mb4](https://blog.csdn.net/qq_17555933/article/details/101445526)
