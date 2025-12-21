# **Imports**

```python
import tkinter as tk
from tkinter import messagebox, ttk
from decimal import Decimal, InvalidOperation
from datetime import date, datetime, timedelta
import mysql.connector
import matplotlib.pyplot as plt
import calendar
```

* `tkinter` → Main GUI module for windows, buttons, labels.
* `messagebox` → For pop-up messages like errors or success.
* `ttk` → For styled widgets like Combobox, Treeview (table-like display).
* `Decimal` → Precise number for money (avoids floating-point errors like 0.1+0.2 problem).
* `InvalidOperation` → Exception if decimal conversion fails (like negative money input).
* `date, datetime, timedelta` → Handling dates for transactions and goals.
* `mysql.connector` → Connect Python to MySQL database.
* `matplotlib.pyplot` → Draw charts, e.g., pie charts for income/expense.
* `calendar` → To check month details like last day of month.

---

# **Database connection**

```python
db = mysql.connector.connect(
    host="localhost",
    user="root",
    password="Radha9959",
    database="fingo"
)
cur = db.cursor()
```

* Connects to MySQL **database `fingo`**.
* `cur` → cursor object, used to execute SQL queries.
* Every time we want data (`SELECT`) or save data (`INSERT/UPDATE`), we use `cur.execute()`.

---

# **Default categories**

```python
DEFAULT_CATEGORIES = [
    "Job","Food", "Transport", "Education", "Entertainment", "Shopping",
    "Health", "Savings", "Travel", "Bills", "Other"
]
```

* Predefined categories for **transactions** and **goals**.
* Used in **dropdown boxes** (`ttk.Combobox`) when the user adds transactions or goals.

---

# **1. Calculate totals function**

```python
def calc_totals(user):
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='income'", (user,))
    inc = cur.fetchone()[0] or 0
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense'", (user,))
    exp = cur.fetchone()[0] or 0
    cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='savings'", (user,))
    sav = cur.fetchone()[0] or 0
    return Decimal(inc), Decimal(exp), Decimal(sav)
```

* Gets **total income, expense, and savings** for a given user.
* **`%s`** → placeholder in SQL query.
* `(user,)` → tuple required by `mysql.connector`. Without comma, it’s just a value, not a tuple.
* `fetchone()[0]` → returns first column of first row (the sum).
* `or 0` → ensures zero if the database has no records.
* Returns values as **Decimal** for precise money calculations.

---

# **2. Check Last Day of Month**

```python
def is_last_day_of_month(d: date):
    return d.day == calendar.monthrange(d.year, d.month)[1]
```

* Checks if a date `d` is **the last day of the month**.
* `calendar.monthrange(year, month)[1]` → returns **number of days in month**.

# **3. Format Money Label**

```python
def money_label(amount):
    return f"₹{amount:,.2f}"
```

* Formats a number as **currency with comma separator and 2 decimals**, e.g., 1234 → `₹1,234.00`.

---

# **4. Pie chart helper function**

```python
def pie_autopct_factory(sizes):
    total = sum(sizes)
    def my_autopct(pct):
        val = pct / 100.0 * total
        return f"₹{val:,.2f}"
    return my_autopct
```

* Used for pie charts.
* Converts percentages to actual ₹ values.
* Example: 40% of ₹1000 → shows ₹400 on chart.
* `sizes` = list of values (Income, Expense+Savings).

---

# **5. Register function**

```python
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
```

* Step-by-step:

  1. Gets user input.
  2. Removes extra spaces (`strip()`).
  3. Checks if input is empty → show error.
  4. Checks if username exists in DB → show error.
  5. Inserts new user into `users` table.
  6. `db.commit()` → saves changes to database.
  7. Shows success pop-up.

---

# **6. Login function**

```python
def login():
    username = entry_user.get().strip()
    password = entry_pass.get().strip()
    cur.execute("SELECT * FROM users WHERE username=%s AND password=%s", (username, password))
    if cur.fetchone():
        open_dashboard(username)
    else:
        messagebox.showerror("Error", "Invalid credentials!")
```

* Checks username & password in `users` table.
* If matched → open dashboard.
* If not → show error.

