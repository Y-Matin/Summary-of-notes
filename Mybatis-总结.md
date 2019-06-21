Mybatis的学习总结
1. 流程：首先添加 一个mybatis-config.xml文件，用于创建sqlsessionFactory，再得到sqlSession对象，通过getMapper(xxx.class)  方法得到 接口的实现类<代理>。通过调用实现类的方法得到返回的数据。
2. 在通过xml文件创建sqlsessionFactory时， 需要注意xml的路径，根据 类路径读取，而且xml文件所在目录须是resources文件夹 ，代    表里面放的是资源文件。   Resources.getResourceAsStream("mybatisConfig.xml")。
3. mybatis提供有两种编程方式： 
        ①. 直接通过sqlSession来完成对数据库操作：sqlession.selectOne("org.mybatis.employMapper.selectEmployee", 1);
        ②. 通过面向接口式编程方式操作数据库。<推荐使用这种>
            EmployeeMapper employeeMapper =sqlSession.getMapper(EmployeeMapper.class);
            Employee employee = employeeMapper.getEmployeeById(1);
3. 接口文件-employeeMapper.java ,里面存放的全是 抽象方法。
4. sql文件-employeeMapper.xml, 里面存放有各种查询数据库的sql语句。
⑴.

            <mapper namespace="com.mybatis.dao.EmployeeMapper"></mapper>
      此处的 namespace-命名空间 一定是是 对应接口文件的-引用路径，将接口文件与mapper.xml文件绑定在一起，
           
     ⑵.在sql标签中，id是sql标签的唯一标示，一定是 该接口中的函数名

                <select id="getEmployeeById" resultType="com.mybatis.bean.Employee">
                    select * from employee where employeeId = #{id}
                </select>
                ***********
                public interface EmployeeMapper {
                    Employee getEmployeeById(int id);
                }
5. sqlSession代表与数据库的一次回话
6. sqlSession与connect一样都是非线程安全，每次使用都应该重新获取新的对象
7. XXmapper接口并没有对应的实现类，但是跟对应的 xml文件的命名空间绑定后，mybatis会为这个接口生成一个代理对象。
        EmployeeMapper employeeMapper =sqlSession.getMapper(EmployeeMapper.class);
8. mybatis有两个重要的配置文件
                ⑴.mybatis-config.xml -全局配置文件,包含数据库连接池信息，事务管理器......
                ⑵.sql映射文件 只包含sql语句
9. 可以本地导入dtd 文件，其作用是对xml文件提供约束条件，本地就提供标签、属性等提示，无需联网 
10. properties标签:propertis 标签用于引入外部配置文件的内容
                属性:resource属性表示  引入在类路径下的资源
                    url属性     表示   引入网络路径 或磁盘路径
11. settings标签：控制很多全局的开关
                属性：cacheEnabled 该属性控制 映射器配置的缓存的全局开关
                    lazyLoadingEnabled 该属性控制 是否延时加载
                    ..........等
12. typeAliases 标签用于起别名，使用场景：比如在sql语句中，返回值类型会很长，会重复很多次，起别名会减少代码量，使代码看起来比较简洁
                1.在mybatis配置文件中，给一个类型其别名，通过typeAlias标签
                2.在mybatis配置文件中，给一个包及其子包中的类批量起一个默认别名<类名小写>
                    需注意一个不同包下 类名相同，此时 默认别名也是相同的
                3.为了解决上面这个问题，在批量起别名情况下，在实体类中，通过注解 给一个 类其别名 ，通过 @Alias(value) 方式
                需注意 默认别名 是不区分大小写的。
13. databaseIdProvider 标签，用来保证mybatis的移植性，应付多个数据库，给不同的数据库区别名。再在对应的 Mpper.xml文件中的sql语句标签上 添加一个 属性 databaseId, databaseId的值为 databaseIdProvider标签中对应的数据库的别名。
    
                <databaseIdProvider type="DB_VENDOR"> 
                  <property name="SQL Server" value="sqlserver"/> 
                  <property name="DB2" value="db2"/>  
                  <property name="Oracle" value="oracle" /> 
                </databaseIdProvider>

    注意一个细节：例如一个函数 在对应的mapper.xml文件中有3个对应的sql语句，第一个没有带 databaseId；第二个databaseId 代表是mysql下的sql；第三个databaseId代表oracle数据库的sql，单 当前连接环境为 mysql时，它会选出带有mysql标识的sql以及没有指定databaseId的语句，再从中选出最精确地那个sql语句。
