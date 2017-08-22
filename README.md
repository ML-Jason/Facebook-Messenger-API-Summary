# 快速建立Chatbot

* 建立一個APP。
* 在APP的設定畫面左方，選擇：【+新增產品】，並選取【Messenger】![](/assets/圖片1.JPG)

> Webhooks之後也會用到，但如果只是要快速建立一個Chatbot，可以先不用選。



* 接下來要設定Facebook呼叫我們程式的Webhooks，在【Webhooks】區域，點選【設定Webhooks】，在【回呼網址】填入我們要接收呼叫的網址，在【驗證權杖】欄位填寫一個我們自訂的字串，Facebook第一次跟我們的Webhook串接時，會用GET的方式呼叫【回呼網址】，這時我們要把【驗證權杖】原封不動的回傳給Facebook，這樣Facebook才會視這個Webhook是正確配對的。![](/assets/圖片2.JPG)

> Facebook所有的事件都是用POST的方式呼叫【回呼網址】，要識別不同的事件就是依照傳入的格式來做判斷。  
> 【回呼網址】一定要是HTTPS，所以如果在開發時，可以考慮使用Cloud9來做為測試平台。

* 上圖的【訂閱欄位】各有不同的功能，之後再簡述。目前可以先訂閱【messages、messaging\_postbacks以及messaging\_potins】三個就好。

* 由於Messenger本身是一個APP，我們要將Messenger跟粉絲頁連結，需要取得粉絲頁的Token，所以在Messenger的設定頁面上，到【權杖產生】的區域，選取一個粉絲頁，並複製產生的Token，這個Token會用來在我們的程式裡做為存取訊息的token，務必要留著。![](/assets/圖片3.JPG)

> 這邊產生的Token一但離開畫面之後就會不見，所以一定要馬上copy起來。  
> 不然也可以用api的方式來產生page token，這個之後會講。

* 回到【Webhooks】區域，在下方選擇要連結的粉絲頁，就完成設定了。剩下的就是Coding的部份。![](/assets/圖片4.JPG)



