# benegearble

benegearble是获取硬件设备数据的蓝牙框架，快速获取ECG125、ECG+、HRV、EEG、Temp等硬件设备的数据。

## 核心功能

- 接收广播包并解析
- 获取设备的硬件信息
- 读取设备的实时数据(波形数据)
- 下载设备的硬盘数据

## 下载地址

> implementation 'com.fjxdkj:benegearble:1.9.8'

## 注意事项

Android工程最低版本要19以上，即build.gradle中的minSdkVersion 至少要19以上

## 用法

### 1.添加权限

```java
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### 2.自定义Application并初始化

#### 2.1 自定义Application

```java
public class MyApplication extends Application {
}
```

#### 2.2 AndroidManifest.xml指定

```java
<application
    android:name=".MyApplication"
    ...
</application>
```

#### 2.3 初始化

重写onCreate方法

```java
@Override
public void onCreate() {
    super.onCreate();
    BeneGearBle.getInstance().init(this);
}
```

#### 2.4 可选的扫描配置

```java
@Override
public void onCreate() {
    super.onCreate();
    BeneGearBle.getInstance().init(this);
    
    BleScanRuleConfig scanRuleConfig = new BleScanRuleConfig.Builder()
            .setScanTimeOut(30*60*1000)  //扫描时间。当输入小于0的数时，无限循环扫描。
            .build()
    BeneGearBle.getInstance().initScanRule(scanRuleConfig);
}
```

#### 2.5 其他函数

```java
/**
 * 设置连接超时时间
 * seconds，单位是秒
 */
BeneGearBle.getInstance().setConnectOverTime(int seconds);
/**
 * 获取连接超时时间
 */
BeneGearBle.getInstance().getConnectOverTime();
/**
 * 开启Debug模式
 */
BeneGearBle.getInstance().enableLog(true);
/**
 * 获取最大可连接数量
 */
BeneGearBle.getInstance().getMaxConnectCount();
/**
 * 是否支持BLE协议
 */
BeneGearBle.getInstance().isSupportBle();
/**
 * 是否开启蓝牙
 */
BeneGearBle.getInstance().isBlueEnable();
/**
 * 指定设备是否连接
 * BaseDevice是所有硬件实体类的基类
 */
BeneGearBle.getInstance().isConnected(BaseDevice device);
/**
 * 指定的mac设备是否连接
 */
BeneGearBle.getInstance().isConnected(String mac);
```

### 3.动态申请权限

蓝牙扫描需要定位的权限。如果Android版本大于等于6，则要申请运行时权限。

```java
//是否已经申请了定位权限
private boolean  isHasPermission(){
        return ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED ;
    }

//申请权限
private void applyPermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, 1);
        }
}


//检查权限是否打开以及蓝牙开关也打开，则可以进行扫描
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode){
        case 1:
            boolean result=true;
            for(int i=0;i<grantResults.length;i++){
                if(grantResults[i]!=PackageManager.PERMISSION_GRANTED){
                    result=false;
                    break;
                }
            }
            if(result && BleManager.getInstance().isBlueEnable()){
                //可以进行扫描操作
            }
            break;
    }
}    
```

### 4.检查手机是否支持BLE协议

```java
if(!BeneGearBle.getInstance().isSupportBle()){
    Toast.makeText(this,"该手机不支持BLE协议",Toast.LENGTH_SHORT).show();
    return;
}
```

### 5.打开蓝牙开关

```java
if(!BeneGearBle.getInstance().isBlueEnable()){
    Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(intent, 0x01);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    switch (requestCode){
        case 0x01:
            if(resultCode==RESULT_OK){
                if(!isHasPermission())
                    applyPermission();
                else{
                   //可以进行扫描操作   
                }
            }
    }
}

```

### 6.完整扫描检查逻辑(仅供参考)

```java
//类成员
private LocationManager locationManager;

    
if (!BeneGearBle.getInstance().isSupportBle()) {
    Toast.makeText(this, "该手机不支持BLE协议", Toast.LENGTH_SHORT).show();
    return;
}
//Android10 申请位置权限需特殊处理
//判断是否打开位置权限
//再判断是否有打开蓝牙
//最后判断是否有申请定位权限
if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.Q){
    locationManager=(LocationManager) getSystemService(Context.LOCATION_SERVICE);
    if(locationManager!=null &&!locationManager.isLocationEnabled()){
        Intent intent = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
        startActivityForResult(intent, 0x02);
    }else{
        if(!BeneGearBle.getInstance().isBlueEnable()){
            Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(intent, 0x01);
        }else {
            if(!isHasPermission())
                applyPermission();
            else{
               //扫描操作
            }
        }
    }
}else{
    if(!BeneGearBle.getInstance().isBlueEnable()){
        Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
        startActivityForResult(intent, 0x01);
    }else {
        if(!isHasPermission())
            applyPermission();
        else{
            //扫描操作
        }
    }
}


