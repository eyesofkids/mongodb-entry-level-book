# 基礎介紹

MongoDB是一個NoSQL資料庫，意思是說不使用SQL指令作為資料庫的查詢指令，那要使用什麼？使用程式的函式呼叫。

由於網路上的教學，大部份都涵蓋過多的內容，看多了反而會搞混，所以才會有這本書的產生，這本書如同像這類的NoSQL資料庫所追求的宗旨一樣 - "極簡"。

極簡 代表的應該是更容易的學習與使用，而不是複雜，或是過多的內容。

極簡 代表的是資料庫的結構應該要更簡單，作簡單的事就好，不需要不斷的互相比較。

而且，不需要有"A就是要來取代B"，或是"A一定比B好"的想法，世界上沒有任何絕對完美的軟體或程式語言，只有比較適合與比較不適合的情況。

## 資料類型

MongoDB使用的是類似JSON的BSON存儲格式，BSON是Binary JSON的簡稱，也就是以二進位編碼儲存的類似JSON格式。不過它並不是單純的JSON，BSON擴充了許多資料類型，提供了豐富的資料類型選擇。

進階的資料類型，都要使用MongoDB所提供的資料類型類別來定義，在程式語言中才能進行轉換的工作，例如:

```js
var ts = new MongoDB.Timestamp(0, Math.floor(new Date().getTime() / 1000));
```

JSON格式即所謂的鍵:值(key:value)對應的資料描述格式，其實它是從JavaScript的物件字面定義描述發展出來的，物件能怎麼定義，資料就可以怎麼定義。這也表示這種鍵:值型的NoSQL資料庫，最適合與JavaScript程式語言搭配使用，尤其是Node.js伺服器的系統。

其他的程式語言也都有提供內建的解析與編碼JSON的函式庫或擴充，這是一種很流行的資料格式。

## 資料結構

資料庫的結構因為是檔案型的結構，而且是平坦化的結構，代表每筆資料各自獨立，沒有連結或像關連資料庫中的JOIN(結合)的作法。

一篇部落格文章會長得像下面這樣的一筆資料，其中的留言與標籤會直接在這篇文章之中:

```
_id: POST_ID
   title: TITLE_OF_POST,
   description: POST_DESCRIPTION,
   author: POST_BY,
   tags: [TAG1, TAG2, TAG3],
   likes: TOTAL_LIKES,
   comments: [  
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      },
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      }
   ]
```

當然，當留言新增多筆時，或是標籤新增或刪減時，直接對這筆資料作更動即可。並沒有限制每筆資料一定要長什麼樣子。

這樣作有一些好處，可以統一管理屬於這篇部落格文章的附加資料，在找出資料時會非常有效率。

如果你覺得某些情況要獨立出其中的附加資料，例如部落格文章其中的作者時，你覺得作者應該要有很多他自己的獨立資料，你可以使用MongoDB提供的_id資料，作跨文件的查詢，所以這需要作兩次的查詢:

```js
var result = db.users.findOne({"titls":"Hello MongoDB"})
var author = db.author.findOne({"_id":result["author_id"]})
```

對於1對多(1:n)的情況，也是類似，例如部落格文章其中的留言時，也是兩次查詢:

```js
var result = db.users.findOne({"name":"Tom Benzamin"},{"comment_ids":1})
var comments = db.comments.find({"_id":{"$in":result["comment_ids"]}})
```

多對多的情況也是有可能發生，例如部落格文章其中的標籤(tags)，對於不同的文章中有很多互相使用的標籤，就是一個多對多的情況。這也是要使用MongoDB提供的_id資料，作跨文件的查詢:

```js
db.blogs.insert({
  "_id": ObjectId("4e54ed9f48dc5922c0094a43"),
  "title": "Hello MongoDB",
  "description": "Hello MongoDB Article...",
  "tags": [
    ObjectId("4e54ed9f48dc5922c0094a42"),
    ObjectId("4e54ed9f48dc5922c0094a41")
  ]
});

db.blogs.insert({
  "_id": ObjectId("4e54ed9f48dc5922c0094a40"),
  "title": "Hello Node.js",
  "description": "Hello Node.js Article...",
  "tags": [
    ObjectId("4e54ed9f48dc5922c0094a42")
  ]
});
```

```js
db.tags.insert({
  "_id": ObjectId("4e54ed9f48dc5922c0094a42"),
  "tagName": "mongoDB",
  "blogs": [
    ObjectId("4e54ed9f48dc5922c0094a43"),
    ObjectId("4e54ed9f48dc5922c0094a40")
  ]
});

db.tags.insert({
  "_id": ObjectId("4e54ed9f48dc5922c0094a41"),
  "tagName": "Node",
  "blogs": [
    ObjectId("4e54ed9f48dc5922c0094a43")
  ]
});
```

各種查詢情況，因為要看起來簡單些，都是直接使用_id查詢，實際上你可以多寫一些程式碼，例如用某個標籤文字值找它的_id值之類的:

```js
// 獲取所有部落格文章(blogs)中有"mongoDB"標籤的
db.blogs.find({"tags": ObjectId("4e54ed9f48dc5922c0094a42")});

// 獲取所有部落格文章(blogs)中有"Node"標籤的
db.blogs.find({"tags": ObjectId("4e54ed9f48dc5922c0094a41")});

// 獲取"Hello MongoDB"文章中所有標籤(tags)
db.tags.find({"blogs": ObjectId("4e54ed9f48dc5922c0094a43")});

// 獲取"Hello Node.js"文章中所有標籤(tags)
db.tags.find({"blogs": ObjectId("4e54ed9f48dc5922c0094a40")});
```

為了增加查詢的效率，一般都會作兩個集合的索引(index):

```js
db.blogs.ensureIndex({"groups": 1});
db.tags.ensureIndex({"persons": 1});
```


## 參考

- https://github.com/ilivebox/the-little-mongodb-book/blob/master/zh-cn/mongodb.markdown
- https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/MongoDB%20%E6%9E%81%E7%AE%80%E5%AE%9E%E8%B7%B5%E5%85%A5%E9%97%A8.md
- http://blog.markstarkman.com/blog/2011/09/15/mongodb-many-to-many-relationship-data-modeling/
