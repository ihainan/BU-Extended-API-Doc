# BU-Extended-API-Doc

## 说明
- __当前接口版本为 v2__。
- 所有 POST 请求，如无特殊说明，一律采用 __UTF-8 编码__，其他请求一律采用 __URL 编码__

## 接口 URL 基本格式
接口地址的根路径为：http://bu.ihainan.me:8080/api/VERSION，其中 VERSION 为接口的版本，如 v1、v2 等。例如添加收藏接口的 URL 可能为：[http://bu.ihainan.me:8080/api/v2/favorite](http://bu.ihainan.me:8080/api/v2/favorite)。

## 基本认证
为避免有人滥用接口，接口采用了 Basic Auth 认证策略，在调用请求时，客户端需要提供用户名和密码。

Basic Auth 的相关介绍参见 [维基百科](https://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)，Android 环境下的实现可以参见 [代码片段](https://github.com/ihainan/BITUnion-for-Android/blob/master/app/src/main/java/me/ihainan/bu/app/utils/network/RequestQueueManager.java) 中 `CustomJsonObjectRequest` 类的实现。__用户名与密码请联系管理员获取__。

## 通用返回报文格式
对于所有的请求，范围的格式均如下所示，其中 `data` 字段可选。

``` JSON
{
    "code": "非负整数",
    "message": "返回信息",
    "data": {
        "field 1": "data 1",
        "field 2": "data 2"
    }
}
```

通用消息格式如下所示，如无特殊说明，__所有请求接口都可能会返回如下格式的数据，客户端需要进行处理__。

1、 正确结果报文，需要注意 `data` 字段可能不存在。

``` JSON
{
    "code": 0,
    "message": "success",
    "data": {
        "field 1": "data 1",
        "field 2": "data 2"
    }
}
```

2、 验证失败
``` JSON
{
    "code": 100,
    "message": "Authentication failed"
}
```

3、非法字段
``` JSON
{
    "code": 102,
    "message": "Invalid filed FIELD_NAME"
}
```

## 收藏相关接口
### 添加收藏
#### 说明
该接口用于用户添加一个主题到收藏列表当中。由于爬虫爬取得到的数据并不完整，所以需要接口调用者提供__主题作者 (`author`)__和__主题名 (`subject`) __两个字段。

需要注意如果一个主题已经被该用户收藏过，__重复添加收藏并不会报错__,如果两次添加收藏操作 `tid` 相同而 `subject` 或者 `author` 不同，则会更新数据库中的对应记录。


#### 接口地址：__/favorite__

#### 请求方法：PUT

#### 请求报文：

``` JSON
{
	"username": "收藏者用户名",
	"tid": "主题编号",
	"subject": "主题标题",
	"author": "主题作者的用户名"
}
```

#### 结果报文：

1、 正确结果报文

``` JSON
{
    "code": 0,
    "message": "success"
}
```

2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/favorite](http://bu.ihainan.me:8080/api/v2/favorite)

__发送报文（POST）：__

``` JSON
{
    "username": "ihainan",
    "tid": "10614199",
    "subject": "研究生出国参加学术会议咨询，求助各位大大",
    "author": "在途中"
}
```

__结果报文：__
```JSON
{
    "code": 0,
    "message": "success"
}
```

### 删除收藏
#### 说明
该接口用于删除特定用户对于特定主题的收藏状态，__需要注意如果主题并未被收藏，该接口并不会报错__。`username` 表示用户名，`tid` 表示主题编号。

#### 接口地址：__/favorite/{username}/{tid}__

#### 请求方法：DELETE

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success"
}
```

2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/favorite/ihainan/2](http://bu.ihainan.me:8080/api/v2/favorite/ihainan/2)

__发送报文（DELETE）__：空。

__结果报文：__
```JSON
{
    "code": 0,
    "message": "success"
}
```

### 获取收藏列表

#### 说明
该接口用于用于获取特定用户的收藏列表。

`username` 表示收藏者的用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：/favorite/list/{username}?from=FROM&to=TO

#### 请求方法：GET

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success",
	"data": [
	    {"fav_id": "收藏编号 1", "username": "收藏者用户名 1", "tid": "主题编号 1", "subject": "主题标题 1", "author": "主题作者 1", "dt_created": "添加收藏时间 1"},
	    {"fav_id": "收藏编号 2", "username": "收藏者用户名 2", "tid": "主题编号 2", "subject": "主题标题 2", "author": "主题作者 2", "dt_created": "添加收藏时间 2"}
	]
}
```