---

# **7. Open dashboard**

```python
def open_dashboard(user):
    app.withdraw()
    dash = tk.Toplevel(app)
    dash.title(f"FinGo Dashboard - {user}")
    dash.geometry("700x450")
```

* `app.withdraw()` → hides login window.
* `Toplevel(app)` → creates new window.
* Sets window **title** and **size**.

---

### **7a. Calculate balance and show labels**

```python
inc, exp, sav = calc_totals(user)
balance = inc - (exp + sav)

tk.Label(dash, text=f"Welcome, {user}", font=("Arial", 16, "bold")).pack(pady=8)
```

* Gets totals for logged-in user.
* Balance = income − (expense + savings).
* Shows **welcome message**.

```python
frame_info = tk.Frame(dash)
frame_info.pack(pady=4)

lbl_income = tk.Label(frame_info, text=f"Income: {money_label(inc)}")
lbl_income.grid(row=0, column=0, sticky="w")
lbl_expense = tk.Label(frame_info, text=f"Expense: {money_label(exp)}")
lbl_expense.grid(row=1, column=0, sticky="w")
lbl_savings = tk.Label(frame_info, text=f"Savings: {money_label(sav)}")
lbl_savings.grid(row=2, column=0, sticky="w")
```

* `Frame` → container to organize widgets.
* `grid(row, column)` → positions widgets in table-like layout.
* `sticky="w"` → aligns text to left (west).

```python
lbl_balance = tk.Label(dash, text=f"Present Balance: {money_label(balance)}",font=("Arial", 14, "bold"), fg="Dark Green")
lbl_balance.pack(pady=8)
```

* Shows **balance** in green color.

---

### **7b. Buttons frame**

```python
btn_frame = tk.Frame(dash)
btn_frame.pack(pady=6)

tk.Button(btn_frame, text="Add Transaction", width=22, command=lambda: add_trans(user, refresh)).grid(row=0, column=0)
tk.Button(btn_frame, text="Add Goal", width=22, command=lambda: add_goal(user, refresh)).grid(row=0, column=1)
tk.Button(btn_frame, text="Add Savings to Goal", width=22, command=lambda: add_savings_to_goal(user, refresh)).grid(row=1, column=0)
tk.Button(btn_frame, text="Show Transactions", width=22, command=lambda: show_transactions(user)).grid(row=1, column=1)
tk.Button(btn_frame, text="Show Summary (Pie Chart)", width=22, command=lambda: show_summary(user)).grid(row=2, column=0)
tk.Button(btn_frame, text="Show Goals (Pie Charts)", width=22, command=lambda: show_goals(user)).grid(row=2, column=1)
```

* **6 buttons**:

  1. Add Transaction → opens transaction window.
  2. Add Goal → opens goal window.
  3. Add Savings → add to existing goal.
  4. Show Transactions → table of transactions.
  5. Show Summary → pie chart of income vs expense+savings.
  6. Show Goals → pie chart for each goal.
* `lambda: add_trans(user, refresh)` → pass user + refresh function as callback.

---

### **7c. Logout and Goals section**

```python
tk.Button(dash, text="Logout", width=18, command=lambda: [dash.destroy(), app.deiconify()]).pack(pady=14)
tk.Label(dash, text="Goals:", font=("Arial", 12, "bold")).pack(pady=(6, 0))
goals_frame = tk.Frame(dash)
goals_frame.pack(padx=6, pady=4, fill="x")
```

* Logout → destroys dashboard and **shows login window again**.
* `goals_frame` → container to display goals dynamically.

---

### **7d. Load goals function**

```python
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
```

* Clears old goal widgets.
* Fetches goals from DB for user.
* If no goals → show “No goals yet”.
* Otherwise → show **goal name, saved amount, target, and % completed**.

---

### **7e. Refresh function**

```python
def refresh():
    nonlocal lbl_income, lbl_expense, lbl_savings, lbl_balance
    i, e, s = calc_totals(user)
    lbl_income.config(text=f"Income: {money_label(i)}")
    lbl_expense.config(text=f"Expense: {money_label(e)}")
    lbl_savings.config(text=f"Savings: {money_label(s)}")
    lbl_balance.config(text=f"Present Balance: {money_label(i - (e + s))}")
    load_goals()
```

