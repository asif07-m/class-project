

import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt


# Database class
class Inventory:

    def __init__(self):
        self.con = sqlite3.connect("inventory.db")
        self.create_tables()

    def create_tables(self):
        cur = self.con.cursor()

        cur.execute("""
            CREATE TABLE IF NOT EXISTS products(
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT,
                category TEXT,
                price REAL,
                stock INTEGER
            )
        """)

        cur.execute("""
            CREATE TABLE IF NOT EXISTS sales(
                product TEXT,
                category TEXT,
                quantity INTEGER,
                total REAL
            )
        """)

        self.con.commit()

    def add_product(self, name, category, price, stock):
        self.con.execute(
            "INSERT INTO products(name,category,price,stock) VALUES(?,?,?,?)",
            (name, category, price, stock)
        )
        self.con.commit()

    def get_products(self):
        return self.con.execute(
            "SELECT * FROM products"
        ).fetchall()

    def sell_product(self, pid, qty):

        cur = self.con.cursor()

        product = cur.execute(
            "SELECT name,category,price,stock FROM products WHERE id=?",
            (pid,)
        ).fetchone()

        if not product:
            return False, "Product not found"

        name, category, price, stock = product

        if qty > stock:
            return False, "Not enough stock"

        total = price * qty

        cur.execute(
            "INSERT INTO sales VALUES(?,?,?,?)",
            (name, category, qty, total)
        )

        cur.execute(
            "UPDATE products SET stock=stock-? WHERE id=?",
            (qty, pid)
        )

        self.con.commit()

        return True, total

    def get_sales(self):
        return self.con.execute(
            "SELECT * FROM sales"
        ).fetchall()


# GUI class
class App:

    def __init__(self, root):

        self.root = root
        self.root.title("Inventory Management System")
        self.root.geometry("800x550")

        self.db = Inventory()

        tk.Label(
            root,
            text="Inventory Management & Sales",
            font=("Arial", 18, "bold")
        ).pack(pady=10)

        # Product input
        frame = tk.Frame(root)
        frame.pack(pady=10)

        tk.Label(frame, text="Name").grid(row=0, column=0)
        tk.Label(frame, text="Category").grid(row=0, column=2)
        tk.Label(frame, text="Price").grid(row=1, column=0)
        tk.Label(frame, text="Stock").grid(row=1, column=2)

        self.name = tk.Entry(frame)
        self.category = tk.Entry(frame)
        self.price = tk.Entry(frame)
        self.stock = tk.Entry(frame)

        self.name.grid(row=0, column=1)
        self.category.grid(row=0, column=3)
        self.price.grid(row=1, column=1)
        self.stock.grid(row=1, column=3)

        tk.Button(
            frame,
            text="Add Product",
            command=self.add
        ).grid(row=2, column=1, pady=10)

        # Product table
        columns = ("ID", "Name", "Category", "Price", "Stock")

        self.table = ttk.Treeview(
            root,
            columns=columns,
            show="headings"
        )

        for col in columns:
            self.table.heading(col, text=col)
            self.table.column(col, width=120)

        self.table.pack(pady=10)

        # Sales section
        sales = tk.Frame(root)
        sales.pack(pady=10)

        tk.Label(sales, text="Product ID").grid(row=0, column=0)
        tk.Label(sales, text="Quantity").grid(row=0, column=2)

        self.pid = tk.Entry(sales, width=10)
        self.qty = tk.Entry(sales, width=10)

        self.pid.grid(row=0, column=1, padx=5)
        self.qty.grid(row=0, column=3, padx=5)

        tk.Button(
            sales,
            text="Record Sale",
            command=self.sell
        ).grid(row=0, column=4, padx=10)

        tk.Button(
            sales,
            text="Show Analytics",
            command=self.analytics
        ).grid(row=0, column=5)

        self.load()


    # Add product
    def add(self):

        try:
            self.db.add_product(
                self.name.get(),
                self.category.get(),
                float(self.price.get()),
                int(self.stock.get())
            )

            messagebox.showinfo(
                "Success",
                "Product added"
            )

            self.load()

        except ValueError:
            messagebox.showerror(
                "Error",
                "Enter valid price and stock"
            )


    # Display products
    def load(self):

        for item in self.table.get_children():
            self.table.delete(item)

        for product in self.db.get_products():
            self.table.insert(
                "",
                "end",
                values=product
            )


    # Record sale
    def sell(self):

        try:

            success, result = self.db.sell_product(
                int(self.pid.get()),
                int(self.qty.get())
            )

            if success:

                messagebox.showinfo(
                    "Sale",
                    f"Sale completed\nAmount: ₹{result:.2f}"
                )

                self.load()

            else:
                messagebox.showerror(
                    "Error",
                    result
                )

        except ValueError:

            messagebox.showerror(
                "Error",
                "Enter valid values"
            )


    # Analytics
    def analytics(self):

        sales = self.db.get_sales()

        if not sales:
            messagebox.showinfo(
                "Analytics",
                "No sales data available"
            )
            return

        df = pd.DataFrame(
            sales,
            columns=[
                "Product",
                "Category",
                "Quantity",
                "Total"
            ]
        )

        revenue = df["Total"].sum()

        best_product = (
            df.groupby("Product")["Quantity"]
            .sum()
            .idxmax()
        )

        messagebox.showinfo(
            "Sales Analytics",
            f"Total Revenue: ₹{revenue:.2f}\n"
            f"Best Selling Product: {best_product}"
        )

        category_sales = (
            df.groupby("Category")["Total"]
            .sum()
        )

        category_sales.plot(
            kind="bar",
            title="Category-wise Sales"
        )

        plt.xlabel("Category")
        plt.ylabel("Sales")
        plt.tight_layout()
        plt.show()


# Start application
root = tk.Tk()
app = App(root)
root.mainloop()