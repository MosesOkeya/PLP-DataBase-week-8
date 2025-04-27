--library.sql

-- Create authors table
CREATE TABLE authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    bio TEXT
);

-- Create books table
CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    isbn VARCHAR(20) UNIQUE NOT NULL,
    published_year YEAR,
    copies_available INT DEFAULT 0
);

-- Create book_authors (many-to-many)
CREATE TABLE book_authors (
    book_id INT,
    author_id INT,
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);

-- Create members table
CREATE TABLE members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    join_date DATE NOT NULL
);

-- Create loans table
CREATE TABLE loans (
    loan_id INT AUTO_INCREMENT PRIMARY KEY,
    book_id INT,
    member_id INT,
    loan_date DATE NOT NULL,
    return_date DATE,
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);

-- Sample Data
INSERT INTO authors (name, bio) VALUES
('J.K. Rowling', 'British author, best known for Harry Potter'),
('George Orwell', 'Known for 1984 and Animal Farm');

INSERT INTO books (title, isbn, published_year, copies_available) VALUES
('Harry Potter and the Sorcerer\'s Stone', '9780747532743', 1997, 5),
('1984', '9780451524935', 1949, 3);

INSERT INTO book_authors (book_id, author_id) VALUES
(1, 1),
(2, 2);

INSERT INTO members (name, email, join_date) VALUES
('Alice Johnson', 'alice@example.com', '2023-01-15'),
('Bob Smith', 'bob@example.com', '2023-02-20');

INSERT INTO loans (book_id, member_id, loan_date, return_date) VALUES
(1, 1, '2024-01-10', '2024-01-20'),
(2, 2, '2024-02-01', NULL);



---

-- a CRUD API



contact_group: M:N between contacts and groups


2. FastAPI Folder Structure

contact_api/
│
├── main.py
├── models.py
├── database.py
├── crud.py
├── requirements.txt
├── contact_db.sql
└── README.md

-- sample requirements.txt

fastapi
uvicorn
mysql-connector-python
sqlalchemy

--basic File Snippets

database.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = "mysql+mysqlconnector://root:password@localhost:3306/contact_db"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

models.py

from sqlalchemy import Column, Integer, String
from database import Base

class Contact(Base):
    __tablename__ = "contacts"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    phone = Column(String(20), unique=True)

crud.py

from sqlalchemy.orm import Session
from models import Contact

def get_contacts(db: Session):
    return db.query(Contact).all()

def create_contact(db: Session, contact_data):
    new_contact = Contact(**contact_data)
    db.add(new_contact)
    db.commit()
    db.refresh(new_contact)
    return new_contact

main.py

from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from database import SessionLocal, engine
from models import Base
import crud

app = FastAPI()
Base.metadata.create_all(bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/contacts")
def read_contacts(db: Session = Depends(get_db)):
    return crud.get_contacts(db)

@app.post("/contacts")
def add_contact(contact: dict, db: Session = Depends(get_db)):
    return crud.create_contact(db, contact)

Sample contact_db.sql

CREATE DATABASE contact_db;

USE contact_db;

CREATE TABLE contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE
);
