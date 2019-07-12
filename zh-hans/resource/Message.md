## 消息中心

消息中心相关的所有功能对应`TuyaSmartMessage`类，支持获取消息列表，批量删除消息，以及是否有消息更新。


### 获取消息列表

`msgType` 表示消息类型，其中：
-  `1` 表示告警消息
-  `2` 表示家庭消息
-  `3` 表示通知消息

`limit` 表示一次获取的消息数量， `offset` 表示偏移量，默认从零开始。

Objc:

```objc
- (void)getMessageList {
    self.smartMessage = [[TuyaSmartMessage alloc] init];
    [self.smartMessage getMessageListWithType:msgType limit:limit offset:offset success:^(NSArray<TuyaSmartMessageListModel *> *list) {
        NSLog(@"get message list success:%@", list);
    } failure:^(NSError *error) {
        NSLog(@"get message list failure:%@", error);
    }];
}
```
Swift:

```swift
let smartMessage: TuyaSmartMessage = TuyaSmartMessage()
func getMessageList() {
    smartMessage.getListWithType(msgType, limit: limit, offset: offset, success: { (list) in
        print("get message list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get message list failure: \(e)")
        }
    })
}
```


### 获取分类消息列表

其中 `msgType` 表示消息类型， 现只有告警类消息支持分类消息，故默认传零。
`msgSrcId` 表示具体的告警消息 `id`，可从 `TuyaSmartMessageListModel` 里获取。
`limit` 表示一次获取的消息数量， `offset` 表示偏移量，默认从零开始。

Objc:

```objc
- (void)getMessageDetailList {
    self.smartMessage = [[TuyaSmartMessage alloc] init];
    [self.smartMessage getMessageDetailListWithType:msgType msgSrcId:msgSrcId limit:limit offset:offset success:^(NSArray<TuyaSmartMessageListModel *> *list) {
        NSLog(@"get message detail list success:%@", list);
    } failure:^(NSError *error) {
        NSLog(@"get message detail list failure:%@", error);
    }];
}
```
Swift:

```swift
let smartMessage: TuyaSmartMessage = TuyaSmartMessage()
func getMessageDetailList() {
    smartMessage.getDetailList(withType: msgType, msgSrcId: msgSrcId, limit: limit, offset: offset, success: { (list) in
        print("get message list success: \(list)")
    }, failure: { (error) in
        if let e = error {
            print("get message list failure: \(e)")
        }
    })
}
```


### 删除消息

批量删除消息， `msgType` 表示消息类型， 其中：
-  `1` 表示告警消息
-  `2` 表示家庭消息
-  `3` 表示通知消息

`ids`是要删除消息的id数组，消息id可以从消息列表中获取。 
`msgSrcIds` 默认为空，若要删除分类消息，则 `msgType` 传 `1`，`ids` 传空， `msgSrcIds` 是要删除分类消息的id数组，消息id可以从 `TuyaSmartMessageListModel` 里获取。
Objc:

```objc
- (void)deleteMessage {
    self.smartMessage = [[TuyaSmartMessage alloc] init];
    [self.smartMessage deleteMessageWithType:msgType ids:["ids"]] msgSrcIds:["msgSrcIds"] success:^{
        NSLog(@"delete message success");
    } failure:^(NSError *error) {
        NSLog(@"delete message failure:%@", error);
    }];
}
```
Swift:

```swift
    func deleteMessage() {
    smartMessage.delete(withType: msgType, ids: ["ids"], msgSrcIds: ["msgSrcIds"], success: {
        print("delete message success")
    }, failure: { (error) in
        if let e = error {
            print("delete message failure: \(e)")
        }
    })
}
```


### 获取是否有最新消息

获取是否有最新消息，返回的字典里包含告警消息、家庭消息、通知消息的最新消息情况，若对应字典的值是 YES，则展示红点，表示有新消息。

Objc:

```objc
- (void)getLatestMessageWithSuccess {
    self.smartMessage = [[TuyaSmartMessage alloc] init];
    [self.smartMessage getLatestMessageWithSuccess:^(NSDictionary *dict) {
        NSLog(@"get latest message success: %@",dict);
    } failure:^(NSError *error) {
        NSLog(@"get latest message failure:%@", error);
    }];
}
```

Swift:

```swift
func getLatestMessage() {
    smartMessage.getLatestMessage(success: { (dict) in
        print("get latest message success :%@", dict);
    }, failure: { (error) in
        if let e = error {
            print("get latest message success: \(e)")
        }
    })
}
```

