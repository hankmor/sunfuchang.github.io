---
layout: post
category : technology
tagline: 
tags : [mybatis, SQL, 动态]
excerpt : 
title_cn: mybatis——动态SQL
---
{% include JB/setup %}

MyBatis的强大特性之一便是它的动态<code>SQL</code>能力。如果你有使用JDBC或其他类似框架的经验，你就能体会到根据不同条件拼接<code>SQL</code>字符串有多么痛苦。拼接的时候要确保不能忘了必要的空格，还要注意省掉列名列表最后的逗号。利用动态<code>SQL</code>这一特性可以彻底摆脱这种痛苦。

通常使用动态SQL不可能是独立的一部分,MyBatis当然使用一种强大的动态<code>SQL</code>语言来改进这种情形,这种语言可以被用在任意映射的<code>SQL</code>语句中。

动态<code>SQL</code>元素和使用<code>JSTL</code>或其他相似的基于<code>XML</code>的文本处理器相似。在MyBatis之前的版本中,有很多的元素需要来了解。MyBatis3大大提升了它们,现在用不到原先一半的元素就能工作了。MyBatis 采用功能强大的基于<code>OGNL</code>的表达式来消除其他元素。

## if, choose (when, otherwise), trim (where, set), foreach, if

动态<code>SQL</code>通常要做的事情是有条件地包含<code>where</code> 子句的一部分。比如:
{% highlight xml %}
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  WHERE state = ‘ACTIVE’ 
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
{% endhighlight %}

这条语句提供了一个可选的文本查找类型的功能。如果没有传入“title”，那么所有处于“ACTIVE”状态的BLOG都会返回；反之若传入了“title”，那么就会把相近“title”的BLOG返回（就这个例子而言，细心的读者会发现其中的参数值是可以包含一些掩码或通配符的）。

如果想可选地通过“title”和“author”两个条件搜索该怎么办呢？首先，改变语句的名称让它更具实际意义；然后只要加入另一个条件即可。

{% highlight xml %}
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’ 
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
{% endhighlight %}

## choose, when, otherwise

有些时候，我们不想用到所有的条件语句，而只想从中择其一二。针对这种情况，MyBatis 提供了<code>choose</code>元素，它有点像<code>Java</code>中的<code>switch</code>语句。

还是上面的例子，但是这次变为提供了“title”就按“title”查找，提供了“author”就按“author”查找，若两者都没有提供，就返回所有符合条件的BLOG（实际情况可能是由管理员策略地选出BLOG列表，而不是返回大量无意义的随机结果）。

{% highlight xml %}
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
{% endhighlight %}

## trim, where, set

前面几个例子已经合宜地解决了一个臭名昭著的动态<code>SQL</code>问题。现在考虑回到“if”示例，这次我们将“ACTIVE=1”也设置成动态的条件，看看会发生什么。

{% highlight xml %}
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  WHERE 
  <if test="state != null">
    state = #{state}
  </if> 
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
{% endhighlight%}

如果这些条件没有一个能匹配上将会怎样？最终这条 SQL 会变成这样：

{% highlight sql %}
SELECT * FROM BLOG WHERE
{% endhighlight %}

这会导致查询失败。如果仅仅第二个条件匹配又会怎样？这条 SQL 最终会是这样:

{% highlight sql %}
SELECT * FROM BLOG WHERE AND title like ‘someTitle’
{% endhighlight %}

这个查询也会失败。这个问题不能简单的用条件句式来解决，如果你也曾经被迫这样写过，那么你很可能从此以后都不想再这样去写。

MyBatis有一个简单的处理，这在90%的情况下都会有用。而在不能使用的地方，你可以自定义处理方式来令其正常工作。一处简单的修改就能得到想要的效果:

{% highlight xml %}
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
{% endhighlight %}

<code>where<code>元素知道只有在一个以上的<code>if</code>条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，<code>where</code>元素也知道如何将他们去除。

如果<code>where</code>元素没有按正常套路出牌，我们还是可以通过自定义<code>trim</code>元素来定制我们想要的功能。比如，和<code>where</code>元素等价的自定义<code>trim</code>元素为：

{% highlight xml %}
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
{% endhighlight %}

<code>prefixOverrides</code>属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它带来的结果就是所有在<code>prefixOverrides</code>属性中指定的内容将被移除，并且插入<code>prefix</code>属性中指定的内容。

