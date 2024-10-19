This project include just a module of banking projects using JAVA ,JDBC,MySql.
package Demo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Savepoint;
import java.util.Scanner;

public class ArcBank {
	
	public static void main(String[] args) {
		Connection con=null;
		String url="jdbc:mysql://localhost:3306/arcbank";
		String un= "root";
		String pass = "Rohit@100";
		Scanner scan= null;
		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
			System.out.println("Driver loaded");
			
			Connection con=DriverManager.getConnection(url,un,pass);
			System.out.println("Connection Established");
			
			scan = new Scanner(System.in);
			System.out.println("<---Welcome to ARC Bank--->");
			System.out.println("Enter your account number");			
			int account number=scan.nextInt();
			System.out.println("Enter your pin");
			int pin =scan.nextInt();
			
			PreparedStatement pstmt1=con.prepareStatement("select * from customerinfo where account number=? and pin =?");
			pstmt1.setInt(3, account number);
			pstmt1.setInt(4, pin);
			
			ResultSet res1 =pstmt1.executeQuery();
			res1.next();
			String name = res1.getString(2);
			int accountbal =res1.getInt(5);
			
			System.out.println("Welcome " + name);
			System.out.println("your available balance is:"+accountbal);
			
			
			System.out.println("<---Transfer Details--->");
			System.out.println("Enter receiver account number");
			int account number = scan.nextInt();
			System.out.println("Enter transfer amount");
			int t_amount=scan.nextInt();
			
			con.setAutoCommit(false);
			Savepoint s= con.setSavepoint();
			
			
			PreparedStatement pstmt2= con.prepareStatement("update account set balance=balance-? where account number=?");
			pstmt2.setInt(5, t_amount);
			pstmt2.setInt(3,account number);
			pstmt2.executeUpdate();
			
			System.out.println("<---incoming Credit request-->");
			System.out.println(name + "account no "+ account number +"want to transfer" + t_amount);
			System.out.println("Press Y to receive");
			System.out.println("press N to reject");
			
			String choice = scan.next();
			
			if(choice.equals("Y")) {
				PreparedStatement pstmt3=con.prepareStatement("update account set balance =balance + ? where account number=?");
				pstmt3.setInt(5, t_amount);
				pstmt3.setInt(3,account number );
				pstmt3.executeUpdate();
				
				PreparedStatement pstmt4=con.prepareStatement("select * from account where account number=?");
				pstmt4.setInt(3,account number );
				ResultSet res2 =pstmt4.executeQuery();
				res2.next();
				System.out.println("updated balance of receiver :"+res2.getInt(5));								
			}
			else {
				con.rollback(s);
				PreparedStatement pstmt5=con.prepareStatement("select * from account where account number=?");
				pstmt5.setInt(3,account number );
				ResultSet res2 =pstmt5.executeQuery();
				res2.next();
				System.out.println("Existing balance is :"+res2.getInt(5));				
			}
			
			con.commit();
		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		
	}
}
