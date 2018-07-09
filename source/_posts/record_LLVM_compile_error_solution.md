---
title: 记录一次 LLVM 编译错误的解决
copyright: true
date: 2018-07-09 22:21:37
tags: [LLVM]
comments:
---

> undefined reference to `llvm::orc::AsynchronousSymbolQuery::handleFullyReady()'

使用官方教程中的命令编译
```bash
clang++ -g -rdynamic toy.cpp -L/usr/local/lib -lLLVMOrcJIT `llvm-config --cxxflags --ldflags --system-libs --libs core mcjit native` -O3 -o toy
```
会得到如下错误：
```bash
warning: unknown warning option '-Wno-maybe-uninitialized'; did you mean '-Wno-uninitialized'? [-Wunknown-warning-option]
1 warning generated.
/tmp/toy-e5b39c.o: In function `llvm::orc::RTDyldObjectLinkingLayer::ConcreteLinkedObject<std::shared_ptr<llvm::RuntimeDyld::MemoryManager> >::finalize()':
/usr/local/include/llvm/ExecutionEngine/Orc/RTDyldObjectLinkingLayer.h:191: undefined reference to `llvm::orc::JITSymbolResolverAdapter::JITSymbolResolverAdapter(llvm::orc::ExecutionSession&, llvm::orc::SymbolResolver&, llvm::orc::MaterializationResponsibility*)'
/tmp/toy-e5b39c.o: In function `llvm::orc::JITSymbolResolverAdapter::~JITSymbolResolverAdapter()':
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:23: undefined reference to `vtable for llvm::orc::JITSymbolResolverAdapter'
/tmp/toy-e5b39c.o: In function `llvm::orc::ExecutionSessionBase::materializeOnCurrentThread(llvm::orc::VSO&, std::unique_ptr<llvm::orc::MaterializationUnit, std::default_delete<llvm::orc::MaterializationUnit> >)':
/usr/local/include/llvm/ExecutionEngine/Orc/Core.h:188: undefined reference to `llvm::orc::MaterializationResponsibility::MaterializationResponsibility(llvm::orc::VSO&, std::map<llvm::orc::SymbolStringPtr, llvm::JITSymbolFlags, std::less<llvm::orc::SymbolStringPtr>, std::allocator<std::pair<llvm::orc::SymbolStringPtr const, llvm::JITSymbolFlags> > >)'
/usr/local/include/llvm/ExecutionEngine/Orc/Core.h:188: undefined reference to `llvm::orc::MaterializationResponsibility::~MaterializationResponsibility()'
/tmp/toy-e5b39c.o: In function `std::set<llvm::orc::SymbolStringPtr, std::less<std::set>, std::allocator<std::set> > llvm::orc::lookupWithLegacyFn<llvm::orc::KaleidoscopeJIT::KaleidoscopeJIT()::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)#1}>(llvm::orc::ExecutionSession&, llvm::orc::AsynchronousSymbolQuery&, std::allocator<std::set> const&, llvm::orc::KaleidoscopeJIT::KaleidoscopeJIT()::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)#1})':
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:90: undefined reference to `llvm::orc::ExecutionSessionBase::failQuery(llvm::orc::AsynchronousSymbolQuery&, llvm::Error)'
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:82: undefined reference to `llvm::orc::AsynchronousSymbolQuery::resolve(llvm::orc::SymbolStringPtr const&, llvm::JITEvaluatedSymbol)'
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:83: undefined reference to `llvm::orc::AsynchronousSymbolQuery::notifySymbolReady()'
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:86: undefined reference to `llvm::orc::ExecutionSessionBase::failQuery(llvm::orc::AsynchronousSymbolQuery&, llvm::Error)'
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:97: undefined reference to `llvm::orc::AsynchronousSymbolQuery::handleFullyResolved()'
/usr/local/include/llvm/ExecutionEngine/Orc/Legacy.h:100: undefined reference to `llvm::orc::AsynchronousSymbolQuery::handleFullyReady()'
/tmp/toy-e5b39c.o:(.data.rel.ro._ZTVN4llvm3orc22LegacyLookupFnResolverIZNS0_15KaleidoscopeJITC1EvEUlRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEE_EE[_ZTVN4llvm3orc22LegacyLookupFnResolverIZNS0_15KaleidoscopeJITC1EvEUlRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEE_EE]+0x30): undefined reference to `llvm::orc::SymbolResolver::anchor()'
clang-7: error: linker command failed with exit code 1 (use -v to see invocation)
```

定位到最下方的一次函数调用`handleFullyReady`，在 `/usr/local/lib` 目录下使用`grep handleFullyReady . -r -n`搜索，发现该函数位于 `LibLLVMOrcJIT.so` 文件中。
```bash
Binary file ./libLLVMOrcJIT.so.7svn matches
```

使用新的命令`clang++ -g -rdynamic toy.cpp -lLLVMOrcJIT `llvm-config --cxxflags --ldflags --system-libs --libs core mcjit native` -O3 -o toy`编译。问题解决。
