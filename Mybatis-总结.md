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
        ⑴.mybatis-config.xml 包含数据库连接池信息，事务管理器......
        ⑵.sql映射文件 只包含sql语句
        