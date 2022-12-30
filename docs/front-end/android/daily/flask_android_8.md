# Mac 10.14  编译Android 8.1源码及刷入nexus 6p

## 1. 环境准备 

[官网](https://source.android.com/setup/build/initializing) 描述得已经相当清楚了 ，这里稍微总结一下： 

###  1.1 创建区分大小写的磁盘映像

mac系统默认是不区分大小写的，所以我们需要创建一个区分大小写的文件系统 

``` shell
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 60g ~/android.dmg 
```

这将创建一个.dmg.sparseimage文件，该文件在装载后可用作具有 Android 开发所需格式的驱动盘。 

>   按官网所说完成编译至少需要 25GB 空间，相信我，其实至少需要60G。当然，空间大小后面还可以通过以下命令修改 
>
>   ```shell
>   hdiutil resize -size <new-size-you-want>g ~/android.dmg.sparseimage
>   ```

为了方便，我们还可以往环境变量配置文件(`~/.bash_profile`--bash，`~/.zshrc`--zsh)添加辅助函数

* 装载函数

  ```shell
  # mount the android file image
  mountAndroid() { hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android; 
  ```

* 卸载函数

  ```shell
  # unmount the android file image
  umountAndroid() { hdiutil detach /Volumes/android; }
  ```
### 1.2 安装所需的软件

* JDK

  各种 Android 版本使用的 Java 版本不一样，请参阅相关[要求](https://source.android.com/setup/build/requirements.html#latest-version)

  我这里是编译Android8.1.0 ，所以使用java1.8

* Xcode 命令行工具

  ```shell
  xcode-select --install
  ```

* MacPorts

  从[macports.org](http://www.macports.org/install.php) 下载安装,请确保 `/opt/local/bin` 在路径中显示在 `/usr/bin` **前面**。否则，请将以下内容添加到环境变量配置文件(`~/.bash_profile`--bash，`~/.zshrc`--zsh)中：

  ```shell
  export PATH=/opt/local/bin:$PATH
  ```

  > 通过 MacPorts 获取 Make、Git 、 GPG、BISON 软件包
  >
  > ```shell
  > POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg bison
  > ```

#### 设置文件描述符数量上限

在 Mac OS 中，可同时打开的文件描述符的默认数量上限太低，在高度并行的编译流程中，可能会超出此上限。要提高此上限，请将下列行添加到环境变量配置文件(`~/.bash_profile`--bash，`~/.zshrc`--zsh)中：

```shell
# set the number of open files to be 1024
ulimit -S -n 1024
```

##2. 下载源代码 

Android 源代码树位于由 Google 托管的 Git 代码库中。为了在 Android 环境中更轻松地使用 Git，Google开发了[Repo](https://source.android.com/setup/develop/repo.html)。

### 2.1 安装 Repo

1. 确保主目录下有一个 `bin/` 目录，并且该目录包含在路径中：

   ```shell
   mkdir ~/bin
   PATH=~/bin:$PATH
   ```

2. 下载 Repo 工具，并确保它可执行

   ```shell
   curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
   chmod a+x ~/bin/repo
   ```
### 2.2 repo 初始化
#### 进入我们一开始创建的文件系统,创建一个空目录

```shell
➜  ~ mountAndroid
/dev/disk3          	GUID_partition_scheme
/dev/disk3s1        	EFI
/dev/disk3s2        	Apple_HFS                      	/Volumes/android
➜  ~ cd /Volumes/android
➜  ~ mkdir aosp
➜  ~ cd aosp
```

#### 指定需要checkout对应的[源代码标记和编译版本](https://source.android.com/setup/start/build-numbers.html#source-code-tags-and-builds)

```shell
repo init -u https://android.googlesource.com/platform/manifest -b android-8.1.0_r50
```

初始化成功后，目录中应包含一个 `.repo` 目录。

### 2.3 下载

这时候就可以开始漫长的下载过程了

```shell
repo sync
```
> 同步操作顺利的话将需要 1 个小时或更长时间完成,

### 2.4 下载驱动

从[官网](https://developers.google.com/android/drivers)下载对应机型驱动即可，下载完成后解压，依次执行里面的sh文件，如：

```shell
$ ./assets/extract-huawei-angler.sh

The license for this software will now be displayed.
You must agree to this license before using this software.

Press Enter to view the licensels
```

执行完毕，驱动文件会释放到vendor目录。

## 3. 编译

### 3.1 清理

```shell
make clobber
```

### 3.2 设置环境

```shell
source build/envsetup.sh
```

### 3.3 选择目标

```shell
➜  ~ lunch

You're building on Darwin

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_mips-eng
     4. aosp_mips64-eng
     5. aosp_x86-eng
     6. aosp_x86_64-eng
     7. aosp_deb-userdebug
     8. aosp_flo-userdebug
     9. full_fugu-userdebug
     10. aosp_fugu-userdebug
     11. mini_emulator_arm64-userdebug
     12. m_e_arm-userdebug
     13. mini_emulator_mips-userdebug
     14. mini_emulator_x86-userdebug
     15. mini_emulator_x86_64-userdebug
     16. aosp_flounder-userdebug
     17. aosp_angler-userdebug
     18. aosp_bullhead-userdebug
     19. aosp_hammerhead-userdebug
     20. aosp_hammerhead_fp-userdebug
     21. aosp_shamu-userdebug
     22. aosp_bullhead-userdebug
     23. aosp_angler-userdebug
```

因为我要编译nexus6p，这里选择23，其他设备可以参考[选择设备编译系统](https://source.android.com/source/running.html#selecting-device-build)

### 3.4 编译代码

```shell
make -j8
```

`-jN` 表示编译并行任务数，这个示电脑情况而定，一般取cpu数的1～2倍就可以

### 3.5 编译遇到问题
#### 找不到对应的macOS.sdk

* 报错日志

  ```shell
  [44/44] bootstrap out/soong/.minibootstrap/build.ninja.in 
  [4/4] out/soong/.bootstrap/bin/minibp out/soong/.bootstrap/build.ninja 
  [860/861] glob vendor///Android.bp 
  [54/54] out/soong/.bootstrap/bin/soong_build out/soong/build.ninja 
  FAILED: out/soong/build.ninja 
  out/soong/.bootstrap/bin/soong_build -t -b out/soong -d out/soong/build.ninja.d -o out/soong/build.ninja Android.bp 
  internal error: Could not find a supported mac sdk: ["10.10" "10.11" "10.12"] 
  ninja: build stopped: subcommand failed. 
  17:53:06 soong failed with: exit status 1
  
  ```

* 解决方法

  修改/build/soong/cc/config/x86_darwin_host.go文件，添加10.14支持，如下

  ```c++
  darwinSupportedSdkVersions = []string{
      "10.10",
      "10.11",
      "10.12",
      "10.14", // 添加mac sdk 10.14
  }
  ```

#### 遇到bison 错误

* 报错日志

  ```shell
  FAILED: out/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/darwin_x86_64/gen/yacc/external/selinux/checkpolicy/policy_parse.c out/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/darwin_x86_64/gen/yacc/external/selinux/checkpolicy/policy_parse.h 
  BISON_PKGDATADIR=external/bison/data prebuilts/misc/darwin-x86/bison/bison -d --defines=out/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/darwin_x86_64/gen/yacc/external/selinux/checkpolicy/policy_parse.h -o out/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/darwin_x86_64/gen/yacc/external/selinux/checkpolicy/policy_parse.c external/selinux/checkpolicy/policy_parse.y 
  [ 0% 309/87784] //external/libcxx:libc++_static header-abi-dumper src/random.cpp [arm] 
  ninja: build stopped: subcommand failed. 
  18:05:05 ninja failed with: exit status 1 
  
  ```

* 解决办法

  ```shell
  cd /Volumes/AOSP/external/bison
  git cherry-pick c0c852bd6fe462b148475476d9124fd740eba160
  mm 
  # 用新生成的bison替换掉原bison文件
  cp /Volumes/AOSP/out/host/darwin-x86/bin/bison /Volumes/AOSP/prebuilts/misc/darwin-x86/bison/ 
  # 重新编译
  make -j8
  ```

## 4. 刷机

经过漫长的等待和反复折腾后，终于到了最后一步---刷机。

``` shell
# 手机连接电脑情况下
adb reboot bootloader 
# 进入源码编译输出的目录 
fastboot flashing unlock 
fastboot flashall -w 

```

**Done**

## 5. 参考链接

[https://source.android.com/setup/build/requirements](https://source.android.com/setup/build/requirements)
[https://www.jianshu.com/p/1c3d47b2031f](https://www.jianshu.com/p/1c3d47b2031f)