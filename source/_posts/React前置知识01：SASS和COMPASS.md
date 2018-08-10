---
layout: posts
title: React前置知识01：SASS和COMPASS
date: 2018-08-10 19:49:23
tags: [sass, compass, react]
category: notes
---

## Sass与Compass 
1. Compass是Sass基础上二次开发的工具。
2. **优点**：写出更优秀的CSS；解决CSS编写过程中的痛点问题（如精灵图合图等）；有效组织样式、图片、字体等项目元素。
3. **应用场景**：重构时自动化CSS；项目周期内更好组织项目内容。
 
#### .sass和.scss
```
// .sass：类ruby语法，空格敏感
h1
    color: #000
    background: #fff
 
// .scss：类css语法，花括号
h1 {color: red;}
```
 
## Sass
#### sass安装及使用
1. 安装ruby和rvm（可选，用于版本管理）
2. 通过gem(ruby安装自带的一个包管理器)安装sass。  
```
gem sources -l // 显示当前的源
gem sources --remove ... // 删除源...
gem sources -a ... // 添加源..
gem install sass // 安装sass
gem install sass --version=3.5.6 // 安装版本为3.5.6的sass
gem uninstall sass // 卸载sass
```
 
#### sass语法介绍
```
sass main.scss main.css // 编译成css文件
sass-convert main.scss main.sass // .sass和.scss
```
1. 变量：存储一些后期可能会修改的通篇使用的量，如字体等等，一般放在文件头部；
    ```
    $headline-ff: Braggadocio, Arial, Verdana, Helvetica, sans-serif;
    ```
2. 文件引入`@import`：类似于全局变量通常会单独放在一个文件中（_variables.scss）需要时通过`import`引入。
    ```
    @import "variables";
    // 多个import可用,分割
    @import "variables", "compass/reset";
    ```
    ==注意==：这里的`@import`并不是css原生的`@import`（缺点：CSS原生的`@import`必须要放在代码的最前边才能生效；当a.css引入了b.css，只有当浏览器将a下载下来，解析渲染时读到import时才回去下载b，此时浏览器处于阻塞过程，大大影响渲染时间，所以不建议使用）。sass在被编译时会将import的文件输出到相应的css文件，并且import指令可以放在任何地方。
    ```
    以下情况使用的是css原生import
    1. 文件名为.css结尾
    @import "variables.css";
    2. "http://"开头的字符串
    @import "http://variables.css";
    3. url()函数
    @import url("variables.css");
    4. 跟有media queries
    @import "variables" projection tv;
    ```
3. 嵌套语法和父类选择器
    ```
    // 类的嵌套
    .head {
        .content { color: red; }
        &:hover { background: green; } // 父类选择器&
        font: { // 属性嵌套
            family: Arial;
            size: 16px;
        }
    }
    ```
4. 变量操作
    - 操作方式
        - 直接操作变量，即变量表达式；
        - 通过函数操作，函数包含以下几种：
            - functions：跟代码块无关的函数，多为内置函数
            - 可重用的代码块（类似于C中的宏macro）
                - 使用时以复制拷贝的方式存在，称为mixin，通过@include调用
                - 使用时以组合声明的形式存在，通过@extend调用
    - 支持的运算操作符包含：`<,>,<=,>=,!=, ==, ()`。sass中的数值计算可以带单位，所以单位并不能混用。
    - sass支持css3中添加的hsl功能，自动转换为16进制色值，解决了不兼容问题。
    - mixin代码块的声明：一般放在页面顶部`@import`之后，或者单独抽离出一个文件，引入方法如下：  
        ```
        @mixin col-6 {
            width: 50%;
            float: left;
        }
        .webdemo {
            @include col-6();
        }
 
        // 一种可以实现但是不建议的方法，
        // 规范建议类名最好有语义化的作用，而非视觉化
        @mixin col-6 {
            .col-6 {
                width: 50%;
                float: left;
            }
        }
 
        @include col-6();
 
        // 带参数的mixin, 50%为默认参数
        @mixin col ($width: 50%) {
            width: $width;
            float: left;
        }
        ```
    - 组合声明：以一种继承的形式来避免CSS的冗余。==工作原理是把继承者的选择器，插入到被继承者选择器所在的位置==。
        ```
        .error.instruction {
            color: #0f0;
        }
 
        .error {
            color: #f00;
        }
 
        .serious-error {
            @extend .error;
            border: 1px #f00;
        }
 
        <div class="serious-error instruction">serious</div> // #0f0
        ```
        **注意**：extend不能继承选择器序列（即`@extend .A .B`不可行，会报错）；使用%可构建仅用于继承的选择器（`%name {...}`），不会出现在生成文件中。   
 
