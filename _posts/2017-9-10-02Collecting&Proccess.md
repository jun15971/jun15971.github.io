---
layout: post
title: 浅析豆瓣电影TOP250榜单
description: Part2:数据收集与处理
date: 2017-09-10 16:03:23 +0800
image: assets/images/Screenshot_douban_top250_csv.png
---
# 数据收集

我们在抓取信息之前先看一下豆瓣网的[robots协议](https://www.douban.com/robots.txt):

```
User-agent: *
Disallow: /subject_search
Disallow: /amazon_search
Disallow: /search
Disallow: /group/search
Disallow: /event/search
Disallow: /celebrities/search
Disallow: /location/drama/search
Disallow: /forum/
Disallow: /new_subject
Disallow: /service/iframe
Disallow: /j/
Disallow: /link2/
Disallow: /recommend/
Disallow: /trailer/
Disallow: /doubanapp/card
Sitemap: https://www.douban.com/sitemap_index.xml
Sitemap: https://www.douban.com/sitemap_updated_index.xml
# Crawl-delay: 5

User-agent: Wandoujia Spider
Disallow: /
```

可以看到我们要抓取的`/top250`并不在禁止之列，那么在不影响服务器性能的前提下，可以合理的运用爬虫来抓取所需的信息。

抓取信息的第一步，引入`Python`的`HTTP`库`requests`用来模拟浏览器登录网页，解析网页`Html`文档的库`lxml`以及用来匹配文本信息的正则表达式库`re`。

> 由于网页结构相对比较简单，所以这里直接使用`xpath`来定位标签，获取对应所需的信息。其实也可以引入`BeautifulSoup`库简化定位标签节点的过程。

```python
import requests
from lxml import html
import re
```
定义一个抓取函数，其中用到`requests`库的`get`方法模拟`http`的`get`请求来获取信息，得到一个名为`r`的`requests`对象。

```python
def get_html_text(url， headers):
    try:
        r = requests.get(url=url, headers=headers)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text  # 响应内容
    except:
        return 'Gather Error'
```
其中：

1. `raise_for_status()`方法的作用是：若`requests`对象的状态码不为`200`，则引发`HTTPError`异常。
2. `r.encoding`为`HTTP header`中猜测的响应编码方式，`r.apparent_encoding`为从内容中分析出的响应内容编码方式。


根据观察可以看出`250`条电影信息存放在`10`个页面内，使用变量`i`计数,在`0～10`个页面内抓取信息。此函数需要使用变量计数，记录抓取电影的个数，此变量设置为`x`，每个循环内的`x`即为当前页面内抓取的信息条数。抓取页面信息使用的是`requests`库的`get`方法，再使用`text`方法得到页面文本内容。


![Screenshot_douban](/assets/Screenshot_douban_source.png)

观察网页源码可以看出，所有的信息都在每个`class`属性为`info`的`div`标签里。依此类推定位到各信息所在标签，代码如下：

```python
def douban_top250_spyder(text, x):  # 用于定位信息
    # 所有的信息都在class属性为info的div标签里
    for j in text.xpath('//div[@class="info"]'):
        title = j.xpath('div[@class="hd"]/a/span[@class="title"]/text()')[0]  # 影片名称
        info = j.xpath('div[@class="bd"]/p[1]/text()')  # 信息段
        rate = 9j.xpath('div[@class="bd"]/div[@class="star"]/span[@class="rating_num"]/text()')[0]  # 评分
        com_count0 = j.xpath('div[@class="bd"]/div[@class="star"]/span[4]/text()')[0]  # 评论人数
        com_count = re.match(r'^\d*', com_count0).group()  # 仅保留数字
        quote0 = j.xpath('div[@class="bd"]/p[@class="quote"]/span[@class="inq"]/text()')  # 短评
        quote = '无' if quote0 == [] else quote0[0].replace(",", "，")  # 若短评不存在则使用‘无’替代，并将短评中的英文逗号替换为中文逗号，避免影响CSV文件的处理
        date = info[1].replace("\n", "").strip(' ').split("\xa0/\xa0")[0]  # 上映日期
        country = info[1].split("\xa0/\xa0")[1]  # 制片国家
        genre = info[1].replace("\n", "").strip(' ').split("\xa0/\xa0")[2]  # 影片类型

```

打印出得到的信息，在控制台核查：

```python
print("x" % str(k), title, rate, com_count, date, country, genre, quote)  # 打印结果
```

```
loop 1
1 肖申克的救赎 9.6 835810 1994 美国 犯罪 剧情 希望让人自由。
2 这个杀手不太冷 9.4 801886 1994 法国 剧情 动作 犯罪 怪蜀黍和小萝莉不得不说的故事。
霸王别姬 9.5 597808 1993 中国大陆 香港 剧情 爱情 同性 风华绝代。
4 阿甘正传 9.4 686379 1994 美国 剧情 爱情 一部美国近现代史。
5 美丽人生 9.5 399229 1997 意大利 剧情 喜剧 爱情 战争 最美的谎言。
 ...
loop 10
 ...
23 彗星来的那一夜 8.3 149338 2013 美国 英国 科幻 悬疑 惊悚 小成本大魅力。
24 黑鹰坠落 8.5 101144 2001 美国 动作 历史 战争 还原真实而残酷的战争。
25 假如爱有天意 8.2 216192 2003 韩国 剧情 爱情 琼瑶阿姨在韩国的深刻版。
```

写入所得到的信息，以逗号分割，存为`csv`文件。

```python
with open("douban_top250_demo.csv", "a") as f:  # 写入文件
    f.write("%s,%s,%s,%s,%s,%s,%s\n" % (title, rate, com_count, date, country, genre, quote))
x += 1  # 每条电影信息打印完后计数加一
```

最后，执行代码主体：
```python
headers_douban = {
        'Accept': '*/*',
        'Accept-Encoding': 'gzip, deflate, sdch, br',
        'Accept-Language': 'zh-CN,zh;q=0.8',
        'Connection': 'keep-alive',
        'Referer': 'http://www.douban.com/',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)\
         Chrome/58.0.3029.110 Safari/537.36'
    }  # 请求头部

if __name__ == '__main__':  # 执行代码
    for i in range(10):  # 每页25个电影，共10页，程序在其中做循环，抓取信息。
        print('loop', i+1)  # 显示第几圈
        url_douban = 'https://movie.douban.com/top250?start={}&filter='.format(i * 25)  # 目标网站迭代形式
        text0 = get_html_text(url_douban, headers_douban)  # 请求得到的网页文本内容
        text_douban = html.fromstring(text0)  # 转换为html类数据，便于xpath处理获取信息
        num_counting = 1  # 计数
        douban_top250_spyder(text_douban, num_counting)
```

得到的效果如下:

![douban_top250_text](/assets/images/Screenshot_douban_top250_csv.png)


# 数据处理

核对数据收集阶段保存的`douban_top250_demo.csv`文件，确认与预期效果一致后，保存为`douban_top250.csv`用于数据处理。
这样可以避免在数据处理阶段反复向豆瓣服务器发出数据请求，被反爬虫机制屏蔽。

数据处理、分析和展示阶段，我们主要任务是格式化数据，根据处理过的数据来制作相应的分析图像。需要导入以下几个`python`库：

```python
import re
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib
from PIL import Image
from collections import Counter
from wordcloud import WordCloud, ImageColorGenerator
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn import linear_model
```

开始处理数据，大致操作思路如下：
先打开文件，再读取整个文件，以逗号分割为列表对象，然后转化为250×7的`ndarray`对象，对应7列数据，每列250个元素，最后将其转换为`DataFrame`对象并加上每列的列名，以方便后续调用。


```python
with open('douban_top250.csv') as f:  # 打开文件
    text_f = f.read().strip()  # 读取文件得到str型文本
list_f = re.split('[,\n]', text_f)  # 分割str文本并转换为list型
array_f = np.array(list_f).reshape(250, 7)  # 转换为250×7的数组
columns_df_f = ['电影名称', '评分', '评分人数', '上映年份', '国家', '类型', '短评']  # 列名
df_f = pd.DataFrame(array_f, columns=columns_df_f)  # 转为DataFrame类
print(df_f)
```

打印得到如下结果：

> ```
  电影名称   评分    评分人数  上映年份   国家    类型  \
0 肖申克的救赎  9.6  833069  1994    美国       犯罪 剧情   
1 这个杀手不太冷  9.4  799185  1994 法国      剧情 动作 犯罪   
2 霸王别姬  9.5  595660  1993   中国大陆 香港   剧情 爱情 同性   
3 阿甘正传  9.4  684515  1994  美国   剧情 爱情   
4 美丽人生  9.5  397936  1997  意大利 剧情 喜剧 爱情 战争   
...      ...      ...   ...    ...   ...    ...
247 彗星来的那一夜  8.3  148590  2013   美国 英国 科幻 悬疑 惊悚   
248 黑鹰坠落  8.5  100794  2001  美国    动作 历史 战争   
249 假如爱有天意  8.2  215610  2003  韩国      剧情 爱情   
               短评  
0             希望让人自由。  
1         怪蜀黍和小萝莉不得不说的故事。  
...              ...
248       还原真实而残酷的战争。  
249          琼瑶阿姨在韩国的深刻版。  
[250 rows x 7 columns]
```

### 片名汇总

`DataFrame`类型的数据在提取列信息方面十分方便。
汇总榜单上所有电影的名字只需将`df_f`对象的`'电影名称'`元素赋给变量即可。

```python
titles = df_f['电影名称']  # 提取片名列  
print(titles)
```

打印结果如下：

> ```
0         肖申克的救赎
1        这个杀手不太冷
2           霸王别姬
3           阿甘正传
4           美丽人生
5           千与千寻
     ...
244         廊桥遗梦
245         罪恶之城
246         两小无猜
247      彗星来的那一夜
248         黑鹰坠落
249       假如爱有天意
Name: 电影名称, Length: 250, dtype: object
```

### 制片国家及影片类型信息处理

处理国家名及影片类型信息的方法与处理片名的方法大同小异。

```python
countries = df_f['国家']  # 提取数组中的国家名数据  
print(countries)
```

这里需要注意观察返回的结果中的：

> ```
2         中国大陆 香港
```

可以看出存在一部影片有多个制作公司的情况，如果要计数则需要解压元素。

```python
list_countries = ' '.join(countries).split()  # 由于存在一部电影有多个制片国家，需要解压出。
print(list_countries)
```
得到列表类的结果如下：  
> ['美国', '法国', '中国大陆', '香港', '美国', '意大利', ... ,'美国', '英国', '美国', '韩国']

对列表元素进行计数,使用内置的`collections`库的`Counter`将列表转化为字典，其中`key`为原列表中的元素，`value`为该元素在列表中出现的次数。    
```python
dict_countries0 = Counter(list_countries)  # 转换成字典类并计数
print(dict_countries0)
```
> Counter({'美国': 143, '英国': 34, '日本': 29, '法国': 27,...,'博茨瓦纳': 1, '爱尔兰': 1})

这样也可以很容易得到总共有多少个不同国家出现：
```python
total_countries = len(dict_countries0)  # 字典长度，即出现的不同国家个数
print(total_countries)
```
> 31

```python
num_countries_show = 12  # 控制参数，显示国家的个数，其余用“其他”来概括
dict_countries = list(dict_countries0.most_common(num_countries_show))  # 提取前12个
countries_rest = dict_countries0.most_common()[num_countries_show:]  # 第12个以后合并为一项
size = [i[1] for i in dict_countries]  #
size.append(sum(i[1] for i in countries_rest))
labels = [i[0] for i in dict_countries]
labels.append('其他')
```

榜单上影片的类型的数据处理与制片国家数据处理类似。  

```python
genres = df_f['类型']
list_genres = ' '.join(genres).split()
dict_genres0 = Counter(list_genres)
total_genres = len(dict_genres0)
num_genres_show = 14  # 控制参数，显示类型的个数，其余用“其他”来概括
dict_genres = list(dict_genres0.most_common(num_genres_show))
genres_rest = dict_genres0.most_common()[num_genres_show:]
size = [i[1] for i in dict_genres]
size.append(sum(i[1] for i in genres_rest))
labels = [i[0] for i in dict_genres]
labels.append('其他')
```


### 上榜年份信息处理  

在年份分析部分除了我们需要注意上榜年份为零的数据是不显示的，需要我们另外添加。  

```python
date_movie = df_f['上映年份']
min_date = int(min(date_movie))
max_date = int(max(date_movie))
range_date = range(min_date, max_date + 1)  # 上榜电影年份范围
dict_date0 = Counter(date_movie)  # 转换为年份与对应出现次数的字典
k = [int(key) for key in dict_date0.keys()]  # 构造上榜年份列表
list_date0 = {key: dict_date0[str(key)] if key in k else 0 for key in range_date}  # 以零填充没有上榜的年份的值
dict_date = Counter(list_date0)  # 转换为字典并计数
keys_date, values_date = list(list_date.keys()), list(list_date.values())
```

### 评分、评论人数处理

抓取得到的评分、评论人数的数据可直接调用，处理方法相对比较简单：  

```python
rate_movie = df_f['评分']
index_rate = rate_movie.index + 1  # 序号即排名（从1开始）
values_rate = rate_movie.values  # 得到评分值列表

comments_count = df_f['评分人数']
values_comments = comments_count.values
```
