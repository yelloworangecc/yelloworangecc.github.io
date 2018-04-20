---
layout: post
title:  "Emacs Org: TODO项"
summary: org采用集成TODO项到笔记文件内的方式管理TODO列表,因为TODO项通常会在记笔记的过程中产生.org模式只需为笔记文件的树结构中某个实体打上标记就可以把它作为TODO项进行管理.
featured-img: sleek
# categories: jekyll update
---
## 基本的TODO功能 ##

org采用集成TODO项到笔记文件内的方式管理TODO列表,因为TODO项通常会在记笔记的过程中产生.在org模式中,只需简单的为树结构中某个实体打上标记就可以把它作为TODO项进行管理.这种方式与TODO列表和笔记分离的方式相比没有重复信息,并且在TODO项出现的地方总是有相关的完整的上下文可供参考.当然这种方式也使得的TODO项分散在整个笔记文件中.org模式通过提供额外的方式汇总显示所有你要做的事.

在任何标题前添加关键字"TODO"就可以生产TODO项,例如:

```
*** TODO 任务1
```

最简单的管理TODO项的方式是使用命令,这些命令包括:

  * `C-c C-t` 在TODO项的各个状态间循环切换,默认状态有TODO和DONE两种,另外还包括未标记的普通标题状态.
  * `C-u C-c C-t` 没有指定关键字时输入特定的关键字.
  * `S-right|left` 改变当前状态为前一个或者后一个状态,与循环切换相似,但在有多个TODO状态时更有用.(命令中的竖线表示"或者")
  * `C-c / t` 在一个稀疏的树中查看TODO项.此命令会折叠其他标题,仅显示TODO状态的标题和包含这些标题的上级标题.
  * `C-c a t` 显示全局的TODO list,命令会收集agenda文件中的所有TODO项到单个buffer中.
  * `S-M-RET` 在当前项的下方插入新的TODO项.

下面的图片显示使用TODO项管理任务的例子:
![org TODO的例子](/pics/org-date-time-example.jpeg)
  
## TODO关键字的扩展 ##

默认情况下,TODO项的标签只有TODO和DONE两种,org模式允许定义更加复杂TODO关键字系统来给你的TODO项分类.不同的TODO关键字系统作用也不同.关键字保存在org-todo-keywords变量中.需要注意的事,加tag是另一种给标题分类的方式,这种方式更加通用.

### 作为工作状态关键字 ###

你可以使用TODO关键字来指示同一个条项顺序的不同的工作过程,下面是一个例子.

```
(setq org-todo-keywords
  '((sequence "TODO" "FEEDBACK" "VERIFY" "|" "DONE" "DELEGATED")))
```

其中的竖线将其他状态与完成状态进行区分,如果不指定,那么只有最后一个状态才是完成状态.你可以使用`C-3 C-c C-t`快速改变到VERIFY状态.每次变化状态,org模式会生成一条时间戳日志,供后续跟踪查询.

### 作为类型关键字 ###

你也可以使用TODO关键字来指示条目的不同类型.例如你可能需要区分工作任务或者家庭任务;或者在多人完成一个项目时,使用人名作为TODO关键字,下面是一个例子:

```
(setq org-todo-keywords '((type "Fred" "Sara" "Lucy" "|" "DONE")))
```

相关的命名有:

  * `C-c C-t` 在各类型间切换.
  * `C-3 C-c / t` 查看Lucy的TODO项.
  * `C-3 C-c a t` 从所有agenda文件中收集Lucy的TODO项.
  
### 多个关键字集合 ###

很多时候你会同时使用多个TODO关键字集合,例如你可能仅需要使用基本的TODO/DONE,也可能还需要一些状态来帮助你处理BUG,甚至还可能需要一个独立的"取消"状态.此时你的配置可能是这样的.

```
(setq org-todo-keywords
           '((sequence "TODO" "|" "DONE")
             (sequence "REPORT" "BUG" "KNOWNCAUSE" "|" "FIXED")
             (sequence "|" "CANCELED")))
```

这种情况下,所有的关键字应该互不相同,`C-c C-t`仅在一个子序列中起作用,因此你需要一种机制来初始化序列,下面是相关命令:

  * `C-u C-u C-c C-t | C-S-right|left` 依次切换子集合.(竖线表示"或者",前后空格表示应用在"完整"的命令,没用空格表示应用在命令的"部分")
  * `S-right|left` 在所有子集合中依次切换关键字.

### 快速访问TODO状态 ###
如果你想快速切换到指定的TODO状态,而不是一个一个依次切换,你可以在配置时为每一个状态分配一个字母做索引,例子如下:

```
(setq org-todo-keywords
           '((sequence "TODO(t)" "|" "DONE(d)")
             (sequence "REPORT(r)" "BUG(b)" "KNOWNCAUSE(k)" "|" "FIXED(f)")
             (sequence "|" "CANCELED(c)")))
```

这样,你在使用`C-c C-t`跟上索引字母就能直接切换到对应状态了.如果跟上空格,那么TODO关键字会被清除.

### 为单个文件设置关键字 ###

在不同的文件使用不同的TODO机制是非常有效的.这时可以在文件内添加特殊行来配置文件本地设置.下面几个例子和之前的例子效果相同,但只对当前文件有效.

```
#+TODO: TODO FEEDBACK VERIFY | DONE CANCELED

#+TYP_TODO: Fred Sara Lucy Mike | DONE

#+TODO: TODO | DONE
#+TODO: REPORT BUG KNOWNCAUSE | FIXED
#+TODO: | CANCELED
```

在输入`#+`后,按`M-TAB`可以补全,避免手动输入出错.

### TODO关键字的样式 ###

org模式使用特殊样式来高亮TODO关键字,`org-todo`是待办状态的样式,`org-done`是完成状态的样式,当你使用两种以上的不同关键字时,你可能需要使用`org-todo-keyword-faces`来配置多种不同样式.例如:

```
(setq org-todo-keyword-faces
           '(("TODO" . org-warning) ("STARTED" . "yellow")
             ("CANCELED" . (:foreground "blue" :weight bold))))
```

### 依赖关系 ###
Org文件的结构化特性使得很容易处理TODO项之间的依赖关系.通常,上级TODO项只有在下级子项全部完成时才能更新到完成状态.有时一系列任务可能有逻辑顺序关系,以至于某个任务不能在其他任务之前完成.如果配置了`org-enforce-todo-dependencies`,上述依赖关系会被检查,违反依赖关系的TODO项会被阻塞(不能设置为完成状态),其中顺序关系通过`ORDERED`属性指定.下面是一些例子:

```
* TODO Blocked until (two) is done
** DONE one
** TODO two

* Parent
  :PROPERTIES:
  :ORDERED: t
  :END:
** TODO a
** TODO b, needs to wait for (a)
** TODO c, needs to wait for (a) and (b)
```

另外,也可以使用`NOBLOCKING`属性来指定某任务绝不会被阻塞.

```
* This entry is never blocked
  :PROPERTIES:
  :NOBLOCKING: t
  :END:
```

相关命令如下:

  * `C-c C-x o` 为当前TODO项触发`ORDERED`属性.
  * `C-u C-u C-u C-c C-t` 避免任何阻塞.
  
其他依赖:
  * `org-agenda-dim-blocked-tasks` 配置日程相关的依赖.
  * `org-enforce-todo-checkbox-dependencies` 配置checkbox相关的依赖.