14. mapper接口文件可以不绑定xml映射文件，可以直接在mapper接口中，在对应的方法前添加注释：

            @Select("select * from employee where employeeId = #{id} ")
            public Employee getEmployeeById(int id);

15. 在增删改操作中，mybatis支持一下类型的返回值：int、Integer、boolean. 无须在xml文件中添加返回类型，而且也不能添加返回类型。只有查才提供returnType属性。
16. 通过sqlSessionFactory.openSession(); 得到的sqlsession不能自动提交更改，需要手动提交。
       也可以通过：sqlSessionFactory.openSession(true); 这种方式 带boolean的参数 来定义是否自动提交。
17. 获取自增主键的值：
            1.useGenneratedKeys 的值设置为 true, 表明 使用自增主键获取主键值策略。
            2.keyProperty 的值 为 javaBean 中对应的 自增主键的成员变量名，表明 将自增主键的值封装给该类的这个成员变量。
18. 对于oracle数据库，不能直接设置为自增，只能通过序列的方式来获取下一个数字。
            所以要插入新的数据时，要先获取id的值。 在<insert> 标签中有一个子标签 <keySelect >这个标签可以用来获取主键的值。

            <insert id="insertAuthor">  
                <selectKey keyProperty="id" resultType="int" order="BEFORE">
                    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
                </selectKey>
                insert into Author    (id, username, password, email,bio, favourite_section)  
                values(#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
            </insert>

19. 对于函数的入参的的处理：
            ⑴.一个入参：通常是通过 #{参数名} 的方式 来实现替换， 实际上 {}中填入任何字符也同样可以正常运行。
            ⑵.多个入参：如果还是按照上面的 #{参数名}的方式 来实现替换，程序在运行时会报错：

                         Cause: org.apache.ibatis.binding.BindingException: 
                                Parameter 'id' not found. Available parameters are [arg1, arg0, param1, param2]   

                        其原因是因为： mybatis把所有的入参都封装到一个map对象中，
                                    key： param1.......paramN
                                    value: 传入的参数值
                                    #{}就是从map 中获取指定的key。
                        解决方法一： 在sql语句中 将#{参数名}的方式改为@{param1}..........@{paramN}
                        解决方法二： 在mapper接口文件中，抽象函数中 明确指定封装参数时 map的key，
                                    key：使用@Param注解指定参数名
                                    value：参数的值
                                    通过#{指定的key}去除对应的参数值。
            ⑶.POJO：如果是参数全是业务逻辑的数据模型，可以直接出入POJO。#{属性名}取出传入POJO中的属性值
            ⑷.TO(Tranfer Object)数据传输对象：参数不是属于数据模型，可以自己新建一个数据对象。
                  如：Page{
                           int pageSize;
                           int PageCurernt;
                           int PageCOunt;
                          }
            ⑸.需注意一个特殊的：但入参是一个集合类型：collection（list，set）或者是数组类型 。Mybatis会做特殊处理。
                会把出入的集合或者数组封装在Map中
                key: collection入参使用 "collection"作为key， 如果是list也可以使用"list"作为key
                      数组 使用"array"作为key
            推荐使用方法 2、3、4。便于确定参数的具体含义。
20. mybatis参数处理源码解析：
====================ParamNameResolver的构造函数,关注 name  ========================================>     
public ParamNameResolver(Configuration config, Method method) {
final Class<?>[] paramTypes = method.getParameterTypes();
final Annotation[][] paramAnnotations = method.getParameterAnnotations();
final SortedMap<Integer, String> map = new TreeMap<>();
int paramCount = paramAnnotations.length;
// get names from @Param annotations
for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
String name = null;
//1.如果 参数 是含有注解的 ，就将 注解的value 赋值给name 
for (Annotation annotation : paramAnnotations[paramIndex]) {
    if (annotation instanceof Param) {
    hasParamAnnotation = true;
    name = ((Param) annotation).value();
    break;
    }
}
if (name == null) {  
    //2.config 是指全局配置中的 useActualParamName 属性 作用是：将方法中实际声明的变量名作为 参数名。
    if (config.isUseActualParamName()) {
    name = getActualParamName(method, paramIndex);
    }
    if (name == null) {

    // use the parameter index as the name ("0", "1", ...)
    // 3.name = 1.....N
    name = String.valueOf(map.size());
    }
}
map.put(paramIndex, name);
}
names = Collections.unmodifiableSortedMap(map);
}
=======================解析总结=========================================>        
        map：<key,value> 含义： 含顺序的参数名
            key：是 Integer类型，代表 参数名的 索引 ，如：0,1... 
            value：name的值
                 ⑴. 如果是含有注解的 ，将注解里面的 值 作为 赋值给name，
                 ⑵. 如果不含有注解：
                        ①. 查看全局变量 ：useActualParamName<jdk 1.8> 是否已配置， name = 参数名
                        ②.name = map.size  相当于当前元素的索引
                {0=id,1=name,2=2......}
