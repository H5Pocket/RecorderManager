# RecorderManager





因为在项目中经常需要使用音视频录制，所以写了一个公共库RecorderManager，欢迎大家使用。

最新0.2.28版本更新：
1.优化视频录制结果获取方式
2.优化代码

0.2.27版本更新： 
1.视频录制界面RecordVideoRequestOption新增RecorderOption和hideFlipCameraButton配置
2.优化代码

0.2.26版本更新： 
1.项目迁移至AndroidX， 引入Kotlin

0.2.25版本更新： 
1.优化权限自动申请，可自动调起视频录制界面
2.规范图片资源命名

## 一.效果展示
仿微信界面视频录制
![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/2/12/168df9c38e4d10d3?w=1080&h=2280&f=jpeg&s=1065098)
![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/1/29/1689916e86ee15a3?w=1080&h=2280&f=jpeg&s=819744)
2.音频录制界面比较简单，就不放图了
## 二.引用
1.Add it in your root build.gradle at the end of repositories
```
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```
2.Add the dependency

```
dependencies {
	        implementation 'com.github.MingYueChunQiu:RecorderManager:0.2.28'
	}
```
## 三.使用
### 1.音频录制
采用默认配置录制
```
mRecorderManager.recordAudio(mFilePath);
```
自定义配置参数录制

```
mRecorderManager.recordAudio(new RecorderOption.Builder()
                    .setAudioSource(MediaRecorder.AudioSource.MIC)
                    .setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
                    .setAudioEncoder(MediaRecorder.AudioEncoder.AAC)
                    .setAudioSamplingRate(44100)
                    .setBitRate(96000)
                    .setFilePath(path)
                    .build());
```
### 2.视频录制
#### (1).可以直接使用RecordVideoActivity，实现了仿微信风格的录制界面
从0.2.18开始改为类似

```
RecorderManagerFactory.getRecordVideoRequest().startRecordVideo(MainActivity.this, 0);
```
RecorderManagerFactory中可以拿到RequestRecordVideoPageable，在RequestRecordVideoPageable接口中

```
/**
     * 以默认配置打开录制视频界面
     *
     * @param activity    Activity
     * @param requestCode 请求码
     */
    void startRecordVideo(@NonNull FragmentActivity activity, int requestCode);

    /**
     * 以默认配置打开录制视频界面
     *
     * @param fragment    Fragment
     * @param requestCode 请求码
     */
    void startRecordVideo(@NonNull Fragment fragment, int requestCode);

    /**
     * 打开录制视频界面
     *
     * @param activity    Activity
     * @param requestCode 请求码
     * @param option      视频录制请求配置信息类
     */
    void startRecordVideo(@NonNull FragmentActivity activity, int requestCode, @Nullable RecordVideoRequestOption option);

    /**
     * 打开录制视频界面
     *
     * @param fragment    Fragment
     * @param requestCode 请求码
     * @param option      视频录制请求配置信息类
     */
    void startRecordVideo(@NonNull Fragment fragment, int requestCode, @Nullable RecordVideoRequestOption option);
```
RecordVideoRequestOption可配置最大时长（秒）和文件保存路径

```
public class RecordVideoRequestOption implements Parcelable {

         private int maxDuration;//最大录制时长
        private String filePath;//文件保存路径
        private RecorderOption recorderOption;//录制参数信息类（在这里配置的文件路径会覆盖filePath，录制视频时设置该属性里的MaxDuration是无效的）
        private boolean hideFlipCameraButton;//隐藏返回翻转摄像头按钮
}
```

RecordVideoActivity里已经配置好了默认参数，可以直接使用，然后在onActivityResult里拿到视频路径的返回值
返回值为RecordVideoResultInfo
```
@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == Activity.RESULT_OK && requestCode == 0) {
        RecordVideoResultInfo info = data.getParcelableExtra(EXTRA_RECORD_VIDEO_RESULT_INFO);
	  
	//从0.2.28版本开始可以使用下面这种方式，更安全更灵活，兼容性强
	RecordVideoResultInfo info = RecorderManagerFactory.getRecordVideoResult(data);
	
	if (info != null) {
                Log.e("MainActivity", "onActivityResult: " + " "
                        + info.getDuration() + " " + info.getFilePath());
            }
        }
    }
```