//检查权限是否打开以及蓝牙开关也打开，则可以进行扫描
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode){
        case 1:
            boolean result=true;
            for(int i=0;i<grantResults.length;i++){
                if(grantResults[i]!=PackageManager.PERMISSION_GRANTED){
                    result=false;
                    break;
                }
            }
            if(result){
              //扫描操作
            }
            break;
    }
}    


@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    switch (requestCode){
        case 1:
            if( BeneGearBle.getInstance().isBlueEnable()){
                if(!isHasPermission())
                    applyPermission();
                else
                  //扫描操作
            }
        case 2:
            if(locationManager.isLocationEnabled()){
                if(!BeneGearBle.getInstance().isBlueEnable()){
                    Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
                    startActivityForResult(intent, 0x01);
                }else {
                    if(!isHasPermission()){
                        applyPermission();
                    } else{
                       //扫描操作
                    }
                }
            }
            break;
    }
}

//是否已经申请了定位权限
private boolean  isHasPermission(){
        return ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED ;
    }

//申请权限
private void applyPermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, 1);
        }
}

```

### 7.扫描相关操作

扫描：

```java
        BeneGearBle.getInstance().scan(new OnScanListener() {
            @Override
            public void onScanStarted() {
                Log.d("benegearble", "onScanStarted");
            }

            @Override
            public void onScanError(String msg) {
                Log.d("benegearble", "onScanError=" + msg);
            }

            @Override
            public void onScanFinish() {
                Log.d("benegearble", "onScanFinish");
            }

            @Override
            public void onECGPlusDiscovery(ECGPlusPackage ecgPlusPackage) {
                Log.d("benegearble", "ECGPlus广播包:" + ecgPlusPackage.toString());
                Log.d("benegearble", "ECGPlusDevice:" + ecgPlusPackage.getDevice());
                Log.d("benegearble", "电量:" + ecgPlusPackage.getBattery());
                Log.d("benegearble", "心率:" + ecgPlusPackage.getHr());
                Log.d("benegearble", "SQ:" + ecgPlusPackage.getSq());
                Log.d("benegearble", "步数:" +ecgPlusPackage.getStep());
                Log.d("benegearble", "运动强度:" + ecgPlusPackage.getExerciseIntensity());
                Log.d("benegearble", "接收时间:" + ecgPlusPackage.getTimestamp());
            }

            @Override
            public void onHRVDiscovery(HRVPackage hrvPackage) {
                Log.d("benegearble", "HRV广播包:" + hrvPackage.toString());
                Log.d("benegearble", "HRV设备:" + hrvPackage.getDevice());
                Log.d("benegearble", "电量:" + hrvPackage.getBattery());
                Log.d("benegearble", "心率:" + hrvPackage.getHr());
                Log.d("benegearble", "SQ:" + hrvPackage.getSq());
                Log.d("benegearble", "步数:" + hrvPackage.getStep());
                Log.d("benegearble", "运动强度:" + hrvPackage.getExerciseIntensity());
                Log.d("benegearble", "SDNN:" +hrvPackage.getSDNN());
                Log.d("benegearble", "TP:" + hrvPackage.getTP());
                Log.d("benegearble", "RMSSD:" + hrvPackage.getRMSSD());
                Log.d("benegearble", "PNN50:" +hrvPackage.getPNN50());
                Log.d("benegearble", "NN50:" + hrvPackage.getNN50());
                Log.d("benegearble", "LF:" + hrvPackage.getLF());
                Log.d("benegearble", "HF:" + hrvPackage.getHF());
                Log.d("benegearble", "LF/HF:" +hrvPackage.getLF_HF());
                Log.d("benegearble", "NLF:" + hrvPackage.getNLF());
                Log.d("benegearble", "NHF:" + hrvPackage.getNHF());
                Log.d("benegearble", "接收时间:" + hrvPackage.getTimestamp());
            }

            @Override
            public void onEEGDiscovery(EEGPackage eegPackage) {
                Log.d("benegearble", "EEG广播包:" + eegPackage.toString());
                Log.d("benegearble", "EEGDevice:" + eegPackage.getDevice());
                Log.d("benegearble", "电量:" + eegPackage.getBattery());
                Log.d("benegearble", "gamma:" + eegPackage.getGamma());
                Log.d("benegearble", "bata:" + eegPackage.getBata());
                Log.d("benegearble", "alpha:" + eegPackage.getAlpha());
                //0:无,1:左转,2:右转
                Log.d("benegearble", "眼动指令:" + eegPackage.getEyeDirection());
                //值越低越专注
                Log.d("benegearble", "专注力值:" + eegPackage.getAttention());
                //0:眼珠向前看且注意力分散，1:眼珠向前看且注意力集中
                //2:眼珠向左转且注意力集中,4:眼珠向左转且注意力分散
                //3:眼珠向右转且注意力集中,5:眼珠向右转且注意力分散
                //6:摇头3次以上(小车会倒退)
                Log.d("benegearble", "控制车指令:" + eegPackage.getCarOrder());
                //值：1到7，越高越专注
                Log.d("benegearble", "专注力等级:" + eegPackage.getAttentionLevel());
                //值：1-20（1最慢，20最快）
                Log.d("benegearble", "小车速度:" + eegPackage.getSpeed());
                Log.d("benegearble", "接收时间:" + eegPackage.getTimestamp());
            }

            @Override
            public void onTEMPDiscovery(TempPackage tempPackage) {
                Log.d("benegearble", "Temp广播包:" + tempPackage.toString());
                Log.d("benegearble", "TempDevice:" + tempPackage.getDevice());
                Log.d("benegearble", "温度:" + tempPackage.getTemp());
                Log.d("benegearble", "接收时间:" + tempPackage.getTimestamp());
            }
        });
