# Banking-Application

SQLQUERY FOR APPLICATION

create schema worshop;
use worshop;
create table account(account_id int primary key auto_increment, customer_id int,
 account_type varchar(220), balance decimal(15,2) not null,
 created_at timestamp default current_timestamp, address varchar(225), mobile varchar(50));
 
 select*from account;
 
 truncate table account;
 
 create table savings_account(account_id int primary key, foreign key
 (`account_id`) references account(account_id),interest_rate decimal(5,2) not null);
 
 select * from savings_account;
 
 create table current_account(account_id int primary key, foreign key
 (`account_id`) references account(account_id),overdraft_limit decimal(5,2) not null);
 
 select*from current_account;
 
 create table transaction
 (transaction_id int primary key auto_increment, 
account_id int,foreign key (`account_id`) references account(account_id),
transaction_type varchar(50) not null,
amount decimal(15,2) not null,
transaction_date timestamp default current_timestamp);

select * from transaction;



JAVA CODE IN ECLIPSE


package com.org.model;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Bank {
 
	 public static Connection db_Connect() throws SQLException
 {
	    	return DriverManager.getConnection("jdbc:mysql://localhost:3306/worshop", "root", "password");
	    	
 }
}


package com.org.Service;

public interface Service {
  void createaccount();
  void viewaccount();
  void updateaccount();
  void withdraw();
  void deposit();
  void amount_trans();
  void viewtrans();
}


package com.org.Service;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Scanner;

import com.org.model.Bank;

public class ServiceImpl implements Service {
 
	private static Scanner sn=new Scanner(System.in);
	
	public void createaccount() {
		Connection conn;
		try {
		conn=Bank.db_Connect();
		System.out.print("Enter Customer id :");
		int c_id=sn.nextInt();
		System.out.print("Enter Account type(Savings/current) :");
		int id=sn.nextInt();
		String type;
		if(id==1) {
			type="Savings";
		}else {
			type="current";
		}
		System.out.print("enter your amount : ");
		Double balance = sn.nextDouble();
		
		System.out.print("enter your address : ");
		String address = sn.next();
		
		System.out.print("enter your mobile : ");
		String mobile = sn.next();
		
		String sql="insert into account (customer_id,account_type,balance,address,mobile) values(?,?,?,?,?)";
		PreparedStatement stmt=conn.prepareStatement(sql);
		stmt.setInt(1,c_id);
		stmt.setString(2,type);
		stmt.setDouble(3,balance);
		stmt.setString(4,address);
		stmt.setString(5,mobile);
		stmt.executeUpdate();
		
		String query="select account_id from account where customer_id=? and account_type=?";
		stmt=conn.prepareStatement(query);
		stmt.setInt(1,c_id);
		stmt.setString(2,type);
		ResultSet rs=stmt.executeQuery();
		int account_id=rs.next() ? rs.getInt("account_id"):0;
		
		if(type.equals("Savings"))
			
		{
			String query1="insert into savings_account(account_id,interest_rate) values(?,?)";
			stmt=conn.prepareStatement(query1);
			stmt.setInt(1,account_id);
			stmt.setDouble(2,0.05);
			stmt.executeUpdate();
		}
		else {
			String query1="insert into current_account(account_id,overdraft_limit) values(?,?)";
			stmt=conn.prepareStatement(query1);
			stmt.setInt(1,account_id);
			stmt.setDouble(2,1000.0);
			stmt.executeUpdate();
		}
		
		System.out.println("Data Insert Successfully..");
		
	}catch(SQLException e)
		{
		e.printStackTrace(); 
		
	}
}
	@Override
	public void viewaccount(){
		// TODO Auto-generated method stub
		try{
			System.out.println("Enter account id: ");
			int a_id=sn.nextInt();
			
			Connection conn=Bank.db_Connect();
			String sql="select*from account where account_id=?";
			PreparedStatement stmt=conn.prepareStatement(sql);
			stmt.setInt(1, a_id);
			ResultSet rs=stmt.executeQuery();
			
			if(rs.next())
			{
				System.out.println("account id: "+rs.getInt("account_id"));
				System.out.println("customer id: "+rs.getInt("customer_id"));
				System.out.println("balance: "+rs.getDouble("balance"));
				System.out.println("account_type: "+rs.getString("account_type"));
				System.out.println("mobile: "+rs.getString("mobile"));
			}
		}catch(SQLException e)
		{
		e.printStackTrace();
	}
		
				
		
	}

