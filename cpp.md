# CODE:
```python
import tkinter as tk
from tkinter import messagebox, ttk
from decimal import Decimal, InvalidOperation
from datetime import date, datetime, timedelta
import mysql.connector
import matplotlib.pyplot as plt
import calendar

db = mysql.connector.connect(
    host="localhost",
    user="root",
    password="Radha9959",
    database="fingo"
)
cur = db.cursor()

DEFAULT_CATEGORIES = [
    "Job","Food", "Transport", "Education", "Entertainment", "Shopping",
    "Health", "Savings", "Travel", "Bills", "Other"
]

def calc_totals(user):
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='income'", (user,))
    inc = cur.fetchone()[0] or 0
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense'", (user,))
    exp = cur.fetchone()[0] or 0
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='savings'", (user,))
    sav = cur.fetchone()[0] or 0
    return Decimal(inc), Decimal(exp), Decimal(sav)

def is_last_day_of_month(d: date):
    return d.day == calendar.monthrange(d.year, d.month)[1]

def money_label(amount):
    return f"₹{amount:,.2f}"

def pie_autopct_factory(sizes):
    total = sum(sizes)
    def my_autopct(pct):
        val = pct / 100.0 * total
        return f"₹{val:,.2f}"
    return my_autopct

def register():
    username = entry_user.get().strip()
    password = entry_pass.get().strip()
    if not username or not password:
        messagebox.showerror("Error", "Enter username and password!")
        return
    cur.execute("SELECT username FROM users WHERE username=%s", (username,))
    if cur.fetchone():
        messagebox.showerror("Error", "Username already exists!")
        return
    cur.execute("INSERT INTO users VALUES (%s, %s)", (username, password))
    db.commit()
    messagebox.showinfo("Success", "Registered successfully!")

def login():
    username = entry_user.get().strip()
    password = entry_pass.get().strip()
    cur.execute("SELECT * FROM users WHERE username=%s AND password=%s", (username, password))
    if cur.fetchone():
        open_dashboard(username)
    else:
        messagebox.showerror("Error", "Invalid credentials!")


def open_dashboard(user):
    app.withdraw()
    dash = tk.Toplevel(app)
    dash.title(f"FinGo Dashboard - {user}")
    dash.geometry("700x450")
    inc, exp, sav = calc_totals(user)
    balance = inc - (exp + sav)

    tk.Label(dash, text=f"Welcome, {user}", font=("Arial", 16, "bold")).pack(pady=8)
    frame_info = tk.Frame(dash)
    frame_info.pack(pady=4)
    lbl_income = tk.Label(frame_info, text=f"Income: {money_label(inc)}")
    lbl_income.grid(row=0, column=0, sticky="w")
    lbl_expense = tk.Label(frame_info, text=f"Expense: {money_label(exp)}")
    lbl_expense.grid(row=1, column=0, sticky="w")
    lbl_savings = tk.Label(frame_info, text=f"Savings: {money_label(sav)}")
    lbl_savings.grid(row=2, column=0, sticky="w")
    lbl_balance = tk.Label(dash, text=f"Present Balance: {money_label(balance)}",font=("Arial", 14, "bold"), fg="Dark Green")
    lbl_balance.pack(pady=8)
    btn_frame = tk.Frame(dash)
    btn_frame.pack(pady=6)
    tk.Button(btn_frame, text="Add Transaction", width=22, command=lambda: add_trans(user, refresh)).grid(row=0, column=0)
    tk.Button(btn_frame, text="Add Goal", width=22, command=lambda: add_goal(user, refresh)).grid(row=0, column=1)
    tk.Button(btn_frame, text="Add Savings to Goal", width=22, command=lambda: add_savings_to_goal(user, refresh)).grid(row=1, column=0)
    tk.Button(btn_frame, text="Show Transactions", width=22, command=lambda: show_transactions(user)).grid(row=1, column=1)
    tk.Button(btn_frame, text="Show Summary (Pie Chart)", width=22, command=lambda: show_summary(user)).grid(row=2, column=0)
    tk.Button(btn_frame, text="Show Goals (Pie Charts)", width=22, command=lambda: show_goals(user)).grid(row=2, column=1)
    tk.Button(dash, text="Logout", width=18, command=lambda: [dash.destroy(), app.deiconify()]).pack(pady=14)
    tk.Label(dash, text="Goals:", font=("Arial", 12, "bold")).pack(pady=(6, 0))
    goals_frame = tk.Frame(dash)
    goals_frame.pack(padx=6, pady=4, fill="x")

    def load_goals():
        for w in goals_frame.winfo_children():
            w.destroy()
        cur.execute("SELECT name, target, saved FROM goals WHERE username=%s", (user,))
        rows = cur.fetchall()
        if not rows:
            tk.Label(goals_frame, text="No goals yet").pack(anchor="w")
        else:
            for name, target, saved in rows:
                perc = (saved / target * 100) if target > 0 else 0
                tk.Label(goals_frame, text=f"{name} — ₹{saved}/{target} ({perc:.0f}%)").pack(anchor="w")

    def refresh():
        nonlocal lbl_income, lbl_expense, lbl_savings, lbl_balance
        i, e, s = calc_totals(user)
        lbl_income.config(text=f"Income: {money_label(i)}")
        lbl_expense.config(text=f"Expense: {money_label(e)}")
        lbl_savings.config(text=f"Savings: {money_label(s)}")
        lbl_balance.config(text=f"Present Balance: {money_label(i - (e + s))}")
        load_goals()
    load_goals()
    show_monthly_report(user)

def show_monthly_report(user):
    today = date.today()
    if not is_last_day_of_month(today):
        return
    start_30 = today - timedelta(days=30)
    start_60 = today - timedelta(days=60)
    start_90 = today - timedelta(days=90)
    cur.execute(" SELECT category, amount FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_30, today))
    rows_30 = cur.fetchall()
    if rows_30:
        list_30 = "\n".join([f"- {cat} — {money_label(amt)}" for cat, amt in rows_30])
        total_30 = sum((Decimal(amt) for _, amt in rows_30), Decimal("0.0"))
    else:
        list_30 = "(No expenses in last 30 days)"
        total_30 = Decimal("0.0")
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_60, today))
    total_2m = Decimal(cur.fetchone()[0] or 0)
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_90, today))
    total_3m = Decimal(cur.fetchone()[0] or 0)
    msg = (
        f"📅 Monthly Spending Report — {today.strftime('%d %B %Y')}\n\n"
        f"🔻 Last 30 days (by transactions):\n{list_30}\n"
        f"➡ Total = {money_label(total_30)}\n\n"
        f"🔻 Last 2 months total = {money_label(total_2m)}\n"
        f"🔻 Last 3 months total = {money_label(total_3m)}\n\n"
        f"Track your spending and try saving more next month"
    )
    messagebox.showinfo("Monthly Expense Reminder", msg)

def add_trans(user, on_done):
    win = tk.Toplevel(app)
    win.title("Add Transaction")
    win.geometry("700x450")
    tk.Label(win, text="Type").pack(pady=4)
    t = ttk.Combobox(win, values=["income", "expense", "savings"], state="readonly")
    t.set("expense"); t.pack()
    tk.Label(win, text="Category").pack(pady=4)
    c = ttk.Combobox(win, values=DEFAULT_CATEGORIES, state="readonly")
    c.set(DEFAULT_CATEGORIES[0]); c.pack()
    tk.Label(win, text="Amount").pack(pady=4)
    a = tk.Entry(win); a.pack()
    tk.Label(win, text="Note (optional)").pack(pady=4)
    n = tk.Entry(win); n.pack()
    def save():
        try:
            amt = Decimal(a.get())
            if amt <= 0: raise InvalidOperation
        except:
            messagebox.showerror("Error", "Enter a valid amount")
            return
        cur.execute("""
            INSERT INTO transactions(username, type, category, amount, note, date)
            VALUES (%s, %s, %s, %s, %s, %s)
        """, (user, t.get(), c.get(), amt, n.get().strip(), date.today()))
        db.commit()
        messagebox.showinfo("Success", "Transaction added!")
        win.destroy()
        on_done()
    tk.Button(win, text="Save", command=save).pack(pady=10)

def add_goal(user, on_done):
    win = tk.Toplevel(app)
    win.title("Add Goal")
    win.geometry("700x450")
    tk.Label(win, text="Goal Name").pack(pady=4)
    name = tk.Entry(win); name.pack()
    tk.Label(win, text="Target Amount").pack(pady=4)
    target = tk.Entry(win); target.pack()
    tk.Label(win, text="Category").pack(pady=4)
    cat = ttk.Combobox(win, values=DEFAULT_CATEGORIES, state="readonly")
    cat.set(DEFAULT_CATEGORIES[0]); cat.pack()

    def save():
        try:
            targ = Decimal(target.get())
            if targ <= 0: raise InvalidOperation
        except:
            messagebox.showerror("Error", "Invalid target amount")
            return
        cur.execute("""
            INSERT INTO goals(username, name, target, saved, category)
            VALUES (%s, %s, %s, %s, %s)
        """, (user, name.get().strip() or "Unnamed", targ, 0, cat.get()))
        db.commit()
        messagebox.showinfo("Success", "Goal added!")
        win.destroy()
        on_done()
    tk.Button(win, text="Save", command=save).pack(pady=10)

def add_savings_to_goal(user, on_done):
    cur.execute("SELECT id, name FROM goals WHERE username=%s", (user,))
    rows = cur.fetchall()
    if not rows:
        messagebox.showinfo("Info", "No goals available")
        return
    win = tk.Toplevel(app)
    win.title("Add Savings to Goal")
    win.geometry("700x450")

    names = [r[1] for r in rows]
    tk.Label(win, text="Choose Goal").pack(pady=6)
    box = ttk.Combobox(win, values=names, state="readonly")
    box.set(names[0]); box.pack()

    tk.Label(win, text="Amount to Add").pack(pady=6)
    amt = tk.Entry(win); amt.pack()

    def save():
        try:
            amount = Decimal(amt.get())
            if amount <= 0: raise InvalidOperation
        except:
            messagebox.showerror("Error", "Invalid amount")
            return

        goal = [r for r in rows if r[1] == box.get()][0]
        goal_id = goal[0]
        cur.execute("UPDATE goals SET saved = saved + %s WHERE id=%s", (amount, goal_id))
        db.commit()
        cur.execute("INSERT INTO transactions(username, type, category, amount, note, date) VALUES (%s,'savings',(SELECT category FROM goals WHERE id=%s),%s,%s,%s)",(user, goal_id, amount, f"ToGoal:{box.get()}", date.today()))
        db.commit()
        cur.execute("SELECT target, saved FROM goals WHERE id=%s", (goal_id,))
        target, saved = cur.fetchone()
        if saved >= target:
            messagebox.showinfo("Goal Completed", f"Goal '{box.get()}' completed 🎉")
            cur.execute("DELETE FROM goals WHERE id=%s", (goal_id,))
            db.commit()
        win.destroy()
        on_done()
    tk.Button(win, text="Save", command=save).pack(pady=10)

def show_transactions(user):
    win = tk.Toplevel(app)
    win.title("All Transactions")
    win.geometry("700x450")

    table = ttk.Treeview(win, columns=("type", "category", "amount", "note", "date"), show="headings", height=15)
    for col in ("type", "category", "amount", "note", "date"):
        table.heading(col, text=col.capitalize())
    table.column("amount", width=80, anchor="e")
    table.pack(expand=True, fill="both", padx=6, pady=6)

    cur.execute("SELECT type, category, amount, note, date FROM transactions WHERE username=%s", (user,))
    for t in cur.fetchall():
        table.insert("", "end", values=(t[0], t[1], money_label(t[2]), t[3], t[4]))

def show_summary(user):
    inc, exp, sav = calc_totals(user)
    balance = inc - (exp + sav)
    sizes = [float(balance), float(exp + sav)]
    if sum(sizes) == 0:
        messagebox.showinfo("Info", "No data to show")
        return
    fig, ax = plt.subplots(figsize=(6, 6))
    ax.pie(sizes, labels=["Income", "Expense + Savings"], autopct=pie_autopct_factory(sizes), startangle=50)
    ax.set_title(f"Income and Expense\nTotal Income:{inc} ")
    plt.show()


def show_goals(user):
    cur.execute("SELECT name, target, saved FROM goals WHERE username=%s", (user,))
    rows = cur.fetchall()
    if not rows:
        messagebox.showinfo("Info", "No goals available")
        return
    for name, target, saved in rows:
        remaining = target - saved
        sizes = [float(saved), float(max(remaining, 0))]
        fig, ax = plt.subplots(figsize=(5, 5))
        ax.pie(sizes, labels=["Saved", "Remaining"], autopct=pie_autopct_factory(sizes), startangle=90)
        ax.set_title(name)
        plt.show()


app = tk.Tk()
app.title("FinGo Login")
app.geometry("700x450")

tk.Label(app, text="FinGo App", font=("Menlo", 18, "bold")).pack(pady=10)
tk.Label(app, text="Username").pack()
entry_user = tk.Entry(app); entry_user.pack()
tk.Label(app, text="Password").pack()
entry_pass = tk.Entry(app, show="*"); entry_pass.pack()
tk.Button(app, text="Login", width=20, command=login).pack(pady=6)
tk.Button(app, text="Register", width=20, command=register).pack()
app.mainloop()```