```

停止扫描：

```java
 BeneGearBle.getInstance().cancelScan();
```

### 8.返回设备的硬件信息

```java
BeneGearBle.getInstance().getDeviceInfo(device, new OnGetDeviceInfoListener() {
    @Override
    public void onStart() {
        Log.d("benegearble","onStart");
    }

    @Override
    public void onSuccess(DeviceInfo deviceInfo) {
        Log.d("benegearble","制造商名称:"+deviceInfo.getManufacturerName());
        Log.d("benegearble","型号:"+deviceInfo.getModelID());
        Log.d("benegearble","序号:"+deviceInfo.getSerialID());
        Log.d("benegearble","硬件版本:"+deviceInfo.getHardwareVersion());
        Log.d("benegearble","固件版本:"+deviceInfo.getFirmwareVersion());
    }

    @Override
    public void onError(String s) {
        Log.d("benegearble","onError="+s);
    }
});
```

### 9.读取实时数据（波形数据）

能够读取实时数据的设备只有HRV、ECG+、ECG125以及EEG。

```java

BeneGearBle.getInstance().readDeviceMemory(device, new OnReadMemoryDataListener() {
    @Override
    public void onStart() {
        Log.d("benegearble","onStart");
    }

    @Override
    public void onError(String msg) {
        Log.d("benegearble","onError="+s);
    }

    @Override
    public void onSuccess(WaveformParam param) {
        Log.d("benegearble","onSuccess");
    }

   /**
     * 每秒回调8-10个波形数值
     * @param originalData 原始数值
     * @param data  真实波形数值
     */
    @Override
    public void onRead(double[] originalData, double[] data) {
        Log.d("benegearble",Arrays.toString(doubles));
    }

  /**
    * 主动停止读取、主动断开连接、设备意外断开连接会回调
    * @param isActiveDisConnected 是否主动断开连接，如果主动停止读取以及主动断开连接，那么isActiveDisConnected为true。意外断开为false
    */
    @Override
    public void onStop(boolean isActiveDisConnected) {
        Log.d("benegearble","onStop");
    }
});
```

暂停读取:

```
BeneGearBle.getInstance().pauseRead(device);
```

PS:与关闭读取的区别在于暂停读取还会保持会传感器的连接



关闭读取:

```java
BeneGearBle.getInstance().stopRead(device);
```

### 10.下载硬盘数据

注意：关闭下载硬盘数据的方法:

```java
BeneGearBle.getInstance().stopDownload(baseDevice);
```

#### 10.2 ECGPlus

ECGPlus共有5种硬盘数据，心率、正常波形、非正常波形、密度强度以及步数。分别有5五种指令对应。当需要只获取指定类型数据的时候，要对照下面的表。

| 指令                        | 对应数据   |
| --------------------------- | ---------- |
| Instruction.HATE            | 心率       |
| Instruction.NOR_WAREFORM    | 正常波形   |
| Instruction.UN_NOR_WAREFORM | 非正常波形 |
| Instruction.DENSITY         | 密度强度   |
| Instruction.STEP            | 步数       |

##### 10.2.1下载ECGPlus全部类型的硬盘数据

```java
BeneGearBle.getInstance().downloadHardDiskData(ecgPlusDevice, new OnDownloadHardDiskDataListener<ECGPlusHardDiskData>() {
        @Override
        public void onStart() {
            Log.d("benegearble","onStart");
        }

        @Override
        public void onError(String s) {
            Log.d("benegearble","onError="+s);
        }

        @Override
        public void onSuccess(ECGPlusHardDiskData ecgPlusHardDiskData) {
            Log.d("benegearble","心率数据="+ecgPlusHardDiskData.getHateDataList());
            Log.d("benegearble","正常波形数据="+ecgPlusHardDiskData.getNorWaveformDataList());
            Log.d("benegearble","非正常波形数据="+ecgPlusHardDiskData.getUnNorWaveformDataList());
            Log.d("benegearble","密度强度="+ecgPlusHardDiskData.getDensityDataList());
            Log.d("benegearble","步数="+ecgPlusHardDiskData.getStepDataList());
        }

        @Override
        public void onProgress(float v) {
            Log.d("benegearble","onProgress="+v);
        }
    });
