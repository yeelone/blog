---
title: "使用正则表达式处理题库文本的Golang实现"
date: 2018-06-07T22:23:30+08:00
tags: ["golang","kotlin","android","正则表达式"]
categories: ["golang","kotlin","Android"]
---

##前言
> 由于企业内部的一些考试需要，有些同事要时常通过一个word格式的题库文件来查询相应题目，但在手机上用word进行查询，总觉得操作上不方便。借着这个契机，应用Golang和Kotlin开发了一个小工具，方便同事可快捷的查询题目。

写这篇文章也算是做个复习吧，因为不知道什么时候就会又一次忘了怎么写正则了。

~~虽然上面说了那么多，其实就是想帮同事作弊~~   ლ(╹◡╹ლ)


##题库文件参考


以下是我的题库文件，可参考：
tiku.txt

```

单选题： (共912小题，总分：1分)

1 . （ ）是银行网点直接和顾客接触的员工，因此在客户建立对银行网点第一印象、维持良好的银行网点服务形象方面起着重要作用 （1.50分）
A　网点负责人
B　大堂经理
C　柜员
D　客户经理
标准答案 ：C
试题解析 ：
1,065 . 柜员柜面营销与柜面服务包括 （2分）
A　负责业务办理间隙识别优质客户的工作，并将客户引导到最合适的服务渠道
B　保持头、脸、手、着装、修整、饰物的清洁、规范，保持职业化服务形象
C　执行站姿挺拔、坐姿端庄、行姿规范、行为检点、微笑亲和、致意得体、道歉真诚的服务要求，保持良好服务行为
D　执行声音亲和、语句清晰、措辞客气、表达明了、称呼准确、问候得体、适当寒暄的服务语言规范
E　负责详细记录客户信息，聆听、发现客户真实需求，为下一步客户经理的关系维护打下基础
标准答案 ：A B C D E
试题解析 ：
1,480 . 根据我社其它物品管理的规定，对于印章和钥匙，对应的调拨机构为上级管理机构，而假币上缴的对应上级机构为现金管理（分）中心。 （1分）
正确
错误
标准答案 ：正确

```

需求就是将这个文件，保存进数据库里，

### 第一步，使用正则表达式解析文件，并拆分出题干，选项，答案几个元素，再分别存入数据库里。
正则我用Golang实现，数据库用sqlite ,因为最后这个数据库是要给Android用的。



一： 先分析题干部分:
> 1 . （ ）是银行网点直接和顾客接触的员工，因此在客户建立对银行网点第一印象、维持良好的银行网点服务形象方面起着重要作用 （1.50分）

可分为三个部分来解析

1) 题序：**[ 1 . ]**  

正则表达：

```
完整的正则： ^\s*?(?P<index>\d+,*?\d*?\s*?\.)\s*?

^\s*? 表示以0或1个以上的空格开头 ,^表示匹配开头

(?P<index>) 是golang的分组语法，在这里不用理会

\d+,*?\d*?\s*?\.
\d+ ： 表示1个以上的数字
,*? :  0或1个逗号 ， 因文本里的题序格式类似这样： 1,234 ，这个正则不支持 1,234,567 ，因为我觉得不太可能需要那么大的数
\d*? : 表示0个以上的数字
\s*?\. : 若干空格并以 . 结束
``` 
Golang 实现：

```
func findSubject(line string) (result bool, s string) {
	reg := regexp.MustCompile(`^\s*?(?P<index>\d+,*?\d*?\s*?\.)\s*?(?P<subject>.*)(?P<score>\s*?（.*分）)$`)
	matches := reg.FindStringSubmatch(line)
	if len(matches) > 0 {
		return true, matches[2]
	}
	return false, ""
}
```

2) 题干正文 : **[（ ）是银行网点直接和顾客接触的员工，因此在客户建立对银行网点第一印象、维持良好的银行网点服务形象方面起着重要作用 ]**

```
分析题干需要同时配合结尾来进行解析，可以明显的看到，每道题都以（1.50分）这样的格式结尾

(?P<subject>.*)(?P<score>\s*?（.*分）)$
```

二： 分析选项：

