# Jquery 总结
1. 删除两个标签之间的数据: 
    
        $('#previe').nextUntil("#next").html("");
    此方法只能将标签的text属性去掉，不能讲该标签真正意义的remove。
2. 在js中创建html标签。使用val()方法给标签的value属性赋值，使用text()方法给标签的文本赋值,使用addClass()方法给标签添加class。
    
        var dept = $("<option></option>")
        .val(item.deptId).text(item.deptName);
3. 将创建的html标签动态的添加到现有的html代码代码中，显示该标签。
   使用append()方法，将 dept 标签添加到id为 DeptId 标签的 内部尾部。
   
        $("#DeptId").append(dept);
4. jquery 中层级选择器有4个， 空格，大于号，加号，波浪号
   1. 空格：在给定的祖先元素下匹配所有的后代元素，包括 儿子，孙子及所有后继。
        
            $("#div2 span") //表示id为div2标签下所有的span标签
   2. 大于号：在给定的父元素下匹配所有的子元素，表示该标签的儿子这一级。

            $("#div2 > span")
   3. 加号：匹配所有紧接在 prev 元素后的 next 元素。同桌关系。

            $("#div2 + span")
   4. 波浪号：匹配 prev 元素之后的所有 siblings 元素,兄弟选择器，只能从当前开始，同级往后。

            $("#div2 ~ div")
5. jquery中使用这则表达式：格式要求：收尾要用"/"标识，不能将表达式放在双引号中。
   
        <!-- 用于校验用户名  -->
        var regName = /(^[a-zA-Z0-9_-]{6,16}$)|(^[\u2E80-\u9FFF]+$)/
6. jquery 中使用 this时，需注意 将this 包含在"$()"中,否则会报错，比如：“this.attr is not a function”。

        $(this).attr("ajax-value") 
7. 在js中 将表单数据清空，可以使用如下：使用的时DOM中的reset方法。
        
        $("#employee_add_from")[0].reset();
8. 在js中 通过this，寻找附近的其他标签：parent()寻找该标签的父标签，find(ele)在当前标签包含内容中寻找目标标签，first()方法返回第一个符合条件的标签，text()返回该标签的文本信息。

        $(this).parent().find("td").first().text()