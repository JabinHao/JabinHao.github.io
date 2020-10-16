---
title: python与shebang
date: 2020-10-16 16:40:14
tags: [python, linux]
---
# 简介
在Python脚本的第一行，常常能看到 `#!/usr/bin/env python3` 或者 `#!/usr/bin/python3` 字样，其中#!符号在计算机行业中叫做 “Shebang”,  其作用是指定由哪个解释器来执行脚本。在这里即是指定python3作为解释器。

# 使用：指定解释器
windows系统是根据文件后缀决定打开方式的，因此首行Shebang是没有用的，只有类unix系统才是根据文件头决定脚本运行方式。

### 在类Unix 系统中 ：

* 通过命令行形式指定解释器： `python3 ./script.py`，这种方式脚本中就可以不添加Shebang行；
* 通过脚本的Shebang来指定解释器： ./script.py，这种方式就需要脚本的第一行如果写上 `#`!/usr/bin/python3` 或者是`#!/usr/bin/env python3`,  shell 会检查脚本的第一行代码, 发现有Shebang, 会按其指定的解释器来执行，在这里就是用python3 解释器来执行;  
* 命令行指定要比Shebang指定优先级更高：当脚本里写上 `#!/usr/bin/python3` 或者是 `#!/usr/bin/env python3`, 但是在命令行输入python2 ./script.py，最终是以python2解释器来执行。


### 注意两点：

* `#!` 之后的空格是可选的, #!/usr/bin/env python3 和 #! /usr/bin/env python3 这两种写法都可以；
* 通过命令行指定解释器执行文件是不必写Shebang的, 只有被直接执行的文件才有必要加入`Shebang`。
# 区别
* #!/usr/bin/python3 采用了绝对路径的写法，即指定了采用/usr/bin/python3该路径下的解释器来执行脚本。如果python3解释器不在该路径下的话（用anaconda安装的话有可能不在），./script.py 就无法运行。
* `*#!/usr/bin/env python3` 的写法指定从PATH环境变量中查找python解释器的位置，因此只要环境变量中存在，该脚本即可执行。所以一般情况下采用 `#!/usr/bin/env python3` 的写法更好，容错率更高。