2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/favorite/list/ihainan?from=0&to=2](http://bu.ihainan.me:8080/api/v2/favorite/list/ihainan?from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

```JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "fav_id":108,
         "username":"ihainan",
         "tid":10614210,
         "subject":"BU for Android 测试帖",
         "author":"ihainan",
         "dt_created":"2016-05-31 08:04:48.0"
      },
      {
         "fav_id":107,
         "username":"ihainan",
         "tid":10614155,
         "subject":"关于本版（开发者必看）",
         "author":"mylangtao",
         "dt_created":"2016-05-31 07:34:05.0"
      }
   ]
}
```

### 收藏状态

#### 说明
该接口用于用于获取特定用户是否已经收藏了特定主题，如果已经收藏返回 `true`，否则返回 `false`。

#### 接口地址：__/favorite/status/{username}/{tid}__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success",
	"data": true
}
```


2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/favorite/status/ihainan/10614210](http://bu.ihainan.me:8080/api/v2/favorite/status/ihainan/10614210)

__发送报文（GET）__：空。

__结果报文：__
```JSON
{
   "code":0,
   "message":"success",
   "data":true
}
```

## 搜索接口
### 根据主题标题搜索
#### 说明
该接口用于根据标题内容去搜索与关键词所匹配对应的主题（Thread）。实际上接口返回的是楼层（`floor`）为 0 的帖子（POST）。

`KEY` 表示搜索的关键词，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

__目前接口尚未支持模糊搜索__。

#### 接口地址：__/search/threads?key=KEY&from=FROM&to=TO__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文，由于返回的字段比较多，下面仅展示部分字段，其余字段请参考实例中的返回结果。

``` JSON
{
	"code": 0,
	"message": "success",
	"data": [
	    {"floor":0, "tid": "主题编号 1", "pname": "主题标题 1", "author": "主题作者 1", "fid": "主题所在板块 ID 1", "fname": "主题所在板块名 1", "dt_created":"爬虫抓取时间 1", "more_fields": "更多字段"},
	    {"floor":0, "tid": "主题编号 2", "pname": "主题标题 2", "author": "主题作者 2", "fid": "主题所在板块 ID 2", "fname": "主题所在板块名 2", "dt_created":"爬虫抓取时间 2", "more_fields": "更多字段"}
	]
}
```


2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/search/threads?key=医院&from=0&to=2](http://bu.ihainan.me:8080/api/v2/search/threads?key=医院&from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

```JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "post_id":32339,
         "floor":0,
         "pid":14733635,
         "fid":14,
         "tid":10613623,
         "aid":0,
         "icon":"0",
         "author":"逍遥小僧",
         "authorid":111596,
         "t_subject":"吐槽下校医院....",
         "subject":"吐槽下校医院....",
         "dateline":1463453624,
         "message":"不知为何，校医院的牙科门诊号的挂号政策如此奇葩。工作日每天三个号没关系，大家早起先来后到去排号也认了，但是为什么非要11点放号呢？大家早起排号估计都要5点或者6点过去，然后就一直在那里干等着，5-6个小时啊！！！这纯粹是浪费大家的时间！！！为什么不能一上班就放号呢？这里面校医院有什么为难之处吗？大家为何不一起向学校反映一下呢？",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1463453624",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":111596,
         "username":"%E9%80%8D%E9%81%A5%E5%B0%8F%E5%83%A7",
         "avatar":"",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-17 11:11:44.0"
      },
      {
         "post_id":32156,
         "floor":0,
         "pid":14733435,
         "fid":108,
         "tid":10613585,
         "aid":0,
         "icon":"0",
         "author":"tracy1988",
         "authorid":71369,
         "t_subject":"求校医院电话~",
         "subject":"求校医院电话~",
         "dateline":1463388565,
         "message":"人在外地，内网上不去，麻烦贴个校医院电话，谢谢~",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1463388565",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":71369,
         "username":"tracy1988",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F71369.jpg%22++border%3D%220%22%3E",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-16 16:51:40.0"
      }
   ]
}
```


### 根据回帖内容搜索
#### 说明
该接口用于根据帖子内容去搜索与关键词索匹配对应的帖子。

`KEY` 表示搜索的关键词，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

__目前接口尚未支持模糊搜索__。

#### 接口地址：__/search/posts?key=KEY&from=FROM&to=TO__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文，由于返回的字段比较多，下面仅展示部分字段，其余字段请参考实例中的返回结果。

``` JSON
{
	"code": 0,
	"message": "success",
	"data": [
	    {"floor": "楼层 1", "tid": "主题编号 1", "pname": "主题标题 1", "author": "主题作者 1", "fid": "主题所在板块 ID 1", "fname": "主题所在板块名 1", "dt_created":"爬虫抓取时间 1", "more_fields": "更多字段"},
        {"floor": "楼层 2", "tid": "主题编号 2", "pname": "主题标题 2", "author": "主题作者 2", "fid": "主题所在板块 ID 2", "fname": "主题所在板块名 2", "dt_created":"爬虫抓取时间 2", "more_fields": "更多字段"}
	]
}
```

2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/search/posts?key=医院&from=0&to=2](http://bu.ihainan.me:8080/api/v2/search/posts?key=医院&from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

```JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "post_id":36649,
         "floor":18,
         "pid":14738570,
         "fid":14,
         "tid":10614221,
         "aid":0,
         "icon":"0",
         "author":"qiang",
         "authorid":133360,
         "t_subject":"究竟是什么造成了这样的情况？？",
         "subject":"",
         "dateline":1464663421,
         "message":"<br><br><center><table border=\"0\" width=\"90%\" cellspacing=\"0\" cellpadding=\"0\"><tr><td>&nbsp;&nbsp;引用[<a href=\"redirect.php?goto=findpost&fid=&ptid=10614221&pid=14738535\">查看原帖</a>]:</td></tr><tr><td><table border=\"0\" width=\"100%\" cellspacing=\"1\" cellpadding=\"10\" bgcolor=\"BORDERCOLOR\"><tr><td width=\"100%\" bgcolor=\"ALTBG2\"><b>changtingwai</b> 2016-5-31 09:29<br />\r\n我连楼上拉椅子的声音都怕，都能吵醒我。夜里经常被吵醒，去了医院神经衰弱。。楼主相比我你已经好很多了 </td></tr></table></td></tr></table></center><br>解决方案呢<br />\r\n昨晚3点睡着，早上不到7点醒了<br />\n<br />\n[ Last edited by qiang on 2016-5-31 at 10:58 ]",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464663526",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":133360,
         "username":"qiang",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F133360.jpg%22++border%3D%220%22%3E",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-31 11:04:15.0"
      },
      {
         "post_id":36611,
         "floor":7,
         "pid":14738535,
         "fid":14,
         "tid":10614221,
         "aid":0,
         "icon":"0",
         "author":"changtingwai",
         "authorid":137583,
         "t_subject":"究竟是什么造成了这样的情况？？",
         "subject":"",
         "dateline":1464658145,
         "message":"我连楼上拉椅子的声音都怕，都能吵醒我。夜里经常被吵醒，去了医院神经衰弱。。楼主相比我你已经好很多了",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464658145",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":137583,
         "username":"changtingwai",
         "avatar":"",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-31 09:34:13.0"
      }
   ]
}
```

## 用户接口
### 用户已发布主题列表
#### 说明
该接口用于获取特定用户发过的主题的列表。

`author` 表示用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：__/user/{author}/threads?from=FORM&to=TO__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文，由于返回的字段比较多，下面仅展示部分字段，其余字段请参考实例中的返回结果。

``` JSON
{
	"code": 0,
	"message": "success",
	"data": [
	    {"floor": 0, "tid": "主题编号 1", "pname": "主题标题 1", "author": "主题作者 1", "fid": "主题所在板块 ID 1", "fname": "主题所在板块名 1", "dt_created":"爬虫抓取时间 1", "more_fields": "更多字段"},
	    {"floor": 0, "tid": "主题编号 2", "pname": "主题标题 2", "author": "主题作者 2", "fid": "主题所在板块 ID 2", "fname": "主题所在板块名 2", "dt_created":"爬虫抓取时间 2", "more_fields": "更多字段"}
	]
}
```


2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/user/ihainan/threads?from=0&to=2](http://bu.ihainan.me:8080/api/v2/user/ihainan/threads?from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "post_id":36634,
         "floor":1,
         "pid":14738558,
         "fid":177,
         "tid":10614210,
         "aid":1248767,
         "icon":"",
         "author":"ihainan",
         "authorid":108263,
         "t_subject":null,
         "subject":"",
         "dateline":1464661429,
         "message":"Volley timeout test.<img src=\"../images/bz/80.gif\" border=\"0\"><br />\n<br />\n<br />\n<b>发自 Unknown Google Nexus 5 - 5.1.0 - API 22 - 1080x1920 @BU for Android</b><br/><br/><span id=\"id_open_api_label\">..:: <a href=http://www.bitunion.org>From BIT-Union Open API Project</a> ::..<br/>",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464661429",
         "postsource":null,
         "aaid":1248767,
         "creditsrequire":0,
         "filetype":"image%2Fjpeg",
         "filename":"BU_30052016_222336.jpg",
         "attachment":"attachments%2Fforumid_177%2F2%2F8%2F28WQ_QlVfMzAwNTIwMTZfMjIyMzM2.jpg",
         "filesize":"408406",
         "downloads":0,
         "uid":108263,
         "username":"ihainan",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F108263.jpg%22++border%3D%220%22%3E",
         "epid":22812,
         "maskpost":0,
         "attachext":"jpg",
         "attachsize":"398.83+K",
         "attachimg":"1",
         "exif":"false",
         "dt_created":"2016-05-31 10:24:14.0"
      },
      {
         "post_id":36556,
         "floor":1,
         "pid":14738458,
         "fid":14,
         "tid":10614214,
         "aid":0,
         "icon":"",
         "author":"ihainan",
         "authorid":108263,
         "t_subject":null,
         "subject":"",
         "dateline":1464619134,
         "message":"我还以为楼主要说家庭出身决定以后的命运……<br />\r\n<br />\r\n我选择信星座，求问风向星座今年运势。",
         "usesig":1,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464619134",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":108263,
         "username":"ihainan",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F108263.jpg%22++border%3D%220%22%3E",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-30 22:44:14.0"
      }
   ]
}
```