#### (2).如果想要界面一些控件的样式，可以继承RecordVideoActivity，里面提供了几个protected方法，可以拿到界面的一些控件

```
/**
     * 获取计时控件
     *
     * @return 返回计时AppCompatTextView
     */
    protected AppCompatTextView getTimingView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getTimingView();
    }

    /**
     * 获取圆形进度按钮
     *
     * @return 返回进度CircleProgressButton
     */
    protected CircleProgressButton getCircleProgressButton() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getCircleProgressButton();
    }

	/**
     * 获取翻转摄像头控件
     *
     * @return 返回翻转摄像头AppCompatImageView
     */
    public AppCompatImageView getFlipCameraView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getFlipCameraView();
    }

    /**
     * 获取播放控件
     *
     * @return 返回播放AppCompatImageView
     */
    protected AppCompatImageView getPlayView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getPlayView();
    }

    /**
     * 获取取消控件
     *
     * @return 返回取消AppCompatImageView
     */
    protected AppCompatImageView getCancelView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getCancelView();
    }

    /**
     * 获取确认控件
     *
     * @return 返回确认AppCompatImageView
     */
    protected AppCompatImageView getConfirmView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getConfirmView();
    }

    /**
     * 获取返回控件
     *
     * @return 返回返回AppCompatImageView
     */
    protected AppCompatImageView getBackView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getBackView();
    }
```
想要替换图标资源的话，提供下列名称图片

```
rm_record_video_flip_camera.png
rm_record_video_cancel.png
rm_record_video_confirm.png
rm_record_video_play.png
rm_record_video_pull_down.png
```

#### (3).同时提供了对应的RecordVideoFragment，实现与RecordVideoActivity同样的功能，实际RecordVideoActivity就是包裹了一个RecordVideoFragment
1.创建RecordVideoFragment

```
/**
     * 获取录制视频Fragment实例（使用默认配置项）
     *
     * @param filePath 存储文件路径
     * @return 返回RecordVideoFragment
     */
    public static RecordVideoFragment newInstance(@Nullable String filePath) {
        return newInstance(filePath, 30);
    }

    /**
     * 获取录制视频Fragment实例（使用默认配置项）
     *
     * @param filePath    存储文件路径
     * @param maxDuration 最大时长（秒数）
     * @return 返回RecordVideoFragment
     */
    public static RecordVideoFragment newInstance(@Nullable String filePath, int maxDuration) {
        return newInstance(new RecordVideoOption.Builder()
                .setRecorderOption(new RecorderOption.Builder().buildDefaultVideoBean(filePath))
                .setMaxDuration(maxDuration)
                .build());
    }

    /**
     * 获取录制视频Fragment实例
     *
     * @param option 录制配置信息对象
     * @return 返回RecordVideoFragment
     */
    public static RecordVideoFragment newInstance(@Nullable RecordVideoOption option) {
        RecordVideoFragment fragment = new RecordVideoFragment();
        fragment.mOption = option;
        if (fragment.mOption == null) {
            fragment.mOption = new RecordVideoOption();
        }
        if (fragment.mOption.getRecorderOption() == null) {
            fragment.mOption.setRecorderOption(new RecorderOption.Builder()
                    .buildDefaultVideoBean(
                            FilePathUtils.getSaveFilePath(fragment.getContext())
                    ));
        }
        if (TextUtils.isEmpty(fragment.mOption.getRecorderOption().getFilePath())) {
            fragment.mOption.getRecorderOption().setFilePath(
                    FilePathUtils.getSaveFilePath(fragment.getContext()));
        }
        return fragment;
    }
```
2.然后添加RecordVideoFragment到自己想要的地方就可以了
3.可以设置OnRecordVideoListener，拿到各个事件的回调

