# personal-finance-tracker-
Python CLI application for tracking income, expanses and spending analysis 
# Personal Finance Tracker
# A command-line application to track income and expenses
# Author: Your Name
# Built with Python 3

import json
import os
from datetime import datetime

DATA_FILE = "finances.json"

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {"transactions": [], "balance": 0}

def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=4)

def add_transaction(data, transaction_type, category, amount, description):
    transaction = {
        "id": len(data["transactions"]) + 1,
        "type": transaction_type,
        "category": category,
        "amount": float(amount),
        "description": description,
        "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }
    data["transactions"].append(transaction)
    if transaction_type == "income":
        data["balance"] += float(amount)
    else:
        data["balance"] -= float(amount)
    save_data(data)
    print(f"\n✓ {transaction_type.capitalize()} of ${amount} added successfully!")

def view_summary(data):
    print("\n" + "="*50)
    print("        FINANCIAL SUMMARY")
    print("="*50)
    total_income = sum(t["amount"] for t in data["transactions"] if t["type"] == "income")
    total_expenses = sum(t["amount"] for t in data["transactions"] if t["type"] == "expense")
    print(f"  Total Income:    ${total_income:.2f}")
    print(f"  Total Expenses:  ${total_expenses:.2f}")
    print(f"  Current Balance: ${data['balance']:.2f}")
    print("="*50)

def view_transactions(data):
    if not data["transactions"]:
        print("\nNo transactions found.")
        return
    print("\n" + "="*70)
    print(f"{'ID':<5} {'Type':<10} {'Category':<15} {'Amount':<10} {'Description':<20} {'Date'}")
    print("="*70)
    for t in data["transactions"]:
        symbol = "+" if t["type"] == "income" else "-"
        print(f"{t['id']:<5} {t['type']:<10} {t['category']:<15} {symbol}${t['amount']:<9.2f} {t['description']:<20} {t['date']}")
    print("="*70)

def view_by_category(data):
    categories = {}
    for t in data["transactions"]:
        cat = t["category"]
        if cat not in categories:
            categories[cat] = {"income": 0, "expense": 0}
        categories[cat][t["type"]] += t["amount"]
    print("\n" + "="*50)
    print("        SPENDING BY CATEGORY")
    print("="*50)
    for cat, amounts in categories.items():
        print(f"  {cat}:")
        if amounts["income"] > 0:
            print(f"    Income:  ${amounts['income']:.2f}")
        if amounts["expense"] > 0:
            print(f"    Expense: ${amounts['expense']:.2f}")
    print("="*50)

def delete_transaction(data, transaction_id):
    for i, t in enumerate(data["transactions"]):
        if t["id"] == transaction_id:
            if t["type"] == "income":
                data["balance"] -= t["amount"]
            else:
                data["balance"] += t["amount"]
            data["transactions"].pop(i)
            save_data(data)
            print(f"\n✓ Transaction {transaction_id} deleted successfully!")
            return
    print(f"\nTransaction {transaction_id} not found.")

def main():
    print("\n" + "="*50)
    print("    PERSONAL FINANCE TRACKER")
    print("    Manage your money smartly")
    print("="*50)

    data = load_data()

    while True:
        print("\nMAIN MENU")
        print("-"*30)
        print("1. Add Income")
        print("2. Add Expense")
        print("3. View All Transactions")
        print("4. View Financial Summary")
        print("5. View Spending by Category")
        print("6. Delete Transaction")
        print("7. Exit")
        print("-"*30)

        choice = input("Enter your choice (1-7): ").strip()

        if choice == "1":
            print("\n--- ADD INCOME ---")
            category = input("Category (salary/freelance/investment/other): ").strip()
            amount = input("Amount ($): ").strip()
            description = input("Description: ").strip()
            add_transaction(data, "income", category, amount, description)

        elif choice == "2":
            print("\n--- ADD EXPENSE ---")
            category = input("Category (food/rent/transport/entertainment/other): ").strip()
            amount = input("Amount ($): ").strip()
            description = input("Description: ").strip()
            add_transaction(data, "expense", category, amount, description)

        elif choice == "3":
            view_transactions(data)

        elif choice == "4":
            view_summary(data)

        elif choice == "5":
            view_by_category(data)

        elif choice == "6":
            view_transactions(data)
            try:
                tid = int(input("\nEnter Transaction ID to delete: "))
                delete_transaction(data, tid)
            except ValueError:
                print("Please enter a valid number.")

        elif choice == "7":
            print("\nGoodbye! Keep tracking your finances.")
            break

        else:
            print("\nInvalid choice. Please enter 1-7.")

if __name__ == "__main__":
    main()
