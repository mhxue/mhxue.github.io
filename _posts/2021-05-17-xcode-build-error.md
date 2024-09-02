---
layout: post
title: Xcode 12.5 build error
date: 2021-05-17 09:41:00-0000
description:
tags: ci&cd
categories: ci&cd
---

# 程序员的自我修养：Xcode 12.5构建失败

编译是我们程序员几乎每天都会去执行的一个任务，部分程序员对这一环节没有特别清晰的感知，主要原因是在现代计算机编程中编译通常会做为构建的一个具体步骤来执行，而一个能够有效应对我们日常开发需要的构建配置是现代IDE所提供的一个基本功能。大多数程序员已经习惯了通过IDE提供的入口添加源代码文件到自己的项目中，修改这些源文件，触发一个构建过程，得到预期的构建产物。纵然整个构建过程很复杂，但是得到一个成功的构建产物又显得那么理所当然，以至于我们来不及更多的去品味构建过程的种种细节。这种现状使得我们在一些预料之外的场景出现构建错误时显得很是那么的错愕和茫然。

## 问题

最近有幸拿到了公司新采购的MackBook Pro一台，带着愉悦的心情安装各种生产力工具期望可以尽快的让指尖在新的妙控键盘上飞舞。出于iOS程序员的基本素质要求，升级Mac OS Big Sur到最新的11.3.1，安装最新的Xcode 14.5，证书同步，配置SSH公钥，拉取项目仓库，哈，一切都很妙不可言，最后来一次全新的构建吧~~，bang!!!

