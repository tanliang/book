# linux后台进程监控脚本

参考:[Python批量下载网易云音乐歌曲](http://flyleaf.fun/2017/10/26/Python批量下载网易云音乐歌曲/)

vi neteasy.py
~~~bash
#!/usr/bin/python
#_*_ coding:utf-8 _*_

import urllib
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
sys.path.append('/usr/local/lib/python2.7/site-packages/')
import requests

gedan_id = raw_input ('请粘贴歌单ID：')
gedan_api = 'http://music.163.com/api/playlist/detail?id=' + gedan_id# 获取歌单API
headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.1 Safari/605.1.15'}
r = requests.get(gedan_api, headers=headers)
arr = r.json()['result']['tracks']    # json解析

for item in arr:
    link = 'http://music.163.com/song/media/outer/url?id=' + str(item['id']) + '.mp3'
    urllib.urlretrieve(link, 'music/' + item['name'].decode('utf-8') + '.mp3')    # 提前要创建 music 文件夹
    print(item['name'].decode('utf-8') + ' 下载完成')

#./neteasy.py
~~~
