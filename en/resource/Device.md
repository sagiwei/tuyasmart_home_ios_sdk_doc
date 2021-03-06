## Device management

The device ID needs to be used to initiate all `TuyaSmartDevice` classes related to all functions for device control. Wrong device Id may cause initiation failure, and the `nil` will be returned.

### Update device information

#### Update single device information

Objc:

```objc
- (void)updateDeviceInfo {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	__weak typeof(self) weakSelf = self;
	[self.device syncWithCloud:^{
		NSLog(@"syncWithCloud success");
		NSLog(@"deviceModel: %@", weakSelf.device.deviceModel);
	} failure:^(NSError *error) {
		NSLog(@"syncWithCloud failure");
	}];
}
```

Swift:

```swift
func updateDeviceInfo() {
    device?.sync(cloud: {
        print("syncWithCloud success")
    }, failure: { (error) in
        if let e = error {
            print("syncWithCloud failure: \(e)")
        }
    })
}
```



### Device Control

Device control is currently divided into **standard device control** and **custom device control**.

#### Standard Device Control (Beta)

##### Standard device feature set

What is the standard device feature set?

The functions of different products are different. For example, in the lighting category, the lamp is used as an example, there are functions such as switching, color adjustment, but for the electrical category, the socket is used as an example, there is no "color" function.

But for a large category, the basic common functions are the same. For example, all lighting categories have the function of switching.

Unify the definition of the function set of similar products and formulate a set of function instruction set rules, which is the standard function set.

> Because it needs to be compatible with a wide variety of product functions, the standard equipment control function is currently in a phase of open product adaptation.
>
> If you want to use this function, you can contact Tuya.



##### Does the device support standardization

The `standard` property (type `BOOL`) of the `TuyaSmartDeviceModel` class defines whether the current device supports standardized control

The `dpCodes` property defines the status of the current device and is called a standard dp code

Each key in the dpCodes dictionary corresponds to a dpCode of a function point, value corresponds to a dpValue of a function point, and dpValue is the value of the function point



##### Standard device control

```objective-c
[self.device publishDpWithCommands:dpCodesCommand success:^{
    NSLog(@"publishDpWithCommands success");
} failure:^(NSError *error) {
    NSLog(@"publishDpWithCommands failure: %@", error);
}];
```

【Commands】

```json
// light 
// Each product has a standard set of rules
// bool: open light
{"switch_led" : true}

// string: set color 
{"colour_data" : "009003e803e8"}

// enum: set white mode
{"work_mode" : "white"}

// value: set brightness 100
{"bright_value" : 100}

// Multiple combinations
{"work_mode" : "colour"}
{"colour_data" : "009003e803e8"}
```

##### Update device status

After the `TuyaSmartDeviceDelegate` delegate protocol is realized, user can update the UI of the App device control in the callback of device status change.

Objc:

```objective-c
- (void)initDevice {
    self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];
    self.device.delegate = self;
}

#pragma mark - TuyaSmartDeviceDelegate

- (void)device:(TuyaSmartDevice *)device dpCommandsUpdate:(NSDictionary *)dpCodes {
    NSLog(@"dpCommandsUpdate: %@", dpCodes);
    // TODO: refresh ui
}

- (void)deviceInfoUpdate:(TuyaSmartDevice *)device {
    // device info update
}

- (void)deviceRemoved:(TuyaSmartDevice *)device {
    // device removed
}
```

Swift:

```objective-c
func initDevice() {
    device = TuyaSmartDevice(deviceId: "your_device_id")
    device?.delegate = self
}

// MARK: - TuyaSmartDeviceDelegate

func device(_ device: TuyaSmartDevice!, dpCommandsUpdate dpCodes: [AnyHashable : Any]!) {
    print("dpCommandsUpdate: \(dpCodes)")
    // TODO: refresh ui
}

func deviceInfoUpdate(_ device: TuyaSmartDevice!) {
    // device info update
}

func deviceRemoved(_ device: TuyaSmartDevice!) {
    // device removed 
}
```


#### Custom Device Control

##### Functions of device

