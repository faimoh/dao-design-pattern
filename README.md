# dao-design-pattern
Discussing the implementation Data Access Object (DAO) Java EE Design Pattern
## Introduction
Ever since I came across the DAO design pattern, I am in love with it. Because, it makes my application design process easier. I still remember the early days of my web application development where I started developing a web app for my team to make daily project reporting easier. The project started out of my personal interest. Initially, I started using the Oracle database. After some days, I began using MySQL on my laptop because it was better working on it when compared with Oracle DB. As soon as I decided to switch to MySQL from Oracle DB, I started facing the mammoth task of changing many parts of my already developed code. It was a frustrating experience. During such period I came across the DAO design pattern and it solved my data access layer issues forever. It remains my favorite to this day. I have been solely developing web applications and DAO is the part of my application that gets developed first, immediately after designing my data model.

DAO design pattern is part of the [core J2EE design patterns](http://www.corej2eepatterns.com/DataAccessObject.htm). The pattern lets you separate the application's data access layer from other parts of the application. Usually, web applications are developed following the Model View Controller (MVC) 2 design pattern, where MVC are three different parts of the same web application. The components of these three parts interact with the application's database for CRUD (Create Read Update Delete) operations. Without having a separate layer for database interactions, we find ourselves using the same JDBC API methods from various components. This results in boilerplate code apart from the tight coupling. Numerous classes that belong to the Model part of our MVC 2 based application would require change if we make changes to our persistent storage. Imagine the scenario of moving from a RDBMS to a file based (XML or plain files for example) system for persistent storage! Without a separate data base access layer, we would definitely end up modifying a big chunk of our existing codebase apart from writing whole lot new!

Having a separate and uniform layer for data base access brings us benefits like:
1. a single layer for all components of the application to interact with database irrespective of the type of the database
2. decoupling the boilerplate database access calls from the other parts of application
3. easy to maintain and increases portability

In this little project, I will demonstrate a fully functional DAO implementation using a simple database.

## Example Implementation
In our example implementation, we first begin with designing our data model. To keep it simple, I will use a database to store names/titles of books. Our database will have a single table called 'Books':

|Book_ID|Title|
|---|---|
|1|Java|
|2|The Java Programming Language|

That's it. Very simple. The books database will be used by both command line and web based applications. So, it should support uniform access. We also want to see the beauty of this 'uniform access layer' while using two different databases - MySQL and Apache Derby. So, those are our two goals for this short project.

The DAO pattern suggests to use an object called 'Data Access Object' which acts as an interface for all our data access needs. This object provides the abstraction and further encapsulates the entire access to the data storage. For our object oriented application, a data access object is another object to interact with - which makes our life a lot easier.

Let's first see, how we would have to interact with our books database without a DAO.

Here's a simple SQL script to create the books database in MySQL 8.0. The script also inserts few books.

```sql
CREATE DATABASE booklib;
CREATE TABLE books (
  book_id UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  title varchar(32) NOT NULL 
);

INSERT INTO books(title) VALUES ('Java'), ('The Java Programming Language');
```
We represent a Book in our Java application as:
```java
public class Book {
  private int bookID;
  private String title;

  public Book() {

  }
}
```

The below code interacts with the database:
```java
import java.sql.*;

public class BooksDBTest {
  public static void main(String[] args) {
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/booklib?user=root&password=root");
        Statement stmt = conn.createStatement();
        String query = "SELECT book_id, title FROM books";
        ResultSet rs = stmt.executeQuery(query);
        while(rs.next()){         
             System.out.println("Book ID: " + rs.getInt("book_id") + " Title: " + rs.getString("title"));         
        }      
        rs.close();
        stmt.close();
        conn.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
  }
}
```
Output:
Book ID: 1 Title: Java
Book ID: 5 Title: The Java Programming Language

Now, let's imagine the underlying database has changed from MySQL to Derby and then later to a file. We should then have to rewrite the whole code. In the above code, we are setting up the low level mechanisms (URL, credentials etc.) to ineract with the database. We can hide this. Database implementor should be able to configure the database without affecting the code.

Following the DAO pattern, we shall first design a data access object that abstracts and encapsulates our persistent storage. Below code shows how we are going to interact with the database through a DAO:
```java
public class DAO {
  public void insertBook(Book b);
  public Book findBook(int bookID);
  public void updateBook(Book b);
  public void deleteBook(int bookID);
}

//Database client program
public class BookDBTest {
  public static void main(String[] args) {
    DAO booksDAO = new DAO();
    Book b = booksDAO.findBook(1);
    b = new Book();
    booksDAO.insertBook(b);
  }
}
```
There's no low level mechanism that we have to care about. It's purely object-oriented. We treat the persistence storage as an object and we interact with the object. We don't even know what the underlying database actually is! DAO has encapsulated the data access mechanisms and also abstracted the way we interact with the database. The client code is happy.
