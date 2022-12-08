#### param



**获取CameraCharacteristics**

```java

CameraManager cameraManager = (CameraManager) getSystemService(Context.CAMERA_SERVICE);

        try {

        	//获取相机id列表

            for (String id : cameraManager.getCameraIdList()) {

            	//根据id 获取相机参数

                CameraCharacteristics characteristics = cameraManager.getCameraCharacteristics(id);



            }

        } catch (CameraAccessException e) {

            e.printStackTrace();

        }

```



**获取相机基本属性**

```java

int requestFlag[] = characteristics.get(CameraCharacteristics.REQUEST_AVAILABLE_CAPABILITIES);

```

requestFlag的返回值对应CameraMetadata.REQUEST_AVAILABLE_CAPABILITIES_XXX



参数参考:https://blog.csdn.net/HUAJUN998/article/details/80367610?utm_source=blogxgwz3



**获取摄像头支持的尺寸和格式**

```java

StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);

```


#### 1、慢动作
通过提高帧率实现
CameraConstrainedHighSpeedCaptureSession

#### 2、hdr
通过setRepeatingBurst连续录制多张图片 每张图片的曝光度不同

参考
API：https://blog.csdn.net/zhangbijun1230/article/details/80556903

