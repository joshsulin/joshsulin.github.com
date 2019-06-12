---
layout: post
title:  "nodejs启动过程和require函数源码分析"
date:   2019-06-12 20:19:20 +0800
categories: Nodejs
tags: Nodejs
---

当我们在命令行中敲下

```nodejs
node a.js
```

之后, 入口文件在 `src/node_main.cc` 中, 主要任务为将参数传入 node::Start 函数:

```nodejs
// src/node_main.cc
// ...

int main(int argc, char *argv[]) {
  setvbuf(stderr, NULL, _IOLBF, 1024);
  return node::Start(argc, argv);
}
```

node::Start 函数定义于 src/node.cc 中, 它进行了必要的初始化工作后, 会调用 StartNodeInstance:

```nodejs
// src/node.cc
// ...

int Start(int argc, char** argv) {
    // ...
    NodeInstanceData instance_data(NodeInstanceType::MAIN,
                                   uv_default_loop(),
                                   argc,
                                   const_cast<const char**>(argv),
                                   exec_argc,
                                   exec_argv,
                                   use_debug_agent);
    StartNodeInstance(&instance_data);
}
```

而在 StartNodeInstance 函数中，又调用了 LoadEnvironment 函数，其中的 ExecuteString(env, MainSource(env), script_name); 步骤，便执行了第一个 JavaScript 文件代码：

```nodejs
// src/node.cc
// ...
void LoadEnvironment(Environment* env) { 
  // ...
  Local<Value> f_value = ExecuteString(env, MainSource(env), script_name);
  // ...
}

static void StartNodeInstance(void* arg) {
  // ...
  {
      Environment::AsyncCallbackScope callback_scope(env);
      LoadEnvironment(env);
  }
  // ...
}

// src/node_javascript.cc
// ...

Local<String> MainSource(Environment* env) {
  return String::NewFromUtf8(
      env->isolate(),
      reinterpret_cast<const char*>(internal_bootstrap_node_native),
      NewStringType::kNormal,
      sizeof(internal_bootstrap_node_native)).ToLocalChecked();
}
```

其中 internal_bootstrap_node_native，即为 lib/internal/bootstrap_node.js 中的代码。（注：很多以前的 Node.js 源码分析文章中，所写的第一个执行的 JavaScript 文件代码为 src/node.js ，但这个文件在 Node.js v5.10 中已被移除，并被拆解为了 lib/internal/bootstrap_node.js 等其他 lib/internal 下的文件，PR 为： https://github.com/nodejs/node/pull/5103 ）

### 正文

作为第一段被执行的 JavaScript 代码，它的历史使命免不了就是进行一些环境和全局变量的初始化工作。代码的整体结构很简单，所有的初始化逻辑都被封装在了 startup 函数中：

```nodejs
// lib/internal/bootstrap_node.js
'use strict';

(function(process) {
  function startup() {
    ......
		const Module = NativeModule.require('module');
		......
		Module.runMain();
		......
  }
  // ...
  startup();
});
```

并且 lib/internal/bootstrap_node.js 会定义NativeModule, 可以看到NativeModule的定义

```nodejs
function NativeModule(id) {
	this.filename = `${id}.js`;
	this.id = id;
	this.exports = {};
	this.loaded = false;
}
```

从入口函数startup中可以看到

```nodejs
const Module = NativeModule.require('module');也就是说会去加载module.js模块。
```

在 lib/internal/bootstrap_node.js 文件的 require 函数中

```nodejs
NativeModule.require = function(id) {
if (id === 'native_module') {
  return NativeModule;
}
/..../
const nativeModule = new NativeModule(id);新建NativeModule对象

nativeModule.cache();
nativeModule.compile();  //主要步骤

return nativeModule.exports;
```

在 compile 中

```nodejs
NativeModule.prototype.compile = function() {
    var source = NativeModule.getSource(this.id);
    source = NativeModule.wrap(source);

    var fn = runInThisContext(source, {
      filename: this.filename,
      lineOffset: 0
    });
    fn(this.exports, NativeModule.require, this, this.filename);

    this.loaded = true;
};
```

wrap会进行文件的包裹

```nodejs
NativeModule.wrap = function(script) {
return NativeModule.wrapper[0] + script + NativeModule.wrapper[1];
};

NativeModule.wrapper = [
'(function (exports, require, module, __filename, __dirname) { ',
'\n});'
];
```

接下来就会执行Module.runMain函数，从而进入module.js中，所以module.js中开始的require是NativeModule.require，并不矛盾。

```nodejs
// bootstrap main module.
Module.runMain = function() {
  // Load the main module--the command line argument.
  Module._load(process.argv[1], null, true);
  // Handle any nextTicks added in the first tick of the program
  process._tickCallback();
};
```

大家看到这个函数, 知道 node 后面的参数, 被Module.-load()方法接收和处理.

```nodejs
// lib/module.js
Module._load = function(request, parent, isMain) {
		......
		var filename = Module._resolveFilename(request, parent);
		......
		var module = new Module(filename, parent);
		......
		module.load(filename);
		......
		return module.exports;		
}
```

接下来, 我们看看 module.load -> Module.prototype.load

```nodejs
Module.prototype.load = function(filename) {
	......
  Module._extensions[extension](this, filename);
	......
};
```

接下来, 我们再来看看Module.-extensions相关的方法

```nodejs
// Native extension for .js
Module._extensions['.js'] = function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  module._compile(internalModule.stripBOM(content), filename);
};


// Native extension for .json
Module._extensions['.json'] = function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  try {
    module.exports = JSON.parse(internalModule.stripBOM(content));
  } catch (err) {
    err.message = filename + ': ' + err.message;
    throw err;
  }
};

//Native extension for .node
Module._extensions['.node'] = function(module, filename) {
  return process.dlopen(module, path._makeLong(filename));
};
```

针对于js后缀的文件, 最终会调用 Module.-extensions['.js'] 方法, 最后会调用 module.-compile() 方法

```nodejs
// lib/module.js
// ...

Module.prototype._compile = function(content, filename) {
  // ...

  var compiledWrapper = runInThisContext(wrapper, {
    filename: filename,
    lineOffset: 0,
    displayErrors: true
  });

  // ...
  const args = [this.exports, require, this, filename, dirname];

  const result = compiledWrapper.apply(this.exports, args);
  // ...
};
```

最后一步， Node.js 会使用 vm.runInThisContext 执行这个拼接完毕的字符串，取得一个 JavaScript 函数，最后带着对应的对象参数执行它们，并将赋值在 module.exports 上的对象返回;

大家细细品味这个执行流程, 一定会有收获的.
