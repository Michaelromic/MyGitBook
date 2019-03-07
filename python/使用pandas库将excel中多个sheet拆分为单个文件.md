# 使用pandas库将excel中多个sheet拆分为单个文件

```
\# -*- coding: utf-8 -*-
import pandas as pd
import os

source_file = 'D:/mmaww/source.xlsx'
target_dir = 'D:/mmaww/targetdir/'

d_read = pd.read_excel(source_file,None)

\# 获取源文件中每个sheet的名字
names=list(d_read.keys())

\# 创建保存目录
if not os.path.exists(target_dir):
    os.mkdir(target_dir)
os.chdir(target_dir)

for name in names:
    print(name)

    # 读取的时候要写 header=None，否则会默认将第0行作为标题，而标题在输出的时候会加粗加框 (源文件没有加粗加框)
    tempsheet=pd.read_excel(source_file, sheet_name = name,header=None)
    savefile = target_dir + name + ".xlsx"
    tempsheet.to_excel(savefile, sheet_name = name, na_rep='', index = False, encoding='utf-8', header=None)
```