# Flutter for web

## 问题

1. commend line

   1. flutter pub upgrade 

      更新依赖至web
   
   2. flutter pub get
   
      更新依赖
   
   3. pub get
   
      下载web依赖
   
   4. webdev serve
   
      编译
   
2. Because tbflutterlite depends on image_picker_saver from path which requires the Flutter SDK, version solving failed. flutter web sdk依赖直接使用git会产生上述错误。

   方案：将sdk源码下载到本地，直接引用本地库

   ```yaml
   dependency_overrides:
     flutter_web:
       path: /Users/diwenchao/FlutterProject/flutter_web/packages/flutter_web
     #    git:
     #      url: https://github.com/flutter/flutter_web
     #      path: packages/flutter_web
     flutter_web_ui:
       path: /Users/diwenchao/FlutterProject/flutter_web/packages/flutter_web_ui
     #    git:
     #      url: https://github.com/flutter/flutter_web
     #      path: packages/flutter_web_ui
     flutter_web_test:
       path: /Users/diwenchao/FlutterProject/flutter_web/packages/flutter_web_test
   #    git:
   #      url: https://github.com/flutter/flutter_web
   #      path: packages/flutter_web_test
   ```

3. 主工程迁移图片资源，将图片资源打包至`web/assets`目录下，可以直接使用。但是package工程迁移时这种方法行不通

   方案：目前还没有办法直接把图片资源放入本package中使用，一个比较geek的方法是将所有的资源文件统一放入主工程，其他package直接引用就行。

4. 切换web分支后，所有的依赖会转为web相关，需要pub upgrade



## 分页面迁移问题

1. 问题：大量插件相关的代码

   方案：代码删除

2. 问题：原来的Atoute基于sourcegen，不能使用

   方案：route方案重做，自行管理页面跳转，webRouter类

3. 错误很难定位到，直接白屏

4. 无用的import不会优化掉，还是会进行编译

5. 问题：build_web_compilers:entrypoint on web/main.dart: Skipping compiling web|web/main.dart with ddc because some of its transitive libraries have sdk dependencies that not supported on this platform:

   web|lib/tbflutterlite/app/first_screen.dart

   https://github.com/dart-lang/build/blob/master/docs/faq.md#how-can-i-resolve-skipped-compiling-warnings

   方案：一般是import的库没有使用web的库，而使用了dart的库，改过来就OK

6. 问题：网络库迁移

   方案：dio是一个package项目，不是插件，所以是可以迁移的，将dio和cookie_jar的源码down到本地，修改依赖至web，整个基础逻辑与内部版master一致
   
7. 问题：webdev serve -v 可以输出因为依赖问题被跳过的类，但是构建过一次之后再次使用此命令不会输出

   方案：删除生成的.dart_tool和build目录

8. 因为web平台对dart:io的一些特性不支持，所以很导致一些类不能转化为js，导致白屏

9. dart:io中File不能使用，file相关逻辑全部删除

10. 网络库contentType相关全部无效，download功能删除

11. web版本太老了，大量的方法不存在

12. 为了迁移网络库，copy出dart:io的子集作为依赖

13. io.dart中引用dart:isolate，dart:isolate需要迁移

14. 问题：SocketException: Failed to create server socket (OS Error: Address already in use, errno = 48), address = 127.0.0.1, port = 8080

    方案：sudo lsof -i:8080；   kill XXXX
    
15. dio向web迁移正式宣告失败，迁移过程如下：

    1. dio网络库基于flutter sdk，迁移至flutter_web需要修改依赖，因此down源码到本地迁移至flutter_web sdk，同时涉及的库还有cookie_jar，同样的处理办法
    2. web编译时会直接略过所有不支持的库，由于dio对dart:io库的强依赖，会导致web编译时直接跳过所有import dart:io的文件，浏览器白屏
    3. 为了解决上述问题，将dart:io库涉及http、socket、io、file等部分的源码copy出来作为dio依赖，所有import dart:io的都改为依赖源码依赖，编译问题解决
    4. 运行时网络请求会有NoSuchMethodError报错，经排查发现Platdorm和RawSocket等类有大量的external方法，但是未实现，部分方法可以自行给值实现，可以解决问题
    5. 直到我看到了RawSocket.startConnect方法，我放弃了。。

16. 网络库迁移方案调整，在保证NetManager上层接口不变的情况下，替换dio为package:http，但是遗留一个问题有待商榷，now直播的同学说package:http在手机上使用不了

17. 问题：web上访问数据出现跨域的问题，数据和图片直接出错

    方案：charles代理，rewrite response，加上允许跨域的header, 更改host解决图片问题
    
18. Nginx配置和命令行

    启动：sudo nginx

    重启:  sudo nginx -s reload

    配置：vim /usr/local/etc/nginx/nginx.conf

19. 其他MAC连接本地环境

    1. 本地启动nginx环境

    2. Chrome下载SwitchyOmega插件，连接代理

    ![b33dbcaa06c2eb334a0421b47](/Users/diwenchao/icecreamco.github.io/assets/images/b33dbcaa06c2eb334a0421b47.png)

    3. Chrome连接diwenchao.baidu.com

20. 其他手机连接本地环境

    1. 连接本地环境代理
    2. 浏览器打开diwenchao.baidu.com

​    

