# Spring boot 使用总结
## 注意点
1. 请求处理：
   
   前台页面中，填入 跳转url时，disPacherServlet 会寻找对应的handler，匹配方法是将 看 url的路径是否与@requestMapping 中的value一致，该url不会经过试图解析器，将 "/main.html"转换为"/main"。如果匹配，则由相关handler处理，如果没有，就进入视图解析器。

   如果没有找到对应的handler，就会经过视图解析器，判断该url是否已经做了映射，如果存在相关配置，就显示对应的映射页面；如果没有相关配置，则报错，"404"。
2. thymeleaf 中 ：预设 单选按钮的选中转态使用 "th:checked";要预设下拉列表的选中状态，使用"th:selected";对输入框预设值,使用"th:value";对展示标签进行赋值,使用"th:text".
3. 在controller中，返回的html路径 都应该在前方加一个"/",如"/emps", 目前已知 如果不加改斜杠，会报错：格式转换错误，不能把string转为 Integer,