The `dps` (`NSDictionary` type) attribute of the `TuyaSmartDeviceModel` class defines the state of the device, and the state is called data point (DP) or function point.
Each `key` in the `dps` dictionary refers to a `dpId` of a function point, and `Value` refers to the `dpValue` of a function point. The `dpValue` is the value of the function point.
Refer to the functions of product in the [Tuya developer platform](https://iot.tuya.com/) for definition of function points of products. See the following figure.

![功能点](./images/ios_dp_sample.png)

The control instructions shall be sent in the format given below:

`{"<dpId>":"<dpValue>"}`

According to the definition of function points of the product in the back end, the example codes are as follows.

Objc:

```objc
- (void)publishDps {
  // self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

  NSDictionary *dps;
  
  // Set bool dp value to true
	dps = @{@"1": @(YES)};

  // Set string dp value to "ff5500"
	dps = @{@"4": @"ff5500"};

  // Set enum dp value to "Medium"
	dps = @{@"5": @"Medium"};

  // Set number dp value to 20
	dps = @{@"6": @(20)};

  // Set byte dp value to "1122"
	dps = @{@"15": @"1122"};

  // Send multiple dp values together
	dps = @{@"1": @(YES), @"4": @"ff5500"};

	[self.device publishDps:dps success:^{
		NSLog(@"publishDps success");

    // Publish dp success. device state change will be reported from deviceDpsUpdate delegate callback.

	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];


}
```
Swift:

```swift
func publishDps() {
    var dps = [String : Any]()
 
    // dps: Please refers to the specific product definition
  
    device?.publishDps(dps, success: {
 	    print("publishDps success")
        
      // Publish dp success. device state change will be reported from deviceDpsUpdate delegate callback.
    }, failure: { (error) in
        if let e = error {
            print("publishDps failure: \(e)")
        }
    })
}
```

##### Precautions:

- Special attention shall be paid to the type of data in sending the control commands. For example, the data type of function points shall be value, and the `@{@"2": @(25)}` instead of `@{@"2": @"25"}` shall be sent for the control command.
- In the transparent transmission, the byte string shall be the string format, and all letters shall be in the lower case, and the string must have even bits. The correct format shall be: `@{@"1": @"011f"}` instead of `@{@"1": @"11f"}`


For more concepts of function points, please refer to the [QuickStart-Related Concepts of Function Points](https://docs.tuya.com/en/product/function.html)


##### Device control

Device control supports three kinds of channel control, LAN control, cloud control, and automatic mode (if LAN is online, first go LAN control, LAN is not online, go cloud control)

##### LAN control:

```objc
	[self.device publishDps:dps mode:TYDevicePublishModeLocal success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```

##### Cloud control

```objc
	[self.device publishDps:dps mode:TYDevicePublishModeInternet success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```

##### Auto mode

```objc
	[self.device publishDps:dps mode:TYDevicePublishModeAuto success:^{
		NSLog(@"publishDps success");
	} failure:^(NSError *error) {
		NSLog(@"publishDps failure: %@", error);
	}];
```


##### Update device status

After the `TuyaSmartDeviceDelegate` delegate protocol is realized, user can update the UI of the App device control in the callback of device status change.

Objc:

```objc
- (void)initDevice {
	self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];
	self.device.delegate = self;
}

#pragma mark - TuyaSmartDeviceDelegate

- (void)device:(TuyaSmartDevice *)device dpsUpdate:(NSDictionary *)dps {
	NSLog(@"deviceDpsUpdate: %@", dps);
	// device dp state was updated
}

- (void)deviceInfoUpdate:(TuyaSmartDevice *)device {
  // device name, device online state change, etc.
}

- (void)deviceRemoved:(TuyaSmartDevice *)device {
  // device was removed.
}

```

Swift:

```swift
func initDevice() {
    device = TuyaSmartDevice(deviceId: "your_device_id")
    device?.delegate = self
}

// MARK: - TuyaSmartDeviceDelegate

func device(_ device: TuyaSmartDevice!, dpsUpdate dps: [AnyHashable : Any]!) {
    print("deviceDpsUpdate: \(dps)")
    // device dp state was updated
}

func deviceInfoUpdate(_ device: TuyaSmartDevice!) {
    // device name, device online state change, etc.
}

func deviceRemoved(_ device: TuyaSmartDevice!) {
    // device was removed.
}
```

### Modify the device name

Objc:

```objc
- (void)modifyDeviceName:(NSString *)mame {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device updateName:name success:^{
		NSLog(@"updateName success");
	} failure:^(NSError *error) {
		NSLog(@"updateName failure: %@", error);
	}];
}
```

Swift:

```swift
func modifyDeviceName(_ name: String) {
    device?.updateName(name, success: {
        print("updateName success")
    }, failure: { (error) in
        if let e = error {
            print("updateName failure: \(e)")
        }
    })
}
```

### Remove device

After a device is removed, it will be in the to-be-network-configured status (quick connection mode).

Objc:

```objc
- (void)removeDevice {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device remove:^{
		NSLog(@"remove success");
	} failure:^(NSError *error) {
		NSLog(@"remove failure: %@", error);
	}];
}
```

Swift:

```swift
func removeDevice() {
    device?.remove({
        print("remove success")
    }, failure: { (error) in
        if let e = error {
            print("remove failure: \(e)")
        }
    })
}
```

### Obtain Wifi signal strength of device

Objc:

```objc
- (void)getWifiSignalStrength {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];
    // self.device.delegate = self;

	[self.device getWifiSignalStrengthWithSuccess:^{
		NSLog(@"get wifi signal strength success");
	} failure:^(NSError *error) {
		NSLog(@"get wifi signal strength failure: %@", error);
	}];
}

#pragma mark - TuyaSmartDeviceDelegate

- (void)device:(TuyaSmartDevice *)device signal:(NSString *)signal {
    NSLog(@" signal : %@", signal);
}
```

Swift:

```swift
func getWifiSignalStrength() {
    self.device?.getWifiSignalStrength(success: {
        print("get wifi signal strength success")
    }, failure: { (error) in
        if let e = error {
            print("get wifi signal strength failure: \(e)")
        }
    })
}

// MARK: - TuyaSmartDeviceDelegate
func device(_ device: TuyaSmartDevice!, signal: String!) {

}
```

### Obtain the sub-device list of a gateway

Objc:

```objc
- (void)getSubDeviceList {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device getSubDeviceListFromCloudWithSuccess:^(NSArray<TuyaSmartDeviceModel *> *subDeviceList) {
        NSLog(@"get sub device list success");
    } failure:^(NSError *error) {
        NSLog(@"get sub device list failure: %@", error);
    }];
}
```

Swift:

```swift
func getSubDeviceList() {
    device?.getSubDeviceListFromCloud(success: { (subDeviceList) in
        print("get sub device list success")
    }, failure: { (error) in
        if let e = error {
            print("get sub device list failure: \(e)")
        }
    })
}
```


### Firmware upgrade

**Firmware upgrade process:**

Obtain device upgrade information -> send module upgrade instructions -> module upgrade succeeds -> send upgrade instructions to the device control module -> the upgrade of device control module succeeds

User obtain device upgrade information interface to get TuyaSmartFirmwareUpgradeModel, you can get firmware type from type property, get type description from typeDesc property.

#### Obtain device upgrade information:

Objc:

```objc
- (void)getFirmwareUpgradeInfo {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device getFirmwareUpgradeInfo:^(NSArray<TuyaSmartFirmwareUpgradeModel *> *upgradeModelList) {
		NSLog(@"getFirmwareUpgradeInfo success");
	} failure:^(NSError *error) {
		NSLog(@"getFirmwareUpgradeInfo failure: %@", error);
	}];
}
```

Swift:

```swift
func getFirmwareUpgradeInfo() {
    device?.getFirmwareUpgradeInfo({ (upgradeModelList) in
        print("getFirmwareUpgradeInfo success")
    }, failure: { (error) in
        if let e = error {
            print("getFirmwareUpgradeInfo failure: \(e)")
        }
    })
}
```

#### Send upgrade instruction:

Objc:

```objc
- (void)upgradeFirmware {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];
	// type: get firmware type from getFirmwareUpgradeInfo interface
	// TuyaSmartFirmwareUpgradeModel - type

	[self.device upgradeFirmware:type success:^{
		NSLog(@"upgradeFirmware success");
	} failure:^(NSError *error) {
		NSLog(@"upgradeFirmware failure: %@", error);
	}];
}
```

Swift:

```swift
func upgradeFirmware() {
    // type: get firmware type from getFirmwareUpgradeInfo interface
    // TuyaSmartFirmwareUpgradeModel - type
  
    device?.upgradeFirmware(type, success: {
        print("upgradeFirmware success")
    }, failure: { (error) in
        if let e = error {
            print("upgradeFirmware failure: \(e)")
        }
    })
}
```

#### Callback interface:
Objc:

```objc
- (void)deviceFirmwareUpgradeSuccess:(TuyaSmartDevice *)device type:(NSInteger)type {
	// firmware upgrade success
}

- (void)deviceFirmwareUpgradeFailure:(TuyaSmartDevice *)device type:(NSInteger)type {
	// firmware upgrade failure
}

- (void)device:(TuyaSmartDevice *)device firmwareUpgradeProgress:(NSInteger)type progress:(double)progress {
	// firmware upgrade progress
}

```

Swift:

```swift
func deviceFirmwareUpgradeSuccess(_ device: TuyaSmartDevice!, type: Int) {
    // firmware upgrade success
}

func deviceFirmwareUpgradeFailure(_ device: TuyaSmartDevice!, type: Int) {
    // firmware upgrade failure
}

func device(_ device: TuyaSmartDevice!, firmwareUpgradeProgress type: Int, progress: Double) {
    // firmware upgrade progress
}
```

