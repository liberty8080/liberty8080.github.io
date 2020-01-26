---
title: java爬虫-webmagic的初步使用
date: 2020-01-25 12:56:13
tags:
	- java
	- 爬虫

---

# java爬虫-webmagic的初步使用

webmagic是一个国人写的java爬虫框架，简单灵活。以一个漫画网站为例，实现一个爬取漫画的爬虫。

表结构：

```sql
use spider;
create table bidongcomic_info
(
    comic_id        varchar(20) not null primary key ,
    comic_title     varchar(100) default null comment '标题',
    author          varchar(20) comment '作者',
    dsc             varchar(1000) comment '作品描述',
    cover_url       varchar(100) comment '封面链接',
    cover_url_local varchar(100) comment '本地链接'
);

create table bidongcomic_chapter
(
    chap_id    varchar(20) not null,
    chap_title varchar(20),
    comic_id varchar(100) primary key
);

create table bidongcomic_img_url(
    chap_id varchar(20),
    img_id int auto_increment primary key ,
    img_url varchar(100)
)

```

表已经建起来了，使用mybatis-generator生成mapper与model

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 配置mysql 驱动jar包路径.用了绝对路径 -->
    <classPathEntry location="/home/jacob/.m2/repository/mysql/mysql-connector-java/8.0.18/mysql-connector-java-8.0.18.jar" />

    <context id="wangyongzhi_mysql_tables" targetRuntime="MyBatis3">
        <!-- 防止生成的代码中有很多注释，加入下面的配置控制 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true" />
            <property name="suppressDate" value="true" />
        </commentGenerator>

        <!-- 数据库连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/spider?useUnicode=true&amp;characterEncoding=UTF-8"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- 数据表对应的model层  -->
        <javaModelGenerator targetPackage="top.liberty3306.spiders.modules.bidongcomic.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- sql mapper 映射配置文件 -->
        <sqlMapGenerator targetPackage="mapping"  targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- mybatis3中的mapper接口 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="top.liberty3306.spiders.modules.bidongcomic.dao"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 数据表进行生成操作 schema:相当于库名; tableName:表名; domainObjectName:对应的DO -->
        <table schema="spider" tableName="bidongcomic_info" domainObjectName="BidongComicInfo"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
        <table schema="spider" tableName="bidongcomic_img" domainObjectName="BidongComicImg"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
        <table schema="spider" tableName="bidongcomic_chapter" domainObjectName="BidongComicChapter"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
    </context>
</generatorConfiguration>
```

mybatis-generator生成了dao，mapper，与model，现在再model上加入注解，匹配字段

chapter.java 省略getter与setter

```java
@HelpUrl("https://www.bidongmh.com/booklist*")
@TargetUrl("https://www.bidongmh.com/book/*")
@ExtractBy(value = "//ul[@class=chapter-list]/li",multi = true)
public class BidongComicChapter {
    @ExtractBy(value = "//li/@data-id",notNull = true)
    private String chapId;
    @ExtractBy("//li/a/text()")
    private String chapTitle;
    @ExtractByUrl(value = "https://www.bidongmh.com/book/(.*/?)",notNull = true)
    private String comicId;
```

自定义pipeLine

```java
public class BidongComicInfoPipeLine implements PageModelPipeline {
    private BidongComicService service;
    public BidongComicInfoPipeLine(BidongComicService service){
        this.service = service;
    }
    @Override
    public void process(Object o, Task task) {
        if(o instanceof BidongComicInfo){
            service.saveComic((BidongComicInfo) o);
        }
        if(o instanceof BidongComicImg){
            service.savImg((BidongComicImg)o);
        }
        if(o instanceof BidongComicChapter){
            service.saveChapter((BidongComicChapter)o);
        }
    }
}
```

启动测试类

```java
@Test
    void start(){
        OOSpider.create(Site.me().setSleepTime(1000),
                new ConsolePageModelPipeline(),
                BidongComicInfo.class, BidongComicChapter.class)
             .addUrl("https://www.bidongmh.com/booklist")
                .thread(6).run();
    }
```

