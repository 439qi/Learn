---
title: 理解 C++ 模块：第一部分：你好，模块，以及模块单元
translation source: "[[CPP/CPP_20/Clippings/Understanding C++ Modules Part 1 Hello Modules, and Module Units|Understanding C++ Modules Part 1 Hello Modules, and Module Units]]"
---
我先前发布的关于模块的帖文收到了很多关注。我很高兴能够抛砖引玉，但同时我也注意到社区的许多人仍不清楚模块*究竟是什么*。

关于模块有很多内容需要讲，无法一次性讲完，而且即使一次性讲完我怀疑有多少人愿意一次性看完。因此我将逐一讲解，从最高级别的方面开始，然后逐步深入，希望这些帖子能够解释清楚模块*是什么*、*能做什么*、*用于做什么*、*不能做什么*以及如何使用。
## 使用模块

如下是一个使用 C++ 模块的 hello-world 程序：

```cpp
export module speech;

export const char* get_phrase() {
    return "Hello, world!";
}
```

```cpp
// main.cpp
import speech;

import <iostream>;

int main() {
    std::cout << get_phrase() << '\n';
}
```

这是一个相当清晰明了的示例。我们有一个导出了模块 `speech` 的源文件，`main.cpp` 中通过 `import` 导入了 `speech` 模块，然后使用了其中定义的函数 `get_phrase` 。

通过 `import` 导入一个模块使该模块中声明的导出实体对导入的翻译单元变得“可见”，仅此而已。

这就引出了如下问题(稍后会做回答)：为什么是 `export module` 而不是 `module`？模块怎么命名？`import` 的规则是什么？`export` 有什么作用？大家一直讨论的 "分区(partition)" 是什么？

## 模块名

模块通过恰当命名的 *模块名(module-name)* 来标识。以下是 *模块名* 的语法：

```lex
module-name:
    [module-name-qualifier] identifier ;

module-name-qualifier:
    identifier "." |
    module-name-qualifier identifier "." ;
```

模块的名称是由点号 `.` 连接的一个或多个的标识符组成。模块名中的标识符规则和其他的标识符规则一致，显而易见的是，`export` 和 `module` 不能用于标识符。

此处的点号 `.` 有什么特别的吗？实际上并*没有*。点号仅仅是为了方便开发者罢了。`boost.asio.async_completion` 比起 `boost_asio_async_completion` 更容易理解其中的逻辑层次，但两种命名规则在语义上没有区别。

> 你可能注意到在 `main.cpp` 我们使用的是 `import <iostream>;`，而 `<iostream>` 并不满足模块名的命名规则。怎么回事呢？答案是：`<iostream>` *不是*模块：它是头文件单元。虽然这是个很重要的话题，但我们在之后的帖文中才会讨论到。
## 一种全新的 C++ 源代码实体

纵观 C++ 的整个历史，一直有一个标准概念来概括 C++ 源代码单元的概念：**翻译单元(translation unit)**。

C++ 模块引入了一种全新的翻译单元，称之为 *模块单元(module unit)*。其定义十分简单：

> 一个*模块单元*是一个包含*模块声明*的翻译单元。

什么是 “模块声明”？我们在先前的示例中已经见过了。其语法也十分简单：

```
module-declaration:
    ["export"] "module" module-name [module-partition] [attribute-specifier-seq] ";" ;

module-partition:
    ":" module-name ;
```

简而言之，任何在开头包含 `module` 行的文件都是 *模块单元*。(下一小节将介绍 `模块分区(module-partition)` 的概念)。

**Important subdivisions:** *模块单元* 分为许多种类，了解不同种类的含义十分重要：
- *模块**接口**单元* 指 *模块声明* 包含 `export` 关键字的模块单元。同一个模块可以有任意个模块接口单元。
- *模块**实现**单元* 指除模块接口单元外其它的模块单元(即模块声明中没有 `export` 关键字)。
- *模块**分区*** 指 *模块声明* 包含 *模块分区* 部分的模块单元。
- *模块**接口分区*** 既是模块接口单元也是模块分区(同时包括 `export` 关键字和 `模块分区` 部分)。
- *模块**实现分区*** 既是模块实现单元也是模块分区(包括 `模块分区` 部分但没有 `export` 关键字)。
- ***主**模块接口单元* 既是非分区模块单元也是模块接口单元。一个模块中有且**仅能有一个**主模块接口单元。所有其他的模块接口单元必须为模块分区。

