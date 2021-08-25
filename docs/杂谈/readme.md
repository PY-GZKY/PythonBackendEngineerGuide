
我在原本的数据表字段命名上采用了 `小驼峰` 命名法。事实上我曾一度将所有的命名都改成了 `驼峰命名`(除了 `Python` 偏好使用的`下划线`，因为它被代表着特殊的意义)

这是一种 `坏习惯`，但我还是这么做了。

原因很简单，因为我是项目的`发起者`，`开发者`和`维护者` ，所以对于项目命名规范和数据库命名规范也就成了我的一家之言。

就像这样:

```python
# 任务日志表
class TaskLog(Base):
    id = Column(Integer, primary_key=True, autoincrement=True)
    taskId = Column(String(50), ForeignKey('tasks.taskId'))
    taskLogMsg = Column(Text, comment='日志信息')
    taskLogLevel = Column(Text, comment='日志等级')
    updateTime = Column(Text, default=str(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())), comment='记录时间')
```

这在程序运行中也没有什么不妥，但是我不能保证每个人都有这种喜好(实际上大多数人都比较喜欢用下划线代替，这样可读性更高)。

更何况项目终究要被放大，如果有新的同事加入或接手了我的项目，我不希望他们在阅读项目代码时显得那么吃力。所以我大概对项目中的`命名`做了一下规范(可能并不是最佳规范，本来也没有什么最优的编码规范)

- 项目目录、文件名: `小驼峰`
- 类名: `大驼峰`
- 类方法名(除魔法方法或自定义的特殊方法): `小驼峰`
- 普通方法名: `小驼峰`
- 实体类字段: `下划线`(包括，`数据表名`，映射到数据表的`字段`名，需要迎合其他同事，特别时对于`共用`的数据表，这一点尤为重要)
- 普通方法名: `小驼峰`


更多的可以参考 `韦世东` 

> `Python 开发团队编码规范参考`: [https://www.weishidong.com/docs/cankao/code_style_reference.html](https://www.weishidong.com/docs/cankao/code_style_reference.html)

> `Python 编程风格指南`: [https://www.weishidong.com/docs/cankao/python_code_style_reference.html](https://www.weishidong.com/docs/cankao/python_code_style_reference.html)