### 用户已发布回帖列表
#### 说明
该接口用于获取特定用户发过的回帖的列表。

`author` 表示用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：__/user/{author}/replies?from=FORM&to=TO__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文，由于返回的字段比较多，下面仅展示部分字段，其余字段请参考实例中的返回结果。

``` JSON
{
	"code": 0,
	"message": "success",
	"data": [
	   {"floor": "楼层 1", "tid": "主题编号 1", "pname": "主题标题 1", "author": "主题作者 1", "fid": "主题所在板块 ID 1", "fname": "主题所在板块名 1", "dt_created":"爬虫抓取时间 1", "more_fields": "更多字段"},
       {"floor": "楼层 2", "tid": "主题编号 2", "pname": "主题标题 2", "author": "主题作者 2", "fid": "主题所在板块 ID 2", "fname": "主题所在板块名 2", "dt_created":"爬虫抓取时间 2", "more_fields": "更多字段"}
	]
}
```


2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/user/ihainan/replies?from=0&to=2](http://bu.ihainan.me:8080/api/v2/user/ihainan/replies?from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "post_id":36634,
         "floor":1,
         "pid":14738558,
         "fid":177,
         "tid":10614210,
         "aid":1248767,
         "icon":"",
         "author":"ihainan",
         "authorid":108263,
         "t_subject":null,
         "subject":"",
         "dateline":1464661429,
         "message":"Volley timeout test.<img src=\"../images/bz/80.gif\" border=\"0\"><br />\n<br />\n<br />\n<b>发自 Unknown Google Nexus 5 - 5.1.0 - API 22 - 1080x1920 @BU for Android</b><br/><br/><span id=\"id_open_api_label\">..:: <a href=http://www.bitunion.org>From BIT-Union Open API Project</a> ::..<br/>",
         "usesig":0,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464661429",
         "postsource":null,
         "aaid":1248767,
         "creditsrequire":0,
         "filetype":"image%2Fjpeg",
         "filename":"BU_30052016_222336.jpg",
         "attachment":"attachments%2Fforumid_177%2F2%2F8%2F28WQ_QlVfMzAwNTIwMTZfMjIyMzM2.jpg",
         "filesize":"408406",
         "downloads":0,
         "uid":108263,
         "username":"ihainan",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F108263.jpg%22++border%3D%220%22%3E",
         "epid":22812,
         "maskpost":0,
         "attachext":"jpg",
         "attachsize":"398.83+K",
         "attachimg":"1",
         "exif":"false",
         "dt_created":"2016-05-31 10:24:14.0"
      },
      {
         "post_id":36556,
         "floor":1,
         "pid":14738458,
         "fid":14,
         "tid":10614214,
         "aid":0,
         "icon":"",
         "author":"ihainan",
         "authorid":108263,
         "t_subject":null,
         "subject":"",
         "dateline":1464619134,
         "message":"我还以为楼主要说家庭出身决定以后的命运……<br />\r\n<br />\r\n我选择信星座，求问风向星座今年运势。",
         "usesig":1,
         "bbcodeoff":0,
         "smileyoff":0,
         "parseurloff":0,
         "score":0,
         "rate":0,
         "ratetimes":0,
         "pstatus":0,
         "lastedit":"1464619134",
         "postsource":null,
         "aaid":0,
         "creditsrequire":0,
         "filetype":null,
         "filename":null,
         "attachment":null,
         "filesize":"0",
         "downloads":0,
         "uid":108263,
         "username":"ihainan",
         "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F108263.jpg%22++border%3D%220%22%3E",
         "epid":0,
         "maskpost":0,
         "attachext":null,
         "attachsize":null,
         "attachimg":null,
         "exif":null,
         "dt_created":"2016-05-30 22:44:14.0"
      }
   ]
}
```

## 关注

### 添加关注
#### 说明

该接口用于添加用户一到用户二的关注关系，关注之后用户一可以在关注时间轴中获取用户二的最新动态。

__目前接口并未对用户名进行检查，所以请保证用户名是正确的。调用该接口之后，用户二会收到一条被关注提醒。__

#### 接口地址：__/follow__

#### 请求方法：POST

#### 请求报文：

``` JSON
{
	"follower": "关注者用户名",
    "following": "被关注者用户名"
}
```

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success"
}
```

