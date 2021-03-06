---
layout: post
title: "效率提升：使用Python配置数据表"
tags:
    - Python
    - Tools
---

唔，首先说一下这是在生产环境，生产环境自然是和自己写着玩的环境是不一样的处理方法，生产环境中我们想要的大概是正确、稳定、高效。

# Python 配置数据表
`jskyzero` `2019/01/18`

## 背景概述

是这样的，出于某种目的，现在我需要配置某个表格，表格是这样的，一部分是配置了一些人数属性，另一部分则需要配置关卡与人物属性的对应关系，用于在关卡中使用，关卡中是需要使用到三位角色，目前是将48名角色随机挑选三位给了关卡，但是这样可能会出现三位某些属性相同的角色的情况，为此需要修改配置。

目前支持的方式是，从若干角色中挑选三位，和若干相同关卡对应角色中，随机一种对应，在和相关同（前）学（辈）沟通过以后拒绝在程序部分增加逻辑，希望维持原有配置方式，考虑到之前是采用第一种方式配置，这次需要采用第二种减少挑选范围，扩大同可能挑选对应方式。

但是仔细一想这炸了，这要是枚举出来过百我都懒得配了，之前一个关卡我只写了一行，现在随便要翻若干倍，显然这里靠蛮力是不行的，好在后台逻辑不改，没事，我可以前台手动枚举，这个本质上数学模型还是很简单，具体涉及到一点excel读取，写入，和数据的概率统计处理方面的知识。

## 模型抽象

再次提及这是在生产环境，甚至没有办法去装某些包，翻了下好歹有个xlrd可以用来读取excel，足够了足够了，暂时是打算写出csv，然后手动拷贝到表中储存，excel中的操作就不太熟悉了，我可以相对模拟输出格式，直接拷贝粘贴，实际上面对几十万行的excel数据，手动修改就会有点不现实了。

那么大概的流程是，读入角色，模拟随机方式，然后去掉不合适的组合，然后写出csv格式文件，然后手工拷贝粘贴至源表格。

## 编码部分

+ 引入包，常量定义（字符常量已经删除，反正是无意义的字符序列）

```python
import xlrd # for excel read
import csv # for writerows
import itertools # for chain

ROLE_NUM = 10
# static string
EXCEL_PATH = 
OUTPUT_PATH = 
PveRobotConf_Name = 
PveRobotAttrs_Name = 
PveRobotAttrs_ID_Name = 
PveRobotAttrs_JOB_Name = 
```


+ 数据读入

xlrd的方式是先获得book，然后book打开sheet，sheet再获取行、列、或者某个格子，要注意的xlrd只是读取，是没有保存相关接口的。

```python
def read_data():
  # read data
  book = xlrd.open_workbook(EXCEL_PATH);
  PveRobotConf = book.sheet_by_name(PveRobotConf_Name)
  PveRobotAttrs = book.sheet_by_name(PveRobotAttrs_Name)

  sh = PveRobotAttrs
  titles = sh.row_values(0)
  # print("numbers of worksheets: {0}".format(book.nsheets))
  # print("worksheet names: {0}".format(book.sheet_names()))
  # print("{0} {1} {2}".format(sh.name, sh.nrows, sh.ncols))
  data = [sh.col_values(titles.index(PveRobotAttrs_ID_Name),4),
          sh.col_values(titles.index(PveRobotAttrs_JOB_Name),4)]
  return data
```

+ 生成与筛选

逻辑相对简单，一些对数据切片和匹配的细节就不再展开说了。

```python
def process_each_level(begin, end, data, result, level):
  # python list process
  # transpose
  data = map(list, zip(*data))
  # map to int
  data = map(lambda l : map(int, l), data)
  # get each role between begin and end
  data = map(lambda x : data[x::ROLE_NUM], range(begin, end))
  # transpose make each sub role together
  data = map(list, zip(*data))
  # join together
  # data = itertools.chain(*data)

  # eight role select 4
  for each_select in itertools.combinations(data, 4):
    process_each_select(each_select, result, level)
  return result



def process_each_select(each_select, result, level):
  for each_result in itertools.product(*each_select):
    ids = map(lambda l: l[0], each_result)
    roles = map(lambda l: l[1], each_result)
    # check role
    if (len(set(roles)) == len(roles)):
      each_result = list(itertools.chain(*[[len(result) + 1], [level], [""], ids]))
      result.append(each_result)
```

+ 输出

因为有大量关卡，这里我们修改config就可以配置关卡与角色起终止一次性生成全部配置。

```python
def process_all():
  result = []
  data = read_data()

  config = [
      [10000, [0,2]],]
  for line in config:
    process_each_level(line[1][0],line[1][1], data, result, line[0]);
  return result


if __name__ == "__main__":
  result = process_all()
  # save to csv file
  with open(OUTPUT_PATH, 'wb') as f:
    csv.writer(f).writerows(result)
```

## 总结

唔，中间的删选部分或许可以再抽象一层，但是也没必要，毕竟是能work就好，生成后大概几十万行的配置，如果手工的话感觉我一辈子就没了。