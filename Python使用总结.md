Python基础知识总结归纳

1.字符串: 判断 字符串 之间是否是 包含 关系 
        ⑴.in    ==> if str1 in str2:
        ⑵.find  ==> if str1.find(str2)>-1:   
        ⑶.index ==> if str1.index(str2)>-1:    #此方法与find作用类似，也是找到字符起始的序号
2.字符串截取
        str = ‘0123456789’
        print str[0:3] #截取第一位到第三位的字符
        print str[:] #截取字符串的全部字符
        print str[6:] #截取第七个字符到结尾
        print str[:-3] #截取从头开始到倒数第三个字符之前
        print str[2] #截取第三个字符
        print str[-1] #截取倒数第一个字符
        print str[::-1] #创造一个与原字符串顺序相反的字符串
        print str[-3:-1] #截取倒数第三位与倒数第一位之前的字符
        print str[-3:] #截取倒数第三位到结尾
        print str[:-5:-3] #逆序截取，具体啥意思没搞明白？