#在cocos2d-x中使用LUA

##说明
1. 这个文章只是工作中用到的简单记录，没有总结、没有升华、也没有深挖，只是工作用到的简单记录，可能会对刚接触的有帮助。

2. 这个文章，是基于cocos2d-x-3.8.1。

##基础
1. 对LUA有基本了解，LUA的学习，个人推荐：“[LUA程序设计](http://product.china-pub.com/40562)“这本书。
2. 对cocos2d-x有基本了解。
3. 项目中会频繁用闭包、和一些语法糖，比如“:”，这对c/c++转过来的，要有个观念改变（当然C++11开始也支持闭包）。

##开发、调试
1. 如果用Visual Studio开发，不牵涉真机调试，就用[BabeLua](https://babelua.codeplex.com/)作为调试插件，非常方便和强大，就是智能提示差点，

2. 如果用XCode或其它开发，就是用[ZeroBraneStudio](http://studio.zerobrane.com/)做远程调试，这个好处就是可真机调试，windows下也可以用。这个个人试验了下，步骤有些多，弄一半，就放弃了，可能是在VS下用[BabeLua](https://babelua.codeplex.com/)太好用了。

3. Android Studio也是有一个[LUA调试插件（更新的比较快）](https://plugins.jetbrains.com/plugin/5055?pr=)，但是个人试验了下无效果，就放弃了，可能是在VS下用[BabeLua](https://babelua.codeplex.com/)太好用了。

##cocos2d-x对LUA封装
###了解部分
1. 如果项目里引入UA脚本引擎，基本就三步：一、把LUA库文件加到项目中，二、把LUA头文件加到项目中，三、把常用的一些LUA操作再封装一层方便调用。
2. 因为cocos2d-x设计的可以支持任意多个脚本引擎，所以有一个脚本引擎管理的类：ScriptEngineManager，不能同时运行两个脚本引擎，但可以在两个脚本引擎间切换，比如动态从LUA切换到JAVASCRIPT。
3. 如果脚本引擎想要由ScriptEngineManager统一管理，那么就要实现ScriptEngineProtocol。
4. 因为脚本引擎是可以取舍的，又脚本引擎管理是可以支持任意多脚本引擎的设计，为了避免放在一起的臃肿和不灵活，所以每个脚本引擎的实际封装实现都由一个单独的工程来完成，对LUA脚本引擎就是cocos2d_lua_bindings这个项目，它会产生一个libluacocos2d库供引擎使用。
5. 如果不需要新增加脚本（比如新增python），上面的可以忽略不看。

###掌握部分
1. 创建一个支持LUA引擎的新项目可以用如下命令：

2. 因为cocos2d引擎可以支持任意多个脚本引擎，这些脚本引擎都有自己的语法格式和函数调用格式， 怎么让cocos2d引擎的函数被任意脚本引擎调用呢，那就是在cocos2d引擎和脚本引擎之间再增加一个桥接层，这样能保持cocos2d引擎函数的独立性，又能通过脚本引擎对应的桥接层被脚本引擎调用。

3. 桥接层就是按脚本引擎调用c/c++的方法，在脚本引擎和cocos2d-x引擎之间增加函数，让脚本引擎调用cocos2d-x的类或功能函数，在cocos2d_lua_bindings这个项目可以看到，桥接层主要是用到LUA的：用户自定义类型这一基础，这个在“[LUA程序设计](http://product.china-pub.com/40562)“的第28章有详细的教学。

4. 通过cocos2d_lua_bindings这个项目可以了解到，cocos2d-x对LUA脚本引擎的桥接层分为两部分，自动生成部分和手动生成部分。
 
5. 桥接层主要是脚本引擎函数调用cocos2d-x引擎的类或函数的一个转换处理，所以这个转换处理可以归纳起来，做成自动化处理，而不用手动去每个处理，这个详细描述看下面的：用coco2d-x工具自动产生LUA桥接函数。

6. 牵涉到c/c++执行脚本引擎回调或需要特殊处理的就不适合通过能通过自动生成来完成，这就需手动了处理，比如cocos2d-x的触摸事件回调和事件回在cocos2d_lua_bindings这个项目，对回调相关的又封装了一个类：ScriptHandlerMgr，在实际项目中需要处理脚本引擎和coco2d-x引擎关联的回调，基本不用这个，因为有些绕，可以看：[C/C++执行LUA回调函数]部分，这个比较直接些。

7. cocos2d_lua_bindings这个项目的external项下有几个第三方类库，简单列举下

	[LuaJIT](http://luajit.org/)：采用C语言写的Lua的解释器的代码（描述来自网络）。
	
	[LuaSocket](http://w3.impa.br/~diego/software/luasocket/)：是一个 Lua 网络扩展库，它由两部分组成：一个用C写的核心和一些Lua模块，前者提供TCP和UDP传输层支持，	后者为上层应用提供网络处理的功能接口（描述来自网络）。
	
	[tolua](https://github.com/dabing1022/Blog/blob/master/tolua/tolua%2B%2B%E4%B8%AD%E6%96%87%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%5B%E5%AE%8C%E6%95%B4%E7%BF%BB%E8%AF%91%5D.md)：个人理解出现在这里，就是方便使用它封装的一些操作LUA函数。
	
	[XXTEA](http://www.cnblogs.com/linzheng/archive/2011/09/14/2176767.html)：是一个快速安全的加密算法（描述来自网络）。

8. ...... 

##LUA和C/C++的相互操作
~~**LUA调用C/C++函数**~~

~~**C/C++调用LUA函数**~~

这两个看“LUA程序设计“和cocos2d_lua_bindings项目代码，cocos2d_lua_bindings项目有一些类和tolua，对LUA和C/C++函数的相互调用做了些封装，调用更方便。

###C/C++执行LUA回调函数
在项目会有这样的需求：一个脚本函数，需要传递给cocos2d-x引擎的延迟回调函数，在cocos2d-x引擎的延迟回调函数执行时，调用相关的脚本函数，最典型的应用就是cocos2d::CallFuncN对应的LuaCallFunc应用，但是这个实现的稍微有些绕，下面列举一个直接的方法。

cocos2d-x-3.8.1版本ActionFloat类对LUA脚本引擎并没有真正实现对接，这个类对接的核心就是上面描述的需求，在github上有一个[完善代码（点这查看）](https://github.com/dbliu91/cocos2d-x/commit/36e16817854ba92141ae5aa9546a515e4032575f)，这个实现就直接在一个函数里设置C++回调LUA函数，回调时执行LUA回调函数（闭包函数）。

##用coco2d-x工具自动产生LUA桥接函数
1. 自动产生桥接函数，至少牵涉3个项目：一，需要把功能函数或类暴露给脚本引擎调用的项目，比如cocos2d_libs。二，实际执行自动生成的项目，对LUA是tolua这个项目，三，需要使用自动生成桥接函数的项目，对LUA是cocos2d_lua_bindings项目。

2. 用tolua自动生成的桥接函数，会默认输出到cocos2d_lua_bindings项目的auto目录

3. 怎样用tolua自动生成桥接函数，在cocos2d-x-3.8.1/tools/tolua/README.mdown文档里，有详细的说明，这里不再赘述。

4. 自动生成的配置文件，就是key/value模式的文本配置文件，非常简单，安心看个半天就知道怎么配置了， 在cocos2d-x-3.8.1/tools/tolua目录下的*.ini都可以用来参考。

###增加新的LUA桥接函数
1. 如果不牵涉c/c++执行脚本引擎回调函数或做特殊处理，在cocos2d-x-3.8.1/tools/tolua配置一个自动生成的ini就可以了。

2. 如果牵涉到c/c++执行脚本引擎回调函数或做特殊处理，可以参考上面【C/C++执行LUA回调函数】

##加密脚本

非常方便，cocos luacompile基本上就满足简单需求了，[别人写的详细用法（点击查看）](http://blog.csdn.net/qq446569365/article/details/39698417)，另外一个可参考的是，[官网上详细参数描述](http://www.cocos2d-x.org/wiki/Cocos_luacompile)

##对象释放
实际项目中，用到的多是从cocos2d::Ref继承的对象，在LUA脚本部分，可以调用removeFromParent，或release做正确的清理，但如果是没有清理函数暴露给LUA的C++类，一定要把析构函数暴露给lua，因为貌似LUA默认的垃圾回收是free，[别人写的详细处理方法（点击查看）](http://www.cnblogs.com/egmkang/archive/2012/07/01/2572064.html)

##弱引用
“[LUA程序设计](http://product.china-pub.com/40562)“这本书对弱引用的说明，看的有些不知所云，看[别人写的一篇文章](http://www.benmutou.com/archives/1808)，个人理解弱引用就是，当一个变量设置为空，那么引用到这个变量的地方也会被设置为空（可能不准确）。

##自动更新
......

##学习的几个阶段

1. 试错：尝试解决问题
2. 找规律：把解决问题的一般性规律找出来
3. 总结：根据找出的规律，做归纳、分析和总结。
4. 提炼：根据总结和分析，做更深一层的处理，比如抽象

##javascript终究会统治一切，所以新立项，还是选择javascript引擎吧
