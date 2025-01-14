## 打包接口

xpack 作为插件形式提供，它的所有 API 我们需要通过 `includes("@builtin/xpack")` 方式来引入。

```lua
includes("@builtin/xpack")

xpack("test")
    set_version("1.0")
    set_homepage("https://xmake.io")
    add_installfiles("...")
```

### xpack:set_version

#### 设置包版本

这个接口用于设置生成的安装包的版本：

```lua
xpack("test")
    set_version("1.0")
    -- ...
```

如果我们没有设置，但是通过 `set_targets` 绑定了安装的目标程序，那么也会使用 target 中的版本配置。


```lua
target("foo")
    set_version("1.0")

xpack("test")
    set_targets("foo")
    -- ...
```


我们也可以使用全局工程的版本，如果没有绑定任何 targets。


```lua
set_version("1.0")

xpack("xmake")
    -- ...
```


### xpack:set_homepage

#### 设置主页信息

```lua
xpack("xmake")
    set_homepage("https://xmake.io")
```

### xpack:set_title

#### 设置标题信息

通常用于配置安装包的简单描述，相比 `set_description` 更加简短。

```lua
xpack("xmake")
    set_title("Xmake build utility ($(arch))")
```

### xpack:set_description

#### 设置详细描述

这个接口可以设置安装包更加详细的描述信息，可以用一到两句话详细描述下包。

```lua
xpack("xmake")
    set_description("A cross-platform build utility based on Lua.")
```

### xpack:set_author

#### 设置作者信息

我们可以设置邮箱，姓名等来描述这个包的作者。

```lua
xpack("xmake")
    set_author("waruqi@gmail.com")
```

### xpack:set_maintainer

#### 设置维护者信息

我们可以设置邮箱，姓名等来描述这个包的维护者。

维护者跟作者有可能是同一个人，也可能不是一个人。

```lua
xpack("xmake")
    set_maintainer("waruqi@gmail.com")
```

### xpack:set_copyright

#### 设置包的版权信息

```lua
xpack("xmake")
    set_copyright("Copyright (C) 2015-present, TBOOX Open Source Group")
```

### xpack:set_licensefile

#### 设置包的 License 文件

我们可以设置 LICENSE 所在的文件路径，像 NSIS 的安装包，它还会额外将 LICENSE 页面展示给安装用户。

```lua
xpack("xmake")
    set_licensefile("../LICENSE.md")
```

### xpack:set_company

#### 设置包所属的公司

我们可以用这个接口设置包所属的公司和组织名。

```lua
xpack("xmake")
    set_company("tboox.org")
```

### xpack:set_inputkind

#### 设置打包的输入源类型

这是个可选接口，可用于标识当前打包的输入源类型

- binary: 从二进制文件作为输入源打包，通常使用 `add_installfiles`
- source: 从源文件作为输入源开始打包，通常使用 `add_sourcefiles`

这一般用于自定义的打包格式，而对于内置的格式，比如: nsis, zip, srczip 等等，
其实已经能够判断获取到当前打包的输入源是从源码开始打包，还是直接从二进制源开始打包。

因此，除非必要（比如要自定义打包格式），通常我们不需要设置它。

而我们在脚本域中，也可以通过 `package:from_source()` 和 `package:from_binary()` 来判断当前的输入源。

```lua
xpack("test")
    set_formats("nsis", "zip", "targz", "srczip", "srctargz", "runself")
    add_installfiles("src/(assets/*.png)", {prefixdir = "images"})
    add_sourcefiles("(src/**)")
    on_load(function (package)
        if package:from_source() then
            package:set("basename", "test-$(plat)-src-v$(version)")
        else
            package:set("basename", "test-$(plat)-$(arch)-v$(version)")
        end
    end)
```

如果上面的打包配置，如果是 nsis 包，默认从二进制文件作为输入源，进行打包，会去打包 `add_installfiles` 配置的文件。

而 `srczip`, `srctargz` 和 `runself` 是从源文件开始打包，会去打包 `add_sourcefiles` 中的文件，然后再执行打包脚本。

### xpack:set_formats

#### 设置打包格式

