---
title: "画像を自動で Google Drive に保存する Line Bot を作成したので紹介"
date: 2020-10-24T06:29:56+09:00
draft: true
---

友人と連絡を取り合う時によく LINE を使うのですが、送信された画像をその時のイベントごとに自動で保存できたら便利だなと考え、line bot を作ることにしました。また、グループメンバーはみんな情報系出身のためコマンドライン風に実装してみました。

{{< picture "eg.png" "eg.png" "example" >}}



## Requirement

- Line Developer の登録
- Google Account の所持



​       

## GAS (Google App Scripts) でコーディング

> GAS でコーディングをするときは、1つの Google Account のみでログインしていないとだめみたいです。複数アカウントでログインしている場合は一度全てのアカウントをログアウトさせてから対象となるアカウントでログインしましょう。

```
var ACCESS_TOKEN = '<your-access-token>';
var MAIL_ADDRESS = ['<your-email>']

function doPost(e) {
  // WebHookで受信した応答用Token
  var replyToken = JSON.parse(e.postData.contents).events[0].replyToken;
  // ユーザーのメッセージを取得
  var userMessage = JSON.parse(e.postData.contents).events[0].message.text;
  // ユーザーがアップロードした画像を取得
  var userImage = JSON.parse(e.postData.contents).events[0].message.id;
  // 応答メッセージ用のAPI URL
  var url = 'https://api.line.me/v2/bot/message/reply';
  
  // メッセージタイプが画像だったら日付を名前として保存する
  if ( JSON.parse(e.postData.contents).events[0].message.type == 'image' ) {
    image = getImage(userImage)
    if ( getTargetFolder() != 'false') {
      var d = new Date();
      var filename = Utilities.formatDate(d, 'Asia/Tokyo', 'yyyyMMddHHmmss') + '.png'
      getTargetFolder().createFile(image.getBlob().getAs("image/png").setName(filename))
    }    
  }
  
  // drive をトリガーとして bot を呼ぶ
  if (userMessage.split(' ')[0] == 'drive') {
    replyTxt = help();
    
    switch( userMessage.split(' ')[1] ) {
      case 'ls':
        replyTxt = driveList()
        break;
        
      case 'mkdir':
        replyTxt = driveMkdir(userMessage.split(' ')[2])
        break;
        
      case 'savedir':
        replyTxt = changeSaveFolder(userMessage.split(' ')[2])
        break;
        
      case 'currentdir':
        replyTxt = getCurrentFolder()
        break;
        
      case 'remaining':
        replyTxt = driveRemaining();
        break;
        
      case 'url':
        replyTxt = getDriveUrl(userMessage.split(' ')[2]);
        break;
        
      case 'help':
        break;
        
      default:
        replyTxt = help();
    }
  }

  UrlFetchApp.fetch(url, {
    'headers': {
      'Content-Type': 'application/json; charset=UTF-8',
      'Authorization': 'Bearer ' + ACCESS_TOKEN,
    },
    'method': 'post',
    'payload': JSON.stringify({
      'replyToken': replyToken,
      'messages': [{
        'type': 'text',
        'text': replyTxt,
      }],
    }),
  });
  return ContentService.createTextOutput(JSON.stringify({'content': 'post ok'})).setMimeType(ContentService.MimeType.JSON);
}



function help() {
  return 'usage:  drive <command> \n----- command一覧 -----\nls: google driveのフォルダーを表示\n\
mkdir: google driveにフォルダーを作成\nsavedir: 写真の保存先を変更\ncurrentdir: 現在の保存先のフォルダーを表示\n\
url: フォルダーのURLを取得\nremaining: google driveの空き容量を確認\nhelp helpを表示\n-------------------------------\n\
e.g.1: drive ls\n\
e.g.2: drive mkdir hoge\n\
e.g.3: drive savedir hoge\n\
e.g.4: drive currentdir\n\
e.g.5: drive url hoge\n\
e.g.6: drive remaining\n\
e.g.7: drive help'
}

function driveList(){
  // drive内のfolderを取得
  var folders = DriveApp.getFolders();
  var replyTxt = '';
  
  if ( !folders.hasNext() ) {
    replyTxt = 'ドライブ内にフォルダは存在しないyo';
    return replyTxt;
  }
  
  while(1) { 
    replyTxt += folders.next(); 
    if ( !folders.hasNext() ) break;
    replyTxt += '\n'; 
  }
  return replyTxt
}

function driveMkdir(newFolderName) {
  // ディレクトリの作成
  var replyTxt = 'フォルダ \'' + newFolderName + '\' は存在しているyo';
  if ( !isFolder(newFolderName) ) {
    DriveApp.createFolder(newFolderName).addEditors(MAIL_ADDRESS);
    replyTxt = 'フォルダ \'' + newFolderName + '\' を作成したyo';
  }
  return replyTxt;
}

function isFolder(newFolderName){
  // drive内のfolderを取得
  var folders = DriveApp.getFolders();
  var checkFolder = false;
  
  while ( folders.hasNext() ) {
    var folder = folders.next();
    if ( folder == newFolderName ) {
      checkFolder = true;
      break;
    }
  }
  return checkFolder;
}

function getCurrentFolder() {
  // 現在保存先に指定しているディレクトリを表示
  var mySheet = SpreadsheetApp.getActiveSheet();
  var folderName = mySheet.getRange(1,1).getValue();
  return '現在の保存先フォルダーは \'' + folderName + '\' だyo';
}

function getTargetFolder() {
  var mySheet = SpreadsheetApp.getActiveSheet();
  var folderName = mySheet.getRange(1,1).getValue();
  var folders = DriveApp.getFolders();
  var checkFolder = false
  
  while ( folders.hasNext() ) {
    var folder = folders.next();
    if ( folder == folderName ) {
      checkFolder = true;
      break;
    }
  }
  if ( checkFolder ) return DriveApp.getFolderById(folder.getId());
  return checkFolder
}


function changeSaveFolder(folderName) {
  // 保存先のディレクトリを変更
  var mySheet = SpreadsheetApp.getActiveSheet();
  var replyTxt = 'フォルダ \'' + folderName + '\' は存在していないyo';
  if ( isFolder(folderName) ) { 
    mySheet.getRange(1,1).setValue(folderName);
    replyTxt = '保存先をフォルダ \'' + folderName + '\' に変更したyo';
  }
  return replyTxt;
}

function driveRemaining(){
  // 残りの容量を確認
  return '残りの容量は ' + DriveApp.getStorageLimit() + 'bytesだyo';
}

function getDriveUrl(folderName) {
  // 保存されている drive の URL を取得
  var folders = DriveApp.getFolders();
  var replyTxt = 'フォルダ \'' + folderName + '\' は存在していないyo';
  
  while ( folders.hasNext() ) {
    var folder = folders.next();
    if ( folder == folderName ) {
      replyTxt = folder.getUrl();
      break;
    }
  }
  return replyTxt; 
}

function getImage(messageId) {
  var url = "https://api.line.me/v2/bot/message/" + messageId + "/content";
  
  return UrlFetchApp.fetch(url, {
    'headers': {
      'Content-Type': 'application/json; charset=UTF-8',
      'Authorization': 'Bearer ' + ACCESS_TOKEN,
    },
    'method': 'get'
  });
}
```



## Usage

| コマンド   | 引数        | 説明                           |
| ---------- | ----------- | ------------------------------ |
| ls         | -           | ドライブ内のフォルダを表示     |
| mkdir      | folder name | ドライブ内にフォルダを作成     |
| savedir    | folder name | 保存先のフォルダを設定         |
| currentdir | -           | 現在の保存先のフォルダーを表示 |
| url        | folder name | フォルダの共有 URL を表示      |
| remaining  | -           | ドライブの残量を表示           |
| help       | -           | Help を表示                    |

