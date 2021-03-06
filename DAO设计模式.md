###一、分层

- **分层实际上是指面向对象的分层，除了数据层操作的是数据外，其他层操作的都是对象**

- **程序分层最重要的就是上一层调用下一层【又分为开发标准（接口）和实现】提供的服务，而不需要知道下一层的具体实现，类比计算机网络的分层结构**
####1、总程序分层

![1553331686131](C:\Users\AlexanderBai\AppData\Roaming\Typora\typora-user-images\1553331686131.png)

####2、业务层划分

![1553423107218](C:\Users\AlexanderBai\AppData\Roaming\Typora\typora-user-images\1553423107218.png)

**业务层可能只有一个业务或可能在总业务下有多个子业务**

####3、项目目录结构

![1553423764593](C:\Users\AlexanderBai\AppData\Roaming\Typora\typora-user-images\1553423764593.png)

**DAO有一个特别之处就是每一个包、接口和类等命名都有特定的规则，以便于实现分层开发**

####4、类结构视图

![1553443076993](C:\Users\AlexanderBai\AppData\Roaming\Typora\typora-user-images\1553443076993.png)

### 二、数据库连接

#### 1、VO类（简单对象类）

- **负责数据的传输与包装：**VO类的对象就是要传递数据的对象  

- **VO类的规则**

  > - 类名称与表名称一致
  >
  > - 便于日后类的操作，必须实现Serializable接口
  >
  > - 不允许出现基本数据类型，只能使用包装类
  >
  > - 类之中的所有属性必须包装（私有属性），必须都编写setter和getter方法
  >
  > - 类之中一定要有一个无参构造器
  >
  >   ~~~java
  >   package cn.alexanderbai.vo;
  >   
  >   import java.io.Serializable;
  >   import java.util.Date;
  >   
  >   /**
  >    * @Author AlexanderBai
  >    * @data 2019/3/23 20:16
  >    */
  >   public class Emp implements Serializable {
  >       private Integer empNo;
  >       private String empName;
  >       private String jod;//岗位
  >       private Double sal;//公资
  >       private Date hireDate;//聘用日期
  >       private String comment;//描述
  >   
  >       public Emp() {
  >       }
  >       
  >   
  >       public void setEmpNo(Integer empNo) {
  >           this.empNo = empNo;
  >       }
  >   
  >       public void setEmpName(String empName) {
  >           this.empName = empName;
  >       }
  >   
  >       public void setJod(String jod) {
  >           this.jod = jod;
  >       }
  >   
  >       public void setComment(String comment) {
  >           this.comment = comment;
  >       }
  >   
  >       public void setSal(Double sal) {
  >           this.sal = sal;
  >       }
  >   
  >       public void setHireDate(Date hireDate) {
  >           this.hireDate = hireDate;
  >       }
  >   
  >       public Integer getEmpNo() {
  >           return empNo;
  >       }
  >   
  >       public String getEmpName() {
  >           return empName;
  >       }
  >   
  >       public String getJod() {
  >           return jod;
  >       }
  >   
  >       public String getComment() {
  >           return comment;
  >       }
  >   
  >       public Double getSal() {
  >           return sal;
  >       }
  >   
  >       public Date getHireDate() {
  >           return hireDate;
  >       }
  >   }
  >   ~~~

####2、DatabaseConnection类

- **负责数据库的连接和关闭**

  ```java
  package cn.alexanderbai.dbc;
  
  import java.sql.Connection;
  import java.sql.DriverManager;
  import java.sql.SQLException;
  
  /**
   * 主要功能是负责数据库的连接与关闭
   * @Author AlexanderBai
   * @data 2019/3/23 20:28
   */
  public class DatabaseConnection {
      private static final String PASSWORD = "ROOT";
      private static final String URL="jdbc:MySQL://127.0.0.1:3306/scott?characterEncoding=UTF-8";
      private String USER ="root";
      private Connection connection=null;
  
      public DatabaseConnection(){
          try {
              //1、加载驱动
              Class.forName("com.mysql.jdbc.Driver");
              //2、获取连接对象
              this.connection= DriverManager.getConnection(URL, USER, PASSWORD);
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          } catch (SQLException e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 获取数据库的一个连接对象，这个对象在构造方法中取得
       * @return Connection 接口对象
       */
      public Connection getConnection() {
          return this.connection;
      }
  
      /**
       * 关闭数据库
       */
      public void close() {
          if (this.connection != null) {
              try {
                  connection.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```