5. sass中的媒体查询：sass中的media query可以内嵌在css规则中，在生成css的时候，media query才会被提到样式的最高层级。避免重复书写选择器，同时避免打乱样式表的流程。  
 
6. sass提供了非常好的嵌套能力，但是嵌套带来的副作用也是不可忽视的：
     - 浏览器解析css文件是按照从右往左的顺序。即对于`.main .headline`会先找到类名为headline的元素，然后再向上查找父级元素是否类名为main，否则继续向上直到查找到类名对应的元素或者html元素。这样导致渲染效率的低下；  
     - 增加了样式修饰的权重；
     - 制造了样式位置的依赖；  
 
    最佳实践是在命名的时候对类名进行语义化的命名，比如`.main-headline`，同时为了保留嵌套清晰易维护的优点，可以通过`at-root`指令指明将嵌套的内容输出到样式表顶层。   
 
7. mixin的参数校验示例
    ```
    @mixin col-sm ($width: 50%) {
        /*输入校验*/
        @if type-of($width) != number {
            @error "$width必须是一个数值类型，目前输入的width是#{$width}.";
        }
 
        @if not unitless($width) { /* 没有单位 */
            @if unit($width) != "%" {
                @warn "$width必须是一个百分值，目前输入的width是#{$width}.";
            }
        } @else {
            @warn "$width必须是一个百分值，目前输入的width是#{$width}.";
            $width: (percentage($width) / 100); /*数值变成百分号表示形式时会增加100倍*/
        }
        @media (min-width: 768px) {
            width: $width;
            float: left;
        } 
    }
    ```
8. sass的四种输出格式`config.rb`中的`output_style`：
    - expanded：默认，样式展开，与手动书写css习惯一致；
    - nested：反映css样式修饰的html的结构，根据嵌套对应缩进样式；
    - compact：将所有属性汇总到一行，关注选择器之间的关系，而非选择器内的属性；
    - compressed：样式表压缩以占用最少的空间。
 
9. 其他：`@each` `@for` `@while`
 
10. 常用网址
    - Sass中的functions详情页：http://sass-lang.com/documentation/Sass/Script/Functions.html
    - Sass和Compass必备技能之Sass篇（视频教程）：https://www.imooc.com/video/7155
 
## Compass
#### compass安装及使用
1. 通过`gem install compass`即可安装成功
2. Compass目录创建  
```
compass create file-name // 初始化工作目录，file-name为生成的文件名
```
3. 目录结构  
```
sass：
    - _*.scss：用于被其他sass文件引入，不会被单独编译
    - *.scss：会被单独编译
    注意：同一目录下，局部文件和非局部文件不能重名
stylesheets：sass文件编译生成的css文件
config.rb：配置项目文件
```
4. 命令
```
compass compile [path/to/project] // 按需编译
compass watch [path/to/project] // 监听目录编译
```
#### compass核心模块
CSS3、Helpers、Typography、Utilities模块通过`@import "compass"`就可以直接引入；而Reset和Layout模块需要分别通过`@import "compass/reset"`和`@import "compass/layout`明确指定引入。  
- CSS3：跨浏览器的CSS3兼容能力  
- Helpers：内含多函数，比较少用到  
- Typography：修饰文本样式  
- Utilities：辅助工具模块，多为mixins  
- Reset：浏览器样式重置模块  
- Layout：提供对页面布局的控制 
 
除了以上六大功能模块之外，还包含browser模块，用于配置compass默认支持的浏览器机器版本，该模块的配置会影响其他模块的输出。    

1. **reset模块**
    所有包含的模块可见：http://compass-style.org/reference/compass/reset/utilities/  
    可以通过调用mixins来调用不同的模块。   
```
    // 如`nested-reset`用于重置页面下某个选择器的所有标签，方式如下：
    .test {
        @include nested-reset;
    }

    // 也可以通过传参的方法，将某个选择器下的样式重置，方式如下：
    // 第一个参数为选择器，第二个参数为是否强制覆盖（!important）
    @include reset-display('.test', true); 
```

    ==使用normalize替代==

    ```
    gem install compass-normalize # 下载normalize
    require 'compass-normalize' # 在config.rb引入

    @import "normalize"; # 在scss文件中替换reset
    ```

    normalize核心模块本身包含八个部分：
    - base: body和html标签的字体文字大小边距等
    - html5: 统一html5中新增的元素样式，如article、section的展现形式
    - links: 统一a标签的展示形式，去掉hover和active时的下划线
    - typography: 统一b, strong, h1, sub, sup等段落文本的样式修饰
    - embeds: img, svg等
    - groups: figure, pre, code等
    - forms: form相关的button, input, textarea等
    - tables: table相关table, td, th等
    这八个部分可以通过子路径单独引入，如`@import "normalize/base";`，通过子类引入的方法需要在前置位置添加`@import "normalize-version";`。
     

