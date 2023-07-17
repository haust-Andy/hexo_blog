---
application.properties和application.yml文件的区别：
---
## application.properties和application.yml文件的区别：
1、application.properties的优先级高于application.yml

【原因：当两者同时存在时，application.yml先执行，而application.properties后执行，后执行的application.properties文件会覆盖先执行的application.yml文件内容（只覆盖相同的内容）】

2、application.properties使用" = "赋值，application.yml使用"  ："赋值，且冒号与属性值之间必须有一个空格（属性:  属性值）

3、application.yml文件需要缩进时，只能使用空格键缩进，不能使用tab键

最后，application.yml文件以树型结构，其的可读性更高，结构清晰明了，建议使用application.yml
————————————————
版权声明：本文为CSDN博主「梅秃头」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_53376718/article/details/129877796