![image](https://user-images.githubusercontent.com/25997299/118444924-5b16f600-b720-11eb-9228-60a725437ba8.png)

看到这个你会做何感想？对于第一次遇到类似问题和有一定经验的程序员来说，他们心中会存在同样的一个观点：一定是哪里出了问题。没错，肯定是哪里出了问题，对于这一点你毋庸置疑。关键是接下来我该怎么做，如何实施有效的措施来解决这一问题以及如何获得这些有效的措施。

## 我这里是好的

在仔细排除犯下低级错误的可能后，我们有很大的理由去相信是外部环境出现了问题，可懝程度从大到小依次排序可能是Xcode，Mac OS，硬件。从经验上来看可能就是Xcode的问题。到此我们似乎可以给出问题原因和解决方案了，新的Xcode版本存在一些未知问题，然后去通知你的团队成员现在不要升级。但是如果想要给出一些更具说服力的原因的话就需要更进一步的探索了。

## 获取帮助

获取帮助的途径是多种多样的。Google? 求助于更资深的同事？去查看最新的Xcode release note? 这些都可能帮你高效的解决问题，不过相对于从正面迎击这个问题来说，我把它们归类为一些侧面解决方式。并且请相信我，如果你能够鼓起勇气从正面迎击这个问题，你或许会有更多意想不到的收获，我这么说并不是否定采取上述侧面手段，相反在正面解决的过程中，你可能随时采取适当的侧面手段来强化自己的能力，两者并不排斥，它们是相辅相成的。

## 定位问题

或许单次构建出现了很多问题，不要被太多的错误和冗长的输出吓到，它们通常有一些内在的关联性，从第一个问题开始着手是一项最佳实践。
![image](https://user-images.githubusercontent.com/25997299/118606801-66ceef00-b7ea-11eb-95ae-245dd8eb5b79.png)
展开第一个问题，我们能提取出以下信息，这是执行一个编译任务，待编译的一个C++类型的源文件，我们的C++源文件引用了一些标准库的头文件，最终在编译标准库头文件时在一个全局命名空间下的`clock_t`缺失定义，导致编译出错。这听起来是一件很不可思议的事情，标准库头文件怎么可能会出错呢？发挥你的想象力去想，脑袋痛么？获取你是个天才一下就能想到问题的答案，不过我们需要一种更科学可具体操作的步骤来得出正确的答案。我们可以从Xcode的构建日志中提取出编译当前出错文件时使用的完整命令。
![image](https://user-images.githubusercontent.com/25997299/118607923-b82bae00-b7eb-11eb-9dad-1c9f9742794b.png)
点击这里获取完整的编译指令。但是这个指令太长以至于我压根没办法把它截取在一张图片上。但是这不妨碍你在自己的终端上重复执行这个命令
![image](https://user-images.githubusercontent.com/25997299/118608520-5c155980-b7ec-11eb-94df-589dddf743ff.png)
至此我们其实对问题做了一定程度的简化。现在我们可以把注意力放在这个编译命令本身的执行上。

## clang

上述很长一段命令其实仅仅是一条clang的编译命令。clang现在已经是Mac OS平台上开发语言的标准前端编译器。我们知道编译一个源文件的输入是源文件本身，输出是一个目标文件，去这条指令里面找找看有没有相关线索？这是一个不错的注意。
经过一番简单的分析后，我决定继续简化这个问题
![image](https://user-images.githubusercontent.com/25997299/118609512-67b55000-b7ed-11eb-90c7-560995a7bd71.png)
令我意想不到的是这条命令是可能正确执行的！！！
完全的指令是有问题的，这个简陋的反而可能正确执行，这让人很困惑，甚至迷失了方向。我能做什么？对比是多出来的哪个编译参数出了问题，对，就这么干。直接对比是要出人命的，你需要对原始命令的显示格式做些优化来更易于阅读。不过还好不是很难，如下我仅仅是做了一些正则替换后就得到了一个不错的展示效果`%s/ -/ -\\\r/g`
![image](https://user-images.githubusercontent.com/25997299/118610173-10fc4600-b7ee-11eb-884b-3a0f511e96fc.png)
接下来怎么做？相信你已经有了答案。逐项确认对编译运行的影响。
我定位到的是这一行。但是这还是不清楚为什么呢？
![image](https://user-images.githubusercontent.com/25997299/118611106-127a3e00-b7ef-11eb-947b-435ba5f3e039.png)
完全的指令是有问题的，这个简陋的反而可能正确执行，这让人很困惑，甚至迷失了方向。我能做什么？对比是多出来的哪个编译参数出了问题，对，就这么干。直接对比是要出人命的，你需要对原始命令的显示格式做些优化来更易于阅读。不过还好不是很难，如下我仅仅是做了一些正则替换后就得到了一个不错的展示效果`%s/ -/ -\\\r/g`
![image](https://user-images.githubusercontent.com/25997299/118612759-abf61f80-b7f0-11eb-83e9-ac7d2cb328f2.png)

## search path

编译器去哪里找头文件？search path。这让我们联想到在上一个步骤中起作用的livabutil。啊是了，头文件冲突了。去libavutil下一年，果不其然
![image](https://user-images.githubusercontent.com/25997299/118613352-422a4580-b7f1-11eb-931d-85c94700167e.png)
到此似乎已经找到原因了，但是还有一个疑点，为什么 之前没问题呢，对比发现

## iphonesimulate 14.5 framework c++ headers

```bash
 clang++ -E -x c++ - -v  -target x86_64-apple-ios8.0-simulator -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator14.5.sdk
```

![image](https://user-images.githubusercontent.com/25997299/118615049-ecef3380-b7f2-11eb-9770-040ae8eb9114.png)

## podspec

1. target search path vs -I
2. subspec

```ruby
# Build settings
  s.pod_target_xcconfig = {
    'HEADER_SEARCH_PATHS' => '${PODS_ROOT}/Headers/Public ${PODS_ROOT}/Headers/Private ${PODS_ROOT}/Headers/Public/IJKMediaPlayer ${PODS_ROOT}/Headers/Private/IJKMediaPlayer ${PODS_ROOT}/IJKMediaPlayer/ijkmedia/** ${PODS_ROOT}/IJKMediaPlayer/ios/Classes/** ${PODS_ROOT}/IJKMediaPlayer/ios/preserves/ffmpeg/include/** ${PODS_ROOT}/../../../ijkmedia/** ${PODS_ROOT}/../../../ios/Classes/** ${PODS_ROOT}/../../../ios/preserves/ffmpeg/include/**',
    'OTHER_LDFLAGS' => '-read_only_relocs suppress',
    'ENABLE_BITCODE' => false,
    'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64'
  }
```

```ruby
   s.subspec 'Core' do  |sb|
    sb.source_files = [
      'ios/Classes/**/*.{h,m,c}',
      'ijkmedia/ijkplayer/*.{h,c}',
      'ijkmedia/ijkplayer/{ijkavformat,ijkavutil,pipeline}/*.{h,c}',
      'ijkmedia/ijksdl/*.{h,c}',
      'ijkmedia/ijksdl/{dummy,ffmpeg,gles2}/**/*.{h,c,m}',
    ]
    sb.exclude_files = [
      'ijkmedia/ijkplayer/ijkavformat/{ijkioandroidio,ijkmediadatasource}.c',
      'ijkmedia/ijksdl/ijksdl_container.h',
      'ijkmedia/ijksdl/ijksdl_extra_log.{h,c}',
    ]
    sb.preserve_paths = [
      'ios/preserves/ffmpeg/include/**/*.h',
      'ijkmedia/ijkplayer/version.sh'
    ]
    sb.public_header_files = [
      'ios/Classes/IJKMediaPlayer/Public/*.h',
    ]
    sb.private_header_files = [
      'ios/Classes/ijkplayer/**/*.h',
      'ios/Classes/ijksdl/**/*.h',
      'ijkmedia/{ijkplayer,ijksdl}/**/*.h',
    ]
    sb.compiler_flags = [
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavcodec',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavfilter',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavformat',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libswresample',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libswscale',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/openssl',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil/arm64',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil/armv7',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil/armv7s',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil/i386',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libavutil/x86_64',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg/arm64',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg/armv7',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg/armv7s',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg/i386',
      '-IIJKMediaPlayer/ios/preserves/ffmpeg/include/libffmpeg/x86_64',
    ]
    sb.dependency 'IJKMediaPlayer/CPP'
  end
  s.subspec 'CPP' do |sb|
    sb.source_files = [
      'ijkmedia/ijkplayer/{ijkavformat,ijkavutil,pipeline}/*.cpp',
    ]
  end
```

## 总结

![image](https://user-images.githubusercontent.com/25997299/118618479-3d1bc500-b7f6-11eb-8057-12b71b7645cf.png)
生活又恢复了平静
能够阻挡你前进脚步的只有你自己。
不要惧怕，事情没你想的那么难。