### 三、数据层

#### 1、定义数据层开发标准

```java
package cn.alexanderbai.dao;

import cn.alexanderbai.vo.Emp;

import java.sql.SQLException;
import java.util.List;
import java.util.Set;

/**
 * @Author AlexanderBai
 * @data 2019/3/23 20:57
 */
public interface IEmpDAO {

    /**
     * 数据库的增加操作，执行insert语句
     * @param vo 包含了要增加的数据
     * @return 若增加成功返回true，否则返回false
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public boolean doCreate(Emp vo) throws Exception;

    /**
     * 数据库的修改操作，执行UPDATE语句，本次的修改将根据ID将所有数据进行变更
     * @param vo 包含了要修改的数据
     * @return 若修改成功，则返回true，否则返回false
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public boolean doUpdate(Emp vo) throws Exception;

    /**
     *执行删除操作，在删除前需要根据删除的编号，拼凑出SQL语句
     * @param ids 所有要删除的编号数据
     * @return 删除成功返回true，否则返回false
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public boolean doRemove(Set<Integer> ids) throws Exception;

    /**
     * 根据编号查询出表一行的完整信息，并将返回结果填充到VO对象中
     * @param id 要查询的数据编号
     * @return 如果查询到则将内容以VO对象的形式返回，如果没有数据在返回null
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public Emp findById(Integer id) throws Exception;

    /**
     * 查询数据表中的所有数据，每行通过VO类包装，而后通过List保存多个返回结果
     * @return 全部的查询数据行，如果没有返回则集合的长度为0（size==0）
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public List<Emp> findAll() throws Exception;

    /**
     * 分页进行数据表的模糊查询操作，每行通过VO类包装，而后通过List保存多个返回结果
     * @param column 要模糊查询的数据列
     * @param KeyWord 要进行查询的关键字
     * @param currentPage 当前所在页
     * @param lineSize 每页显示的数据行数
     * @return 全部的查询数据行，如果没有数据则返回集合的长度为0（size==0）
     *@throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public List<Emp> findAllSplit(String column, String KeyWord, Integer currentPage, Integer lineSize) throws Exception;

    /**
     * 使用Count()函数统计数据表中符合查询要求的数据量
     * @param cloumn 要模糊查询的数据列
     * @param keyWord 要进行查询的关键字
     * @return 返回Count()的统计结果，如果没有数据则返回的结果为0
     * @throws Exception 若数据库没有连接则出现NullPointerException,如果SQL语句初五则出现SQLException
     */
    public Integer getAllCount(String cloumn, String keyWord) throws Exception;
}
```



#### 2、定义数据层开发标准的实现类

