title: cmake_dependent_option 详解
date: 2019-12-12 16:01:00
tags: [技术, C++, cmake]
categories: 技术

------

`cmake_dependent_option` 可以说是一个理解起来十分头疼的一条 cmake 命令了。我们先看看 [cmake 的文档是怎么说的：](https://cmake.org/cmake/help/v3.5/module/CMakeDependentOption.html)

> Macro to provide an option dependent on other options.
>
> This macro presents an option to the user only if a set of other conditions are true. When the option is not presented a default value is used, but any value set by the user is preserved for when the option is presented again. Example invocation:
>
> ```cmake
> CMAKE_DEPENDENT_OPTION(USE_FOO "Use Foo" ON
>                        "USE_BAR;NOT USE_ZOT" OFF)
> ```
>
> If USE_BAR is true and USE_ZOT is false, this provides an option called USE_FOO that defaults to ON. Otherwise, it sets USE_FOO to OFF. If the status of USE_BAR or USE_ZOT ever changes, any value for the USE_FOO option is saved so that when the option is re-enabled it retains its old value.

这段英文绕来绕去，越看越头疼，我就不按原文翻译了，只说一下我的理解。我先把这个命令的具体形式再重复一下：

```cmake
cmake_dependent_option(OPT_VAR "OPT_VAR_DES" DEF_VAL_1 "CONDITION_EXP" DEF_VAR_2)
```

这个命令带有 5 个参数:

1. OPT_VAR
2. OPT_VAR_DES
3. DEF_VAL_1
4. CONDITION_EXP
5. DEF_VAR_2

`cmake_dependent_option`的目的是要定义一个`option`，这个 `option` 就是 `OPT_VAR`，这个 `option` 的描述是 `OPT_VAR_DES`，这个 `option` 的默认值不是常量，而是 `DEF_VAL_1` 或者 `DEF_VAL_2`（`DEF_VAL_1`和`DEF_VAL_2`不同，但只能是`ON`或者`OFF`之一），具体是哪一个，取决于表达式 `CONDITION_EXP`，如果表达式 `CONDITION_EXP` 为 `true`，则默认值是 `DEF_VAL_1`，如果表达式 `CONDITION_EXP` 为 `false`，则默认值是 `DEF_VAL_2`。

如果我们把这个命令自己来实现一把的话，可能要需要下面这一大段代码：

```cmake
if(CONDITION_EXP)
	set(OPT_VAR_DEF DEF_VAL_1)
else()
	set(OPT_VAR_DEF DEF_VAL_2)
endif()

option(OPT_VAR "OPT_VAR_DES" OPT_VAR_DEF)
```

使用 `cmake_dependent_option`的时候需要导入`CMakeDependentOption`这个模块，也就是需要包含下面的语句：

```cmake
include(CMakeDependentOption)
```

否则会出现类似 `Unknown CMake command "cmake_dependent_option".`的报错。