It isn’t named explicitly, and it isn’t entirely clear from the above reading, but it is possible to have a module implementation unit that is not a partition. A module-declaration without `export` and without a `module-partition` label is a module implementation unit. There is no way to import this module unit from another file, but it is useful to provide the implementation of entities declared in different module units.

## 模块分区

就像 C++ 的头文件一样，模块不一定必须划分为多个文件。然而，
Just like with C++ headers, there is no requirement that modules be split and subdivided into multiple files. Nevertheless, large source files can become intractable, so C++ modules also have a way to subdivide a single module into distinct translation units that are merged together to form the total module. These subdivisions are known as *partitions*.

Suppose we have two *enormous, cumbersome, and unwieldy* functions that we don’t want to include in the same module:

```
export module speech;

export const char* get_phrase_en() {
    return "Hello, world!";
}

export const char* get_phrase_es() {
    return "¡Hola Mundo!";
}
```

Whoa! That’s a ton of code! Let’s subdivide it using *partitions*:

```
// speech.cpp
export module speech;

export import :english;
export import :spanish;
```

```
// speech_english.cpp
export module speech:english;

export const char* get_phrase_en() {
    return "Hello, world!";
}
```

```
// speech_spanish.cpp
export module speech:spanish;

export const char* get_phrase_es() {
    return "¡Hola Mundo!";
}
```

```
// main.cpp
import speech;

import <iostream>;
import <cstdlib>;

int main() {
    if (std::rand() % 2) {
        std::cout << get_phrase_en() << '\n';
    } else {
        std::cout << get_phrase_es() << '\n';
    }
}
```

What’s going on here?

- We have **one module** named `speech`
- Speech has **two partitions**: `english` and `spanish`
- The syntax `export module <module-name>:<part-name>` declares that the given module unit is a *module interface partition* belonging to `<module-name>` with the partition named by `<part-name>`.
- The syntax `import :<part-name>` (with a leading colon) imports the partition named by `<part-name>`. The given `<part-name>` must belong to the importing module. Translation units that are not module units of a module `A` are not allowed to import partitions from `A`.
- The syntax `export import :<part-name>` makes the exported entities in the module partition visible as part of the module interface.

The name of a module partition follows the same rules as module names, except `private` is not allowed.

When a user imports a module, then all entities described in all *module interface units* for that module become visible in the importing file. Remember: *module interface partitions* are *module interface units*.

The module unit that contains `export module <module-name>` (with no partition name) is known as the **primary** *module interface unit.* There must be exactly one *primary module interface unit*, but any number of *module interface partitions*.

In the above example, `get_phrase_en` and `get_phrase_es` both live in the `speech` module. The subdivision into partitions is not exposed to users.

The module interface is defined as the union of all module interface units within that module.

We can classify the above source files as such:

- `speech.cpp` is the *primary module interface unit*.
- `speech_english.cpp` and `speech_spanish.cpp` are *module interface partitions*.
- `main.cpp` is a regular translation unit.

## “Submodules” are not a Thing (Technically)

Another way we could have subdivided the prior example might look like this:

```
// speech.cpp
export module speech;

export import speech.english;
export import speech.spanish;
```

```
// speech_english.cpp
export module speech.english;

export const char* get_phrase_en() {
    return "Hello, world!";
}
```

```
// speech_spanish.cpp
export module speech.spanish;

export const char* get_phrase_es() {
    return "¡Hola Mundo!";
}
```

```
// main.cpp
import speech;

import <iostream>;
import <cstdlib>;

int main() {
    if (std::rand() % 2) {
        std::cout << get_phrase_en() << '\n';
    } else {
        std::cout << get_phrase_es() << '\n';
    }
}
```

Instead of using partitions, we move the declarations of `get_phrase_en` and `get_phrase_es` into their own modules, and the `speech` module `export import` s them. The `export import <name>` syntax declares that users who import the module will transitively import the module of the given name.

The content of `main.cpp` is unaffected between the two layouts. It *implicitly* imports `speech.english` and `speech.spanish` by its import of `speech`.