```java
package cn.alexanderbai.dao;

import cn.alexanderbai.vo.Emp;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

/**
 * @Author AlexanderBai
 * @data 2019/3/23 21:26
 */
public class EmpDAOImpl implements IEmpDAO {

    private Connection connection; //数据库连接对象
    private PreparedStatement preparedStatement;//数据库操作对象

    /**
     * 实例化数据层子类对象，同时传入一个数据库连接对象
     * @param connection Connection连接对象，如果为null表示数据库没有打开
     */
    public EmpDAOImpl(Connection connection) {
        this.connection = connection;
    }


    @Override
    public boolean doCreate(Emp vo) throws Exception {
        String sql = "insert into emp(empNo,empName,job,sal,hireDate,comment)values(?,?,?,?,?,?)";
        this.preparedStatement=this.connection.prepareStatement(sql);
        this.preparedStatement.setInt(1,vo.getEmpNo());
        this.preparedStatement.setString(2,vo.getEmpName());
        this.preparedStatement.setString(3,vo.getJod());
        this.preparedStatement.setDouble(4,vo.getSal());
        this.preparedStatement.setDate(5,new java.sql.Date(vo.getHireDate().getTime()));
        this.preparedStatement.setString(6,vo.getComment());
        return this.preparedStatement.executeUpdate()>0;
    }

    @Override
    public boolean doUpdate(Emp vo) throws Exception {
        String sql = "update emp set empName=?,job=?,sal=?,hireDate=?,comment=? where empNo=?";
        this.preparedStatement = this.connection.prepareStatement(sql);

        this.preparedStatement.setString(1,vo.getEmpName());
        this.preparedStatement.setString(2,vo.getJod());
        this.preparedStatement.setDouble(3,vo.getSal());
        this.preparedStatement.setDate(4,new java.sql.Date(vo.getHireDate().getTime()));
        this.preparedStatement.setString(5,vo.getComment());
        this.preparedStatement.setInt(6,vo.getEmpNo());
        return this.preparedStatement.executeUpdate()>0;
    }

    @Override
    public boolean doRemove(Set<Integer> ids) throws Exception {
        StringBuffer sql = new StringBuffer();
        sql.append("delete from emp where empNO in(");
        Iterator<Integer> iterator = ids.iterator();
        while (iterator.hasNext()) {
            sql.append(iterator.next()).append(",");
        }
        sql.delete(sql.length() - 1, sql.length());//删除掉最后的“，”
        sql.append(")");
        this.preparedStatement=this.connection.prepareStatement(sql.toString());
        return this.preparedStatement.executeUpdate()==ids.size();
    }

    @Override
    public Emp findById(Integer id) throws Exception {
        Emp vo=null;
        String sql="select empNo,empName,job,sal,hireDate,comment from emp where empNo=?";
        this.preparedStatement=this.connection.prepareStatement(sql);
        this.preparedStatement.setInt(1,id);
        ResultSet resultSet=this.preparedStatement.executeQuery();
        if (resultSet.next()) {
            vo = new Emp();
            vo.setEmpNo(resultSet.getInt(1));
            vo.setEmpName(resultSet.getString(2));
            vo.setJod(resultSet.getString(3));
            vo.setSal(resultSet.getDouble(4));
            vo.setHireDate(resultSet.getDate(5));
            vo.setComment(resultSet.getString(6));
        }
        return vo;
    }

    @Override
    public List<Emp> findAll() throws Exception {
        List<Emp> all = new ArrayList<>();
        String sql = "select empNo,empName,job,sal,hireDate,comment from emp";
        this.preparedStatement=this.connection.prepareStatement(sql);
        ResultSet resultSet=this.preparedStatement.executeQuery();
        while (resultSet.next()) {
            Emp vo = new Emp();
            vo.setEmpNo(resultSet.getInt(1));
            vo.setEmpName(resultSet.getString(2));
            vo.setJod(resultSet.getString(3));
            vo.setSal(resultSet.getDouble(4));
            vo.setHireDate(resultSet.getDate(5));
            vo.setComment(resultSet.getString(6));
            all.add(vo);
        }
        return all;
    }

    @Override
    public List<Emp> findAllSplit(String column, String keyWord, Integer currentPage, Integer lineSize) throws Exception {
        List<Emp> all = new ArrayList<>();
        String sql = "select * from"
                + " ( select empNo,empName,job,sal,hireDate,comment,rownum rn"
                + "from emp where " + column + " like ? and rownum<=?)temp "
                + "where temp.rn>?";//这个SQL语句是Oracle语法，单元测试中无法运行
        this.preparedStatement=this.connection.prepareStatement(sql);
        this.preparedStatement.setString(1,"%"+keyWord+"%");
        this.preparedStatement.setInt(2,currentPage*lineSize);
        this.preparedStatement.setInt(3,(currentPage-1)*lineSize);
        ResultSet resultSet=this.preparedStatement.executeQuery();
        while (resultSet.next()) {
            Emp vo = new Emp();
            vo.setEmpNo(resultSet.getInt(1));
            vo.setEmpName(resultSet.getString(2));
            vo.setJod(resultSet.getString(3));
            vo.setSal(resultSet.getDouble(4));
            vo.setHireDate(resultSet.getDate(5));
            vo.setComment(resultSet.getString(6));
            all.add(vo);
        }
        return all;
    }

    @Override
    public Integer getAllCount(String cloumn, String keyWord) throws Exception {
        String sql = "select COUNT(*) from emp where" + cloumn + "like ?";
        this.preparedStatement=this.connection.prepareStatement(sql);
        this.preparedStatement.setString(1,"%"+keyWord+"%");
        ResultSet resultSet=this.preparedStatement.executeQuery();
        if (resultSet.next()) {
            return resultSet.getInt(1);
        }
        return null;
    }
}
```



#### 3、定义DAO工厂类

