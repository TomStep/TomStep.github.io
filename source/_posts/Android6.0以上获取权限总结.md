---
title: Android6.0以上获取权限总结
date: 2016-11-08 15:08:55
tags: Android
---

## Android6.0中的权限分类：

1、普通权限（一般权限）
>   这些权限对于用户隐私和设备操作不会造成太多危险，系统会自动授予权限。
>   （普通权限类型在文章最后）
>    **  用法：和之前声明权限方法相同，直接在Manifest中声明就好了。** 


2、运行时权限（危险权限）
>   对于低版本来说，在Mainfest中声明了运行时权限，系统会自动授予。
>   *但是在6.0以上版本时，系统会明确的让用户来决定是否授予这些权限。*


运行时权限分为9组：（每组只要有一个权限申请成功了，就默认整组权限都可以使用）
<!-- more -->  
```java
        //联系人权限组
        group:android.permission-group.CONTACTS 
            permission:android.permission.WRITE_CONTACTS
            permission:android.permission.GET_ACCOUNTS    
            permission:android.permission.READ_CONTACTS
        //电话权限组
        group:android.permission-group.PHONE   
            permission:android.permission.READ_CALL_LOG
            permission:android.permission.READ_PHONE_STATE 
            permission:android.permission.CALL_PHONE
            permission:android.permission.WRITE_CALL_LOG
            permission:android.permission.USE_SIP
            permission:android.permission.PROCESS_OUTGOING_CALLS
            permission:com.android.voicemail.permission.ADD_VOICEMAIL
        //日历权限组
        group:android.permission-group.CALENDAR 
            permission:android.permission.READ_CALENDAR
            permission:android.permission.WRITE_CALENDAR
        //拍照权限组
        group:android.permission-group.CAMERA  
            permission:android.permission.CAMERA
        //传感器权限组
        group:android.permission-group.SENSORS  
            permission:android.permission.BODY_SENSORS
        //位置权限组
        group:android.permission-group.LOCATION  
            permission:android.permission.ACCESS_FINE_LOCATION
            permission:android.permission.ACCESS_COARSE_LOCATION
        //储存卡权限组
        group:android.permission-group.STORAGE  
            permission:android.permission.READ_EXTERNAL_STORAGE
            permission:android.permission.WRITE_EXTERNAL_STORAGE
        //麦克风权限组
        group:android.permission-group.MICROPHONE 
            permission:android.permission.RECORD_AUDIO
        //sms卡权限组
        group:android.permission-group.SMS       
            permission:android.permission.READ_SMS
            permission:android.permission.RECEIVE_WAP_PUSH
            permission:android.permission.RECEIVE_MMS
            permission:android.permission.RECEIVE_SMS
            permission:android.permission.SEND_SMS
            permission:android.permission.READ_CELL_BROADCASTS
```


## Android6.0获取权限方法：
>在6.0以上的系统中，允许用户关闭危险权限，同时也需要用户允许使用权限。
我们获取的方式：
    检查是否授权-->请求授权-->回调查看状态做相应操作

### 判断是否授权：
```java
    //核对授权情况：
    //  1、当系统版本<23时，会有：
    //      ActivityCompat.checkSelfPermission（）== PERMISSION_GRANTED
    //  2、当系统版本>=23时，会判断是否授权
    //  3、当targetSdkVersion < 23。运行在6.0版本上时会出现：
    //      ActivityCompat.checkSelfPermission（）== PERMISSION_GRANTED
          
    ActivityCompat.checkSelfPermission(activity, requestPermission);
```

```java
    //参考网上做法：
    public boolean selfPermissionGranted(String permission) {
        // For Android < Android M, self permissions are always granted.
        boolean result = true;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (targetSdkVersion >= Build.VERSION_CODES.M) {
                // targetSdkVersion >= Android M, we can
                // use Context#checkSelfPermission
                result = context.checkSelfPermission(permission)
                        == PackageManager.PERMISSION_GRANTED;
            } else {
                // targetSdkVersion < Android M, we have to use PermissionChecker
                result = PermissionChecker.checkSelfPermission(context, permission)
                        == PermissionChecker.PERMISSION_GRANTED;
            }
        }
        return result;
    }
```

### 请求权限：

