# Jquery 总结
1. 删除两个标签之间的数据: 
    
        $('#previe').nextUntil("#next").html("");
    此方法只能将标签的text属性去掉，不能讲该标签真正意义的remove。