2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/follow](http://bu.ihainan.me:8080/api/v2/follow)

__发送报文（POST）__：

``` JSON
{
  	"follower":"ihainan",
  	"following":"我与路口"
}
```

__结果报文：__

``` JSON
{
	"code": 0,
	"message": "success"
}
```

### 取消关注
#### 说明

该接口用于取消用户一到用户二的关注关系，取消关注之后用户一再关注列表中再也看不到用户二的相关动态。如果用户一到用户二并没有关注关系，接口不会返回错误。

#### 接口地址：__/follow/{follower}/{following}__

#### 请求方法：DELETE

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success"
}
```

2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/follow/ihainan/%e6%88%91%e4%b8%8e%e8%b7%af%e5%8f%a3](http://bu.ihainan.me:8080/api/v2/follow/ihainan/%e6%88%91%e4%b8%8e%e8%b7%af%e5%8f%a3)

__发送报文（DELETE）__：空

__结果报文：__

``` JSON
{
	"code": 0,
	"message": "success"
}
```

### 获取关注者列表
#### 说明
该接口用于获取用户所关注的人的列表。

`follower` 表示关注者的用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：__/follow/list/{follower}?from=FROM&to=TO__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文。

``` JSON
{
	"code": 0,
	"message": "success",
    "data":[
    {
        "fl_id": "关注编号 1",
        "follower":"关注者",
        "following":"被关注者 1",
        "dt_created":"关注时间 1"
    },
    {
        "fl_id": "关注编号 2",
        "follower":"关注者",
        "following":"被关注者 2",
        "dt_created":"关注时间 2"
    }
   ]
}
```


2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/follow/list/ihainan?from=0&to=2](http://bu.ihainan.me:8080/api/v2/follow/list/ihainan?from=0&to=2)

__发送报文（GET）__：空。

__结果报文：__

``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "fl_id":91,
         "follower":"ihainan",
         "following":"琴心三叠",
         "dt_created":"2016-05-31 07:40:16.0"
      },
      {
         "fl_id":90,
         "follower":"ihainan",
         "following":"wuxu01",
         "dt_created":"2016-05-30 22:58:29.0"
      }
   ]
}
```

