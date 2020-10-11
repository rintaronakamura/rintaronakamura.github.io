---
layout: post
title:  "Railsでフォーマットの変更（日付型→文字列型）"
date:   2019-04-29 17:00:00 +0900
categories: rails
---

`config/initializers/time_formats.rb`を作成し、使いたいフォーマットを登録。

```
Time::DATE_FORMATS[:ja] = "%Y年%m月%d日 %H時%M分"
Time::DATE_FORMATS[:ymd] = "%Y%/m%/d"
```

使い方は、
```
Time.new(2019,4,29).to_s(:ja)
=>  2019年04月29日 12時09分
```

ちなみに、デフォルトで登録されているフォーマットは下記の通り。

```
Time.new(2017,9,1).to_s(:default)=> "2017-09-01 00:00:00 +0900"
Time.new(2017,9,1).to_s(:long)=> "September 01, 2017 00:00"
Time.new(2017,9,1).to_s(:short)=> "01 Sep 00:00"
Time.new(2017,9,1).to_s(:db)=> "2017-09-01 00:00:00"
```
