# flutter_inapp_purchase
<p align="left">
  <a href="https://pub.dartlang.org/packages/flutter_inapp_purchase"><img alt="pub version" src="https://img.shields.io/pub/v/flutter_inapp_purchase.svg?style=flat-square"></a>
</p>

In App Purchase plugin for flutter. This project has been `forked` from [react-native-iap](https://github.com/dooboolab/react-native-iap). We are trying to share same experience of `in-app-purchase` in `flutter` as in `react-native`.
We will keep working on it as time goes by just like we did in `react-native-iap`.
Please give stars in github repository because we want to share this project in [opencollective](https://opencollective.com/react-native-iap). We need at least 100 stars to be there. Please help us to maintain and get more sponsors and backers.

`PR` is always welcomed.

## Breaking Changes
* There was parameter renaming in `0.5.0` to identify different parameters sent from the device. Please check the readme.

## Getting Started
Follow the [Medium Blog](https://medium.com/@dooboolab/flutter-in-app-purchase-7a3fb9345e2a) for the configuration.

For help getting started with Flutter, view our online
[documentation](https://flutter.io/).

For help on editing plugin code, view the [documentation](https://flutter.io/developing-packages/#edit-plugin-package).


## Methods
| Func  | Param  | Return | Description |
| :------------ |:---------------:| :---------------:| :-----|
| ~prepare~ |  | `String` | *Deprecated* Use initConnection instead. |
| initConnection |  | `String` | Prepare IAP module. Must be called on Android before any other purchase flow methods. In ios, it will simply call `canMakePayments` method and return value.|
| getProducts | `List<String>` Product IDs/skus | `List<IAPItem>` | Get a list of products (consumable and non-consumable items, but not subscriptions). Note: On iOS versions earlier than 11.2 this method _will_ return subscriptions if they are included in your list of SKUs. This is because we cannot differentiate between IAP products and subscriptions prior to 11.2.  |
| getSubscriptions | `List<String>` Subscription IDs/skus | `List<IAPItem>` | Get a list of subscriptions. Note: On iOS  this method has the same output as `getProducts`. Because iOS does not differentiate between IAP products and subscriptions.  |
| getPurchaseHistory | | `List<IAPItem>` | Gets an invetory of purchases made by the user regardless of consumption status (where possible) |
| getAvailablePurchases | | `List<PurchasedItem>` | (aka restore purchase) Get all purchases made by the user (either non-consumable, or haven't been consumed yet)
| buySubscription | `string` Subscription ID/sku, `string` Old Subscription ID/sku (on Android) | `PurchasedItem` | Create (buy) a subscription to a sku. For upgrading/downgrading subscription on Android pass second parameter with current subscription ID, on iOS this is handled automatically by store. |
| buyProduct | `string` Product ID/sku | `PurchasedItem` | Buy a product |
| buyProductWithoutFinishTransaction | `string` Product ID/sku | `PurchasedItem` | Buy a product without finish transaction call (iOS only) |
| finishTransaction | `void` | `String` | Send finishTransaction call to Apple IAP server. Call this function after receipt validation process |
| consumePurchase | `String` Purchase token | `String` | Consume a product (on Android.) No-op on iOS. |
| endConnection | | `String` | End billing connection (on Android.) No-op on iOS. |
| consumeAllItems | | `String` | Manually consume all items in android. Do NOT call if you have any non-consumables (one time purchase items). No-op on iOS. |


## Data Types
* IAPItem
  ```dart
  final String productId;
  final String price;
  final String currency;
  final String type;
  final String localizedPrice;
  final String title;
  final String description;
  final String introductoryPrice;
  /// ios only
  final String introductoryPricePaymentModeIOS;
  final String introductoryPriceNumberOfPeriodsIOS;
  final String introductoryPriceSubscriptionPeriodIOS;
  final String subscriptionPeriodNumberIOS;
  final String subscriptionPeriodUnitIOS;
  /// android only
  final String subscriptionPeriodAndroid;
  final String introductoryPriceCyclesAndroid;
  final String introductoryPricePeriodAndroid;
  final String freeTrialPeriodAndroid;
  ```

* PurchasedItem
  ```dart
  final dynamic transactionDate;
  final String transactionId;
  final String productId;
  final String transactionReceipt;
  final String purchaseToken;
  // Android only
  final bool autoRenewingAndroid;
  final String dataAndroid;
  final String signatureAndroid;
  // iOS only
  final dynamic originalTransactionDateIOS;
  final String originalTransactionIdentifierIOS;
  ```


## Install
Add ```flutter_inapp_purchase``` as a dependency in pubspec.yaml

For help on adding as a dependency, view the [documentation](https://flutter.io/using-packages/).


## Configuring in app purchase
- Please refer to [Blog](https://medium.com/@dooboolab/react-native-in-app-purchase-121622d26b67).


## Usage Guide
#### Android `connect` and `endConnection`
* You should start the billing service in android to use its funtionalities. We recommend you to use `initConnection` getter method in `initState()`. 
  ```dart
    /// start connection for android
    @override
    void initState() {
      super.initState();
      asyncInitState(); // async is not allowed on initState() directly
    }

    void asyncInitState() async {
      await FlutterInappPurchase.initConnection;
    }
  ```
* You should end the billing service in android when you are done with it. Otherwise it will be keep running in background. We recommend to use this feature in `dispose()`.
  ```
    /// start connection for android
    @override
    void dispose() async{
      super.dispose();
      await FlutterInappPurchase.endConnection;
    }
  ```
#### Get IAP items
  ```dart
  void getItems () async {
    List<IAPItem> items = await FlutterInappPurchase.getProducts(_productLists);
    for (var item in items) {
      print('${item.toString()}');
      this._items.add(item);
    }
  }
  ```

#### Purchase Item
  ```dart
  void purchase() async {
    PurchasedItem purchased = await FlutterInappPurchase.buyProduct(item.productId);
    print('purcuased - ${purchased.toString()}');
  }
  ```


## Example
Below code is just a `cp` from example project. You can test this in real example project.
```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => new _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final List<String>_productLists = Platform.isAndroid
      ? [
    'android.test.purchased',
    'point_1000',
    '5000_point',
    'android.test.canceled',
  ]
      : ['com.cooni.point1000','com.cooni.point5000'];

  String _platformVersion = 'Unknown';
  List<IAPItem> _items = [];

  @override
  void initState() {
    super.initState();
    initPlatformState();
  }

  // Platform messages are asynchronous, so we initialize in an async method.
  Future<void> initPlatformState() async {
    String platformVersion;
    // Platform messages may fail, so we use a try/catch PlatformException.
    try {
      platformVersion = await FlutterInappPurchase.platformVersion;
    } on PlatformException {
      platformVersion = 'Failed to get platform version.';
    }

    // initConnection
    var result = await FlutterInappPurchase.initConnection;
    print ('result: $result');

    // If the widget was removed from the tree while the asynchronous platform
    // message was in flight, we want to discard the reply rather than calling
    // setState to update our non-existent appearance.
    if (!mounted) return;

    setState(() {
      _platformVersion = platformVersion;
    });

    // refresh items for android
    String msg = await FlutterInappPurchase.consumeAllItems;
    print('consumeAllItems: $msg');
  }

  Future<Null> _buyProduct(IAPItem item) async {
    try {
      PurchasedItem purchased= await FlutterInappPurchase.buyProduct(item.productId);
      print('purcuased - ${purchased.toString()}');
    } catch (error) {
      print('$error');
    }
  }

  Future<Null> _getProduct() async {
    List<IAPItem> items = await FlutterInappPurchase.getProducts(_productLists);
    for (var item in items) {
      print('${item.toString()}');
      this._items.add(item);
    }

    setState(() {
      this._items = items;
    });
  }

  _renderInapps() {
    List<Widget> widgets = this._items.map((item) => Container(
      margin: EdgeInsets.symmetric(vertical: 10.0),
      child: Container(
        child: Column(
          children: <Widget>[
            Container(
              margin: EdgeInsets.only(bottom: 5.0),
              child: Text(
                item.toString(),
                style: TextStyle(
                  fontSize: 18.0,
                  color: Colors.black,
                ),
              ),
            ),
            FlatButton(
              color: Colors.orange,
              onPressed: () {
                this._buyProduct(item);
              },
              child: Row(
                children: <Widget>[
                  Expanded(
                    child: Container(
                      height: 48.0,
                      alignment: Alignment(-1.0, 0.0),
                      child: Text('Buy Item'),
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    )).toList();
    return widgets;
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Inapp Plugin by dooboolab'),
        ),
        body:
        Container(
          padding: EdgeInsets.all(10.0),
          child: ListView(
            children: <Widget>[
              Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisAlignment: MainAxisAlignment.start,
                children: <Widget>[
                  Container(
                    child: Text(
                      'Running on: $_platformVersion\n',
                      style: TextStyle(
                          fontSize: 18.0
                      ),
                    ),
                  ),
                  Container(
                    height: 60.0,
                    margin: EdgeInsets.only(bottom: 10.0),
                    child: ListView(
                      scrollDirection: Axis.horizontal,
                      children: <Widget>[
                        Row(
                          children: <Widget>[
                            Container(
                              margin: EdgeInsets.symmetric(horizontal: 15.0),
                              child: FlatButton(
                                color: Colors.green,
                                padding: EdgeInsets.all(0.0),
                                onPressed: () async {
                                  await FlutterInappPurchase.initConnection;
                                },
                                child: Container(
                                  padding: EdgeInsets.symmetric(horizontal: 20.0),
                                  alignment: Alignment(0.0, 0.0),
                                  child: Text(
                                    'Connect Billing',
                                    style: TextStyle(
                                      fontSize: 16.0,
                                    ),
                                  ),
                                ),
                              ),
                            ),
                            FlatButton(
                              color: Colors.green,
                              padding: EdgeInsets.all(0.0),
                              onPressed: () {
                                this._getProduct();
                              },
                              child: Container(
                                padding: EdgeInsets.symmetric(horizontal: 20.0),
                                alignment: Alignment(0.0, 0.0),
                                child: Text(
                                  'Get Items',
                                  style: TextStyle(
                                    fontSize: 16.0,
                                  ),
                                ),
                              ),
                            ),
                            Container(
                              margin: EdgeInsets.only(left: 15.0),
                              child: FlatButton(
                                color: Colors.green,
                                padding: EdgeInsets.all(0.0),
                                onPressed: () async {
                                  await FlutterInappPurchase.endConnection;
                                  setState(() {
                                    this._items = [];
                                  });
                                },
                                child: Container(
                                  padding: EdgeInsets.symmetric(horizontal: 20.0),
                                  alignment: Alignment(0.0, 0.0),
                                  child: Text(
                                    'End Connection',
                                    style: TextStyle(
                                      fontSize: 16.0,
                                    ),
                                  ),
                                ),
                              ),
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                  Column(
                    children: this._renderInapps(),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Help Maintenance
I've been maintaining quite many repos these days and burning out slowly. If you could help me cheer up, buying me a cup of coffee will make my life really happy and get much energy out of it.
<br/><a href="https://www.buymeacoffee.com/dooboolab" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/purple_img.png" alt="Buy Me A Coffee" style="height: auto !important;width: auto !important;" ></a>
