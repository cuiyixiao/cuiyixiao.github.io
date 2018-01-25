# teeworlds for my Graduation desig
## /src
### /src/base 
- detect.h 运行环境等自定义
- math.h   简单数学计算
- vmath.h  vecotr之间的数学计算
- system.h system.c 包括debug相关日志生成，底层网络的封装，空间的分配等
###### /src/base/tl
- alorithm.h 算法组例如排序，查找等基础算法
- allocator.h 空间配置器，分配空间
- array.h 数组容器
- base.h assert的封装
- range.h 一种类似队列的容器
- sorted_array.h array的封装
- string.h string容器，一个类似string的类

### /src/mastersrv
- mastesrv.h mastersrv.cpp  服务器端的数据发送封装，以及检查服务器，以及服务器的创建。

### /src/versionsrv
- versionsrv.h versionsrv   版本相关服务器

### /src/engine
- key.h 键盘宏定义
- kernel.h 接口注册头文件
- storage.h 存储文件相关管理的头文件
###### /src/engine/client
- keynames.h 服务器端监听的按件

###### /src/engine/shared
- protocal.h 网络消息的协议。
- huffman.h huffman.cpp 一棵哈夫曼树
- memheap.h memheap.cpp  成块儿的分配内存(chunk_size)
- message.h 基类，两个虚函数，打包与解包。
- kernel.cpp  接口注册,头文件在上层目录
- linereader.h linereader.cpp io成行的读取数据的封装
- storage.cpp 存储文件的相关管理，头文件在上层目录

###### /src/engine/external
- pnglite图片链接相关外部库
- wavpack 音频压缩相关外部库
- zlib 数据压缩相关外部库






# server涉及模块而
- base/system.h
- engine/config.h
- engine/console.h
- engine/map.h
- engine/masterserver.h
- engine/server.h
- engine/storage.h
- engine/shared/compression.h
- engine/shared/config.h
- engine/shared/demo.h
- engine/shared/mapchecker.h
- engine/shared/network.h
- engine/shared/packer.h
- engine/shared/protocol.h
- engine/shared/snapshot.h
- mastersrv/mastersrv.h
- register.h
- detect.h
