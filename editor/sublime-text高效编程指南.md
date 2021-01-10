# sublime text高效编辑
sublime text最大的特色就是批量编辑，而批量编辑的前提是能够快速选中相应的内容，常用的可以用`cmd+d (windows下用ctrl+d)`即可依次选中相同的内容，如下图所示。
![sublime text依次选择](https://tva1.sinaimg.cn/large/00831rSTgy1gdiombzttgg31390j5q5o.gif)
以上这种方法适合少量相同的选中，但对于一个大段落中要选中这个段落内很多相同的词，不妨用以下这种方法。
将以下这部分代码拷贝到preferences下的key bindings，保存一下。下次选中一段文本，按下快捷键`super+shift+s`即可查找你想批量编辑的单词，然后按下`alt+enter`即可在选中范围内去批量编辑。
```
{ "keys": ["super+shift+s"], "command": "show_panel", "args": {"panel": "replace", "in_selection": true}},
    { "keys": ["ctrl+f"], "command": "show_panel", "args": {"panel": "replace", "in_selection": false},
    "context":
        [
            { "key": "selection_empty", "operator": "equal", "operand": true}
        ]
    }
```
![sublime text在选中范围内查找](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomd4q31g31gk0q6nhw.gif)
按下`cmd+k+u`即可对选中部分大写，`cmd+k+l`则小写，u代表upper，l代表lower。
![替换大小写](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomdz3s1g30xn0m6q87.gif)
前面说的是批量对相同的部分进行编辑，sublime text也可以很方便进行多行编辑。只需选中需要多行编辑的段落，然后按下`ctrl+shift+l`即可进入多行编辑的状态，按下`ctrl+l`即可返回之前的段落选择。当然也可以通过`ctrl+shift+↑/↓`(windows下是`ctrl+alt+↑/↓`)来扩展光标，达到相同目的。如果你只是需要选中一行，用`cmd+l`即可(windows下是ctrl+l)。
![多行编辑](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomf1x5lg30xn0m6ti6.gif)
sublime text还可以快速选中括号内的内容，不限于小括号，还有中括号，大括号等都照选择不误。只需将鼠标的光标挪到括号内的内容的任何一处，然后按下`ctrl+shift+m`即可快速选中括号内容，十分有利于快速替换括号内的内容或者是复制括号内的内容。
![选中括号内的内容](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomfshhrg30xn0m6wi3.gif)
vim快速删除行可以用按两下dd解决，sublime text也不遑多让，无需选中行，只需要光标在该行，即可用`ctrl+shift+k`删除该行。
![删除行和复制行](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomgvl7vg30xn0m612m.gif)  
## 快速切换多个文件夹
## 多个视图
我觉得sublime text的多个不同的视图就像是蝙蝠侠适合不同作战状态下的战服一样，随着需要能够切换到最能够保证工作效率的状态。可以
![不同视图下的快捷键](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomhs4uwj308505eglv.jpg)
![切换视图](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomiqh3ng30qa0o9qiv.gif)
## 快速在多个文件跳转
如果你打开了多个文件的话，切换过来另外一个文件，我们希望能够在当前和之前几个文件下跳转，不妨使用`cmd+p`(windows下使用`ctrl+p`)在多个文件下跳转，并且sublime text很体贴的一点是，所列出的文件的顺序是按照你最近使用的顺序打开的。
![跳转文件](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomk51j5g30qa0o9gwo.gif)
当然你可以通过按`cmd+num`(num是你的文件标签的序号, windows下是`ctrl+num`)来跳转。
## 在多个文件夹内找具有相关关键字的文件
如果现在领导要你在一堆文件夹里面找一个具有reg_exp关键字的文件出来，估计你平时可能没有好好管理文档的习惯，现在怎么办呢？如果你没有Mac OSX下的Alfred的话真不好办，但是sublime text可以解决这个问题。你只需要按下`cmd+shift+f`开启全局查找，添加文件可能存放的文件夹，然后添加搜索关键字查找即可，返回find results文件即是所有包含该关键字的文件，双击即可打开。是不是很方便？用来作为查找相关关键字的软件也很不错。  
![在文件夹里查找关键字](https://tva1.sinaimg.cn/large/00831rSTgy1gdioml7qilg30qa0o9x1f.gif)
# 以项目或者文件夹的形式来进行工作
如果我们要进行一个项目的话，必定会需要多个文件，我们需要多个文件在同一个窗口下，这样我们可以方便按照项目来组织文件。这种情况下有两种方式，一个是使用sublime text的open folder，就可以打开该路径，并且里面的文件依次列出。
![打开文件夹](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomlzgq4g30qa0o912b.gif)
除此以外，如果需要的文件并不在同一个文件夹下，也可以在同一个窗口下将所有文件保存为一个project类型的文件，下次直接open project打开这个文件，相应的文件就会像上次一样在同样一个窗口下打开。
![保存为project](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomne092g30qa0o94qp.gif)

# 插件部分
以下主要总结一下sublime text一些好用的插件，非常有利于提高效率。

# 搜索

## googlesearch

这款插件可以方便地在sublime text里面打开搜索框，回车以后就可以跳转到浏览器的谷歌搜索界面，也可以选中文本之后右键用googlesearch。
![谷歌搜索](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomohmnug30qa0eke3u.gif)



# 对齐插件

## Alignment

![alignment](https://tva1.sinaimg.cn/large/00831rSTgy1gdiompz68sg30qa0ekmyl.gif)

如果写的代码左右有非常不工整和不对齐的地方，不妨尝试用一下Alignment这个傻瓜式对齐的插件。Alignment这款插件可以选中之后根据默认的符号去对齐，比如在这里就是"="符号，当然你可以设置更多的符号，比如"<",">"等等，你也可以用cmd加上鼠标右键的方式选中多个地方，然后按一下"ctrl+shift+a"就可以快速对齐。

# 选择和移动类

## ace-jump

ace-jump可以很方便地去根据自己输入的内容去选择对应跳转的位置。快捷键“shift+cmd+.”选择行跳转，"shift+cmd+;"选择字符跳转。

![acejump](https://tva1.sinaimg.cn/large/00831rSTgy1gdioms6zvwg30u00keto6.gif)

可以选择行跳转，或者是按照字符去跳转，只要先输入对应的字符，然后按照光标提示输入要跳转的位置字符即可。

## moveByParagraph

moveByParagraph顾名思义，就是可以上下按照一段一段的跨越速度去跳转，选择，从此之后再也不用鼠标拉动去选择了。

![movebyparagraph](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomtnenzg30u00ke1kx.gif)


## Expand-selection-to_quotes

Expand-selection-to-quotes可以选中引号内的全部文本，对于引号内有多个单词的不方便用cmd+d选中的，可以用这种方法选中。

![expand_selection_to_quotes](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomuwkrrg30u00ke0vz.gif)


# 编辑类

## FileDiffs

FileDiffs用来比较两个文件或者是剪贴板之间的差异十分方便。

![filediffs](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomxfdzog30u00ke4qp.gif)


## DeleteBlankLines

DeleteBlankLines如其名所示，就是用来删除空行的，选中要删除其中的空行的对应的部分，然后按下默认快捷键“ctrl+shift+alt+delete”即可删除其中的空行。

![deleteBlanklines](https://tva1.sinaimg.cn/large/00831rSTgy1gdiomyjy50g30u00ke4qp.gif)

## Text Pastry
![text pastry](https://tva1.sinaimg.cn/large/00831rSTgy1gdion1dfh2g30u00k8k1p.gif)
text pastry是很好用的批量编辑的助手，安装以后，比如我们需要生成好几个dataframe类似的变量，只是用序号或者字母标识出区别，这个时候text pastry就起到很大的作用，在mac下输入`cmd+shift+p`命令，输入相关提示语`text pastry from`等任何一个单词即可，就有相应的text pastry命令出现，选中回车，就自然会根据你批量编辑的行数去添加相应的序号或者字母，这在开发中往往需要同时开发多个相类似的变量当中十分高效，效率是翻倍的。

## 使用markdown进行写作
sublime text作为一款正经的编辑器，当然也是可以写markdown文本的，虽然看起来sublime text看起来并没有写markdown文本编辑器的优势，既没有typora漂亮的书写界面，也没有bear一样的方便的云同步的功能。但是我觉得sublime text如果单纯论在批量编辑方面的功能，完全可以将其应用在编辑表格方面上。要知道markdown的表格其实是让我蛮头疼的一个编辑内容，因为有太多`|`，`:-----:`这类的符号，这些工作交给sublime text来完成最合适不过了
![edit markdown](https://tva1.sinaimg.cn/large/00831rSTgy1gdion3gq1zg30u00k8npd.gif)
虽然tyora也可以很方便地像excel一样拉出一个表格来，但可惜不支持保持表格格式复制过来，所以用sublime text是一个批量编辑的好工具。
此外，你还可以安装一个叫`MarkdownLiveShow`的插件，可以实时显示格式化后的markdown文本。
![markdownLiveShow](https://tva1.sinaimg.cn/large/00831rSTgy1gdion484yrg30u00k8gzt.gif)
个人觉得还是很不错的，通过`cmd+shift+p`调出命令后输入`MarkdownLiveShow`即可调出编辑和即时浏览界面。



以上就是我觉得比较好用的sublime text插件，其实还有很多很多插件我觉得还需要学习，以前并不觉得写代码的速度有关系，现在我觉得能不能用代码快速实现自己的想法对一个程序员来说十分之关键，而要实现这样高速代码的效果，一个是状态很重要，第二个是一些高效的操作，包括快捷键和插件，以及有意识去减少自己的重复工作，十分之重要。
