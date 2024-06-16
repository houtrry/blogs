使用方式： addr2line.exe路径 参数 so路径 要查询的地址

示例：   
崩溃信息如下：
```
2024-06-15 20:38:28.617 15876-15896/com.houtrry.openglsample A/libc: Fatal signal 5 (SIGTRAP), code 1 in tid 15896 (GLThread 777)
2024-06-15 20:38:28.686 15916-15916/? A/DEBUG: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
2024-06-15 20:38:28.686 15916-15916/? A/DEBUG: Build fingerprint: 'Xiaomi/oxygen/oxygen:7.1.1/NMF26F/V11.0.2.0.NDDCNXM:user/release-keys'
2024-06-15 20:38:28.686 15916-15916/? A/DEBUG: Revision: '0'
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG: ABI: 'arm64'
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG: pid: 15876, tid: 15896, name: GLThread 777  >>> com.houtrry.openglsample <<<
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG: signal 5 (SIGTRAP), code 1 (TRAP_BRKPT), fault addr 0x7f73066e8c
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG:     x0   000000000000002a  x1   0000007f726b4c90  x2   0000000000000005  x3   0000000000000003
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG:     x4   7472617473207761  x5   0000000000800000  x6   0000007f92e6e000  x7   0000000000000000
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG:     x8   0000000000000000  x9   0000007f726b4ce8  x10  0000007f726b4e20  x11  000000000000001a
2024-06-15 20:38:28.687 15916-15916/? A/DEBUG:     x12  0000000000000018  x13  0000000000000000  x14  0000000000000000  x15  0024c48cb44d3d22
2024-06-15 20:38:28.688 15916-15916/? A/DEBUG:     x16  0000007f90760a48  x17  0000007f8fd213dc  x18  0000000000004022  x19  0000007f819fbc00
2024-06-15 20:38:28.688 15916-15916/? A/DEBUG:     x20  0000007f8eaabcd0  x21  0000007f819fbc00  x22  0000007f726b56fc  x23  0000007f739da008
2024-06-15 20:38:28.688 15916-15916/? A/DEBUG:     x24  000000000000000c  x25  ccf02861c03e2452  x26  0000007f819fbc98  x27  ccf02861c03e2452
2024-06-15 20:38:28.688 15916-15916/? A/DEBUG:     x28  0000000000000003  x29  0000007f726b5420  x30  0000007f73066e8c
2024-06-15 20:38:28.688 15916-15916/? A/DEBUG:     sp   0000007f726b5390  pc   0000007f73066e8c  pstate 0000000060000000
2024-06-15 20:38:28.691 15916-15916/? A/DEBUG: backtrace:
2024-06-15 20:38:28.691 15916-15916/? A/DEBUG:     #00 pc 0000000000015e8c  /data/app/com.houtrry.openglsample-1/lib/arm64/liblopengl.so (Java_com_houtrry_lopengl_view_MapView_ndkReadAssertManagers+352)
2024-06-15 20:38:28.691 15916-15916/? A/DEBUG:     #01 pc 0000000000b42d18  /data/app/com.houtrry.openglsample-1/oat/arm64/base.odex (offset 0xa5e000)

```
崩溃信息显示崩溃在#00 pc 0000000000015e8c  /data/app/com.houtrry.openglsample-1/lib/arm64/liblopengl.so (Java_com_houtrry_lopengl_view_MapView_ndkReadAssertManagers+352)
但是尾部352并不是出问题的行号，352行也不是ndkReadAssertManagers方法  
因此，使用addr2line查询崩溃具体的行号信息。  
使用示例如下  
```
D:\develop\tools\androidsdk\ndk\20.1.5948944\toolchains\aarch64-linux-android-4.9\prebuilt\windows-x86_64\bin>  .\aarch64-linux-android-addr2line.exe -C -f -e D:\develop\code\OpenglSample\lopengl\build\intermediates\cmake\debug\obj\arm64-v8a\liblopengl.so 0000000000015e8c
Java_com_houtrry_lopengl_view_MapView_ndkReadAssertManagers
D:/develop/code/OpenglSample/lopengl/src/main/cpp/lopengl.cpp:486
```
查询到的具体信息显示，崩溃在lopengl.cpp的486行。



注意：
1. window与linux使用的addr2line目录不同
2. 手机架构不同，使用的so也不同，需要先确定手机的使用架构
3. 要查询的地址是崩溃信息里带的