```
public class RecordVideoOption：

	private RecorderOption option;//录制配置信息
        private int maxDuration;//最大录制时间（秒数）
        private boolean hideFlipCameraButton;//隐藏返回翻转摄像头按钮
        private OnRecordVideoListener listener;//录制视频监听器

	/**
     * 录制视频监听器
     */
    public interface OnRecordVideoListener {
		/**
         * 当完成一次录制时回调
         *
         * @param filePath      视频文件路径
         * @param videoDuration 视频时长（毫秒）
         */
        void onCompleteRecordVideo(String filePath, int videoDuration);

        /**
         * 当点击确认录制结果按钮时回调
         *
         * @param filePath      视频文件路径
         * @param videoDuration 视频时长（毫秒）
         */
        void onClickConfirm(String filePath, int videoDuration);

  /**
         * 当点击取消按钮时回调
         *
         * @param filePath      视频文件路径
         * @param videoDuration 视频时长（毫秒）
         */
        void onClickCancel(String filePath, int videoDuration);

        /**
         * 当点击返回按钮时回调
         */
        void onClickBack();
    }
```
4.RecorderOption是具体的录制参数配置类
```
	private int audioSource;//音频源
        private int videoSource;//视频源
        private int outputFormat;//输出格式
        private int audioEncoder;//音频编码格式
        private int videoEncoder;//视频编码格式
        private int audioSamplingRate;//音频采样频率（一般44100）
        private int bitRate;//视频编码比特率
        private int frameRate;//视频帧率
        private int videoWidth, videoHeight;//视频宽高
        private int maxDuration;//最大时长
        private long maxFileSize;//文件最大大小
        private String filePath;//文件存储路径
        private int orientationHint;//视频录制角度方向
```

#### (4).如果想自定义自己的界面，可以直接使用RecorderManagerable类
1.通过RecorderManagerFactory获取RecorderManagerable
```
public class RecorderManagerFactory {

/**
     * 创建录制管理类实例（使用默认录制类）
     *
     * @return 返回录制管理类实例
     */
    @NonNull
    public static RecorderManagerable newInstance() {
        return newInstance(new RecorderHelper());
    }

    /**
     * 创建录制管理类实例（使用默认录制类）
     *
     * @param intercept 录制管理器拦截器
     * @return 返回录制管理类实例
     */
    @NonNull
    public static RecorderManagerable newInstance(RecorderManagerInterceptable intercept) {
        return newInstance(new RecorderHelper(), intercept);
    }

    /**
     * 创建录制管理类实例
     *
     * @param recorderable 实际录制类
     * @return 返回录制管理类实例
     */
    @NonNull
    public static RecorderManagerable newInstance(Recorderable recorderable) {
        return newInstance(recorderable, null);
    }

    /**
     * 创建录制管理类实例
     *
     * @param recorderable 实际录制类
     * @param intercept    录制管理器拦截器
     * @return 返回录制管理类实例
     */
    @NonNull
    public static RecorderManagerable newInstance(Recorderable recorderable, RecorderManagerInterceptable intercept) {
        return new RecorderManager(recorderable, intercept);
    }

    @NonNull
    public static RequestRecordVideoPageable getRecordVideoRequest() {
        return new RecordVideoPageRequest();
    }
    
    @Nullable
    public static RecordVideoResultInfo getRecordVideoResult(@Nullable Intent data) {
        RecordVideoResultInfo info = null;
        if (data != null) {
            info = data.getParcelableExtra(EXTRA_RECORD_VIDEO_RESULT_INFO);
        }
        return info;
    }

}
```
它们返回的都是RecorderManagerable 接口类型，RecorderManager 是默认的实现类，RecorderManager 内持有一个真正进行操作的Recorderable。