	@Override
	public void updateaccount() {
		// TODO Auto-generated method stub
		try {
			System.out.println("Enter account id :");
			int a_id=sn.nextInt();
			System.out.println("Enter address :");
			String address=sn.next();
			System.out.println("Enter mobile number :");
			String mobile=sn.next();
			Connection conn=Bank.db_Connect();
			String sql="update account set address=?,mobile=? where account_id=?";
			PreparedStatement stmt= conn.prepareStatement(sql);
			
			stmt.setString(1, address);
			stmt.setString(2, mobile);
			stmt.setInt(3, a_id);
			stmt.executeUpdate();
			System.out.println("Data updated successfully!");
		}catch(SQLException e) {
			e.printStackTrace(); 
		}
		
	}

	@Override
	public void withdraw() {
		try {
			
			Connection conn = Bank.db_Connect();
			System.out.print("Enter account id :");
			int acc_id=sn.nextInt();
			
			System.out.print("Enter withdraw amount :");
			double amount=sn.nextDouble();
			
			String query1="select*from account where account_id=?";
			PreparedStatement stmt= conn.prepareStatement(query1);
			stmt.setInt(1, acc_id);
			 ResultSet rs = stmt.executeQuery();
		        if(rs.next()) {
		        	String account  =rs.getString("account_type");
		        	if(!account.equals("current")) {
		        		System.out.println(account);
		        		Double balance = rs.getDouble("balance");
		        		if(balance >= amount) {
		        			query1 = "INSERT INTO transaction (account_id, transaction_type, amount) VALUES (?, ?, ?)";
		        	        PreparedStatement pstmt1 = conn.prepareStatement(query1);
		        	        pstmt1.setInt(1, acc_id);
		        	        pstmt1.setString(2, "Withdraw"); 
		        	        pstmt1.setDouble(3, amount);
		        	        pstmt1.executeUpdate();
		        	        
		        	        query1 = "UPDATE account SET balance = balance - ? WHERE account_id = ?";
		        	        PreparedStatement pstmt11 = conn.prepareStatement(query1);
		        	        pstmt11.setDouble(1, amount);
		        	        pstmt11.setInt(2, acc_id); 
		        	        pstmt11.executeUpdate();
		        	        
		        	        System.out.println("Withdraw  successful...");
		        		}else {
		        			System.out.println("Insufficient balance!");
		        		}
		        		rs.close();
		        	}
		        
		        }
		    	}catch (SQLException e) {
		        e.printStackTrace();
		    }
		    	
		    }
	
	@Override
	public void deposit() {
	    try {
	        Connection conn = Bank.db_Connect();
	        
	        System.out.print("Enter account id: ");
	        int acc_id = sn.nextInt();

	        // Check if account_id exists in the account table
	        String checkAccountQuery = "SELECT account_id FROM account WHERE account_id = ?";
	        PreparedStatement Stmt = conn.prepareStatement(checkAccountQuery);
	        Stmt.setInt(1, acc_id);
	        ResultSet rs = Stmt.executeQuery();

	        if (!rs.next()) {
	            System.out.println("Account ID does not exist. Please check the account ID and try again.");
	            return;
	        }
	        
	        System.out.print("Enter amount to deposit: ");
	        double amount = sn.nextDouble();
	        
	        // Proceed with inserting the transaction if account_id exists
	        String query = "INSERT INTO transaction (account_id, transaction_type, amount) VALUES (?, ?, ?)";
	        PreparedStatement stmt = conn.prepareStatement(query);
	        stmt.setInt(1, acc_id);
	        stmt.setString(2, "Deposit");
	        stmt.setDouble(3, amount);
	        stmt.executeUpdate();
	        
	        // Update account balance
	        query = "UPDATE account SET balance = balance + ? WHERE account_id = ?";
	        stmt = conn.prepareStatement(query);
	        stmt.setDouble(1, amount);
	        stmt.setInt(2, acc_id);
	        stmt.executeUpdate();
	        
	        System.out.println("Deposit successful...");

	    } catch (SQLException e) {
	        e.printStackTrace();
	    }
	}