If you familiar with some other languages’ module designs, it should be noted that *this* is **not valid**:

```
// speech.cpp
export module speech;

// NOT OK:
export import .english;
export import .spanish;
```

Python users might be familiar with such syntax as a *qualified relative import*, where the leading `.` tells the module mechanism to look for a sibling module with the given name. C++ Modules do not work like this as their is no intrinsic hierarchy. The compiler sees no relationship between modules `speech`,`speech.english`, and `speech.spanish`. The above snippet is completely nonsensical in the eyes of the language.

`speech.english` and `speech.spanish` *are not submodules* of `speech`. They are completely disjoint modules.

So what’s the deal? If both of these appear identical to downstream users, why would you choose one over the other? The answer is, of course, tradeoffs:

*When using partitions*, every entity in the interface partitions is part of the same module. **The module that owns an entity is intended to be part of that entity’s ABI!** This means that moving an entity from one module to another is potentially **ABI breaking**. (It should be noted that this author strongly discourages mixing and matching library versions in such a way that this would be a problem, but that’s another topic.)

*When using “submodules”*, you give users the ability to be more granular in what they import. Despite the potential speed-up from modules, an `import boost;` that imports the entirety of Boost could be deathly expensive to compile times!

## Module Implementation Units

So far we’ve poked at module *interface* units, but there’s another type of module unit: The module *implementation* unit.

A *module implementation unit* is a module unit which does not have the `export` keyword before the `module` keyword in its *module-declaration*. The implementation units belong to a named module. Entities declared within an implementation unit are visible *only* to the module of which they belong. This has the probable advantage of keeping details hidden and possible benefit of helping accelerate incremental builds as modification to the implementation units might not effect downstream modules. The details of when these benefits kick in and when they are inhibited will be a subject of a future post.

Here’s what our example would look like if we use *implementation units*:

```
// speech.cpp
export module speech;

import :english;
import :spanish;

export const char* get_phrase_en();
export const char* get_phrase_es();
```

```
// speech_english.cpp
module speech:english;

const char* get_phrase_en() {
    return "Hello, world!";
}
```

```
// speech_spanish.cpp
module speech:spanish;

const char* get_phrase_es() {
    return "¡Hola Mundo!";
}
```

This looks similar to before, but has a few changes:

- The `import` of the partitions in the primary interface unit no longer has the `export` keyword.
- The `module` declarations in the partitions are also missing the `export` keyword. This makes the partitions *implementation* partitions.
- The functions are defined in the implementation units without the `export` keyword. Their exported-ness is not relevant in the implementation units.
- The functions are *declared* in the primary interface unit with the `export` keyword. This makes the functions part of the module interface, even if not defined.

This is analogous to having a header file that declares two functions and two source files that define those functions. Changes to the implementation units do not effect the interface, and there is no way to modify the module interface by making changes to the implementation units.

I’ve been thinking about it, and it doesn’t seem like we have too much code to divide up into multiple files. Still, I’d like to be able to save on incremental build times when I only modify the implementation. Can I do that without partitions? Yes!

```
// speech.cpp
export module speech;

export const char* get_phrase_en();
export const char* get_phrase_es();
```

```
// speech_impl.cpp
module speech;

const char* get_phrase_en() {
    return "Hello, world!";
}

const char* get_phrase_es() {
    return "¡Hola Mundo!";
}
```

The partition syntax has disappeared, and we only have two source files for our module. The `module speech;` declaration (without `export` ) declares that module unit to be an *implementation unit* (not a partition). There is no way to import this file separately, nor is there reason to do so. These two files will be used to create the module `speech`.

## Restrictions on \[export\] {import,module} and Partitions

The token sequence `export import` may look weird at first, but it’s meaning is just what it sounds like: Import the given module, and then export that import so that downstream importers will also import the same module transitively.

Because of the way they are defined, there are a few important restrictions on how you can import and export module partitions.

### export import is only allowed for interface partitions

The following is not allowed:

```
module A:Foo;
```

```
export module A;

export import :Foo;  // NO! :Foo is not an interface unit!
```

The reasoning is fairly simple: Because `A:Foo` does not contribute to the interface of `A`, it is nonsensical to propagate the entities of `A:Foo` to importers.