### 关注状态
#### 说明
该接口用于用户一是否关注了用户二，如果已经关注返回 `true`，否则返回 `false`。

#### 接口地址：__/follow/status/{follower}/{following}__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文

``` JSON
{
	"code": 0,
	"message": "success",
	"data": true
}
```

2、 错误结果报文：参见通用返回报文格式。

#### 实例：

__地址：__  [http://bu.ihainan.me:8080/api/v2/follow/status/ihainan/%e6%88%91%e4%b8%8e%e8%b7%af%e5%8f%a3](http://bu.ihainan.me:8080/api/v2/favorite/status/ihainan/%e6%88%91%e4%b8%8e%e8%b7%af%e5%8f%a3)

__发送报文（GET）__：空。

__结果报文：__
```JSON
{
   "code":0,
   "message":"success",
   "data":true
}
```

## 时间轴
### 获取特定用户的动态时间轴
#### 说明
该接口用于获取特定用户的动态时间轴。动态包括三类，分别为：

- 发主题 / 回帖时间轴，表示为类型 1。
- 收藏时间轴，表示为类型 2。
- 关注时间轴，表示为类型 3。

`username` 表示特定用户的用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：__/spec/{username}__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文。
``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "tl_id": "时间轴编号 1",
         "type":1,
         "content":{
            "post_id":"帖子编号",
            "floor":"发帖楼层",
            "tid":"主题编号",
            "dt_created":"发帖 / 回帖时间"
         }
      },
      {
         "tl_id": "时间轴编号 2",
         "type":2,
         "content":{
            "fav_id": "收藏编号",
            "username":"收藏者用户名",
            "tid":"收藏主题编号",
            "subject":"主题标题",
            "author":"主题作者",
            "dt_created":"收藏时间"
         }
      },
      {
         "tl_id": "时间轴编号 3",
         "type":3,
         "content":{
            "fl_id": "关注编号",
            "follower":"关注者",
            "following":"被关注者",
            "dt_created":"关注时间"
         }
      }
   ]
}
```

2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/spec/ihainan?from=0&to=3](http://bu.ihainan.me:8080/api/v2/spec/ihainan?from=0&to=3)

__发送报文（GET）__：空。

__结果报文：__

``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "tl_id":109,
         "type":2,
         "content":{
            "fav_id":109,
            "username":"ihainan",
            "tid":10614209,
            "subject":"求校内的同学帮找一下 极速之星下载软件的序列号",
            "author":"pc200709",
            "dt_created":"2016-05-31 12:58:21.0"
         }
      },
      {
         "tl_id":93,
         "type":3,
         "content":{
            "fl_id":93,
            "follower":"ihainan",
            "following":"我与路口",
            "dt_created":"2016-05-31 12:44:54.0"
         }
      },
      {
         "tl_id":36634,
         "type":1,
         "content":{
            "post_id":36634,
            "floor":1,
            "pid":14738558,
            "fid":177,
            "tid":10614210,
            "aid":1248767,
            "icon":"",
            "author":"ihainan",
            "authorid":108263,
            "t_subject":"BU for Android 测试帖",
            "subject":"",
            "dateline":1464661429,
            "message":"Volley timeout test.<img src=\"../images/bz/80.gif\" border=\"0\"><br />\n<br />\n<br />\n<b>发自 Unknown Google Nexus 5 - 5.1.0 - API 22 - 1080x1920 @BU for Android</b><br/><br/><span id=\"id_open_api_label\">..:: <a href=http://www.bitunion.org>From BIT-Union Open API Project</a> ::..<br/>",
            "usesig":0,
            "bbcodeoff":0,
            "smileyoff":0,
            "parseurloff":0,
            "score":0,
            "rate":0,
            "ratetimes":0,
            "pstatus":0,
            "lastedit":"1464661429",
            "postsource":null,
            "aaid":1248767,
            "creditsrequire":0,
            "filetype":"image%2Fjpeg",
            "filename":"BU_30052016_222336.jpg",
            "attachment":"attachments%2Fforumid_177%2F2%2F8%2F28WQ_QlVfMzAwNTIwMTZfMjIyMzM2.jpg",
            "filesize":"408406",
            "downloads":0,
            "uid":108263,
            "username":"ihainan",
            "avatar":"%3Cimg+src%3D%22images%2Fupavatars%2F108263.jpg%22++border%3D%220%22%3E",
            "epid":22812,
            "maskpost":0,
            "attachext":"jpg",
            "attachsize":"398.83+K",
            "attachimg":"1",
            "exif":"false",
            "dt_created":"2016-05-31 10:24:14.0"
         }
      }
   ]
}
```

