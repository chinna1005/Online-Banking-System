# Online-Banking-System
import java.sql.*;
import java.util.Scanner;

public class OnlineBanking {
    Scanner sc = new Scanner(System.in);

    // Create new account
    public void createAccount() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Name: ");
            String name = sc.nextLine();
            System.out.print("Enter Initial Balance: ");
            double balance = sc.nextDouble();

            String sql = "INSERT INTO accounts(name, balance) VALUES(?, ?)";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setString(1, name);
            ps.setDouble(2, balance);
            ps.executeUpdate();

            System.out.println("✅ Account created successfully.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Deposit money
    public void deposit() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Account ID: ");
            int id = sc.nextInt();
            System.out.print("Enter Amount to Deposit: ");
            double amount = sc.nextDouble();

            String sql = "UPDATE accounts SET balance = balance + ? WHERE account_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setDouble(1, amount);
            ps.setInt(2, id);
            ps.executeUpdate();

            logTransaction(con, id, "Deposit", amount);
            System.out.println("✅ Deposit successful.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    //check balance
    public void balance() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Account ID: ");
            int id = sc.nextInt();

            String sql = "SELECT balance FROM accounts WHERE account_id= ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, id);

            ResultSet rs = ps.executeQuery();
            if (rs.next()) {
                System.out.println("Current Balance: " + rs.getDouble("balance"));
            } else {
                System.out.println("Account not found.");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Withdraw money
    public void withdraw() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Account ID: ");
            int id = sc.nextInt();
            System.out.print("Enter Amount to Withdraw: ");
            double amount = sc.nextDouble();

            String checkSql = "SELECT balance FROM accounts WHERE account_id = ?";
            PreparedStatement checkPs = con.prepareStatement(checkSql);
            checkPs.setInt(1, id);
            ResultSet rs = checkPs.executeQuery();

            if (rs.next()) {
                double balance = rs.getDouble("balance");
                if (balance >= amount) {
                    String sql = "UPDATE accounts SET balance = balance - ? WHERE account_id = ?";
                    PreparedStatement ps = con.prepareStatement(sql);
                    ps.setDouble(1, amount);
                    ps.setInt(2, id);
                    ps.executeUpdate();

                    logTransaction(con, id, "Withdraw", amount);
                    System.out.println("✅ Withdrawal successful.");
                } else {
                    System.out.println("❌ Insufficient balance.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Transfer funds
    public void transfer() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Sender Account ID: ");
            int senderId = sc.nextInt();
            System.out.print("Enter Receiver Account ID: ");
            int receiverId = sc.nextInt();
            System.out.print("Enter Amount to Transfer: ");
            double amount = sc.nextDouble();

            con.setAutoCommit(false);

            String withdrawSql = "UPDATE accounts SET balance = balance - ? WHERE account_id = ? AND balance >= ?";
            PreparedStatement withdrawPs = con.prepareStatement(withdrawSql);
            withdrawPs.setDouble(1, amount);
            withdrawPs.setInt(2, senderId);
            withdrawPs.setDouble(3, amount);

            String depositSql = "UPDATE accounts SET balance = balance + ? WHERE account_id = ?";
            PreparedStatement depositPs = con.prepareStatement(depositSql);
            depositPs.setDouble(1, amount);
            depositPs.setInt(2, receiverId);

            int rows1 = withdrawPs.executeUpdate();
            int rows2 = depositPs.executeUpdate();

            if (rows1 > 0 && rows2 > 0) {
                logTransaction(con, senderId, "Transfer Out", amount);
                logTransaction(con, receiverId, "Transfer In", amount);
                con.commit();
                System.out.println("✅ Transfer successful.");
            } else {
                con.rollback();
                System.out.println("❌ Transfer failed. Check balances and IDs.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // View transaction history
    public void viewTransactions() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Account ID: ");
            int id = sc.nextInt();

            String sql = "SELECT * FROM transactions WHERE account_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, id);
            ResultSet rs = ps.executeQuery();

            System.out.println("Transaction History:");
            while (rs.next()) {
                System.out.println(rs.getInt("id") + " | " + rs.getString("type") + " | " + rs.getDouble("amount") + " | " + rs.getTimestamp("date"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    //Delete account
 // deleteAccount
    public void deleteAccount() {
        try (Connection con = DBConnection.getConnection()) {
            System.out.print("Enter Account ID to delete: ");
            int id = sc.nextInt();

            String sql = "DELETE FROM accounts WHERE account_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, id);

            int rowsDeleted = ps.executeUpdate();
            if (rowsDeleted > 0) {
                System.out.println("✅ Account deleted successfully.");
            } else {
                System.out.println("⚠ No account found with the given ID.");
            }
        } catch (SQLException e) {
            System.out.println("❌ Error deleting account: " + e.getMessage());
        }
    }


    // Helper method to log transactions
    private void logTransaction(Connection con, int accountId, String type, double amount) throws SQLException {
        String sql = "INSERT INTO transactions(account_id, type, amount) VALUES(?, ?, ?)";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, accountId);
        ps.setString(2, type);
        ps.setDouble(3, amount);
        ps.executeUpdate();
    }
}