```

##### 10.2.2 重载方法

```java
        /**
         * 下载ECGPlus指定单个类型的硬盘数据
         */
        BeneGearBle.getInstance().downloadHardDiskData(ECGPlusDevice ecgplusDevice, Instruction intruction, OnDownloadHardDiskDataListener<ECGPlusHardDiskData> linster);
        /**
         * 下载ECGPlus指定多个类型的硬盘数据
         */
        BeneGearBle.getInstance().downloadHardDiskData(ECGPlusDevice ecgplusDevice, Instruction[] intructions, OnDownloadHardDiskDataListener<ECGPlusHardDiskData> linster);
```

#### 10.3 HRV

HRV是ECGPLus的升级版，相对于ECGPlus多了一个心率变异的数据。当需要只获取指定类型数据的时候，要对照下面的表。

| 指令                        | 对应数据   |
| --------------------------- | ---------- |
| Instruction.HATE            | 心率       |
| Instruction.NOR_WAREFORM    | 正常波形   |
| Instruction.UN_NOR_WAREFORM | 非正常波形 |
| Instruction.DENSITY         | 密度强度   |
| Instruction.STEP            | 步数       |
| Instruction.HATE_VARIATION  | 心率变异   |

##### 10.3.1 下载HRV全部类型的硬盘数据

```java
BeneGearBle.getInstance().downloadHardDiskData(hrvDevice, new OnDownloadHardDiskDataListener<HRVHardDiskData>() {
    @Override
    public void onStart() {
        Log.d("benegearble","onStart");
    }

    @Override
    public void onError(String msg) {
        Log.d("benegearble","onError="+msg);
    }

    @Override
    public void onSuccess(HRVHardDiskData data) {
        Log.d("benegearble","所有笔的心率列表="+data.getHateList());
        List<HateData> hateDataList=data.getHateDataList();
        for(HateData item :hateDataList){
           Log.d("benegearble","该笔的心率列表:"+item.getmList().size());
           Log.d("benegearble","该笔不正常波形数据个数:"+item.getmList().size());
        }
        Log.d("benegearble","正常波形数据="+data.getNorWaveformDataList());
        Log.d("benegearble","非正常波形数据="+data.getUnNorWaveformDataList());
        Log.d("benegearble","密度强度="+data.getDensityDataList());
        Log.d("benegearble","步数="+data.getStepDataList());
        Log.d("benegearble","心率变异="+data.getHateVariationList());
    }

    @Override
    public void onProgress(float progress) {
        Log.d("benegearble","onProgress="+progress);
    }
});
```

##### 10.3.2  只下载其中一个类型的全部数据

```java
 BeneGearBle.getInstance().downloadHardDiskData(hrvDevice,Instruction.HATE, new OnDownloadHardDiskDataListener<HRVHardDiskData>() {});
```

##### 10.3.3 只下载其中一个类型以及指定时间的数据

注：2021-01-28 13:48:47的时间戳是1611812927000L

PS：获取数据是此时此刻至2021-01-28 13:48:47的HRV心率数据，不在该范围内的数据不会获取到

```java
        BeneGearBle.getInstance().downloadHardDiskData(hrvDevice,Instruction.HATE,1611812927000L, new OnDownloadHardDiskDataListener<HRVHardDiskData>() {});