2. **layout模块**（使用率低）  
http://compass-style.org/reference/compass/layout/
内部分了三大模块grid-background，sticky-footer，stretching。都可以通过子目录的方式如`@import "compass/layout/grid-background"`的方式显示引入。   
    - stretch：拉伸填充。通过`@include stretch(top, right, bottom, left)`调用，将元素拉伸填充屏幕，参数可缺省。  
    - sticky-footer: 使页脚始终处于最底部。需符合一定的html结构（详见官网）。
    - grid-background: 使用css3中的grid定宽定高自适应宽高   
   

3. **CSS3模块&Browser Support 模块**     
    - [CSS3模块](http://compass-style.org/reference/compass/css3/)：封装了CSS3新属性，通过调用对应的新属性即可，如`@include box-shadow(1px 1px 2px 2px #aaa);`，可以自动添加浏览器前缀（除此之外，提供部分CSS3属性在IE下的兼容处理，如inline-block, opacity等）。
    - Browser Support模块：通过`@import "compass/support"`可直接引入，实际上CSS3已引入了该模块。
        - `browser()`：通过`@debug browser()`可显示出当前支持的浏览器；
        - `browser-version('chrome')`：显示当前对应浏览器考虑的所有版本；
        - `$supported-browsers: chrome firefox`：限制当前支持的浏览器；
        - `$browser-minmum-versions: ("ie": "8")`：通过键值对的方法限制最低支持的版本；
        - `$graceful-usage-threshold=0.1`：如果对于某个属性（若不支持仅仅影响美观度）在支持的浏览器中该属性的使用率达到了0.1%，则对其进行兼容。假如显式声明了兼容的浏览器版本，则不会根据该设置判断属性是否需要兼容。   
        - `$critical-usage-threshold=0.01`：如果对于某个属性（若不支持会导致页面混乱无法阅读等等较大问题）在支持的浏览器中该属性的使用率达到了0.01%，则对其进行兼容。假如显式声明了兼容的浏览器版本，则不会根据该设置判断属性是否需要兼容。      
           

4. **Typography模块**   
    - Links
        - `@include hover-link();`：正常态下去掉下划线，在hover或者focus时才显示
        - `@include  link-colors(normal, hover, active, visited, focus)`：设置不同状态下的颜色值，只有第一个参数是必须参数
        - `@include unstyle-link()`：抹平超链接样式
    - Lists
        - `@include no-bullets()`： 去掉列表前的list-style，包含ul和li下
        - `@include no-bullet()`： 去掉单个li元素前的list-style
        - `@include inline-list()`：使得list横向布局，通过设置为`inline`实现
        - `@include horizontal-list(padding, float)`：使得list横向布局，通过float实现，第一个参数为padding值，第二个参数为float的方向。
        - `@include inline-block-list(padding)`：目的同上，通过设置li的display值为inline-block实现。
        
    - Text
        - `@include force-wrap()`：连续长文本强制换行（如url等）
        - `@include nowrap()`：连续长文本强制不换行，其实等同于`white-space: nowrap;`
        - `@include ellipsis()`：文本超出容器宽度，在其后添加省略号，为了兼容firefox低版本，可通过安装`compass install ellipsis`，通过`$use-mozilla-ellipsis-binding: true;`开启对firefox低版本的支持。   
        - `@include hide-text()`：隐藏文本，text-indent实现
        - `@include squish-text()`：隐藏文本，通过font-size/opacity实现
        - `@include replace-text(url, bg-pos-x, bg-pos-y)`：设置背景图片
        - `@include replace-text-with-dimensions(本地图片地址)`：设置本地图片为背景，自动检测宽高使得容器大小与背景图片一致

    - Vertical Rhythm
        行内容间的留白，让所有文本元素的高度是基准高的整数倍   
   

5. **Helpers模块**  
     Helpers中为函数，而不是mixins，不需要include，可以直接调用，其中的许多函数都于config.rb中的配置项对应。
    - Data URI：可以在Web 页面中包含图片但无需任何额外的HTTP 请求的一类URI（通过base64编码实现）。然而比直接使用图片资源相比要多使用50%的CPU资源，多4倍的内存，且不支持IE6/7。`background-image: inline-image('image.png')` 图片参数是相对于在cofig文件中设定的image目录设置的目录的相对路径。
    - `image-url('image.png')`：对于某些分布在CDN上的图片，后面常常跟着一个时间戳（cache buster）来标示这个图片更新的版本或者时间，对于CSS背景图片设置不友好，而且经常更改。image-url可以很好地解决这个问题，同时引用路径也只需要关注图片相对于配置的图片目录image_dir的相对路径。`stylesheet-url`和`font-url`用于管理指向CSS目录和font目录的资源文件路径，用法类似。   
    - `compass-env()`：compass编译环境，只有两个返回值development和production。
    - `image-width()`和`image-height()`：用于计算图片的宽和高   
    - `append-selector($selector, $to-append)`：将第二个参数叠加组合到第一个参数中生成学则器。    

   
6. **Utilities模块**
   Utilities模块主要是一些无法分类到其他模块中的功能，其中又分为5个类别：`Color`、`Print`、`Tables`、`General`、`Sprites`。
    - Color
        - `brightness($color)`：计算颜色的亮度
        - contrasted($background-color, [$dark], [$light], [$threshold])：根据输入的bgColor自动生成background-color属性，并在输入的dark和light两个色值之间选取一个和背景颜色对比度更大的设为color属性。   
    - Print
        必须在两个文件中协同使用，`pirnt.scss`和`screen.scss`都需要引入print模块，主要是应用于适配打印设备。
        - `print-utilities`：在`screen.scss`中调用需要传参`@include print-utilities(screen)`不传默认为`print.scss`中。
    - Tables
        - `outer|inner-tables-borders($width, $color)`：分别用于设置table内外边框
        - `table-scaffolding()`：单元格文本对齐和padding初始化，`th`居中，`.numberic`右对齐。
        - `alternating-rows-and-columns($evenColor, $oddColor, $相邻两列颜色差值，$thColor, $tfootColor)`：对奇偶行|相隔列进行不同的颜色修饰。
    - General
        -`clearfix()`：通过在`.clearfix`中设置`overflow:hidden`清除浮动。
        -`legacy-pie-clearfix()`：通过伪类清除浮动。
        -`float($direction)`：根据配置设置是否启用对IE6的hack解决double float-margin bug。
        -`min-height|width($length)`：设置min-height/min-width，兼容IE。
        -`tag-cloud($baseFontSize)`：生成大小不同的字体，类名为xs,xxs, s, l, xl, xxl。
    - Sprites
        主要是通过`sprite helpers`实现。
        ```
        @import "compass/utilities/sprites;
        @import "logo/*.png";  /* compass据此生成sass样式文件，默认不会存储在硬盘，可通过在控制台输入compass sprites "images/logo/*.png"生成文件查看，生成sprites */
        @include all-logo-sprites();   /* 图片使用，中间的logo为目录名，只取路径的最后一个文件夹名 */
        ```
        图片引用的方法有两种，一个是直接在对应元素中添加对应类名`logo-imageName`；另一个则是通过`@include logo-sprite("imageName")`的方法。   
        - 对于button类型的图标，希望在hover, active等状态下使用不同的图片，可以通过在对图片命名为`imageName_hover.png`等，compass就会自动生成对应样式。可通过设置`$disable-magic-sprite-selectors:true`关闭该特性。
        - `$logo-layout: vertical|horizontal|diagonal|smart`：生成精灵图的布局方式设置。

   
#### compass其他知识
1. config.rb中`require 'compass/import-once/activate'`：启用import-once，对于多次import同一文件只会引入一次，避免冗余。如果确实是需要多次引入，则可以通过 @import "compass/reset!"来引入后面的重复引用文件。  
2. 当使用`compressed`作为sass的输出格式时，默认会去掉所有注释文字。而对于一些必要的注释内容（如copy rights），可以通过如`/*! hahaha */`来避免被去除。
3. `@debug`可以在控制台显示出mixins函数等对应的输出，如`@debug browser()`显示出的当前compass支持的浏览器，类似于console.log()的功能；
4. 通过在控制台输入`compass interactive`可以进入console，直接调用mixins。   
5. compass生成地址的时候，默认生成的都是绝对地址，认为`config.rb`所在的位置为根路径。假如有专门的服务器地址来存储相关的文件等，可以在config.rb中设置（http_path等），也可以通过relative_assets设置为true来生成相对路径。  
6. `compile compass -e production --force`：强制compass在production环境下编译，也可以在config.rb中配置`environment = :development`。   
7. 在选择器，字符串或SASS变量中如果需要引用函数，需要使用形如#{fn()}。