```
	^\s?(?P<index>[A-Z]\s?)\s?(?P<content>.*)
	选项总是以字母开头，所以用 [A-Z] 匹配即可。
```

Golang 实现：

```
func findOptions(line string) (result bool, s string) {
	//判断选择题
	reg := regexp.MustCompile(`^\s?(?P<index>[A-Z]\s?)\s?(?P<content>.*)`)
	matches := reg.FindStringSubmatch(line)
	if len(matches) > 0 {
		return true, line
	}

	reg = regexp.MustCompile(`^(正确)|^(错误)`) //这里需要注意，判断题并不是以[A-Z]开头的。
	matches = reg.FindStringSubmatch(line)

	if len(matches) > 0 {
		return true, matches[0]
	}

	return false, ""
}
```

分析答案就太简单了：

```
func findAnswer(line string) (result bool, s string) {
	matched, _ := regexp.MatchString(`^(标准答案 ：)`, line)
	if matched {
		return true, line
	}
	return false, ""
}
```

不多说了，最后就是保存进sqlite了

```
func saveToDB() {
	db, err := sql.Open("sqlite3", "./yee.db")
	checkErr(err)

	for _, q := range questionList {
		stmt, err := db.Prepare(`INSERT INTO question(subject, options, answer) values(?,?,?)`)
		checkErr(err)

		res, err := stmt.Exec(q.Subject, strings.Join(q.Options, "||"), q.Answer) //选项之间用 || 分隔即可，简单处理
		checkErr(err)
		id, err := res.LastInsertId()

		checkErr(err)

		fmt.Println(id)
	}
}

```
![image](https://wx3.sinaimg.cn/mw690/6547935dgy1fs2z37h8nhj20tb0ilte8.jpg)

完整代码请参考Gayhub: https://github.com/yeelone/bankolin-demo


### 第二步，实现Android App 
将第一步生成的 yee.db 数据库文件放进新建的Android项目里，目录为 src/main/res/raw ，

接着我们需要在APP启动时，将yee.db文件拷进android的数据文件地址

先实现一个Helper类并为context进行扩展：

```kotlin
package com.example.elone.myapplication

import android.content.Context
import android.database.sqlite.SQLiteDatabase
import org.jetbrains.anko.db.ManagedSQLiteOpenHelper

class DatabaseHelper(ctx: Context) : ManagedSQLiteOpenHelper(ctx, "yee.db", null, 1) {
    companion object {
        private var instance: DatabaseHelper? = null

        @Synchronized
        fun Instance(context: Context): DatabaseHelper {
            if (instance == null) {
                instance = DatabaseHelper(context.applicationContext)
            }
            return instance!!
        }
    }

    override fun onCreate(database: SQLiteDatabase) {
    }

    override fun onUpgrade(database: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
    }
}

val Context.database: DatabaseHelper
    get() = DatabaseHelper.Instance(applicationContext)
```

再接着回到MainActivity

```kotlin
private fun importDatabase() {
        // 存放数据库的目录
        var dirPath = "/data/data/"+PACKAGE_NAME+"/databases"
        var dir = File(dirPath)
        if (!dir.exists()) {
            dir.mkdir()
        }
        // 数据库文件
        var file = File(dir, "yee.db")
        try {
            if (!file.exists()) {
                file.createNewFile()
            }
            // 加载需要导入的数据库
            var instream: InputStream = this.applicationContext.resources.openRawResource(R.raw.yee)
            var fos = FileOutputStream(file)
            var buffer  = ByteArray(instream.available())
            instream.read(buffer)
            fos.write(buffer)
            instream.close()
            fos.close()
        } catch ( e: FileNotFoundException) {
            e.printStackTrace()
        }
    }
```

anko sqlite 查询语法

```
    private fun searchQuestion() {
        val editText = findViewById<EditText>(R.id.searchText)
        val rows = database.use {
            select("question")
                    .whereSimple("(subject like '%"+editText.text+"%') ")
                    .exec {
                        parseList(classParser<Question>())
                    }
        }

        mListView.adapter = MainAdapter(rows)
    }
```

![image](https://wx2.sinaimg.cn/mw690/6547935dgy1fs2z6ovwmlj20bt0jf781.jpg)