	@Override
	public void amount_trans() {
	    try {
	        Connection conn = Bank.db_Connect();
	        
	        System.out.println("Enter source account id: ");
	        int sourceAcc = sn.nextInt();
	        
	        System.out.println("Enter destination account id: ");
	        int dest = sn.nextInt();
	        
	        System.out.println("Enter transfer amount: ");
	        double amount = sn.nextDouble();
	        
	        // Check balance in source account
	        String checkBalanceQuery = "SELECT balance FROM account WHERE account_id = ?";
	        PreparedStatement stmt = conn.prepareStatement(checkBalanceQuery);
	        stmt.setInt(1, sourceAcc);
	        ResultSet rs = stmt.executeQuery();
	        
	        if (rs.next()) {
	            double balance = rs.getDouble("balance");
	            
	            if (balance >= amount) {
	                // Deduct from source account
	                String deductQuery = "UPDATE account SET balance = balance - ? WHERE account_id = ?";
	                PreparedStatement deductStmt = conn.prepareStatement(deductQuery);
	                deductStmt.setDouble(1, amount);
	                deductStmt.setInt(2, sourceAcc);
	                deductStmt.executeUpdate();

	                // Add to destination account
	                String addQuery = "UPDATE account SET balance = balance + ? WHERE account_id = ?";
	                PreparedStatement addStmt = conn.prepareStatement(addQuery);
	                addStmt.setDouble(1, amount);
	                addStmt.setInt(2, dest);
	                addStmt.executeUpdate();

	                // Record transaction
	                String recordTransaction = "INSERT INTO transaction (account_id, transaction_type, amount) VALUES (?, ?, ?)";
	                
	                PreparedStatement transStmt = conn.prepareStatement(recordTransaction);
	                transStmt.setInt(1, sourceAcc);
	                transStmt.setString(2, "Transfer Out");
	                transStmt.setDouble(3, amount);
	                transStmt.executeUpdate();

	                transStmt.setInt(1, dest);
	                transStmt.setString(2, "Transfer In");
	                transStmt.setDouble(3, amount);
	                transStmt.executeUpdate();

	                System.out.println("Transfer successful...");
	            } else {
	                System.out.println("Insufficient balance in source account.");
	            }
	        } else {
	            System.out.println("Source account not found.");
	        }
	        
	    } catch (SQLException e) {
	        e.printStackTrace();
	    }
	}

	@Override
	public void viewtrans() {
		// TODO Auto-generated method 
		try {
		Connection conn=Bank.db_Connect();
		System.out.print("enter account id: ");
		int acc_id=sn.nextInt();
		String query="select * from transaction where account_id=?";
		PreparedStatement stmt = conn.prepareStatement(query);
		stmt.setInt(1, acc_id);
		ResultSet rs=stmt.executeQuery();
		
		while(rs.next()) {
			System.out.println("transaction id: "+rs.getInt("transaction_id"));
			System.out.println("account id: "+rs.getInt("account_id"));
			System.out.println("transaction type: "+rs.getString("transaction_type"));
			System.out.println("amount: "+rs.getDouble("amount"));
		}
		
		
		
	}catch(SQLException e) {
		e.printStackTrace();
	}
	
	}
}


package com.org.Controller;

import java.sql.SQLException;
import java.util.Scanner;

import com.org.Service.ServiceImpl;
import com.org.model.Bank;
public class BankController {
  private static Scanner sn=new Scanner(System.in);
  public static void main(String[] args) {
	  ServiceImpl sc=new ServiceImpl();
	  Bank n=new Bank();
	  try {
		  
		  ServiceImpl sv=new ServiceImpl();
		  n.db_Connect();
		  boolean exe=true;
		  while(exe) {
		  System.out.println("Connection Created Successfully...");
		  System.out.println("1.Create Account");
		  System.out.println("2.View Account");
		  System.out.println("3.Update Account Info");
		  System.out.println("4.Deposit Amount");
		  System.out.println("5.Withdraw Amount");
		  System.out.println("6.Transaction Amount");
		  System.out.println("7.View Transaction");
		  System.out.println("8.Exit..");
		  int input=sn.nextInt();
		  switch(input) {
		  case 1:
			  sv.createaccount();
			  //System.out.println("1");
			  break;
		  case 2:
			  sv.viewaccount();
			  //System.out.println("2");
			  break;
		  case 3:
			  sv.updateaccount();
			  //System.out.println("3");
			  break;
		  case 4:
			  sv.deposit();
			  //System.out.println("4");
			  break;
		  case 5:
			  sv.withdraw();
			  //System.out.println("5");
			  break;
		  case 6:
			  sv.amount_trans();
			  //System.out.println("6");
			  break;
		  case 7:
			  sv.viewtrans();
			 // System.out.println("7");
			  break;
		  case 8:
			  exe=false;
			  System.out.println("Exited from the Account...");
			  break;
		 default:
				  System.out.println("Invalid..");
			  
		}
	}
 }
	  catch(SQLException e) {
		  //to auto generate catch block
		  e.printStackTrace();
	  }
  }

}
	




