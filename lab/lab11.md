1. JDBC体系结构和简单的查询
```java
/* 请在适当的位置补充代码，完成指定的任务 
   提示：
      try {


      } catch
    之间补充代码  
*/
import java.sql.*;

public class Client {
        
    public static void main(String[] args) {
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;

        try {
            // String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
            String URL = "jdbc:postgresql://localhost:5432/postgres";
            String USER = "gaussdb";
            String PASS = "Passwd123@123";
            Class.forName("org.postgresql.Driver");
            connection = DriverManager.getConnection(URL, USER, PASS);
            statement = connection.createStatement();
            resultSet = statement.executeQuery("select c_name, c_mail, c_phone from client where c_mail is not null");
            System.out.println("姓名\t邮箱\t\t\t\t电话");
            while (resultSet.next()) {  
                System.out.print(resultSet.getString("c_name") + "\t");  
                System.out.print(resultSet.getString("c_mail") + "\t\t");  
                System.out.println(resultSet.getString("c_phone"));  
            }
 
         } catch (ClassNotFoundException e) {
            System.out.println("Sorry,can`t find the JDBC Driver!"); 
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            try {
                if (resultSet != null) {
                    resultSet.close();
                }
                if (statement != null) {
                    statement.close();
                }

                if (connection != null) {
                    connection.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```

2. 用户登录
```java
    import java.sql.*;
import java.util.Scanner;

public class Login {
    public static void main(String[] args) {
        Connection connection = null;
        //申明下文中的resultSet, statement
        Statement statement = null;
        ResultSet resultSet = null;
        Scanner input = new Scanner(System.in);
        System.out.print("请输入用户名：");
        String loginName = input.nextLine();
        System.out.print("请输入密码：");
        String loginPass = input.nextLine();
        try {
            Class.forName("org.postgresql.Driver");
            String userName = "gaussdb";
            String passWord = "Passwd123@123";
            String url = "jdbc:postgresql://localhost:5432/postgres";
           
            connection = DriverManager.getConnection(url, userName, passWord);
          
            connection = DriverManager.getConnection(url, userName, passWord);
            // 补充实现代码:
            // statement = connection.createStatement();
            // String SQL = "select * from client where c_mail = '"+loginName+"' ;";
            // resultSet = statement.executeQuery(SQL);
            // if(resultSet.next()){
            //     if(resultSet.getString("c_password").equals(loginPass))
            //         System.out.println("登录成功。");
            //     else System.out.println("用户名或密码错误！");
            // }
            // else System.out.println("用户名或密码错误！");
            statement = connection.createStatement();
            String sql = "select * from client where c_mail = ? and c_password = ?";
            PreparedStatement ps = connection.prepareStatement(sql);
            ps.setString(1, loginName);
            ps.setString(2, loginPass);
            resultSet = ps.executeQuery();
            if (resultSet.next())
                System.out.println("登录成功。");
            else
                System.out.println("用户名或密码错误！");

         } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            try {
                if (resultSet != null) {
                    resultSet.close();
                }
                if (statement != null) {
                    statement.close();
                }

                if (connection != null) {
                    connection.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```

3. 添加新客户
```java
import java.sql.*;
import java.util.Scanner;

public class AddClient {
    static final String JDBC_DRIVER = "org.postgresql.Driver";
    static final String DB_URL = "jdbc:postgresql://127.0.0.1:5432/postgres?";
    static final String USER = "gaussdb";
    static final String PASS = "Passwd123@123";
    
    /**
     * 向Client表中插入数据
     *
     * @param connection 数据库连接对象
     * @param c_id 客户编号
     * @param c_name 客户名称
     * @param c_mail 客户邮箱
     * @param c_id_card 客户身份证
     * @param c_phone 客户手机号
     * @param c_password 客户登录密码
     */
    public static int insertClient(Connection connection,
                                   int c_id, String c_name, String c_mail,
                                   String c_id_card, String c_phone, 
                                   String c_password){
 String sql = "insert into client values (?, ?, ?, ?, ?, ?)";
        try {
            PreparedStatement ps = connection.prepareStatement(sql);
            ps.setInt(1, c_id);
            ps.setString(2, c_name);
            ps.setString(3, c_mail);
            ps.setString(4, c_id_card);
            ps.setString(5, c_phone);
            ps.setString(6, c_password);
            return ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return 0;
    }
    // 不要修改main() 
    public static void main(String[] args) throws Exception {

        Scanner sc = new Scanner(System.in);
        Class.forName(JDBC_DRIVER);

        Connection connection = DriverManager.getConnection(DB_URL, USER, PASS);

        while(sc.hasNext())
        {
            String input = sc.nextLine();
            if(input.equals(""))
                break;
            String[]commands = input.split(" ");
            if(commands.length ==0)
                break;
            int id = Integer.parseInt(commands[0]);
            String name = commands[1];
            String mail = commands[2];
            String idCard = commands[3];
            String phone = commands[4];
            String password = commands[5];
            insertClient(connection, id, name, mail, idCard, phone, password);
        }
    }
}
```

4. 银行卡销户
```java
import java.sql.*;
import java.util.Scanner;

public class RemoveCard {
    static final String JDBC_DRIVER = "org.postgresql.Driver";
    static final String DB_URL = "jdbc:postgresql://127.0.0.1:5432/postgres?";
    static final String USER = "gaussdb";
    static final String PASS = "Passwd123@123";
     /**
     * 删除bank_card表中数据
     *
     * @param connection 数据库连接对象
     * @param b_c_id 客户编号
     * @param c_number 银行卡号
     */
    public static int removeBankCard(Connection connection,
                                   int b_c_id, String b_number){
        String sql = "delete from bank_card where b_c_id = ? and b_number = ?";
        try {
            PreparedStatement ps = connection.prepareStatement(sql);
            ps.setInt(1, b_c_id);
            ps.setString(2, b_number);
            return ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return 0;
    }
    // 不要修改main() 
    public static void main(String[] args) throws Exception {

        Scanner sc = new Scanner(System.in);
        Class.forName(JDBC_DRIVER);

        Connection connection = DriverManager.getConnection(DB_URL, USER, PASS);

        while(sc.hasNext())
        {
            String input = sc.nextLine();
            if(input.equals(""))
                break;
            String[]commands = input.split(" ");
            if(commands.length ==0)
                break;
            int id = Integer.parseInt(commands[0]);
            String carNumber = commands[1];
            
            int n = removeBankCard(connection, id, carNumber);
            if (n > 0) {
               System.out.println("已销卡数：" + n);
            } else {
               System.out.println("销户失败，请检查客户编号或银行卡号！" );
            }
        }
    }

}
```

5. 客户修改密码
```java
import java.sql.*;
import java.util.Scanner;

public class ChangePass {
    static final String JDBC_DRIVER = "org.postgresql.Driver";
    static final String DB_URL = "jdbc:postgresql://127.0.0.1:5432/postgres?";
    static final String USER = "gaussdb";
    static final String PASS = "Passwd123@123";
    
    /**
     * 修改客户密码
     *
     * @param connection 数据库连接对象
     * @param mail 客户邮箱,也是登录名
     * @param password 客户登录密码
     * @param newPass  新密码
     * @return
     *   1 - 密码修改成功
     *   2 - 用户不存在
     *   3 - 密码不正确
     *  -1 - 程序异常(如没能连接到数据库等）
     */
    public static int passwd(Connection connection,
                             String mail,
                             String password, 
                             String newPass){
        PreparedStatement pstmt = null;
        ResultSet resultSet = null;
        int n = 0;
        try {
            String sql = "select * from client where c_mail = ?;";
            pstmt = connection.prepareStatement(sql);
            pstmt.setString(1,mail);
            resultSet = pstmt.executeQuery();
            if(resultSet.next()==false) n = 2;
            else{
                sql = "select * from client where c_mail = ? and c_password = ?;";
                pstmt = connection.prepareStatement(sql);
                pstmt.setString(1,mail);
                pstmt.setString(2,password);
                resultSet = pstmt.executeQuery();
                if(resultSet.next()==false) n = 3;
                else{
                    sql = "update client set c_password = ? where c_mail = ?;";
                    pstmt = connection.prepareStatement(sql);
                    pstmt.setString(1,newPass);
                    pstmt.setString(2,mail);
                    n = pstmt.executeUpdate();
                }
            }
            
        } catch (SQLException throwables) {
            throwables.printStackTrace();
            n = -1;
        } finally {
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
                if (resultSet != null) {
                    resultSet.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
                n = -1;
            }
        }
        return n;
    }
    // 不要修改main() 
    public static void main(String[] args) throws Exception {

        Scanner sc = new Scanner(System.in);
        Class.forName(JDBC_DRIVER);
        Connection connection = DriverManager.getConnection(DB_URL, USER, PASS);

        while(sc.hasNext())
        {
            String input = sc.nextLine();
            if(input.equals(""))
                break;
            String[]commands = input.split(" ");
            if(commands.length ==0)
                break;
            String email = commands[0];
            String pass = commands[1];
            String pwd1 = commands[2];
            String pwd2 = commands[3];
            if (pwd1.equals(pwd2)) {
              int n = passwd(connection, email, pass, pwd1);  
              System.out.println("return: " + n);
            } else {
              System.out.println("两次输入的密码不一样!");
            }
        }
    }
}
```

6. 事务与转账操作
``` java
import java.sql.*;
import java.util.Scanner;

public class Transfer {
    static final String JDBC_DRIVER = "org.postgresql.Driver";
    static final String DB_URL = "jdbc:postgresql://127.0.0.1:5432/postgres?";
    static final String USER = "gaussdb";
    static final String PASS = "Passwd123@123";
    /**
     * 转账操作
     *
     * @param connection 数据库连接对象
     * @param sourceCard 转出账号
     * @param destCard 转入账号
     * @param amount  转账金额
     * @return boolean
     *   true  - 转账成功
     *   false - 转账失败
     */
    public static boolean transferBalance(Connection connection,
                             String sourceCard,
                             String destCard, 
                             double amount){
PreparedStatement pstmt = null;
        ResultSet resultSet = null;
        boolean n = true;
        try {
			connection.setAutoCommit(false);
			connection.setTransactionIsolation(4);
            String sql = "update bank_card set b_balance = b_balance - ? where b_number = ?;";
            pstmt = connection.prepareStatement(sql);
            pstmt.setDouble(1,amount);
			pstmt.setString(2,sourceCard);
            pstmt.executeUpdate();
			sql = "update bank_card set b_balance = b_balance + ? where b_number = ? and b_type = '储蓄卡';";
			pstmt = connection.prepareStatement(sql);
            pstmt.setDouble(1,amount);
			pstmt.setString(2,destCard);
            pstmt.executeUpdate();
			sql = "update bank_card set b_balance = b_balance - ? where b_number = ? and b_type = '信用卡';";
			pstmt = connection.prepareStatement(sql);
            pstmt.setDouble(1,amount);
			pstmt.setString(2,destCard);
            pstmt.executeUpdate();

			sql = "select * from bank_card where b_number = ? and b_type = '储蓄卡';";
			pstmt = connection.prepareStatement(sql);
			pstmt.setString(1,sourceCard);
            resultSet = pstmt.executeQuery();
            if(resultSet.next()==false) {
				n = false;
				connection.rollback();
			}
			else {
				if(resultSet.getDouble("b_balance") < 0){
					n = false;
					connection.rollback();
				}
				else{
					sql = "select * from bank_card where b_number = ?;";
					pstmt = connection.prepareStatement(sql);
					pstmt.setString(1,destCard);
					resultSet = pstmt.executeQuery();
					if(resultSet.next()==false) {
						n = false;
						connection.rollback();
					}
					else connection.commit();
				}
			}
 
        } catch (SQLException throwables) {
            throwables.printStackTrace();
            n = false;
        } finally {
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
                if (resultSet != null) {
                    resultSet.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
                n = false;
            }
        }
        return n;
    }
    // 不要修改main() 
    public static void main(String[] args) throws Exception {

        Scanner sc = new Scanner(System.in);
        Class.forName(JDBC_DRIVER);

        Connection connection = DriverManager.getConnection(DB_URL, USER, PASS);

        while(sc.hasNext())
        {
            String input = sc.nextLine();
            if(input.equals(""))
                break;

            String[]commands = input.split(" ");
            if(commands.length ==0)
                break;
            String payerCard = commands[0];
            String  payeeCard = commands[1];
            double  amount = Double.parseDouble(commands[2]);
            if (transferBalance(connection, payerCard, payeeCard, amount)) {
              System.out.println("转账成功。" );
            } else {
              System.out.println("转账失败,请核对卡号，卡类型及卡余额!");
            }
        }
    }
}
```

7. 把稀疏表格转为键值对存储
``` java
import java.sql.*;

public class Transform {
  static final String JDBC_DRIVER = "org.postgresql.Driver";
    static final String DB_URL = "jdbc:postgresql://127.0.0.1:5432/postgres?";
    static final String USER = "gaussdb";
    static final String PASS = "Passwd123@123";
    
    /**
     * 向sc表中插入数据
     *
     */
    public static int insertSC(Connection conn,int sno, String type, int score){
		PreparedStatement pstmt = null;
		int n = 0;
		try {
            String sql = "insert into sc values (?,?,?);";
            pstmt = conn.prepareStatement(sql);
            pstmt.setInt(1,sno);
            pstmt.setString(2,type);
            pstmt.setInt(3,score);
            n = pstmt.executeUpdate();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        return n;
    }
    public static void main(String[] args) {
		Connection conn = null;
        Statement stmt = null;
        ResultSet rest = null;
        try {
            Class.forName(JDBC_DRIVER);
            conn = DriverManager.getConnection(DB_URL, USER, PASS);
            
			stmt = conn.createStatement();
            String sql = "select * from entrance_exam;";
            rest = stmt.executeQuery(sql);
            while(rest.next()){
                int sno = rest.getInt("sno");
				int chinese = rest.getInt("chinese");
				int math = rest.getInt("math");
				int english = rest.getInt("English");
				int physics = rest.getInt("physics");
				int chemistry = rest.getInt("chemistry");
				int biology = rest.getInt("biology");
				int history = rest.getInt("history");
				int geography = rest.getInt("geography");
				int politics = rest.getInt("politics");
				if(chinese != 0){
					insertSC(conn,sno,"chinese",chinese);
				}
				if(math != 0){
					insertSC(conn,sno,"math",math);
				}
				if(english != 0){
					insertSC(conn,sno,"english",english);
				}
				if(physics != 0){
					insertSC(conn,sno,"physics",physics);
				}
				if(chemistry != 0){
					insertSC(conn,sno,"chemistry",chemistry);
				}
				if(biology != 0){
					insertSC(conn,sno,"biology",biology);
				}
				if(history != 0){
					insertSC(conn,sno,"history",history);
				}
				if(geography != 0){
					insertSC(conn,sno,"geography",geography);
				}
				if(politics != 0){
					insertSC(conn,sno,"politics",politics);
				}
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            try {
                if (rest != null) {
                    rest.close();
                }
                if (stmt != null) {
                    stmt.close();
                }
                if (conn != null) {
                    conn.close();
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```
