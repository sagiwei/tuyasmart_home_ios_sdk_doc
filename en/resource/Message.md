## Message center

All functions related to the message center are realized by using the `TuyaSmartMessage` class. Obtaining message list, deleting messages in batches and checking for message update are supported. 


### Obtain message list

In this case, `msgType` means MessageType.  It contains three enumerated values:
-  `1` means Alarm Message
-  `2` means Family Message
-  `3` means Notice Message

`limit` indicates the number of messages acquired at a time, `offset` default is zero.

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


### Obtain message list by msgType

In this case, `msgType` means MessageType.  Only the alarm message supports there，so default is zero。 `msgSrcId` means Detail Message Id, you can obtain it form `TuyaSmartMessageListModel`.
`limit` indicates the number of messages acquired at a time, `offset` default is zero.

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


### Delete messages

Delete messages in batches. In this case, `msgType` means MessageType.  It contains three enumerated values:
-  `1` means Alarm Message
-  `2` means Family Message
-  `3` means Notice Message

`ids` denotes the id group of messages to be deleted, and the message id can be attained from the message list. 
`msgSrcIds` default is nil，if you want to delete detail messages， `msgType` has to be `1`, and `ids` has to be nil.
`msgSrcIds` denotes the id group of detail messages to be deleted, and the message id can be attained from the `TuyaSmartMessageListModel`. 


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


### Obtain the latest news

Obtain the latest news, this returned dict contains three vaule: alarm message, family message, notice message, a red dot will be displayed to hint the new message. 

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