```

##### 10.3.4 只下载多个类型以及指定时间的数据

注：2021-01-28 13:48:47的时间戳是1611812927000L

PS：获取数据是此时此刻至2021-01-28 13:48:47的HRV心率、心率变异以及不正常波形数据，不在该范围内的数据不会获取到

```java
        Instruction[] instructions=new Instruction[]{Instruction.HATE,Instruction.HATE_VARIATION,Instruction.UN_NOR_WAREFORM};
        BeneGearBle.getInstance().downloadHardDiskData(hrvDevice,instructions,1611812927000L, new OnDownloadHardDiskDataListener<HRVHardDiskData>() {});
```

##### 10.3.5 下载多个类型、指定时间的数据以及下完后是否断开连接

注：2021-01-28 13:48:47的时间戳是1611812927000L

PS：获取数据是此时此刻至2021-01-28 13:48:47的HRV心率、心率变异以及不正常波形数据，不在该范围内的数据不会获取到。false表示下载完后继续保持连接

```java
        Instruction[] instructions=new Instruction[]{Instruction.HATE,Instruction.HATE_VARIATION,Instruction.UN_NOR_WAREFORM};

        BeneGearBle.getInstance().downloadHardDiskData(hrvDevice,instructions,1611812927000L,false, new OnDownloadHardDiskDataListener<HRVHardDiskData>() {});
```

#### 10.4 EEG

EEG设备只有脑波数据，对应一个指令。

| 指令            | 对应数据 |
| --------------- | -------- |
| Instruction.EEG | 脑波     |

##### 10.4.1 下载EEG硬盘数据

```java
BeneGearBle.getInstance().downloadHardDiskData(eegDevice, new OnDownloadHardDiskDataListener<EEGHardDiskData>() {
    @Override
    public void onStart() {
        Log.d("benegearble","onStart");
    }

    @Override
    public void onError(String msg) {
        Log.d("benegearble","onError="+msg);
    }

    @Override
    public void onSuccess(EEGHardDiskData data) {
        List<BrainWave> list=data.getBrainWaveList();
        Log.d("benegearble","脑波数据="+data.getBrainWaveList().size());
        for(BrainWave item:list){
            Log.d("benegearble","alphaLowPower="+item.getAlphaLowPower());
            Log.d("benegearble","alphaMidPower="+item.getAlphaMidPower());
            Log.d("benegearble","alphaHighPower="+item.getAlphaHighPower());
            Log.d("benegearble","betaLowPower="+item.getBetaLowPower());
            Log.d("benegearble","betaMidPower="+item.getBetaMidPower());
            Log.d("benegearble","betaHighPower="+item.getBetaHighPower());
            Log.d("benegearble","deltaPower="+item.getDeltaPower());
            Log.d("benegearble","gammaPower="+item.getGammaPower());
            Log.d("benegearble","thetaPower="+item.getThetaPower());
            Log.d("benegearble","接收时间="+item.getTimestamp());
        }
    }

    @Override
    public void onProgress(float progress) {
        Log.d("benegearble","onProgress="+progress);
    }
});

```

##### 10.4.2 重载方法

```java
        /**
         * 下载指定时间EEG的脑波
         */
        BeneGearBle.getInstance().downloadHardDiskData(EEGDevice eegDevice,,int seconds, OnDownloadHardDiskDataListener<EEGHardDiskData> linster);
```

##### 10.5 Temp

Temp只有一种温度数据，对应一条指令。

| 指令             | 对应数据 |
| ---------------- | -------- |
| Instruction.TEMP | 温度     |

##### 10.5.1 下载EEG硬盘数据

```java
BeneGearBle.getInstance().downloadHardDiskData(tempDevice, new OnDownloadHardDiskDataListener<List<Temperature>>() {
    @Override
    public void onStart() {
        Log.d("benegearble","onStart");
    }

    @Override
    public void onError(String msg) {
        Log.d("benegearble","onError="+msg);
    }

    @Override
    public void onSuccess(List<Temperature> data) {
        for(Temperature item:data){
            Log.d("benegearble","温度值="+item.getTemp());
            Log.d("benegearble","接收时间="+item.getTimestamp());
        }
    }

    @Override
    public void onProgress(float progress) {
        Log.d("benegearble","onProgress="+progress);
    }
});
```

##### 10.5.2 重载方法

```java
        /**
         * 下载指定时间Temp的温度值
         */
        BeneGearBle.getInstance().downloadHardDiskData(TempDevice tempDevice,,int seconds, OnDownloadHardDiskDataListener<List<Temperature> linster);
```

## 问题咨询

如果在开发过程中出现问题，请联系

QQ：879768021

微信：everyguo0828

手机:18250000332