配置当前 XPack 包需要生成的打包格式，可以同时配置多个，`xmake pack` 命令会一次性全部生成。

!> 有些格式如果当前平台不支持生成，会自动忽略。

```lua
xpack("test")
    set_formats("nsis", "zip", "targz", "srczip", "srctargz", "runself")
```

我们也可以通过命令，指定生成其中部分格式，而不是一次性全部生成。

```bash
$ xmake pack -f "nsis,zip"
```

通过逗号分隔，指定生成 NSIS 和 zip 包，暂时忽略其他格式包。

目前支持的格式有：

| 格式     | 说明                              |
| ----     | ----                              |
| nsis     | Windows NSIS 安装包，二进制安装   |
| zip      | 二进制 zip 包，不包含安装脚本     |
| targz    | 二进制 tar.gz 包，不包含安装脚本  |
| srczip   | zip 源码包                        |
| srctargz | tar.gz 源码包                     |
| runself  | 自运行 shell 脚本包，源码编译安装 |
| srpm     | rpm 源码安装包（待支持）          |
| deb      | deb 二进制安装包 （待支持）       |
| 其他     | 可自定义格式和安装脚本            |

### xpack:set_basename

#### 设置包文件名

设置生成包的文件名，但不包含后缀名。

```lua
xpack("xmake")
    set_basename("xmake-v$(version)")
```

我们也可以在其中配置 `$(version)`, `$(plat)`, `$(arch)` 等变量。

另外，想要更灵活的配置，可以再 on_load 脚本中去配置它。

```lua
xpack("xmake")
    on_load(function (package)
        package:set("basename", "xmake-v" .. package:version())
    end)
```

### xpack:set_extension

#### 设置安装包的扩展名

通常我们并不需要修改生成包的扩展名，因为指定了 `nsis`, `zip` 等格式后，都会有一个默认的后缀名，例如：`.exe`, `.zip`。

但是，如果我们正在自定义包格式，需要生成一个自定义的包，那么我们可能需要配置它。

```lua
xpack("mypack")
    set_format("myformat")
    set_extension(".myf")
    on_package(function (package)
        local outputfile = package:outputfile()
        -- TODO
    end)
```

例如，这里我们自定义了一个 myformat 包格式，采用 `.myf` 的自定义后缀名，然后我们就可以在 on_package 中生成它，

`package:outputfile()` 返回的包输出文件名中就会包含这个后缀名。

### xpack:add_targets

#### 关联目标程序

我们可以通过这个接口，配置关联需要被安装的目标 target。

```lua
target("foo")
    set_kind("shared")
    add_files("src/*.cpp")
    add_headerfiles("include/(*.h)")

xpack("test")
    set_formats("nsis")
    add_targets("foo")
```

当生成 test 安装包的时候，被关联的 foo 目标的可执行程序，动态库等待都会被一起打包安装。
另外，target 中通过 `add_headerfiles` 和 `add_installfiles` 配置的自定义安装文件也会被打入安装包，一起被安装。

而且我们还可以在 target 和它的 rules 中通过 `on_installcmd`, `after_installcmd` 等自定义打包安装脚本，也会被一起执行。

### xpack:add_components
### xpack:set_bindir
### xpack:set_libdir
### xpack:set_includedir
### xpack:set_prefixdir
### xpack:set_nsis_displayicon
### xpack:set_specfile
### xpack:set_iconfile
### xpack:add_sourcefiles
### xpack:add_installfiles
### xpack:on_load
### xpack:before_package
### xpack:on_package
### xpack:after_package
### xpack:before_installcmd
### xpack:before_uninstallcmd
### xpack:on_installcmd
### xpack:on_uninstallcmd
### xpack:after_installcmd
### xpack:after_uninstallcmd
### xpack:set_specvar

## 组件接口

### xpack_component:set_title
### xpack_component:set_description
### xpack_component:set_default
### xpack_component:on_load
### xpack_component:before_installcmd
### xpack_component:before_uninstallcmd
### xpack_component:on_installcmd
### xpack_component:on_uninstallcmd
### xpack_component:after_installcmd
### xpack_component:after_uninstallcmd
### xpack_component:add_sourcefiles
### xpack_component:add_installfiles