* Updates **income, expense, savings, balance, and goals**.
* `nonlocal` → allows updating outer-scope labels.
* Called after any **transaction/goal change**.


# **8. Monthly Expense Reminder**

```python
def show_monthly_report(user):
    today = date.today()
    if not is_last_day_of_month(today):
        return
```

* Checks if **today is last day of month**.
* If not → return, do nothing.

```python
start_30 = today - timedelta(days=30)
start_60 = today - timedelta(days=60)
start_90 = today - timedelta(days=90)
```

* Calculate **date ranges** for last 30, 60, 90 days.

```python
cur.execute(" SELECT category, amount FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_30, today))
rows_30 = cur.fetchall()
```

* Fetch **all expenses in last 30 days**.

```python
if rows_30:
    list_30 = "\n".join([f"- {cat} — {money_label(amt)}" for cat, amt in rows_30])
    total_30 = sum((Decimal(amt) for _, amt in rows_30), Decimal("0.0"))
else:
    list_30 = "(No expenses in last 30 days)"
    total_30 = Decimal("0.0")
```

* Formats list of expenses.
* Computes **total 30-day expense**.

```python
cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_60, today))
total_2m = Decimal(cur.fetchone()[0] or 0)
cur.execute("SELECT SUM(amount) FROM transactions WHERE username=%s AND type='expense' AND date BETWEEN %s AND %s", (user, start_90, today))
total_3m = Decimal(cur.fetchone()[0] or 0)
```

* Calculates **total expense for last 2 months and 3 months**.

```python
msg = (
    f"📅 Monthly Spending Report — {today.strftime('%d %B %Y')}\n\n"
    f"🔻 Last 30 days (by transactions):\n{list_30}\n"
    f"➡ Total = {money_label(total_30)}\n\n"
    f"🔻 Last 2 months total = {money_label(total_2m)}\n"
    f"🔻 Last 3 months total = {money_label(total_3m)}\n\n"
    f"Track your spending and try saving more next month"
)
messagebox.showinfo("Monthly Expense Reminder", msg)
```

* Creates **user-friendly message** with emojis.
* Shows **pop-up with monthly spending summary**.


---

# **9. Add Transaction window**

```python
win = tk.Toplevel(app)
win.title("Add Transaction")
win.geometry("700x450")
```

* Opens a new window.

```python
t = ttk.Combobox(win, values=["income", "expense", "savings"], state="readonly")
t.set("expense"); t.pack()
```

* Dropdown for transaction type.
* Default → expense.

```python
c = ttk.Combobox(win, values=DEFAULT_CATEGORIES, state="readonly")
c.set(DEFAULT_CATEGORIES[0]); c.pack()
```

* Dropdown for category.

```python
a = tk.Entry(win); a.pack()
n = tk.Entry(win); n.pack()
```

* `a` → amount input
* `n` → note input

```python
def save():
    try:
        amt = Decimal(a.get())
        if amt <= 0: raise InvalidOperation
    except:
        messagebox.showerror("Error", "Enter a valid amount")
        return
```

* Converts input to **Decimal**.
* Validates positive number.

```python
cur.execute("""
    INSERT INTO transactions(username, type, category, amount, note, date)
    VALUES (%s, %s, %s, %s, %s, %s)
""", (user, t.get(), c.get(), amt, n.get().strip(), date.today()))
db.commit()
```

* Saves transaction in DB.

```python
messagebox.showinfo("Success", "Transaction added!")
win.destroy()
on_done()
```

* Pop-up → closes window → refresh dashboard.

# **10. Add Goal (`add_goal`)**

```python
def add_goal(user, on_done):
    win = tk.Toplevel(app)
    win.title("Add Goal")
    win.geometry("700x450")
```

* Opens a **new window** (`Toplevel`) for adding a goal.
* `win.title` → window title.
* `win.geometry` → size 700x450 pixels.

```python
tk.Label(win, text="Goal Name").pack(pady=4)
name = tk.Entry(win); name.pack()
```

