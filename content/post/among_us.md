---
title: "Among us のマップを送信してくれる bot を作成したので紹介"
date: 2021-01-20T03:12:45+09:00
draft: false
---

最近、大学のメンバーで [among us](https://ja.wikipedia.org/wiki/Among_Us) というゲームをするのにハマっております。among us とはいわゆる人狼ゲームであり、クルーとインポスターという二つの役職に分かれて生存を目指すゲームです。 詳しい説明は省略しますが、議論はマップを見ながら行うことが一般的であるため、ゲームを進めるにあたってマップは必須なアイテムです。ただ、ライングループで議論を行う場合、マップを探すのが面倒だったりするので bot を作って楽しようと考えました。今回は、その bot を紹介します。

![amoug_us_demo.png](https://s-kaisei.github.io/Ikath-Tech-Blog/images/amoug_us_demo.png)



## Requirement

・ Line Developer の登録と使いかた

・ Google Account の所持

・ Google App Scripts の使い方

   　　　

## ソースコード

参考: [https://www.pre-practice.net/2017/10/line-bot_82.html](https://www.pre-practice.net/2017/10/line-bot_82.html)

```
var CHANNEL_ACCESS_TOKEN = "Your Accsess Token"

function doPost(e) {
  var contents = e.postData.contents;
  var obj = JSON.parse(contents);
  var events = obj["events"];
  for (var i = 0; i < events.length; i++) {
    if (events[i].type == "message") {
      reply_message(events[i]);
    } else if (events[i].type == "postback") {
      post_back(events[i]);
    }
  }
}

function reply_message(e) {
  var input_text = e.message.text;
  if (input_text == "@map") {
    var postData = {
      "replyToken": e.replyToken,
      "messages": [{
        "type": "template",
        "altText": "this is a carousel template",
        "template": {
          "type": "carousel",
          "columns": [
            {
              "thumbnailImageUrl": "https://1.bp.blogspot.com/-uxIsaN0S5lQ/X-FcrvAAInI/AAAAAAABdD4/6uw_qNUh9dQrG0aUzIExybt84yTEmXOPwCNcBGAsYHQ/s200/onepiece01_luffy.png",
              "imageBackgroundColor": "#000000",
              "title": "THE SKELD",
              "text": "First Map. The large spaceship.",
              "actions": [
                {
                  "type": "postback",
                  "label": "display",
                  "data": "first"
                }
              ]
            },
            
            {
              "thumbnailImageUrl": "https://1.bp.blogspot.com/-rzRcgoXDqEg/YAOTCKoCpPI/AAAAAAABdOI/5Bl3_zhOxm07TUGzW8_83cXMOT9yy1VJwCNcBGAsYHQ/s200/onepiece02_zoro_bandana.png",
              "imageBackgroundColor": "#000000",
              "title": "MILA＿HQ",
              "text": "Second Map. A high-rise building.",
              "actions": [
                {
                  "type": "postback",
                  "label": "display",
                  "data": "second"
                }
              ]
            },
            {
              "thumbnailImageUrl": "https://1.bp.blogspot.com/-VACf4WfKNOk/X-Fcx7DWraI/AAAAAAABdE8/KT0UudaIHrMtbFvjC02yLuPLNBZurGYpQCNcBGAsYHQ/s200/onepiece17_doflamingo.png",
              "imageBackgroundColor": "#000000",
              "title": "POLUS",
              "text": "Third Map. A big station on a chilly planet.",
              "actions": [
                {
                  "type": "postback",
                  "label": "display",
                  "data": "third"
                }
              ]
            }
          ],
          "imageAspectRatio": "rectangle",
          "imageSize": "cover"
        }
      }]
    };
  }
  fetch_data(postData);
}

function post_back(e) {
  var data = e.postback.data;
  
  if (data == "first") {
    var imageUrl = "https://drive.google.com/uc?id=1HNhxL5Gww_hdQJs33I43vCkOX4FZeDiP";
  } else if (data == "second") {
    var imageUrl = "https://drive.google.com/uc?id=1tQGpWYGFwSyzplID284c661FC2phJhXF";
  } else if (data == "third") {
    var imageUrl = "https://drive.google.com/uc?id=10uggVpylGheCcifYZSAaBFVY-UflbvcH";
  }

  var postData = {
    "replyToken": e.replyToken,
    "messages": [{
      'type': 'image',
      'originalContentUrl': imageUrl,
      'previewImageUrl': imageUrl,
    }]
  };
  fetch_data(postData);
}

function fetch_data(postData) {
  var options = {
    "method": "post",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer " + CHANNEL_ACCESS_TOKEN
    },
    "payload": JSON.stringify(postData)
  };
  UrlFetchApp.fetch("https://api.line.me/v2/bot/message/reply", options);
}
```

## MAPs

参照: [https://among-us.fandom.com/ja/wiki/%E3%83%9E%E3%83%83%E3%83%97%E3%82%AC%E3%82%A4%E3%83%89](https://among-us.fandom.com/ja/wiki/%E3%83%9E%E3%83%83%E3%83%97%E3%82%AC%E3%82%A4%E3%83%89)

{{< picture "the_skeld.jpg" "the_skeld.jpg" "the skeld" >}}

{{< picture "mira_hq.jpg" "mira_hq.jpg" "mira hq" >}}

{{< picture "polus.jpg" "polus.jpg" "polus" >}}



