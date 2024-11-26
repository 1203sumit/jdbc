import java.sql.*;
import java.util.Scanner;

 class LibraryManagementSystem {
    // Database connection details
    private static final String DB_URL = "jdbc:mysql://localhost:3306/library_db";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Initialize the database
        initializeDatabase();

        while (true) {
            System.out.println("\n=== Library Management System ===");
            System.out.println("1. Add a Book");
            System.out.println("2. View All Books");
            System.out.println("3. Search Book by Title");
            System.out.println("4. Delete Book by ID");
            System.out.println("5. Exit");
            System.out.print("Choose an option: ");
            int choice = scanner.nextInt();
            scanner.nextLine(); // Consume newline

            switch (choice) {
                case 1:
                    addBook(scanner);
                    break;
                case 2:
                    viewBooks();
                    break;
                case 3:
                    searchBookByTitle(scanner);
                    break;
                case 4:
                    deleteBookById(scanner);
                    break;
                case 5:
                    System.out.println("Exiting... Goodbye!");
                    return;
                default:
                    System.out.println("Invalid option. Please try again.");
            }
        }
    }

    private static void initializeDatabase() {
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             Statement stmt = conn.createStatement()) {

            String createDatabaseSQL = "CREATE DATABASE IF NOT EXISTS library_db";
            stmt.executeUpdate(createDatabaseSQL);

            String useDatabaseSQL = "USE library_db";
            stmt.executeUpdate(useDatabaseSQL);

            String createTableSQL = "CREATE TABLE IF NOT EXISTS Books ("
                    + "id INT AUTO_INCREMENT PRIMARY KEY, "
                    + "title VARCHAR(255) NOT NULL, "
                    + "author VARCHAR(255) NOT NULL, "
                    + "year INT NOT NULL)";
            stmt.executeUpdate(createTableSQL);

            System.out.println("Database initialized successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void addBook(Scanner scanner) {
        System.out.print("Enter Book Title: ");
        String title = scanner.nextLine();

        System.out.print("Enter Author: ");
        String author = scanner.nextLine();

        System.out.print("Enter Year of Publication: ");
        int year = scanner.nextInt();

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement(
                     "INSERT INTO Books (title, author, year) VALUES (?, ?, ?)")) {

            pstmt.setString(1, title);
            pstmt.setString(2, author);
            pstmt.setInt(3, year);
            pstmt.executeUpdate();

            System.out.println("Book added successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void viewBooks() {
        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM Books")) {

            System.out.println("\n=== All Books ===");
            while (rs.next()) {
                System.out.println("ID: " + rs.getInt("id")
                        + ", Title: " + rs.getString("title")
                        + ", Author: " + rs.getString("author")
                        + ", Year: " + rs.getInt("year"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void searchBookByTitle(Scanner scanner) {
        System.out.print("Enter Book Title to Search: ");
        String title = scanner.nextLine();

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM Books WHERE title LIKE ?")) {

            pstmt.setString(1, "%" + title + "%");
            ResultSet rs = pstmt.executeQuery();

            System.out.println("\n=== Search Results ===");
            while (rs.next()) {
                System.out.println("ID: " + rs.getInt("id")
                        + ", Title: " + rs.getString("title")
                        + ", Author: " + rs.getString("author")
                        + ", Year: " + rs.getInt("year"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void deleteBookById(Scanner scanner) {
        System.out.print("Enter Book ID to Delete: ");
        int id = scanner.nextInt();

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement("DELETE FROM Books WHERE id = ?")) {

            pstmt.setInt(1, id);
            int rowsAffected = pstmt.executeUpdate();

            if (rowsAffected > 0) {
                System.out.println("Book deleted successfully!");
            } else {
                System.out.println("No book found with the given ID.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
