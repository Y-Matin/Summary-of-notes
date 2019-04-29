Mybatis的学习总结
    1.流程：首先添加 一个mybatis-config.xml文件，用于创建sqlsessionFactory，再得到sqlSession对象，通过getMapper(xxx.class)  方法得到 接口的实现类<代理>。通过调用实现类的方法得到返回的数据。
    2.在通过xml文件创建sqlsessionFactory时， 需要注意xml的路径，根据 类路径读取，而且xml文件所在目录须是resources文件夹 ，代    表里面放的是资源文件。   Resources.getResourceAsStream("mybatisConfig.xml")。
    3.mybatis提供有两种编程方式： 
        ①.直接通过sqlSession来完成对数据库操作：sqlession.selectOne("org.mybatis.employMapper.selectEmployee", 1);
        ②.通过面向接口式编程方式操作数据库。<推荐使用这种>
            EmployeeMapper employeeMapper =sqlSession.getMapper(EmployeeMapper.class);
            Employee employee = employeeMapper.getEmployeeById(1);
    3.接口文件-employeeMapper.java ,里面存放的全是 抽象方法。
    4.sql文件-employeeMapper.xml, 里面存放有各种查询数据库的sql语句，
            ⑴.<mapper namespace="com.mybatis.dao.EmployeeMapper"></mapper> 此处的 namespace-命名空间 一定是是 对应接口文件的-引用路径，将接口文件与mapper.xml文件绑定在一起，
            ⑵.在sql标签中，id是sql标签的唯一标示，一定是 该接口中的函数名
                <select id="getEmployeeById" resultType="com.mybatis.bean.Employee">
                    select * from employee where employeeId = #{id}
                </select>
                ***********
                public interface EmployeeMapper {
                    Employee getEmployeeById(int id);
                }
    5.sqlSession代表与数据库的一次回话
    6.sqlSession与connect一样都是非线程安全，每次使用都应该重新获取新的对象
    7.XXmapper接口并没有对应的实现类，但是跟对应的 xml文件的命名空间绑定后，mybatis会为这个接口生成一个代理对象。
        EmployeeMapper employeeMapper =sqlSession.getMapper(EmployeeMapper.class);
    8.mybatis有两个重要的配置文件
                ⑴.mybatis-config.xml -全局配置文件,包含数据库连接池信息，事务管理器......
                ⑵.sql映射文件 只包含sql语句
    9.可以本地导入dtd 文件，其作用是对xml文件提供约束条件，本地就提供标签、属性等提示，无需联网 
    10.properties标签:propertis 标签用于引入外部配置文件的内容
                属性:resource属性表示  引入在类路径下的资源
                    url属性     表示   引入网络路径 或磁盘路径
    11.settings标签：控制很多全局的开关
                属性：cacheEnabled 该属性控制 映射器配置的缓存的全局开关
                    lazyLoadingEnabled 该属性控制 是否延时加载
                    ..........等
    12.typeAliases 标签用于起别名，使用场景：比如在sql语句中，返回值类型会很长，会重复很多次，起别名会减少代码量，使代码看起来比较简洁
                1.在mybatis配置文件中，给一个类型其别名，通过typeAlias标签
                2.在mybatis配置文件中，给一个包及其子包中的类批量起一个默认别名<类名小写>
                    需注意一个不同包下 类名相同，此时 默认别名也是相同的
                3.为了解决上面这个问题，在批量起别名情况下，在实体类中，通过注解 给一个 类其别名 ，通过 @Alias(value) 方式
                需注意 默认别名 是不区分大小写的。
    13.databaseIdProvider 标签，用来保证mybatis的移植性，应付多个数据库，给不同的数据库区别名。再在对应的 Mpper.xml文件中的sql语句标签上 添加一个 属性 databaseId, databaseId的值为 databaseIdProvider标签中对应的数据库的别名。
                <databaseIdProvider type="DB_VENDOR"> 
                  <property name="SQL Server" value="sqlserver"/> 
                  <property name="DB2" value="db2"/>  
                  <property name="Oracle" value="oracle" /> 
                </databaseIdProvider>
        注意一个细节：例如一个函数 在对应的mapper.xml文件中有3个对应的sql语句，第一个没有带 databaseId；第二个databaseId 代表是mysql下的sql；第三个databaseId代表oracle数据库的sql，单 当前连接环境为 mysql时，它会选出带有mysql标识的sql以及没有指定databaseId的语句，再从中选出最精确地那个sql语句。
    14.mapper接口文件可以不绑定xml映射文件，可以直接在mapper接口中，在对应的方法前添加注释：
            @Select("select * from employee where employeeId = #{id} ")
            public Employee getEmployeeById(int id);
    