* Label prompts user to **enter goal name**.
* `Entry` → text input box.
* `pack(pady=4)` → adds vertical spacing of 4 pixels.

```python
tk.Label(win, text="Target Amount").pack(pady=4)
target = tk.Entry(win); target.pack()
```

* Label prompts user to **enter target money** for the goal.
* `Entry` stores input as string; later converted to Decimal.

```python
tk.Label(win, text="Category").pack(pady=4)
cat = ttk.Combobox(win, values=DEFAULT_CATEGORIES, state="readonly")
cat.set(DEFAULT_CATEGORIES[0]); cat.pack()
```

* Label prompts user to **choose category**.
* `ttk.Combobox` → dropdown menu.
* `state="readonly"` → user cannot type custom value.
* Default value is first category from `DEFAULT_CATEGORIES`.

---

### **10a. Save function inside Add Goal**

```python
def save():
    try:
        targ = Decimal(target.get())
        if targ <= 0: raise InvalidOperation
    except:
        messagebox.showerror("Error", "Invalid target amount")
        return
```

* Converts target input to `Decimal` → precise money type.
* Checks **positive number**.
* If fails → shows error pop-up.

```python
cur.execute("""
    INSERT INTO goals(username, name, target, saved, category)
    VALUES (%s, %s, %s, %s, %s)
""", (user, name.get().strip() or "Unnamed", targ, 0, cat.get()))
db.commit()
```

* Inserts goal into **goals table**.
* `saved` = 0 initially.
* `name.get().strip() or "Unnamed"` → if user leaves name blank, use `"Unnamed"`.
* `db.commit()` → saves to database.

```python
messagebox.showinfo("Success", "Goal added!")
win.destroy()
on_done()
```

* Success pop-up.
* Closes add goal window.
* Calls `on_done()` → refresh dashboard to show new goal.

```python
tk.Button(win, text="Save", command=save).pack(pady=10)
```

* Button to save goal → triggers `save()` function.

---

# **11. Add Savings to Goal (`add_savings_to_goal`)**

```python
cur.execute("SELECT id, name FROM goals WHERE username=%s", (user,))
rows = cur.fetchall()
if not rows:
    messagebox.showinfo("Info", "No goals available")
    return
```

* Fetch **all goals** of user from DB.
* If no goals → show info pop-up → exit function.

```python
win = tk.Toplevel(app)
win.title("Add Savings to Goal")
win.geometry("700x450")
```

* Opens **new window** for adding money to a goal.

```python
names = [r[1] for r in rows]
tk.Label(win, text="Choose Goal").pack(pady=6)
box = ttk.Combobox(win, values=names, state="readonly")
box.set(names[0]); box.pack()
```

* Label → prompts user to **select goal**.
* Combobox → dropdown with goal names.
* Default selection → first goal in list.

```python
tk.Label(win, text="Amount to Add").pack(pady=6)
amt = tk.Entry(win); amt.pack()
```

* Entry to input **amount to add**.

---

### **11a. Save function inside Add Savings**

```python
def save():
    try:
        amount = Decimal(amt.get())
        if amount <= 0: raise InvalidOperation
    except:
        messagebox.showerror("Error", "Invalid amount")
        return
```

* Converts input to `Decimal`.
* Validates **positive number**.

```python
goal = [r for r in rows if r[1] == box.get()][0]
goal_id = goal[0]
```

* Finds the **selected goal** from DB results.
* `goal_id` → unique goal ID in table.

```python
cur.execute("UPDATE goals SET saved = saved + %s WHERE id=%s", (amount, goal_id))
db.commit()
```

* Updates `saved` field → adds the entered amount to goal.
* `db.commit()` → saves change.

```python
cur.execute("INSERT INTO transactions(username, type, category, amount, note, date) VALUES (%s,'savings',(SELECT category FROM goals WHERE id=%s),%s,%s,%s)",
            (user, goal_id, amount, f"ToGoal:{box.get()}", date.today()))
db.commit()
```

* Adds **transaction of type 'savings'** for record keeping.
* Note = `ToGoal:GoalName`.
* Category is taken from goal category.