```
public interface RecorderManagerable extends Recorderable {

/**
     * 设置录制对象
     *
     * @param recorderable 录制对象实例
     */
    void setRecorderable(Recorderable recorderable);

    /**
     * 获取录制对象
     *
     * @return 返回录制对象实例
     */
    Recorderable getRecorderable();

    /**
     * 初始化相机对象
     *
     * @param holder Surface持有者
     * @return 返回初始化好的相机对象
     */
    Camera initCamera(SurfaceHolder holder);

    /**
     * 初始化相机对象
     *
     * @param cameraType 指定的摄像头类型
     * @param holder     Surface持有者
     * @return 返回初始化好的相机对象
     */
    Camera initCamera(Constants.CameraType cameraType, SurfaceHolder holder);

    /**
     * 翻转摄像头
     *
     * @param holder Surface持有者
     * @return 返回翻转并初始化好的相机对象
     */
    Camera flipCamera(SurfaceHolder holder);

    /**
     * 翻转到指定类型摄像头
     *
     * @param cameraType 摄像头类型
     * @param holder     Surface持有者
     * @return 返回翻转并初始化好的相机对象
     */
    Camera flipCamera(Constants.CameraType cameraType, SurfaceHolder holder);

    /**
     * 获取当前摄像头类型
     *
     * @return 返回摄像头类型
     */
    Constants.CameraType getCameraType();

    /**
     * 释放相机资源
     */
    void releaseCamera();
}
```
RecorderManagerIntercept实现RecorderManagerInterceptable接口，用户可以直接继承RecorderManagerIntercept，它里面所有方法都是空实现，可以自己改写需要的方法

```
public interface RecorderManagerInterceptable extends RecorderManagerable, CameraInterceptable {
}
```
Recorderable是一个接口类型，由实现Recorderable的子类来进行录制操作，默认提供的是RecorderHelper，RecorderHelper实现了Recorderable。

```
public interface Recorderable {

/**
     * 录制音频
     *
     * @param path 文件存储路径
     * @return 返回是否成功开启录制，成功返回true，否则返回false
     */
    boolean recordAudio(String path);

    /**
     * 录制音频
     *
     * @param option 存储录制信息的对象
     * @return 返回是否成功开启录制，成功返回true，否则返回false
     */
    boolean recordAudio(RecorderOption option);

    /**
     * 录制视频
     *
     * @param camera  相机
     * @param surface 表面视图
     * @param path    文件存储路径
     * @return 返回是否成功开启录制，成功返回true，否则返回false
     */
    boolean recordVideo(Camera camera, Surface surface, String path);

    /**
     * 录制视频
     *
     * @param camera  相机
     * @param surface 表面视图
     * @param option  存储录制信息的对象
     * @return 返回是否成功开启视频录制，成功返回true，否则返回false
     */
    boolean recordVideo(Camera camera, Surface surface, RecorderOption option);

    /**
     * 释放资源
     */
    void release();

    /**
     * 获取录制器
     *
     * @return 返回实例对象
     */
    MediaRecorder getMediaRecorder();

    /**
     * 获取配置信息对象
     *
     * @return 返回实例对象
     */
    RecorderOption getRecorderOption();
}
```
2.拿到后创建相机对象

```
		if (mCamera == null) {
            mCamera = mManager.initCamera(mCameraType, svVideoRef.get().getHolder());
            mCameraType = mManager.getCameraType();
        }
```
3.录制

```
isRecording = mManager.recordVideo(mCamera, svVideoRef.get().getHolder().getSurface(), mOption.getRecorderOption());
```
4.释放

```
	    mManager.release();
            mManager = null;
            mCamera = null;
```
## 四.总结
目前来说，大体流程就是这样，更详细的信息请到Github上查看， 后期将添加闪光灯等更多功能，敬请关注，github地址为 https://github.com/MingYueChunQiu/RecorderManager ，码云地址为 https://gitee.com/MingYueChunQiu/RecorderManager ，如果它能对你有所帮助，请帮忙点个star，有什么建议或意见欢迎反馈。
