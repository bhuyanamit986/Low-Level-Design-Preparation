# 📚 Project: Library Management System

> **Learn by Building** - A complete library management system

## 📋 Requirements

- Manage books and members
- Book borrowing and returns
- Fine calculation for overdue books
- Search functionality
- Multiple copies per book

## 🏗️ Key Classes

```java
public class Book {
    private final String isbn;
    private final String title;
    private final String author;
    private final String publisher;
    private final List<BookCopy> copies;
}

public class BookCopy {
    private final String barcode;
    private final Book book;
    private CopyStatus status;
    private Loan currentLoan;
}

public class Member {
    private final String memberId;
    private final String name;
    private final MemberType type;
    private final List<Loan> loans;
    
    public int getMaxBooksAllowed() {
        return type.getMaxBooks();
    }
}

public class Loan {
    private final BookCopy copy;
    private final Member member;
    private final LocalDate borrowDate;
    private final LocalDate dueDate;
    private LocalDate returnDate;
    
    public boolean isOverdue() {
        return returnDate == null && LocalDate.now().isAfter(dueDate);
    }
    
    public double calculateFine(double finePerDay) {
        if (!isOverdue()) return 0;
        long days = ChronoUnit.DAYS.between(dueDate, LocalDate.now());
        return days * finePerDay;
    }
}

public class Library {
    private final Map<String, Book> booksByIsbn;
    private final Map<String, Member> members;
    private final List<Loan> activeLoans;
    
    public Loan borrowBook(String memberId, String barcode);
    public double returnBook(String barcode);
    public List<Book> searchByTitle(String title);
    public List<Book> searchByAuthor(String author);
}
```

## 📝 Implementation Guide

See full implementation in the source files.

## 🧪 Concepts Covered

- ✅ Entity Relationships
- ✅ Domain Modeling
- ✅ Business Rules
- ✅ Search Functionality
