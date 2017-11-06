# HyperLog Android

[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://opensource.org/licenses/MIT) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Download](https://api.bintray.com/packages/piyushgupta27/maven/smart-scheduler/images/download.svg) ](https://bintray.com/piyushgupta27/maven/smart-scheduler/_latestVersion)
## Overview
A utility logger library for Android on top of standard Android `Log` class for debugging purpose. This is a simple library that will allow Android apps or library to store `log` into `database` so that developer can pull the logs from the database into `File` or push the logs to your `remote server` for `debugging` purpose. Want to know more on this and wondering why you should prefer using this library over doing it yourself. Check out the blog post.

<p align="center">
<kbd>
<img src="asset/device_logger.gif" alt="Device Logger" width="380" height="633">
</kbd>
</p>

## Log Format
```
timeStamp + " | " + appVersion + " : " + osVersion + " | " + deviceUUID + " | [" + logLevelName + "]: " + message
```
```
2017-10-05T14:46:36.541Z 1.0 | 0.0.1 : Android-7.1.1 | 62bb1162466c3eed | [INFO]: Log has been pushed
```
## Download
Download the latest version or grab via Gradle.

The library is available on [`mavenCentral()`](https://dl.bintray.com/piyushgupta27/maven/com/hypertrack/hyperlog/) and `jcenter()`. In your module's `build.gradle`, add the following code snippet and run the `gradle-sync`.


```
dependencies {
    ...
    compile 'com.hypertrack:hyperlog:0.0.5'
    ...
}
```

## Initialize
Inside `onCreate` of Application class or Launcher Activity. 
```
HyperLog.initialize(this);
HyperLog.setLogLevel(Log.VERBOSE);
```

## Usage
```
HyperLog.d(TAG,"Debug Log");
```

## Get Logs in a File
```
File file = HyperLog.getDeviceLogsInFile(this);
```

## Push Logs Files to Remote Server
Logs file can be pushed to your remote server or `RequestBin`(for testing) or to [`Logstash`](https://www.elastic.co/products/logstash).

**Steps:**

1. Set the API Endpoint URL `HyperLog.setURL` before calling `HyperLog.pushLogs` method otherwise `exception` will be thrown. Developers can also set a testing endpoint using [`RequestBin`](https://requestb.in/).
```
HyperLog.setURL("API URL");
```

2. Push Logs file to the given endpoint.
```
HyperLog.pushLogs(this, false, new HyperLogCallback() {
            @Override
            public void onSuccess(@NonNull String response) {

            }

            @Override
            public void onError(@NonNull VolleyError errorResponse) {

            }
});
```

**Follow steps to setup testing endpoint at [`RequestBin`](https://requestb.in/)**

1. Visit the [`RequestBin`](https://requestb.in/) site and create a `RequestBin`.
2. Once you have the bin created, copy the URL and set it to the `HyperLog.setURL`.
3. Now call `HyperLog.pushLogs` to push the logs file to given endpoint. This is asynchronous call.
4. After `HyperLog.pushLogs` success, reload the requestbin page to view exactly which requests were made, what headers were sent, and raw body and so on, all in a pretty graphical setting like below image.
5. Once you get the logs on `RequestBin` create your own endpoint on your server and start receiving logs on to your server for debugging.
6. (Optional) From your server you can directly push those to [`Logstash`](https://www.elastic.co/products/logstash) (part of [ELK](https://www.elastic.co/webinars/introduction-elk-stack) stack). We have discussed ELK in one of our [blog](https://www.hypertrack.com/blog/2017/02/10/centralized-logging-at-hypertrack/).

<p align="center">
<kbd>
<img src="asset/request_bin.png" alt="RequestBin" ->
</kbd>
</p>
<p align="center">
<em>Request Bin Sample Response</em>
</p>

**Note:** 
* Push logs file to server in compressed form to reduce the data consumption and response time.
```
HyperLog.pushLogs(Context mContext, boolean compress, HyperLogCallback callback);
```
* Logs will be compressed using `GZIP` encoding.
* Logs will be deleted from database after successful push.
* Logs will push to the server in batches. Each batch can have maximum of `5000 logs`.

## Custom Log Message Format
Default Log Message that will store in database.
```
timeStamp + " | " + appVersion + " : " + osVersion + " | " + deviceUUID + " | [" + logLevelName + "]: " + message
```
```
2017-10-05T14:46:36.541Z 1.0 | 0.0.1 : Android-7.1.1 | 62bb1162466c3eed | [INFO]: Log has been pushed
```
This message can easily be customize.
1. Create a new class extending `LogFormat`.
2. Override `getFormattedLogMessage` method.
3. Now return the formatted message.
```
class CustomLogMessageFormat extends LogFormat {

    CustomLog(Context context) {
        super(context);
    }

    public String getFormattedLogMessage(String logLevelName, String message, String timeStamp,
                                         String senderName, String osVersion, String deviceUUID) {
                                         
        //Custom Log Message Format                                
        String formattedMessage = timeStamp + " : " + logLevelName + " : " + deviceUUID + " : " + message;
        
        return formattedMessage;
    }
}

```
Custom Log Message Format example
```
2017-10-05T14:46:36.541Z 1.0 | INFO | 62bb1162466c3eed | Log has been pushed

```

4. Above created class instance then needs to be passed while initializing `HyperLog` or can be set later.
```
HyperLog.initialize(this,new CustomLogMessageFormat(this));
 
OR
 
HyperLog.initialize(this);
HyperLog.setLogFormat(new CustomLogMessageFormat(this));
```
## Additional Methods
* Different types of log.
```
HyperLog.d(TAG,"debug");
HyperLog.i(TAG,"information");
HyperLog.e(TAG,"error");
HyperLog.v(TAG,"verbose");
HyperLog.w(TAG,"warning");
HyperLog.a(TAG,"assert");
HyperLog.exception(TAG,"exception",throwable);
```

* To check whether any device logs are available.
```
HyperLog.hasPendingDeviceLogs();
```

* Get the count of stored device logs.
```
HyperLog.logCount();
```

* Developer can pass additional headers along with API call while pushing logs to server.
```
HashMap<String, String> additionalHeaders = new HashMap<>();
additionalHeaders.put("Authorization","Token");
HyperLog.pushLogs(this, additionalHeaders, false, hyperLogCallback);
```

* By default, seven days older logs will get delete automatically from the database. You can change the expiry period of logs by defining `expiryTimeInSeconds`.
```
HyperLog.initialize(@NonNull Context context, int expiryTimeInSeconds);
```
* Developers can also get the device log as a list of `DeviceLog` model or list of `String` .By default, fetched logs will delete from the database. Developers can override to change the default functionality.
```
HyperLog.getDeviceLogs(boolean deleteLogs);
HyperLog.getDeviceLogsInFile(Context mContext, boolean deleteLogs);
```
* By default, every get calls return data from first batch if there are one or more batch.
* If there are more than one batches then developer can gets the specific batch data by providing batch number.
```
HyperLog.getDeviceLogs(boolean deleteLogs, int batchNo);
```
* Get the number of batches using `HyperLog.getDeviceLogBatchCount`.
* Developer can manually delete the logs using `HyperLog.deleteLogs`.

## Contribute
Please use the [issues tracker](https://github.com/hypertrack/hyperlog-android/issues) to raise bug reports and feature requests. We'd love to see your pull requests, so send them in!

## About HyperTrack
Developers use [HyperTrack](https://www.hypertrack.com) to build location features, not infrastructure. We reduce the complexity of building and operating location features to a few APIs that just work.
Check it out. [Sign up](https://dashboard.hypertrack.com/signup/) and start building! Join our [Slack community](http://slack.hypertrack.io) for instant responses. You can also email us at help@hypertrack.io

## License

```
MIT License

Copyright (c) 2016 HyperTrack

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
