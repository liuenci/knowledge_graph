```java
<where>  
        <if test="username !=null ">  
            u.username LIKE CONCAT(CONCAT('%', #{username, jdbcType=VARCHAR}),'%')  
            </if>  
        <if test="sex != null and sex != '' ">  
            AND u.sex = #{sex, jdbcType=INTEGER}  
</if>  
        <if test="birthday != null ">  
            AND u.birthday = #{birthday, jdbcType=DATE}  
</if> 
    </where>
```