```java
package cn.alexanderbai.factory;

import cn.alexanderbai.dao.EmpDAOImpl;
import cn.alexanderbai.dao.IEmpDAO;

import java.sql.Connection;

/**
 *定义DAO工厂类
 * @Author AlexanderBai
 * @data 2019/3/23 21:28
 */
public class DAOFactory {
    /**
     * @param connection Connection连接对象，如果为null表示数据库没有打开
     * @return 生成一个EmpDAOImpl对象并返回
     */
    public static IEmpDAO getIEmpDAOInstance(Connection connection) {
        return new EmpDAOImpl(connection);
    }
}
```



### 四、业务层

####1、定义业务层开发标准

```java
package cn.alexanderbai.service;

import cn.alexanderbai.vo.Emp;

import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * 业务层标准
 * @Author AlexanderBai
 * @data 2019/3/23 21:40
 */
public interface IEmpService {

    /**
     * 实现雇员数据的增加操作，在本操作之前需要使用IEmpDAO接口中的如下方法：<br>
     *     <li>首先利用IEmpDAO.findById()方法判断要增加的雇员编号是否存在。</li>
     *     <li>如果雇员不存在则使用IEmpDAO.doCreat()方法保存雇员信息</li>
     * @param vo 包含了要增加数据的VO类对象
     * @return 数据增加成功返回true，否则返回false
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public boolean insert(Emp vo) throws Exception;

    /**
     * 实现数据的修改操作，调用的是IEmpDAO.doUpdate()方法，此操作属于全部修改
     * @param vo 包含要修改的所有信息
     * @return 修改成功返回true，否则返回false
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public boolean update(Emp vo) throws Exception;

    /**
     * 实现数据的批量删除操作，在本操作中需要执行如下调用:<br>
     *     <li>需要判断要删除数据传入的集合是否为空（判断null以及size())</li>
     *     <li>如果确定有删除的数据，则调用IEmpDAO.doRemove()方法删除</li>
     * @param ids 包含了要删除数据的所有id内容
     * @return 删除成功返回true，否则返回false
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public boolean delete(Set<Integer> ids) throws Exception;

    /**
     * 根据雇员编号，查询一个雇员的完整信息，调用的是IEmpDAO.findById()方法查询
     * @param id 要查询的雇员编号信息
     * @return 如果能查询到雇员信息则以Vo对象方式返回，若查询不到则返回null
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public Emp get(int id) throws Exception;

    /**
     * 查询雇员的全部数据，调用的是IEmpDAO.findAll()方法查询
     * @return 所在的查询记录以List集合返回
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public List<Emp> list() throws Exception;

    /**
     * 实现数据的模糊查询操作，同时会返回符合查询要求的数据量，在本次操作中要调用以下的功能：<br>
     *     <li>调用IEmpDAO.findSplit()方法，分页查询要显示的数据：</li>
     *     <li>调用IEmpDAO.getAllCount()方法，统计数据的个数：</li>
     * @param column 模糊查询的字段
     * @param keyWord 模糊查询的关键字
     * @param currentPage 当前所在页
     * @param lineSize 每页的长度
     * @return 本方法返回两个数据，所以使用Map集合，出现的内容如下：<br>
     *     <li>key=allEmps、value=IEmpDAO.findAllSplit(),返回List&it;Emp&gt;</li>
     *     <li>key=empCount、value=IEmpDao.getCount(),返回的是Integer</li>
     * @throws Exception IEmpDAO接口中抛出的异常
     */
    public Map<String, Object> listSplit(String column, String keyWord, int currentPage, int lineSize) throws Exception;

}
```

#### 2、定义业务层开发标准的实现类

