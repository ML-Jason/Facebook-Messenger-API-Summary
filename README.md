# 快速建立一個echo的Chatbot

* 建立一個APP。
* 在APP的設定畫面左方，選擇：【+新增產品】，並選取【Messenger】![](/assets/圖片1.JPG)

> Webhooks之後也會用到，但如果只是要快速建立一個Chatbot，可以先不用選。



* 接下來要設定Facebook呼叫我們程式的Webhooks，在【Webhooks】區域，點選【設定Webhooks】，在【回呼網址】填入我們要接收呼叫的網址，在【驗證權杖】欄位填寫一個我們自訂的字串，Facebook第一次跟我們的Webhook串接時，會用GET的方式呼叫【回呼網址】，這時我們要把【驗證權杖】原封不動的回傳給Facebook，這樣Facebook才會視這個Webhook是正確配對的。![](/assets/圖片2.JPG)

> Facebook所有的事件都是用POST的方式呼叫【回呼網址】，要識別不同的事件就是依照傳入的格式來做判斷。  
> 【回呼網址】一定要是HTTPS，所以如果在開發時，可以考慮使用Cloud9來做為測試平台。



* 上圖的【訂閱欄位】各有不同的功能，可以得到不同的資訊，之後再簡述。目前可以先訂閱【messages、messaging\_postbacks以及messaging\_potins】三個就好。

* 由於Messenger本身是一個APP，我們要將Messenger跟粉絲頁連結，需要取得粉絲頁的Token，所以在Messenger的設定頁面上，到【權杖產生】的區域，選取一個粉絲頁，並複製產生的Token，這個Token會用來在我們的程式裡做為存取訊息的token，務必要留著。![](/assets/圖片3.JPG)

> 這邊產生的Token一但離開畫面之後就會不見，所以一定要馬上copy起來。  
> 不然也可以用api的方式來產生page token，這個之後會講。



* 回到【Webhooks】區域，在下方選擇要連結的粉絲頁，就完成設定了。剩下的就是Coding的部份。![](/assets/圖片4.JPG)

* 開始寫code，以下以Node.js做範例，首先要先寫一個讓Facebook用GET的方式驗證【驗證權杖】的進入點：

```
router.get('/webhook', (req, res) => {
  if (req.query['hub.mode'] === 'subscribe' &&
      req.query['hub.verify_token'] === <剛剛填入的驗證權杖>) {
    console.log('Validating webhook');
    res.status(200).send(req.query['hub.challenge']);
  } else {
    console.error('Failed validation. Make sure the validation tokens match.');
    res.sendStatus(403);
  }
};
```

* 然後再寫一個接收POST的進入點，所有的資料Facebook都會從這個進入點傳入：

```
router.post('/webhook', (req, res) => {
  const data = req.body;

  // 從資料格式先確定資料是屬於page的
  if (data.object === 'page') {
    // Facebook是用批次的方式將資料傳入，所以data.entry是一個陣列，一次可能會有很多筆資料
    data.entry.forEach((entry) => {
      const pageID = entry.id; // 粉絲頁的ID，如果Chatbot訂閱不止一個粉絲頁，就可以用這個值來做判斷
      const timeOfEvent = entry.time;
      // 如果entry的內容有messaging，表示這是一個訊息(可能是使用者輸入的訊息或是我們定義好的按鈕訊息)
      if (entry.messaging) {
        // 一樣，訊息會是陣列，有可能不止一筆
        entry.messaging.forEach((event) => {
          if (event.message) {
            // 收到使用者輸入的訊息
            receivedMessage(event);
          } else if (event.postback) {
            // 收到postback的訊息(這通常是我們定義好的一些按鈕，後述)
            // 目前先不處理
          } else {
            console.log('Webhook received unknown event: ', event);
          }
        });
      }
    });

    // 無論如何，除非程式真的出錯，否則一定要在20秒之內回傳200，不然Facebook會認為這個webhook失效，
    // 如果Facebook沒收到200，它會定期再次傳送，一但一段時間之後都沒收到200，就會把這個webhook給關掉。
    res.sendStatus(200);
  }
};
```

* 編寫處理訊息的function：

```
const receiveMessage = (event) => {
    // 取得送出訊息者的PSID
    const senderID = event.sender.id;
    // 取得使用者送出的文字訊息
    const messageText = event.message.text;
    
    // 把訊息送回去給使用者
    sendTextMessage(senderID, messageText);
};

const sendTextMessage = (recipientId, messageText) => {
  // 將收文者的PSID以及文字組成傳送物件
  const messageData = {
    recipient: {
      id: recipientId,
    },
    message: {
      text: messageText,
    },
  };
  
  // 使用粉絲頁Token呼叫Facebook messager api
  request({
    uri: 'https://graph.facebook.com/v2.6/me/messages',
    qs: { access_token: <粉絲頁Token> },
    method: 'POST',
    json: messageData,
  }, (error, response, body) => {
    if (!error && response.statusCode === 200) {
      const recipientId = body.recipient_id;
      const messageId = body.message_id;
      
      console.log('Successfully sent message with id %s to recipient %s', messageId, recipientId);
    } else {
      console.error('Unable to send message.');
      console.error(response.body);
    }
  });
};
```

* 這樣就完成了一個會echo的Chatbot。



