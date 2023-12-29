# Wherami Android SDK Tutorial

## Import Dependency
To use Wherami Android SDK, you need to import the library from the maven. Remember to [configure Maven repository](https://github.com/MTrec-PathAdvisor/HKUST-Path-Advisor-Localization.github.io/blob/main/Android%20SDK%20gradle%20setting.md) first.
```gradle
// app/src/build.gradle
implementation 'com.mtrec.wherami:offlinesdk:1.1.0-b5'

//if this import failed, please check if the maven repo is configured properly. Or, use the static library provided by Admin.
```

## Usage
Step1a: Configure the SDK: There are two parameters:

● CMS_HOST​ The domain/IP hosting the latest dataset for localization

● SITE_NAME ​The name of the site the dataset to be loaded for localization
```java
Client​.Configure(CMS_HOST, SITE_NAME, getApplicationContext());
//For example,
Client​.Configure("http://43.252.40.60", "HKUST_fusion", getApplicationContext());
```

Step1b: Configure Localization Engine type
```java
Map<String, Object> config = new HashMap<>();
config.put("wherami.lbs.sdk.core.MapEngineFactory:EngineType", "wherami.lbs.sdk.core.NativeMapEngine");
Client.ConfigExtra(config);
```

Step 2a: Get the fingerprint version:
```java
Client​.GetDataVersion()
```
If null is returned, it must download the fingerprint to continue (See Step 3)

Step 2b: Check if any fingerprint update is available
```java
Client​.CheckDataUpdate(new ​Client​.​DataUpdateQueryCallback​() { @Override
    public void onQueryFailed(​Exception​ e) { //Query failed
    }
    @Override
    public void onUpdateAvailable(​String​ newVersion) {
        //Update is available
    }
    @Override
    public void onLatestVersion() {
        //Already on latest version
    }
}, getApplicationContext());
```

Step 3: Update the fingerprint data if needed:
```java
Client​.UpdateData(new ​Client​.​DataUpdateCallback​() { @Override
    public void onProgressUpdated(​int​ percentage) { //Update progress
    }
    @Override
    public void onCompleted() {
        //Update completed
    }
    @Override
    public void onFailed(​Exception​ e) {
        //Update failed
    }
}, getApplicationContext());
```

Step 4a: Start localization in UI-less mode (with optional UI) In this mode, the engine is managed by the application itself.
```java
//Construct a MapEngine which will exist for the whole application lifetime
MapEngine ​engine = MapEngineFactory.Create(getApplicationContext());
//Alternatively construct a MapEngine to run as foreground service
MapEngine ​engine = MapEngineFactory.Create(getApplicationContext(), MapEngineFactory.MODE_FOREGROUND_SERVICE);
//Initialize the engine
engine.initialize();
//Implement the LocationUpdateCallback and pass to engine to receive location update.
engine.attachLocationUpdateCallback(this);
//Implement the PoiChangeCallback and pass to the engine to receive POI change update.
engine.attachPoiChangeCallback(this);
//Start the engine
engine.start();
```
A UI can be created optionally by adding MapFragment programmatically:
```java
FragmentManager​ fragMan = getFragmentManager();
FragmentTransaction​ fragTransaction = fragMan.beginTransaction();
//Pass the engine instance to the MapFragment
MapFragment ​mapFragment = MapFragment.newInstance(engine); fragTransaction.add(destLayout.getId(),mapFragment, “mapFragment”).commit();

//To stop localization, call:
engine.stop();
```

Step 4b: Start localization in UI mode
In this mode the MapFragment manages MapEngine, i.e. the MapEngine stops positioning once the MapFragment is not active. This provides a simple mean for localization feature drop-in by including the MapFragment. However the localization stops as the fragment is detached or paused.

Firstly, include the MapFragment in the Activity XML or programmatically:
```java
FragmentManager​ fragMan = getFragmentManager();
FragmentTransaction​ fragTransaction = fragMan.beginTransaction();
//Not passing the engine instance to the MapFragment
MapFragment ​mapFragment = MapFragment.newInstance();
fragTransaction.add(destLayout.getId(),mapFragment, “mapFragment”).commit();
```
The hosting Activity should implement the MapFragment.MapEngineReadyCallback
```java
public void onMapEngineReady(​MapEngine​ engine){
//Attach the necessary callbacks
//Implement the LocationUpdateCallback and pass to engine to receive location update.
engine.attachLocationUpdateCallback(this);
//Implement the PoiChangeCallback and pass to the engine to receive POI change update.
engine.attachPoiChangeCallback(this);
}
```