```java
package cn.alexanderbai.service;

import cn.alexanderbai.dbc.DatabaseConnection;
import cn.alexanderbai.factory.DAOFactory;
import cn.alexanderbai.vo.Emp;

import java.util.*;

/**
 * 业务层标准实现类，当取得本类对象时，就意味着可以进行数据库操作了
 * @Author AlexanderBai
 * @data 2019/3/23 21:41
 */
public class EmpServiceImpl implements IEmpService {

    private DatabaseConnection databaseConnection = new DatabaseConnection();

    @Override
    public boolean insert(Emp vo) throws Exception {
        try {
            if (DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(vo.getEmpNo()) == null) {
                return DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).doCreate(vo);
            }
            System.out.println("雇员已经存在，不能增加");
            return false;
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }

    }

    @Override
    public boolean update(Emp vo) throws Exception {
        try {
            return DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).doUpdate(vo);
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }
    }

    @Override
    public boolean delete(Set<Integer> ids) throws Exception {
        try {
            if (ids.size() == 0) {
                return false;
            }
            return DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).doRemove(ids);
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }
    }

    @Override
    public Emp get(int id) throws Exception {
        try {
            System.out.println(
                    DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getEmpNo()
                    + " "+DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getEmpName()
                    + " "+DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getJod()
                    +" "+ DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getSal()
                    + " "+DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getHireDate()
                    + " "+DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id).getComment()
            );
            return DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findById(id);
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }
    }

    @Override
    public List<Emp> list() throws Exception {
        try {
            if (DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findAll()!=null) {
                Iterator iterator =DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findAll().iterator();
                while (iterator.hasNext()) {
                    Emp emp= (Emp) iterator.next();
                    System.out.println(
                            emp.getEmpNo()
                                    +" " +emp.getEmpName()
                                    + " " +emp.getJod() + emp.getSal()
                                    + " " +emp.getHireDate()
                                    + " " +emp.getComment()
                    );
                }
            }
            return DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findAll();
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }
    }

    @Override
    public Map<String, Object> listSplit(String column, String keyWord, int currentPage, int lineSize) throws Exception {
        try {
            Map<String, Object> map = new HashMap<>();
            map.put("allEmps", DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).findAllSplit(column, keyWord, currentPage, lineSize));
            map.put("empCount", DAOFactory.getIEmpDAOInstance(this.databaseConnection.getConnection()).getAllCount(column, keyWord));
            return map;
        } catch (Exception e) {
            throw e;
        }finally {
            this.databaseConnection.close();
        }
    }
}
```

#### 3、定义Service工厂类

~~~java
package cn.alexanderbai.factory;

import cn.alexanderbai.dbc.DatabaseConnection;
import cn.alexanderbai.service.EmpServiceImpl;
import cn.alexanderbai.service.IEmpService;
import cn.alexanderbai.vo.Emp;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * 定义service工厂类
 * @Author AlexanderBai
 * @data 2019/3/23 21:43
 */
public class ServiceFactory{
    /**
     *
     * @return 返回一个EmpServiceImpl实例对象
     */
    public static IEmpService getIEmpServiceInstance() {
        return new EmpServiceImpl();
    }
}
~~~

### 五、单元测试

~~~java
package cn.alexanderbai.test.junit;

import cn.alexanderbai.factory.ServiceFactory;
import cn.alexanderbai.vo.Emp;
import junit.framework.TestCase;
import org.junit.Test;

import java.util.*;

import static org.junit.Assert.*;

/**
 * @Author AlexanderBai
 * @data 2019/3/24 13:45
 */
public class EmpServiceImplTest {

    @org.junit.Test
    public void insert() {
        Emp vo = new Emp();
        vo.setEmpNo(8);
        vo.setEmpName("AlexanderBai");
        vo.setJod("架构师");
        vo.setSal(40000.0);
        vo.setHireDate(new Date());
        vo.setComment("希望大家都能通过自己的努力，成为一名优秀的架构师！！！");
        try {
            TestCase.assertTrue(ServiceFactory.getIEmpServiceInstance().insert(vo));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void update() {
        Emp vo=new Emp();
        vo.setEmpNo(8);
        vo.setEmpName("AlexanderBai");
        vo.setJod("架构师");
        vo.setSal(40000.0);
        vo.setHireDate(new Date());
        vo.setComment("希望大家都通过自己的努力，能成为一名优秀的架构师！！！");
        try {
            TestCase.assertTrue(ServiceFactory.getIEmpServiceInstance().update(vo));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void delete() {
        Set<Integer> all = new HashSet<>();
        all.add(1);
        try {
            TestCase.assertTrue(ServiceFactory.getIEmpServiceInstance().delete(all));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void get() {
        try {
            TestCase.assertNotNull(ServiceFactory.getIEmpServiceInstance().get(8));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void list() {
        try {
            TestCase.assertTrue(ServiceFactory.getIEmpServiceInstance().list().size()>0);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void listSplit() {//此方法由于在findAllSplit()方法中使用Oracle语法，故在此无法运行
        try {
            Map<String, Object> map = ServiceFactory.getIEmpServiceInstance().listSplit("empName", "s", 1, 10);
            List<Emp> allEmps = (List<Emp>) map.get("allEmps");
            Integer empCounts= (Integer) map.get("empCount");
            TestCase.assertTrue(allEmps.size()>0 && empCounts>0);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
~~~

学了这么累，是吧，放松一下

![3](C:\Users\AlexanderBai\Pictures\images\3.jpg)