### 获取关注用户的动态时间轴
#### 说明
该接口用于获取特定用户所关注用户的动态时间轴。动态包括三类，分别为：

- 发主题 / 回帖时间轴，表示为类型 1。
- 收藏时间轴，表示为类型 2。
- 关注时间轴，表示为类型 3。

`username` 表示特定用户的用户名，`FROM` 表示起始位置，`TO` 表示终点位置，接口返回 [`FROM`, `TO`) 区间内的列表数据。`FROM` 最小为 0，`TO` 最小为 1，`FROM` 必须小于 `TO` ，并且单次获取的数据最大为 30 条（`FROM` - `TO` <= 30）。

#### 接口地址：__/focus/{username}__

#### 请求方法：GET

#### 结果报文：

1、正确结果报文。
``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "tl_id": "时间轴编号 1",
         "type":1,
         "content":{
            "post_id":"帖子编号",
            "floor":"发帖楼层",
            "tid":"主题编号",
            "dt_created":"发帖 / 回帖时间"
         }
      },
      {
         "tl_id": "时间轴编号 2",
         "type":2,
         "content":{
            "fav_id": "收藏编号",
            "username":"收藏者用户名",
            "tid":"收藏主题编号",
            "subject":"主题标题",
            "author":"主题作者",
            "dt_created":"收藏时间"
         }
      },
      {
         "tl_id": "时间轴编号 3",
         "type":3,
         "content":{
            "fl_id": "关注编号",
            "follower":"关注者",
            "following":"被关注者",
            "dt_created":"关注时间"
         }
      }
   ]
}
```

2、 错误结果报文：

- `TO` 小于等于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than 0"
}
```

