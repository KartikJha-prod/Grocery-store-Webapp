# Grocery-store-Webapp
Grocery Store Web App (Flask + MySQL)
A small teaching project that shows how to build a database-driven web app with Python, Flask and MySQL. The app displays four database tables (products, orders, suppliers, customers) on one page and lets you add new records through simple forms.
This code accompanies a 3-part tutorial series on DEV Community:
Part 1: How to use MySQL
Part 2: How to connect a database to Python
Part 3: How to use Flask to make the actual web app
Note: This is sample code made for learning, not a production-ready application.
Features
Live view of the products, orders, suppliers and customers tables
Forms to add a product, order, supplier or customer
Orders pick up the product's current price automatically
Output is escaped with html.escape so user input can't inject HTML
Database errors are caught, rolled back and reported
Tech Stack
Python 3.9+
Flask
pandas
mysql-connector-python
MySQL 8.0.13 or newer (the customers table uses DEFAULT (CURRENT_DATE))
Project Structure
.
├── app.py                      # Flask application (routes and HTML)
├── database_grocery_store.py   # connecting_database() helper
└── README.md
Setup
1. Clone the repository and create a virtual environment
git clone <your-repo-url>
cd <your-repo-folder>

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
2. Install dependencies
pip install flask pandas mysql-connector-python
3. Create the database
Log in to MySQL and create the database and tables:
CREATE DATABASE Grocery_store_db;
USE Grocery_store_db;

CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    join_date DATE DEFAULT (CURRENT_DATE)
);

CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    description VARCHAR(255)
);

CREATE TABLE suppliers (
    supplier_id INT AUTO_INCREMENT PRIMARY KEY,
    supplier_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20)
);

CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category_id INT NOT NULL,
    supplier_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (category_id) REFERENCES categories(category_id),
    FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
Sample data is included in Part 1 of the tutorial. Add at least one category and one supplier before adding products, because products reference them through foreign keys.
4. Add the connection helper
Create database_grocery_store.py next to app.py. It must expose a connecting_database() function that returns a MySQL connection:
import os

import mysql.connector


def connecting_database():
    return mysql.connector.connect(
        host=os.environ.get("DB_HOST", "localhost"),
        user=os.environ.get("DB_USER", "root"),
        password=os.environ.get("DB_PASSWORD", ""),
        database=os.environ.get("DB_NAME", "Grocery_store_db"),
        port=int(os.environ.get("DB_PORT", 3306)),
    )
Set your credentials as environment variables instead of hardcoding them:
export DB_USER="your_username"
export DB_PASSWORD="your_password"
On Windows (PowerShell): $env:DB_USER = "your_username"
5. Run the app
python app.py
Open http://127.0.0.1:5000 in your browser.
Routes
Route
Method
Description
/
GET
Shows all four tables and the "add record" forms
/add-product
POST
Inserts a new product
/add-order
POST
Creates an order and one order line at the current product price
/add-supplier
POST
Inserts a new supplier
/add-customer
POST
Inserts a new customer
After each successful insert, the app redirects back to / so the tables refresh.
Troubleshooting
ModuleNotFoundError: No module named 'database_grocery_store': create the helper file from step 4 in the same folder as app.py.
Access denied for user: check your DB_USER and DB_PASSWORD values.
Unknown database 'Grocery_store_db': run the SQL in step 3 first.
Foreign key error when adding a product: the category ID or supplier ID you entered doesn't exist yet.
pandas shows a UserWarning about SQLAlchemy: this is harmless. The app still works.
Security Notes
app.run(debug=True) is for development only. The debugger can let others run code on your server. For production, use a WSGI server such as Gunicorn and set debug=False.
Never commit real database passwords to Git. Use environment variables or a .env file listed in .gitignore.
Ideas for Next Steps
Add a delete or edit option for each table
Move the HTML into Flask templates (Jinja2) instead of an f-string
Let an order include several products and link it to a customer
Add product images and styling
Author
Written by KartikJha-prod. Feedback and suggestions are welcome in the tutorial comments on DEV Community