## Android Marshmallow 新permission module対応について
### 前提
外部ストレージにデータを記録することでアプリの再インストール時により正確にインストール計測するため、
```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
上記のpermissionが必要となります。
Android Marshmallowの端末から`sendConversion`の実行する前に、上記のpermissionの確認をしなければなりません。

### 対応例
```java
private final int PERMISSION_REQUEST_CODE_FOX = 1; // 任意のユニックなリクエストコード

@TargetApi(Build.VERSION_CODES.M)
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);

  // Mの端末で実行する時に、インストール計測前にcheckSelfPermissionでpermissionsをチェックする
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED
        && checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED) {
      // 既にpermissionを得られた場合
      sendFoxConversion();
    } else {
      // ユーザーにpermissionを要求する
      requestPermissions(
          new String[]{Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
          PERMISSION_REQUEST_CODE_FOX);
    }
  } else {
    // M以外の端末はinstall時にpermissionを得たため、直ちにinstall計測を行う。
    sendFoxConversion();
  }
}

@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
  super.onRequestPermissionsResult(requestCode, permissions, grantResults);
  switch (requestCode) {
    case PERMISSION_REQUEST_CODE_FOX:
      if (grantResults.length > 1
          && grantResults[0] == PackageManager.PERMISSION_GRANTED
          && grantResults[1] == PackageManager.PERMISSION_GRANTED) {
        // ユーザーの承認を得てinstll計測を行う
        sendFoxConversion();
      }
      break;
    default:
      break;
  }
}


```