- `FROM` 小于 0。

``` JSON
{
    "code": 102,
    "message": "Invalid field from: FROM should be greater than or equal to 0"
}
```

- `FROM` 小于 `to`。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO should be greater than or equal to FROM"
}
```

- `FROM` - `TO` > 30。

``` JSON
{
    "code": 102,
    "message": "Invalid field to: TO minus FROM should be less than or equal to 30"
}
```

#### 实例：

__地址：__ [http://bu.ihainan.me:8080/api/v2/timeline/focus/ihainan?from=0&to=1](http://bu.ihainan.me:8080/api/v2/timeline/focus/ihainan?from=0&to=1)

__发送报文（GET）__：空。

__结果报文：__

``` JSON
{
   "code":0,
   "message":"success",
   "data":[
      {
         "tl_id":36703,
         "type":1,
         "content":{
            "post_id":36703,
            "floor":2,
            "pid":14738628,
            "fid":14,
            "tid":10614234,
            "aid":0,
            "icon":"0",
            "author":"lanqiang",
            "authorid":106368,
            "t_subject":"闲聊 未来三年资产保值计划（5）",
            "subject":"",
            "dateline":1464670073,
            "message":"全是高大上的经济学专业术语啊，我看得一头雾水。<br />\r\n<br />\r\n还是讲大白话比较好： 有钱就是王道，管它房价怎么涨，都不怕！<br />\r\n<br />\r\n然后怎么才能有钱： 努力+选择+运气。<br />\r\n<br />\r\n努力不够，你就什么都做不成。 选择错了，你也会一事无成。 努力和选择都有了，如果没有机遇和运气，也照样是一败涂地。<br />\r\n<br />\r\n我说，你讲那么多高深的词语干什么。能不能别每次都装经济学砖家？能不能接地气一点？",
            "usesig":1,
            "bbcodeoff":0,
            "smileyoff":0,
            "parseurloff":0,
            "score":0,
            "rate":0,
            "ratetimes":0,
            "pstatus":0,
            "lastedit":"1464670093",
            "postsource":null,
            "aaid":0,
            "creditsrequire":0,
            "filetype":null,
            "filename":null,
            "attachment":null,
            "filesize":"0",
            "downloads":0,
            "uid":106368,
            "username":"lanqiang",
            "avatar":"",
            "epid":0,
            "maskpost":0,
            "attachext":null,
            "attachsize":null,
            "attachimg":null,
            "exif":null,
            "dt_created":"2016-05-31 12:54:14.0"
         }
      }
   ]
}
```