### export module must appear once per module

Suppose we are defining a module `Cats`. In order to define it, we’ll need at least one module interface unit that is not a partition of `Cats`.

```cpp
export module Cats;

export void meow();
export void purr();
```

What happens if we add another module unit that extends `Cats`?

```
export module Cats;

export void hiss();
```

Not allowed! The `export module` without a partition name is the *primary* interface unit, and there can only be one.

The current nature of compilers could not cope with such a design. Two compiler invocations would be unable to know if these should be “merged,” or if you’ve very quickly rewritten and moved the definition of `Cats` to another file. A compiler could theoretically see both files simultaneously and do the merge, but then you have questions about how the two files interact. Supporting such a design would be very complex with little benefit.

Additionally, supporting multiple primary interface units raises another question: What stops a user from injecting things into someone else’s module by defining another `Cats` file?

### export is not allowed in implementation units

This one is self-explanatory. Allowing an implementation unit to export an entity (or module import) is senseless.

### All interface partitions must be re-exported from the primary interface unit

Because an interface partition may extend the interface of a module (i.e. introduce entities not declared in the primary interface unit), it is essential that a compiler be able to see the entirety of a module’s interface just by looking into the primary interface unit.

```
export module Cats:Sounds;

export void meow();
export void hiss();
```

```
export module Cats:Behaviors;

export void eat();
export void sleep();
```

```
export module Cats;

export import :Sounds;
```

```
// importer.cpp
import Cats;

void foo() {
    meow(); // Okay
    hiss(); // Okay
    eat();  // Er...
}
```

`importer.cpp` uses `eat`, and `eat` is defined with `export` in an interface partition, but how is the compiler supposed to know that `eat` is a member of `Cats`? The primary `Cats` unit does not export-import `:Behaviors`!

The program above is *ill-formed*, **no diagnostic required**! NDR is one of the most frightening terms in the C++ standard. If often means *undefined behavior*.

However: You can reasonably expect that this issue will most often result in a compile error, as the definition of `eat` is not visible. It may simply be unclear *why* `eat` is not visible when you can clearly see `export void eat();` defined in `export module Cats:Behaviors`. So you’ll most likely see a diagnostic, just not a diagnostic related to the missing re-export.

### Module implementation units are spooky beasts

One can define a module interface and implementation in separate files without the need for partitions. The may be unintuitive, but is perfectly reasonable upon inspection.

```
export module Cats;

export void dream();
export void sleep();
```

```
// cats_sleep.cpp
module Cats;

import sleep_info;

void sleep() {
    if (is_rem_sleep()) {
        dream();
    }
}
```

In the above, `cats_sleep.cpp` is an implementation unit for `Cats`. The primary interface unit makes no mention of it, so you might expect this to be an issue. What’s going on?

The answer is that the `Cats` interface unit contains sufficient information that downstream users can import and use the module successfully. They need not see the implementation of `sleep` and `dream`, so there is no need to mention where it is defined. It can be assumed that the definitions will be resolved by the linker.

Another thing you may notice: `cats_sleep.cpp` calls `dream`, but we never import or declare it! This is perfectly fine: non-partition module implementation units implicitly import the module of which they are a member. Since there is no way for the interface to import this anonymous implementation unit, there is no risk of cyclic imports.

This begs a question: Why make a module implementation partition at all? You could just make anonymous implementation units instead, right?

Not always. Even though the entities in implementation units are not visible to importers, they *are* visible to other members of the module as long as they are imported. This is one possible way you might use PIMPL with modules (and before you ask: There are still good reasons to use PIMPL with C++ modules)

```
module Gadgets:PrivWidget;

struct PrivWidget {
    // ...
};
```

```
export module Gadgets;

import :PrivWidget;

export class Widget {
    std::unique_ptr<PrivWidget> _data;
    // ...
};

export Widget get_widget() {
    auto priv = std::make_unique<PrivWidget>(1, 2, 3);
    return Widget{std::move(priv)};
}
```

## Enough for Now

C++ modules are a huge change. To discuss every aspect of them will take more than a single post. We’ve covered the bare basics, but there is so much more to discuss.

Watch this space for follow-ups!