类似的用于动态更新语句的解决方案叫做<code>set</code>。<code>set</code>元素可以被用于动态包含需要更新的列，而舍去其他的。比如：

{% highlight xml %}
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
{% endhighlight %}

这里，<code>set</code>元素会动态前置<code>SET</code>关键字，同时也会消除无关的逗号，因为用了条件语句之后很可能就会在生成的赋值语句的后面留下这些逗号。

若你对等价的自定义<code>trim</code>元素的样子感兴趣，那这就应该是它的真面目：

{% highlight xml %}
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
{% endhighlight %}

注意这里我们忽略的是后缀中的值，而又一次附加了前缀中的值。

## foreach

动态<code>SQL</code>的另外一个常用的必要操作是需要对一个集合进行遍历，通常是在构建<code>IN</code>条件语句的时候。比如：

{% highlight xml %}
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
{% endhighlight %}

<code>foreach</code>元素是非常强大的，它允许你指定一个集合，声明可以用在元素体内的集合项和索引变量。它也允许你指定开闭匹配的字符串以及在迭代之间放置分隔符。这个元素是很智能的，因此它不会偶然地附加多余的分隔符。

**注意** 你可以将一个<code>List</code>实例或者数组作为参数对象传给MyBatis，当你这么做的时候，<code>MyBatis</code>会自动将它包装在一个 Map 中并以名称为键。<code>List</code>实例将会以“list”作为键，而数组实例的键将是“array”。

到此我们已经完成了涉及<code>XML</code>配置文件和<code>XML</code>映射文件的讨论。下一部分将详细探讨<code>Java</code>API，这样才能从已创建的映射中获取最大利益。

## bind
<code>bind</code>元素可以从<code>OGNL</code>表达式中创建一个变量并将其绑定到上下文。比如：

{% highlight xml %}
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
{% endhighlight %}

## Multi-db vendor support

一个配置了“_databaseId”变量的<code>databaseIdProvider</code>对于动态代码来说是可用的，这样就可以根据不同的数据库厂商构建特定的语句。比如下面的例子：

{% highlight xml %}
<insert id="insert">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    <if test="_databaseId == 'oracle'">
      select seq_users.nextval from dual
    </if>
    <if test="_databaseId == 'db2'">
      select nextval for seq_users from sysibm.sysdummy1"
    </if>
  </selectKey>
  insert into users values (#{id}, #{name})
</insert>
{% endhighlight %}

## Pluggable Scripting Languages For Dynamic SQL

MyBatis从3.2开始支持可插拔的脚本语言，因此你可以在插入一种语言的驱动（language driver）之后来写基于这种语言的动态<code>SQL</code>查询。

可以通过实现下面接口的方式来插入一种语言：

{% highlight java %}
public interface LanguageDriver {
  ParameterHandler createParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql);
  SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType);
  SqlSource createSqlSource(Configuration configuration, String script, Class<?> parameterType);
}
{% endhighlight %}

一旦有了自定义的语言驱动，你就可以在<code>mybatis-config.xml</code>文件中将它设置位默认语言：

{% highlight xml %}
<typeAliases>
  <typeAlias type="org.sample.MyLanguageDriver" alias="myLanguage"/>
</typeAliases>
<settings>
  <setting name="defaultScriptingLanguage" value="myLanguage"/>
</settings>
{% endhighlight %}

除了设置默认语言，你也可以针对特殊的语句指定特定语言，这可以通过如下的<code>lang</code>属性来完成：

{% highlight xml %}
<select id="selectBlog" lang="myLanguage">
  SELECT * FROM BLOG
</select>
{% endhighlight %}

或者在你正在使用的映射中加上注解<code>@Lang</code>来完成：

{% highlight java %}
public interface Mapper {
  @Lang(MyLanguageDriver.class)
  @Select("SELECT * FROM BLOG")
  List<Blog> selectBlog();
}
{% endhighlight %}

**注意** 可以将Apache Velocity作为动态语言来使用，更多细节请参考MyBatis-Velocity项目。

你前面看到的所有<code>xml</code>标签都是默认MyBatis语言提供的，它是由别名为<code>xml</code>语言驱动器<code>org.apache.ibatis.scripting.xmltags.XmlLanguageDriver</code>驱动的。