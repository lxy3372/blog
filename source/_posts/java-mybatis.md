title: "MyBatis动态SQL优化方案之@UpdateProvider"
date: 2015-03-31 06:09:31
tags: [Java,MyBatis,UpdateProvider]
description: MyBatis动态SQL优化方案之@UpdateProvider

---

   最近开始学习纯注解形式的MyBatis，我很是喜爱（从动态的PHP转到Java对配置XML这种配置的从个人感情上很是厌恶,不喜勿喷哈），网上也有不少教程，但普遍性的就都是`@Select`、`@Update`、`@Delete`以及`@SelectKey`的一些简单的用法的介绍。
   
   当然这些以满足了简单的SQL逻辑。但是当遇到多个简单的更新不同的字段的时候，便显得显得冗余了。一般的解决的办法是，不断重载一个更新方法，更新简单的字段。还有更甚的是一次更新全部字段。其实如果用Xml配置的话，是有相关的语法通过判断字段是否为空来判断是否需要更新的。
类似： 
```Java
...
<select id="findByPage" parameterType="com.user.util.PageUtil" resultType="com.user.dao.UserDaoImpl" >
		select * from user where 1 = 1 
		<if email="email!= "" || email!= null">
			    and ename = #{user.email}
		</if>
		...
```

其实注解形式通过`@UpdateProvider`也是能够解决这个问题的。
<!--more-->
当使用Mapper时：
```Java
public interface UserMapper{
	
	@Select("SELECT * FROM user WHERE user_id = #{id}")
	User getUserById(@Param("id") long id);
	
	@UpdateProvider(type = UserSql.class, method = "update")
	int update(User user);
	
}



//建立对应的UserSql.java文件：

public class UserSql{

	public String update(final User user){
		return new SQL(){
			{
				UPDATE("user");
				
				//通过条件 判断是否需要更新该字段
				if (StringUtils.isNotBlank(user.getName())){
					SET("name = #{name}")
				}
				
				if (StringUtils.isNotBlank(user.getNickName())){
					SET("name = #{nickName}")
				}
				
				if (StringUtils.isNotBlank(user.getEmail())){
					SET("name = #{email}")
				}
				
				...
				
				WHERE("id = #{id}")
				
			}
		}
	}

}

```

这样基本上就实现了通过对字段的判断实现动态的拼接了。