```python
cur.execute("SELECT target, saved FROM goals WHERE id=%s", (goal_id,))
target, saved = cur.fetchone()
if saved >= target:
    messagebox.showinfo("Goal Completed", f"Goal '{box.get()}' completed 🎉")
    cur.execute("DELETE FROM goals WHERE id=%s", (goal_id,))
    db.commit()
```

* Checks if **goal is completed**.
* If completed → shows pop-up → deletes goal from DB.

```python
win.destroy()
on_done()
```

* Closes window → refresh dashboard.

```python
tk.Button(win, text="Save", command=save).pack(pady=10)
```

* Button triggers `save()`.

---

# **12. Show Transactions (`show_transactions`)**

```python
win = tk.Toplevel(app)
win.title("All Transactions")
win.geometry("700x450")
```

* Opens **new window** to display all transactions.

```python
table = ttk.Treeview(win, columns=("type", "category", "amount", "note", "date"), show="headings", height=15)
for col in ("type", "category", "amount", "note", "date"):
    table.heading(col, text=col.capitalize())
table.column("amount", width=110, anchor="e")
table.pack(expand=True, fill="both", padx=6, pady=6)
```

* `Treeview` → table-like widget.
* Columns: type, category, amount, note, date.
* `anchor="e"` → right-align amount column.
* `pack(expand=True, fill="both")` → fills window.

```python
cur.execute("SELECT type, category, amount, note, date FROM transactions WHERE username=%s", (user,))
for t in cur.fetchall():
    table.insert("", "end", values=(t[0], t[1], money_label(t[2]), t[3], t[4]))
```

* Fetch all transactions → insert rows into table.
* Amount formatted using `money_label`.

---

# **13. Show Summary (`show_summary`)**

```python
inc, exp, sav = calc_totals(user)
sizes = [float(inc), float(exp + sav)]
if sum(sizes) == 0:
    messagebox.showinfo("Info", "No data to show")
    return
```

* Calculate totals.
* Sizes → [Income, Expense+Savings].
* If no data → show pop-up.

```python
fig, ax = plt.subplots(figsize=(6, 6))
ax.pie(sizes, labels=["Total Income", "Expense + Savings"], autopct=pie_autopct_factory(sizes), startangle=90)
ax.set_title("Income and Expense")
plt.show()
```

* Draw **pie chart** with Matplotlib.
* `autopct` → shows ₹ values using helper function.
* `startangle=90` → rotates start of pie chart.

---

# **14. Show Goals (`show_goals`)**

```python
cur.execute("SELECT name, target, saved FROM goals WHERE username=%s", (user,))
rows = cur.fetchall()
if not rows:
    messagebox.showinfo("Info", "No goals available")
    return
```

* Fetch all goals of user.
* If none → show pop-up → exit function.

```python
for name, target, saved in rows:
    remaining = target - saved
    sizes = [float(saved), float(max(remaining, 0))]
    fig, ax = plt.subplots(figsize=(5, 5))
    ax.pie(sizes, labels=["Saved", "Remaining"], autopct=pie_autopct_factory(sizes), startangle=90)
    ax.set_title(name)
    plt.show()
```

* Loops through each goal.
* Pie chart → **Saved vs Remaining amount**.
* Shows a separate chart for each goal.

---

# **Main App (Login Window)**

```python
app = tk.Tk()
app.title("FinGo Login")
app.geometry("700x450")
```

* Creates **main login window**.
* `Tk()` → main root window.
* Sets title & size.

```python
tk.Label(app, text="FinGo App", font=("Menlo", 18, "bold")).pack(pady=10)
tk.Label(app, text="Username").pack()
entry_user = tk.Entry(app); entry_user.pack()
tk.Label(app, text="Password").pack()
entry_pass = tk.Entry(app, show="*"); entry_pass.pack()
```

* Labels → app name, username, password.
* Entries → user inputs.
* `show="*"` → hide password text.

```python
tk.Button(app, text="Login", width=20, command=login).pack(pady=6)
tk.Button(app, text="Register", width=20, command=register).pack()
```

* Login button → calls `login()` function.
* Register button → calls `register()` function.

```python
app.mainloop()
```

* Starts GUI **event loop**, waits for user interaction.

