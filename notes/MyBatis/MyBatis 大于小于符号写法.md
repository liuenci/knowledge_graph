<a name="ZhN7R"></a>
# 第一种写法
​

原符号       <        <=      >       >=       &        '        "<br />替换符号    &lt;    &lt;=   &gt;    &gt;=   &amp;   &apos;  &quot;<br />​

例如：sql如下：<br />​

create_date_time &gt;= #{startTime} and  create_date_time &lt;= #{endTime}<br />​

## 第二种写法<br />大于等于<br /><![CDATA[ >= ]]><br />小于等于<br /><![CDATA[ <= ]]><br />例如：sql如下：<br />create_date_time <![CDATA[ >= ]]> #{startTime} and  create_date_time <![CDATA[ <= ]]> #{endTime}
