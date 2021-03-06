---
layout: post
title:  "ai BehaviorTree介绍（1）"
categories: ai
tags: ai
---

* content
{:toc}

## 简介

AI开发是游戏开发中非常重要模块，如何开发出一个性能高效、可玩性强、扩展性强AI是
游戏框架设计的关键，今天起将主要介绍如何使用行为树来开发一个高效有趣且扩展性强
的AI系统，第一篇主要介绍行为树的基本概念。


## LuaState

想要更好的理解脚本执行流程，就必须先了解LuaState相关初始化，源码里简称L，lua
状态机是lua框架的核心，无论是lua的编译和lua的执行都离不开这个lua状态机，这里
使用UniLua作为源码参考，方便大家理解，初始化lua状态的方式很简单：

```
ILuaState Lua = LuaAPI.NewState();
Lua.L_OpenLibs();
```

需要注意的是在初始化完lua状态机后，需要初始化lua标准库，也就是上面的第二行代码
否则lua标准库的函数是无法调用的。

## 执行流程

lua脚本执行流程是在LuaState初始化完成的基础上进行的，核心可以分为两部分，即lua
的编译和lua的执行。

### 编译

编译lua源码主要调用这些函数进行源码编译，可以跟踪源码这些方法进行查看：

```
ThreadStatus L_DoFile(string filename);
public ThreadStatus L_LoadFileX(string filename, string mode);
ThreadStatus Load(ILoadInfo loadinfo, string name, string mode);
private ThreadStatus D_PCall<T>(PFuncDelegate<T> func, ref T ud, int oldTopIndex, int errFunc);
private ThreadStatus D_RawRunProtected<T>(PFuncDelegate<T> func, ref T ud);
private static void F_Load(ref LoadParameter param);
```

上面函数除了最后一个是进入实际编译解析阶段，其他函数都是初始化数据阶段，这里关心
F_Load函数的实现是什么？

```
private static void F_Load(ref LoadParameter param) {
	var L = param.L;
	LuaProto proto;
	var c = param.LoadInfo.PeekByte();
	if(c == LuaConf.LUA_SIGNATURE[0]){
		L.CheckMode( param.Mode, "binary" );
		proto = Undump.LoadBinary(L, param.LoadInfo, param.Name);
	} else {
		L.CheckMode( param.Mode, "text" );
		proto = Parser.Parse(L, param.LoadInfo, param.Name);
	}
	var cl = new LuaLClosureValue( proto );
	Utl.Assert(cl.Upvals.Length == cl.Proto.Upvalues.Count);
	L.Top.V.SetClLValue(cl);
	L.IncrTop();
}
```

该函数关键是调用Parser.Parse函数进入编译，具体代码实现可以去Parser的内部实现，
函数返回LuaProto，然后生成一个LuaLClosureValue对象cl，最终的结果是将函数编译
后的cl类型Push到栈顶（L.Top.V.SetClLValue(cl)），lua脚本的编译阶段到此全部结
束。

### 执行

lua执行是以lua脚本编译后的输出作为输入交给虚拟机进行运行的过程，执行主要涉及
下面这些脚本。

```
ThreadStatus PCall(int numArgs, int numResults, int errFunc);
private ThreadStatus D_PCall<T>(PFuncDelegate<T> func, ref T ud, int oldTopIndex, int errFunc);
private void D_Call(StkId func, int nResults, bool allowYield);
private void V_Execute();
```

上面函数只有最后一个是进入执行虚拟机指令阶段，虚拟机执行字节码指令的过程是很简单
的，这里简单给出函数原型：

```
private void V_Execute() {
	while(true){
		Instruction i
		switch(i){
			case OpCpde.OP_MOVE:
			//....
			case OpCpde.OP_LOADK:
			//....
			default:
			break
		}
	}
}
```

可以看出该函数使用一个无限循环加switch方式执行字节码，知道字节码执行完毕，lua脚本执行
也结束了。

## 参考资料

[UniLua源码](https://github.com/xebecnan/UniLua)

[lua源码](https://github.com/lua/lua)



	