=========================一下是参数的值与对应的参数名绑定的处理========================================>
public Object getNamedParams(Object[] args) {
    //args 里面的值就是具体入参的值，可能包含 基本类型和对象类型
    //names 是一个Map类型，其值来源为上面构造方法。 形如：{0=id,1=name,2=2......}
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
      return null;
    } else if (!hasParamAnnotation && paramCount == 1) {
        //1.如果只有一个入参，就直接把该值返回，并没有封装在map中 
      return args[names.firstKey()];
    } else {
      final Map<String, Object> param = new ParamMap<>();
      int i = 0;
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
          //names集合中的value作为key，names集合中的key只是作为args数组取值的参考
        param.put(entry.getValue(), args[entry.getKey()]);
        // add generic param names (param1, param2, ...)
        //GENERIC_NAME_PREFIX:param
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        // ensure not to overwrite parameter named with @Param
        //同时也额外提供paramN的参数名。
        //效果：#{指定的key} 或者 #{paraN} 
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
===================================解析总结=====================================================================>
        //args 里面的值就是具体入参的值，可能包含 基本类型和对象类型
        //names 是一个Map类型，其值来源为上面构造方法。 形如：{0=id,1=name,2=2......}
            //1.如果只有一个入参，就直接把该值返回，并没有封装在map中 
            //names集合中的value作为key，names集合中的key只是作为args数组取值的参考
            //同时也额外提供paramN的参数名。
            //效果：#{指定的key} 或者 #{paraN} 
=================================================================================================================>
21. #{} 与${}的区别和相似之处：
            相同点：#{}和${}都可以从参数中取值，功能是一样的。
            不同： #{}采用了预编译的形式，在sql语句中采用了 "?" 占位符， proparedStatment，可以防止sql注入
                  ${}采用了直接替换的方式。会存在安全隐患。
            使用方法：
                  大多数情况下，我们都是用#{}从参数中取值
                  原生jdbc不支持占位符的地方，我们可以用${}来进行取值。例如:表名.....
22. select标签：查询语句如果返回一个list，select标签中的resultType不是list类型，而是 list中元素的类型。
23. 批量插入：  insert + foreach  动态sql 
        <!--批量导入list 的使用 ，结合foreach 语句-->
        <insert id="insertEmployeeList" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="employeeId">
            insert into employee(departmentId,positionId,employeeName,employeeSex,employeeTel,employeeAddress,level)
            values
            <foreach collection="list" index="index" item="item" separator=",">
                (
                #{item.departmentId},
                #{item.positionId},
                #{item.employeeName},
                #{item.employeeSex},
                #{item.employeeTel},
                #{item.employee_address},
                #{item.level}
                )
            </foreach>
        </insert>
    