```java
    //请求权限
    public static void requestPermission(final Activity activity, final int requestCode, PermissionUtil.PermissionGrant permissionGrant) {
        if (activity == null) {
            return;
        }

        Log.i(TAG, "requestPermission requestCode:" + requestCode);
        if (requestCode < 0 || requestCode >= requestPermissions.length) {
            Log.w(TAG, "requestPermission illegal requestCode:" + requestCode);
            return;
        }

        final String requestPermission = requestPermissions[requestCode];

        //判断权限是否授予
        if(selfPermissionGranted(activity,requestPermission)){
            Log.d(TAG, "ActivityCompat.checkSelfPermission ==== PackageManager.PERMISSION_GRANTED");
            //已获取授权，直接外部回调，做自己想做的事
            permissionGrant.onPermissionGranted(requestCode);
        }else {
            Log.i(TAG, "ActivityCompat.checkSelfPermission != PackageManager.PERMISSION_GRANTED");
            //未获取授权
            //判断shouldShowRequestPermissionRationale：
            if (ActivityCompat.shouldShowRequestPermissionRationale(activity, requestPermission)) {
                Log.i(TAG, "requestPermission shouldShowRequestPermissionRationale");
                shouldShowRationale(activity, requestCode, requestPermission);
            } else {
                Log.d(TAG, "requestCameraPermission else");
                ActivityCompat.requestPermissions(activity, new String[]{requestPermission}, requestCode);
            }
        }
    }

```

**对应逻辑图：**
<img src="http://og5n67ybk.bkt.clouddn.com/android%E6%9D%83%E9%99%90%E7%94%B3%E8%AF%B7%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg">

**主要坑点：**
>当用户选择了“不在询问”拒绝之后，再次点击功能就不会出现系统提示框了。这时我们需要自己在onRequestPermissionsResult()中捕获并且提示用户再次打开权限。

### 重写onRequestPermissionsResult方法：
>通过
ActivityCompat.requestPermissions(activity, new String[]{requestPermission}, requestCode);
最后会调用onRequestPermissionsResult()方法，其中申请的权限和相应的状态都会返回。

```java
 @Override
  public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
      //permissions是申请的权限
      //grantResults是对应权限的申请结果（用来二次判断）
      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
       PermissionUtil.requestPermissionsResult(this,requestCode,permissions,grantResults, new PermissionUtil.PermissionGrant() {
            @Override
            public void onPermissionGranted(int requestCode) {
                Toast.makeText(TakePhotoActivity.this, "回调后做相应的动作", Toast.LENGTH_SHORT).show();
            }
        });
  }
```

```java
    //回调后检查权限状态 
    //  如果权限被用户拒绝了，再次询问用户（openSettingActivity()方法内）
    //  如果允许了，做自己想做事(permissionGrant.onPermissionGranted(requestCode);
)
    public static void requestPermissionsResult(final Activity activity, final int requestCode, @NonNull String[] permissions,@NonNull int[] grantResults, PermissionGrant permissionGrant) {

        if (activity == null) {
            return;
        }
        Log.d(TAG, "requestPermissionsResult requestCode:" + requestCode);

        if (requestCode == CODE_MULTI_PERMISSION) {
            requestMultiResult(activity, permissions, grantResults, permissionGrant);
            return;
        }

        if (requestCode < 0 || requestCode >= requestPermissions.length) {
            Log.w(TAG, "requestPermissionsResult illegal requestCode:" + requestCode);
            Toast.makeText(activity, "illegal requestCode:" + requestCode, Toast.LENGTH_SHORT).show();
            return;
        }

        Log.i(TAG, "onRequestPermissionsResult requestCode:" + requestCode + ",permissions:" + permissions.toString()
                + ",grantResults:" + grantResults.toString() + ",length:" + grantResults.length);

        if (grantResults.length == 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            Log.i(TAG, "onRequestPermissionsResult PERMISSION_GRANTED");
            //TODO success, do something, can use callback
            permissionGrant.onPermissionGranted(requestCode);

        } else {
            //TODO hint user this permission function
            Log.i(TAG, "onRequestPermissionsResult PERMISSION NOT GRANTED");
            //TODO
            String[] permissionsHint = logs;
            openSettingActivity(activity,permissionsHint[requestCode]);
        }

    }
```

**主要坑点：**
>这里只讲了在Activity中的使用情况。在fragment中运行时权限要做特殊处理：
1、在Fragment中申请权限，去掉ActivityCompat.requestPermissions(activity, new String[]{requestPermission}, requestCode)。使用Fragment自带的requestPermissions()方法。*最后都会回调一次在activity中的onRequestPermissionsResult方法*
2、如果在Fragment中嵌套Fragment，在子Fragment中使用requestPermissions方法，onRequestPermissionsResult也不会回调回来。
*建议使用：getParentFragment().requestPermissions方法*可以回调到父类的onRequestPermissionsResult中。
加入下面代码可以传递到子类：


```java
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
      List<Fragment> fragments = getChildFragmentManager().getFragments();
      if (fragments != null) {
          for (Fragment fragment : fragments) {
              if (fragment != null) {
                  fragment.onRequestPermissionsResult(requestCode,permissions,grantResults);
              }
          }
      }
  }
```


## 最后：

1、上述代码已封装在一个工具类中。[（有兴趣的可以在我的Github上查看）][1]
2、当前也有比较好的权限框架：[RxPermissions 基于RxJava的运行时权限检测框架][2]




[1]:https://github.com/TomStep/PermissionsUtil
[2]:https://github.com/tbruyelle/RxPermissions


