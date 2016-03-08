# Bluetooth Low Energy

> Android 4.3 (API Level 18) introduces built-in platform support for Bluetooth Low Energy in the central role and provides APIs that apps can use to discover devices, query for services, and read/write characteristics. In contrast to [Classic Bluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth.html), Bluetooth Low Energy (BLE) is designed to provide significantly lower power consumption. This allows Android apps to communicate with BLE devices that have low power requirements, such as proximity sensors, heart rate monitors, fitness devices, and so on.

Android4.3(API Level 18)よりBLE向けとなるCentral role APIの提供が開始された. 
このAPIを使用することにより, アプリはBLEデバイスを探索・発見することができ, serviceを問い合わせ, characteristicsのread/writeが可能となる. 
[従来のBluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth.html),と比べてBLEは大幅に低い消費電力となるよう設計されている. これはAndroidアプリケーションがBLEデバイスとcommunicateするのに近接センサーや心拍数モニタ, フィットネス機器のように少ない電力で済むことを意味する. 


## Key Terms and Concepts

> Here is a summary of key BLE terms and concepts:

> - **Generic Attribute Profile (GATT)** —The GATT profile is a general specification for sending and receiving short pieces of data known as "attributes" over a BLE link. All current Low Energy application profiles are based on GATT.
>     - The Bluetooth SIG defines many [profiles](https://www.bluetooth.org/en-us/specification/adopted-specifications) for Low Energy devices. A profile is a specification for how a device works in a particular application. Note that a device can implement more than one profile. For example, a device could contain a heart rate monitor and a battery level detector.

> - **Attribute Protocol (ATT)** —GATT is built on top of the Attribute Protocol (ATT). This is also referred to as GATT/ATT. ATT is optimized to run on BLE devices. To this end, it uses as few bytes as possible. Each attribute is uniquely identified by a Universally Unique Identifier (UUID), which is a standardized 128-bit format for a string ID used to uniquely identify information. The *attributes* transported by ATT are formatted as *characteristics* and *services*.

> - **Characteristic** —A characteristic contains a single value and 0-n descriptors that describe the characteristic's value. A characteristic can be thought of as a type, analogous to a class. 

> - **Descriptor** —Descriptors are defined attributes that describe a characteristic value. For example, a descriptor might specify a human-readable description, an acceptable range for a characteristic's value, or a unit of measure that is specific to a characteristic's value.

> - **Service** —A service is a collection of characteristics. For example, you could have a service called "Heart Rate Monitor" that includes characteristics such as "heart rate measurement." You can find a list of existing GATT-based profiles and services on [bluetooth.org](https://www.bluetooth.org/en-us/specification/adopted-specifications).

これらはBLEのキーワードとコンセプトである. 

 - **Generic Attribute Profile (GATT)** GATTプロファイルはBLE通信によるデータの送信や既知の"attributes"の断片を受信するための基本仕様である. 全てのBLEアプリケーションのプロファイルはGATTプロファイルがベースになる.
	 - Bluetooth SIGではいつくかのBLE[プロファイル](https://www.bluetooth.org/en-us/specification/adopted-specifications)が定義されている. プロファイルはデバイスが特定のアプリケーションでどのように動作するかの仕様である. デバイスは複数のプロファイルを実装できる点には注意すること. 例えばあるデバイスは心拍数モニターと電力消費モニターを含むことができる. 

 - **Attribute Protocal (ATT)** - GATTはAttribute Prorotocol(ATT)の上位に組み込まれる. これはGATT/ATTと呼ばれる. ATTはBLEデバイスでの実行に最適化されており, 少ないバイトの使用で実行できるよう作られている. それぞれのattributeは標準化された128bit形式の文字列IDであるUniversally Unique Identifier(UUID)で識別される. attributesはATTによりcharacteristicsとserviceの形式にされて転送される. 

 - **Characteristic** - characteristicは1つの値と0-nのcharacteristicに言及するdescriptorを含んでいる. characteristicはタイプやクラスに似たものと考えることができる. 

 - **Descriptor** - descriptorはcharacteristicの値を説明するattributeとして定義される. 例えば, characteristicの値が許容可能な範囲を人間が読めるdescriptionとして定義されるものであったり, characteristicの値の単位を表すものであったりする. 

 - **Service** - serviceはcharacteristicのコレクションである. 例えば, "Heart rate Monitor"と呼ばれるサービスには"heart rate measurement"という名のcharacteristicが含まれる. GATT-ベースのプロファルとサービスは[bluetooth.org](https://www.bluetooth.org/en-us/specification/adopted-specifications)で見つけることができる. 


### Roles and Responsibilities

> Here are the roles and responsibilities that apply when an Android device interacts with a BLE device:

> - Central vs. peripheral. This applies to the BLE connection itself. The device in the central role scans, looking for advertisement, and the device in the peripheral role makes the advertisement.
> - GATT server vs. GATT client. This determines how two devices talk to each other once they've established the connection.

> To understand the distinction, imagine that you have an Android phone and an activity tracker that is a BLE device. The phone supports the central role; the activity tracker supports the peripheral role (to establish a BLE connection you need one of each—two things that only support peripheral couldn't talk to each other, nor could two things that only support central).

> Once the phone and the activity tracker have established a connection, they start transferring GATT metadata to one another. Depending on the kind of data they transfer, one or the other might act as the server. For example, if the activity tracker wants to report sensor data to the phone, it might make sense for the activity tracker to act as the server. If the activity tracker wants to receive updates from the phone, then it might make sense for the phone to act as the server.

> In the example used in this document, the Android app (running on an Android device) is the GATT client. The app gets data from the GATT server, which is a BLE heart rate monitor that supports the [Heart Rate Profile](http://developer.bluetooth.org/TechnologyOverview/Pages/HRP.aspx). But you could alternatively design your Android app to play the GATT server role. See [BluetoothGattServer](https://developer.android.com/reference/android/bluetooth/BluetoothGattServer.html) for more information.

ここではAndroidデバイスがBLEデバイスと作用し合う際の役割と応答性について述べる. 

 - Central vs. peripheral. BLEコネクションに適用される形態である. centralなデバイスはスキャンし, advertisementを探す. peripheralなデバイスはadvertisementを作成する.
 - GATT server vs. GATT client. 2つのデバイス接続確立後の通信方法を決定する. 

各役割の違いを理解するためにAndroid PhoneとBLEデバイスであるActivity trackerを例に挙げる. 
PhoneはCentralの役割を果たす. Activity trackerはperipheralの役割を果たす(BLE接続するにはPeripheralだけでは成り立たないし, Centralだけでも成り立たない. 両方の役割をもつデバイスが揃う必要がある.)

PhoneとActivity trackerが接続を確立した後, デバイスは互いにGATT metadataの転送を開始する. 転送データの種類に応じてどちらかがserverとして動作する. 例えば, もしActivity trackerがセンサーデータをPhoneにレポートしたい場合, Activity trackerがserverとなるかもしれない. 一方でもしActivity trackerがPhoneからデータを受信してデータをアップデートしたい場合はPhoneがserverになるかもしれない. 

このドキュメントではAndroidデバイスで実行されるAndroid アプリケーションはGATT clientとして使用される. アプリは[Heart Rate Profile](http://developer.bluetooth.org/TechnologyOverview/Pages/HRP.aspx)をサポートするBLE heart rate monitorのデータをGATT serverから受信する. もしGATT serverとして振る舞うアプリを作成したい場合は[BluetoothGattServer](https://developer.android.com/reference/android/bluetooth/BluetoothGattServer.html) により詳しい情報が書いてある. 


## BLE Permissions

> In order to use Bluetooth features in your application, you must declare the Bluetooth permission [BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH). You need this permission to perform any Bluetooth communication, such as requesting a connection, accepting a connection, and transferring data.

> If you want your app to initiate device discovery or manipulate Bluetooth settings, you must also declare the [BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN) permission. **Note:** If you use the [BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN) permission, then you must also have the [BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH) permission.

BLEアプリケーションはBluetooth機能を使用する. そのためには[BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH)パーミッションを宣言する必要がある. アプリがBluetoothコミュニケーション(接続要求/接続受理/データ転送)をする場合にこのパーミッションを宣言する必要がある. 

もしBluetoothに関する設定を操作するのであれば[BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN)のパーミッションも宣言する必要がある. **NOTE:** もし[BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN)のパーミッションを使用するのであれば同時に[BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH)のパーミッションも必要である. 

> Declare the Bluetooth permission(s) in your application manifest file. For example:

アプリケーションのAndroidManifestでこれらのパーミッションを宣言するには下記を参照. 

```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

> If you want to declare that your app is available to BLE-capable devices only, include the following in your app's manifest:

もしBLE機能を備えたデバイスでのみアプリを有効にしたいのであれば次の1行をAndroidManifestに加えると良い. これによりBLE機能を備えていないデバイスではアプリは無効となりインストールされない. 

```xml
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

> However, if you want to make your app available to devices that don't support BLE, you should still include this element in your app's manifest, but set `required="false"`. Then at run-time you can determine BLE availability by using [PackageManager.hasSystemFeature()](https://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)):

しかし, BLE機能を備えていないデバイスでもアプリは有効にしインストールまでは進めたい場合は`required="false"`で宣言しておく. アプリ実行時にBLE機能の有無を調べる場合は[PackageManager.hasSystemFeature()](https://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))を使用する. 

```java
// Use this check to determine whether BLE is supported on the device. Then
// you can selectively disable BLE-related features.
if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
    Toast.makeText(this, R.string.ble_not_supported, Toast.LENGTH_SHORT).show();
    finish();
}
```


## Setting Up BLE

> Before your application can communicate over BLE, you need to verify that BLE is supported on the device, and if so, ensure that it is enabled. Note that this check is only necessary if `<uses-feature.../>` is set to false.

> If BLE is not supported, then you should gracefully disable any BLE features. If BLE is supported, but disabled, then you can request that the user enable Bluetooth without leaving your application. This setup is accomplished in two steps, using the [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html).

アプリケーションでBLE通信を始める前に, デバイスがBLE機能を有しているか検証する必要ががる. これはAndroidManifestの`<uses-feature.../>`の`required="false"`を設定している場合に必要である. 

デバイスがBLEをサポートしていないのであればそれに関わる機能を全て無効かする必要がある. BLEをサポートしているものの機能が無効かされている場合, ユーザにこれを有効とするようリクエストすることができる. これには[BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)を使って2つの手順を実行する. 

> 1. Get the [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)
The [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html) is required for any and all Bluetooth activity. The [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html) represents the device's own Bluetooth adapter (the Bluetooth radio). There's one Bluetooth adapter for the entire system, and your application can interact with it using this object. The snippet below shows how to get the adapter. Note that this approach uses [getSystemService()](https://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String)) to return an instance of [BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html), which is then used to get the adapter. Android 4.3 (API Level 18) introduces [BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html):
>  
> 2. Enable Bluetooth
Next, you need to ensure that Bluetooth is enabled. Call [isEnabled()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#isEnabled()) to check whether Bluetooth is currently enabled. If this method returns false, then Bluetooth is disabled. The following snippet checks whether Bluetooth is enabled. If it isn't, the snippet displays an error prompting the user to go to Settings to enable Bluetooth:

 1. [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)を取得する
[BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)はBluetoothを使用する場合に必要となる. [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)はデバイス自身のBluetoothアダプタ(Bluetooth radio)に相当する. システムが持つBluetoothアダプタを使用してアプリケーションは通信することができる. 次のスニペットはadapterを取得するものである. これは[getSystemService()](https://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String))を使用して[BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html)を得るアプローチである. [BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html)はAndroid 4.3(API Level 18)から導入されている. 

	```java
	// Initializes Bluetooth adapter.
	final BluetoothManager bluetoothManager =
	        (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
	mBluetoothAdapter = bluetoothManager.getAdapter();
	```

 2. Bluetoothの有効化
次にBluetoothを有効化する. [isEnabled()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#isEnabled())でBluetoothが現在有効かどうかを検証できる. もしこのメソッドがfalseを返せばBluetoothは無効化されている. 次のスニペットはBluetoothが有効かどうかを検証し, 無効であればユーザにBluetoothを有効にするよう促すプロンプトを表示する. 

	```java
	private BluetoothAdapter mBluetoothAdapter;
	...
	// Ensures Bluetooth is available on the device and it is enabled. If not,
	// displays a dialog requesting user permission to enable Bluetooth.
	if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
	    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
	    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
	}
	```


## Finding BLE Device

> To find BLE devices, you use the [startLeScan()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback)) method. This method takes a [BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html) as a parameter. You must implement this callback, because that is how scan results are returned. Because scanning is battery-intensive, you should observe the following guidelines:

> - As soon as you find the desired device, stop scanning.
> - Never scan on a loop, and set a time limit on your scan. A device that was previously available may have moved out of range, and continuing to scan drains the battery.

> The following snippet shows how to start and stop a scan:

BLEデバイスを見つけるために[startLeScan()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback))メソッドが使用できる. このメソッドは[BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html)をパラメータに取る. あなたはこのcallbackを実装し, デバイススキャン結果を確認する. デバイススキャンはバッテリーを多く消費するため次のガイドラインに従うこと. 

 - 期待するデバイスが見つかったらスキャンを停止する. 
 - スキャンし続けないこと. timeoutを設けること. 接続されたデバイスが圏外移動し, スキャンを続けているとバッテリーを消費し続けることになる. 

次はスキャンの開始と停止のスニペットである. 

```java
/**
 * Activity for scanning and displaying available BLE devices.
 */
public class DeviceScanActivity extends ListActivity {

    private BluetoothAdapter mBluetoothAdapter;
    private boolean mScanning;
    private Handler mHandler;

    // Stops scanning after 10 seconds.
    private static final long SCAN_PERIOD = 10000;
    ...
    private void scanLeDevice(final boolean enable) {
        if (enable) {
            // Stops scanning after a pre-defined scan period.
            mHandler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    mScanning = false;
                    mBluetoothAdapter.stopLeScan(mLeScanCallback);
                }
            }, SCAN_PERIOD);

            mScanning = true;
            mBluetoothAdapter.startLeScan(mLeScanCallback);
        } else {
            mScanning = false;
            mBluetoothAdapter.stopLeScan(mLeScanCallback);
        }
        ...
    }
...
}
```

> If you want to scan for only specific types of peripherals, you can instead call [startLeScan(UUID[], BluetoothAdapter.LeScanCallback)](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback)), providing an array of [UUID](https://developer.android.com/reference/java/util/UUID.html) objects that specify the GATT services your app supports.

> Here is an implementation of the [BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html), which is the interface used to deliver BLE scan results:

もし特定のperipheralのみを探す場合, 代わりに[startLeScan(UUID[], BluetoothAdapter.LeScanCallback)](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback))を使うことができる. アプリケーションがサポートしているGATT serviceの[UUID](https://developer.android.com/reference/java/util/UUID.html)のarrayを提供すればよい. 

下記は[BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html)を実装し, BLEデバイスのスキャン結果を使用するサンプルである. 

```java
private LeDeviceListAdapter mLeDeviceListAdapter;
...
// Device scan callback.
private BluetoothAdapter.LeScanCallback mLeScanCallback =
        new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(final BluetoothDevice device, int rssi,
            byte[] scanRecord) {
        runOnUiThread(new Runnable() {
           @Override
           public void run() {
               mLeDeviceListAdapter.addDevice(device);
               mLeDeviceListAdapter.notifyDataSetChanged();
           }
       });
   }
};
```

> **Note:** You can only scan for Bluetooth LE devices or scan for Classic Bluetooth devices, as described in [Bluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth.html). You cannot scan for both Bluetooth LE and classic devices at the same time.

**NOTE:** アプリはBLEデバイスがClassic BTデバイスどちらか片方しか一度にスキャンできない. これについては[Bluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth.html)で述べている.


## Connecting to a GATT Server

> The first step in interacting with a BLE device is connecting to it— more specifically, connecting to the GATT server on the device. To connect to a GATT server on a BLE device, you use the [connectGatt()](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#connectGatt(android.content.Context, boolean, android.bluetooth.BluetoothGattCallback)) method. This method takes three parameters: a [Context](https://developer.android.com/reference/android/content/Context.html) object, `autoConnect` (boolean indicating whether to automatically connect to the BLE device as soon as it becomes available), and a reference to a [BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html):

BLEデバイスと通信するためのfirst stepとしてそれと接続すること. すなわちデバイスからGATT serverに接続する. BLEデバイスがGATT serverに接続するには[connectGatt()](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#connectGatt(android.content.Context, boolean, android.bluetooth.BluetoothGattCallback))メソッドを使用する. このメソッドは[Context](https://developer.android.com/reference/android/content/Context.html), `autoConnect`(利用可能になったBLEデバイスに自動接続するかどうかのフラグ), そして[BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html)をパラメータにとる. 

```java
mBluetoothGatt = device.connectGatt(this, false, mGattCallback);
```

> This connects to the GATT server hosted by the BLE device, and returns a [BluetoothGatt](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html) instance, which you can then use to conduct GATT client operations. The caller (the Android app) is the GATT client. The [BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html) is used to deliver results to the client, such as connection status, as well as any further GATT client operations.

 > In this example, the BLE app provides an activity (`DeviceControlActivity`) to connect, display data, and display GATT services and characteristics supported by the device. Based on user input, this activity communicates with a [Service](https://developer.android.com/reference/android/app/Service.html) called `BluetoothLeService`, which interacts with the BLE device via the Android BLE API:

これはBLEデバイスによってホストされたGATT serverへの接続と, [BluetoothGatt](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html)のインスタンスを返却する. BluetoothGattインスタンスでGATT clientとしての操作が可能になる. 呼び出し元であるAndroid アプリがGATT clientとして振る舞う時, [BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html)は接続状態や他のGATT client操作を伝送するために使用される. 

今回の例では, BLEアプリは`DeviceControlActivity`でBLEデバイスに接続し, データを表示し, GATT serviceとデバイスがサポートしているcharacteristicsを表示する. ユーザ入力を契機に
Android BLE APIを使用してBLEデバイスと通信する`BluetoothLeService`とコミュニケーションする.

```java
// A service that interacts with the BLE device via the Android BLE API.
public class BluetoothLeService extends Service {
    private final static String TAG = BluetoothLeService.class.getSimpleName();

    private BluetoothManager mBluetoothManager;
    private BluetoothAdapter mBluetoothAdapter;
    private String mBluetoothDeviceAddress;
    private BluetoothGatt mBluetoothGatt;
    private int mConnectionState = STATE_DISCONNECTED;

    private static final int STATE_DISCONNECTED = 0;
    private static final int STATE_CONNECTING = 1;
    private static final int STATE_CONNECTED = 2;

    public final static String ACTION_GATT_CONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_CONNECTED";
    public final static String ACTION_GATT_DISCONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_DISCONNECTED";
    public final static String ACTION_GATT_SERVICES_DISCOVERED =
            "com.example.bluetooth.le.ACTION_GATT_SERVICES_DISCOVERED";
    public final static String ACTION_DATA_AVAILABLE =
            "com.example.bluetooth.le.ACTION_DATA_AVAILABLE";
    public final static String EXTRA_DATA =
            "com.example.bluetooth.le.EXTRA_DATA";

    public final static UUID UUID_HEART_RATE_MEASUREMENT =
            UUID.fromString(SampleGattAttributes.HEART_RATE_MEASUREMENT);

    // Various callback methods defined by the BLE API.
    private final BluetoothGattCallback mGattCallback =
            new BluetoothGattCallback() {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status,
                int newState) {
            String intentAction;
            if (newState == BluetoothProfile.STATE_CONNECTED) {
                intentAction = ACTION_GATT_CONNECTED;
                mConnectionState = STATE_CONNECTED;
                broadcastUpdate(intentAction);
                Log.i(TAG, "Connected to GATT server.");
                Log.i(TAG, "Attempting to start service discovery:" +
                        mBluetoothGatt.discoverServices());

            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                intentAction = ACTION_GATT_DISCONNECTED;
                mConnectionState = STATE_DISCONNECTED;
                Log.i(TAG, "Disconnected from GATT server.");
                broadcastUpdate(intentAction);
            }
        }

        @Override
        // New services discovered
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);
            } else {
                Log.w(TAG, "onServicesDiscovered received: " + status);
            }
        }

        @Override
        // Result of a characteristic read operation
        public void onCharacteristicRead(BluetoothGatt gatt,
                BluetoothGattCharacteristic characteristic,
                int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
            }
        }
     ...
    };
...
}
```

> When a particular callback is triggered, it calls the appropriate `broadcastUpdate()` helper method and passes it an action. Note that the data parsing in this section is performed in accordance with the Bluetooth Heart Rate Measurement [profile specifications](http://developer.bluetooth.org/gatt/characteristics/Pages/CharacteristicViewer.aspx?u=org.bluetooth.characteristic.heart_rate_measurement.xml):

特定のcallbackをトリガした際に, 適切な`broadcastUpdate()`ヘルパメソッドを呼び出しactionを渡す. このセクションでBluetooth Heart Rate Measurementプロファイルの使用に従ってデータをパースする. 

```java
private void broadcastUpdate(final String action) {
    final Intent intent = new Intent(action);
    sendBroadcast(intent);
}

private void broadcastUpdate(final String action,
                             final BluetoothGattCharacteristic characteristic) {
    final Intent intent = new Intent(action);

    // This is special handling for the Heart Rate Measurement profile. Data
    // parsing is carried out as per profile specifications.
    if (UUID_HEART_RATE_MEASUREMENT.equals(characteristic.getUuid())) {
        int flag = characteristic.getProperties();
        int format = -1;
        if ((flag & 0x01) != 0) {
            format = BluetoothGattCharacteristic.FORMAT_UINT16;
            Log.d(TAG, "Heart rate format UINT16.");
        } else {
            format = BluetoothGattCharacteristic.FORMAT_UINT8;
            Log.d(TAG, "Heart rate format UINT8.");
        }
        final int heartRate = characteristic.getIntValue(format, 1);
        Log.d(TAG, String.format("Received heart rate: %d", heartRate));
        intent.putExtra(EXTRA_DATA, String.valueOf(heartRate));
    } else {
        // For all other profiles, writes the data formatted in HEX.
        final byte[] data = characteristic.getValue();
        if (data != null && data.length > 0) {
            final StringBuilder stringBuilder = new StringBuilder(data.length);
            for(byte byteChar : data)
                stringBuilder.append(String.format("%02X ", byteChar));
            intent.putExtra(EXTRA_DATA, new String(data) + "\n" +
                    stringBuilder.toString());
        }
    }
    sendBroadcast(intent);
}
```

> Back in `DeviceControlActivity`, these events are handled by a [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html):

`DeviceControlActivity`に戻って, これらのイベントは[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)でハンドルされる. 

```java
// Handles various events fired by the Service.
// ACTION_GATT_CONNECTED: connected to a GATT server.
// ACTION_GATT_DISCONNECTED: disconnected from a GATT server.
// ACTION_GATT_SERVICES_DISCOVERED: discovered GATT services.
// ACTION_DATA_AVAILABLE: received data from the device. This can be a
// result of read or notification operations.
private final BroadcastReceiver mGattUpdateReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();
        if (BluetoothLeService.ACTION_GATT_CONNECTED.equals(action)) {
            mConnected = true;
            updateConnectionState(R.string.connected);
            invalidateOptionsMenu();
        } else if (BluetoothLeService.ACTION_GATT_DISCONNECTED.equals(action)) {
            mConnected = false;
            updateConnectionState(R.string.disconnected);
            invalidateOptionsMenu();
            clearUI();
        } else if (BluetoothLeService.
                ACTION_GATT_SERVICES_DISCOVERED.equals(action)) {
            // Show all the supported services and characteristics on the
            // user interface.
            displayGattServices(mBluetoothLeService.getSupportedGattServices());
        } else if (BluetoothLeService.ACTION_DATA_AVAILABLE.equals(action)) {
            displayData(intent.getStringExtra(BluetoothLeService.EXTRA_DATA));
        }
    }
};
```


## Reading BLE Attributes

> Once your Android app has connected to a GATT server and discovered services, it can read and write attributes, where supported. For example, this snippet iterates through the server's services and characteristics and displays them in the UI:

AndroidアプリがGATT serverへの接続を保持し, serviceを発見し, それのattributeを読み書きでき, それらがサポートされている場合. 例えば次のスニペットはGATT serverがもつserviceとcharacteristicsを反復しながらUI表示するものである. 

```java
public class DeviceControlActivity extends Activity {
    ...
    // Demonstrates how to iterate through the supported GATT
    // Services/Characteristics.
    // In this sample, we populate the data structure that is bound to the
    // ExpandableListView on the UI.
    private void displayGattServices(List<BluetoothGattService> gattServices) {
        if (gattServices == null) return;
        String uuid = null;
        String unknownServiceString = getResources().
                getString(R.string.unknown_service);
        String unknownCharaString = getResources().
                getString(R.string.unknown_characteristic);
        ArrayList<HashMap<String, String>> gattServiceData =
                new ArrayList<HashMap<String, String>>();
        ArrayList<ArrayList<HashMap<String, String>>> gattCharacteristicData
                = new ArrayList<ArrayList<HashMap<String, String>>>();
        mGattCharacteristics =
                new ArrayList<ArrayList<BluetoothGattCharacteristic>>();

        // Loops through available GATT Services.
        for (BluetoothGattService gattService : gattServices) {
            HashMap<String, String> currentServiceData =
                    new HashMap<String, String>();
            uuid = gattService.getUuid().toString();
            currentServiceData.put(
                    LIST_NAME, SampleGattAttributes.
                            lookup(uuid, unknownServiceString));
            currentServiceData.put(LIST_UUID, uuid);
            gattServiceData.add(currentServiceData);

            ArrayList<HashMap<String, String>> gattCharacteristicGroupData =
                    new ArrayList<HashMap<String, String>>();
            List<BluetoothGattCharacteristic> gattCharacteristics =
                    gattService.getCharacteristics();
            ArrayList<BluetoothGattCharacteristic> charas =
                    new ArrayList<BluetoothGattCharacteristic>();
           // Loops through available Characteristics.
            for (BluetoothGattCharacteristic gattCharacteristic :
                    gattCharacteristics) {
                charas.add(gattCharacteristic);
                HashMap<String, String> currentCharaData =
                        new HashMap<String, String>();
                uuid = gattCharacteristic.getUuid().toString();
                currentCharaData.put(
                        LIST_NAME, SampleGattAttributes.lookup(uuid,
                                unknownCharaString));
                currentCharaData.put(LIST_UUID, uuid);
                gattCharacteristicGroupData.add(currentCharaData);
            }
            mGattCharacteristics.add(charas);
            gattCharacteristicData.add(gattCharacteristicGroupData);
         }
    ...
    }
...
}
```


## Receiving GATT Notifications

> It's common for BLE apps to ask to be notified when a particular characteristic changes on the device. This snippet shows how to set a notification for a characteristic, using the [setCharacteristicNotification()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#setCharacteristicNotification(android.bluetooth.BluetoothGattCharacteristic, boolean)) method:

BLEアプリに共通することとして, デバイス上で特定のcharacteristicの変更通知を受け取ることがある. 次のスニペットは[setCharacteristicNotification()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#setCharacteristicNotification(android.bluetooth.BluetoothGattCharacteristic, boolean))メソッドを用いてcharacteristicの変更通知を受け取る方法である. 

```java
private BluetoothGatt mBluetoothGatt;
BluetoothGattCharacteristic characteristic;
boolean enabled;
...
mBluetoothGatt.setCharacteristicNotification(characteristic, enabled);
...
BluetoothGattDescriptor descriptor = characteristic.getDescriptor(
        UUID.fromString(SampleGattAttributes.CLIENT_CHARACTERISTIC_CONFIG));
descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
mBluetoothGatt.writeDescriptor(descriptor);
```

> Once notifications are enabled for a characteristic, an [onCharacteristicChanged()](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicChanged(android.bluetooth.BluetoothGatt, android.bluetooth.BluetoothGattCharacteristic)) callback is triggered if the characteristic changes on the remote device:

characteristicの変更通知が有効になると, リモートデバイスのcharacteristicが変更されると[onCharacteristicChanged()](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicChanged(android.bluetooth.BluetoothGatt, android.bluetooth.BluetoothGattCharacteristic))が呼ばれる. 

```java
@Override
// Characteristic notification
public void onCharacteristicChanged(BluetoothGatt gatt,
        BluetoothGattCharacteristic characteristic) {
    broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
}
```


## Closing the Client App

> Once your app has finished using a BLE device, it should call [close()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#close()) so the system can release resources appropriately:

アプリがBLEデバイスの使用を終える時, [close()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#close())メソッドを呼び出し, システムが適切にリソースを解放できるようにすること. 

```java
public void close() {
    if (mBluetoothGatt == null) {
        return;
    }
    mBluetoothGatt.close();
    mBluetoothGatt = null;
}
```


## 参考 

 - [Qiita - Bluetooth Low Energy（BLE）/ iBeaconとは by miyatay](http://qiita.com/miyatay/items/4d4ce4ccd7905ddc0144)

> **License:**
> Portions of this page are modifications based on work created and shared by the Android Open Source Project and used according to terms described in the Creative Commons 2.5 Attribution License.

original link: https://developer.android.com/guide/topics/connectivity/bluetooth-le.html#notification