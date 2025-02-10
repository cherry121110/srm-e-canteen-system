Code for the project:
from flask import Flask, render_template, request, redirect, url_for
import mysql.connector
from datetime import datetime, timedelta
from flask import session
app = Flask(_name_)
# Database configuration
db_config = {
 'host': 'localhost',
 'port': 3306,
 'user': 'root',
 'password': 'aseena',
 'database': 'e_canteen_database'
}
# Connect to the database
conn = mysql.connector.connect(**db_config)
cursor = conn.cursor()
# Predefined admin password
ADMIN_PASSWORD = "1234@"
56
@app.route('/')
def home():
 return render_template('home.html')
@app.route('/login', methods=['POST'])
def login_post():
 global username
 username = request.form.get('username')
 password = request.form.get('password')
 
 # Check if user is admin
 sql = "SELECT * FROM admin WHERE username = %s AND password = 
%s"
 cursor.execute(sql, (username, password))
 admin = cursor.fetchone()
 if admin:
 return 'Admin login successful'
 else:
 # Check username and password against regular users table
 sql = "SELECT * FROM USER WHERE username = %s AND password = 
%s"
 cursor.execute(sql, (username, password))
 user = cursor.fetchone()
 
 if user:
 return render_template('shop.html')
 else:
57
 return 'Invalid username or password'
@app.route('/login', methods=['GET'])
def login():
 return render_template('login.html')
@app.route('/cart')
def display_cart():
 try:
 # Fetch data from the cart table
 cursor.execute("SELECT * FROM cart")
 cart_items = cursor.fetchall()
 
 # Create a list to store the details of cart items
 cart_details = []
 # Fetch item details from the items table for each item in the cart
 for item in cart_items:
 item_id = item[0]
 
 # Fetch item details using item_id
 cursor.execute("SELECT * FROM items WHERE item_id = %s", 
(item_id,))
 item_details = cursor.fetchone()
 
 # Append item details to the cart_details list
 if item_details:
 cart_details.append(item_details)
 else:
58
 print("No item details found for item_id:", item_id)
 
 # Pass the cart details to the template for rendering
 return render_template('cart.html', cart_details=cart_details)
 except Exception as e:
 print("Error fetching cart details:", e)
 return "An error occurred while fetching cart details."
@app.route('/add_to_cart', methods=['POST'])
def add_to_cart():
 if request.method == 'POST':
 # Retrieve data from the form
 shop_id = request.form.get('shop_id')
 item_id = request.form.get('item_id')
 
 # Retrieve username from session data
 
 
 if username:
 # Insert the data into the cart table
 sql_insert_cart = "INSERT INTO cart (shop_id, item_id, username, 
timestamp) VALUES (%s, %s, %s, %s)"
 timestamp = get_current_time()
 values = (shop_id, item_id, username, timestamp)
 cursor.execute(sql_insert_cart, values)
 conn.commit()
 
 return 'Item added to cart successfully'
59
 else:
 return 'User not logged in'
 else:
 return 'Invalid request method'
@app.route('/checkout',)
def checkout():
 
 return render_template('checkout.html')
 
@app.route('/new')
def new_user():
 return render_template('create_user_form.html')
@app.route('/new_user', methods=['POST'])
def add_user():
 if request.method == 'POST':
 # Get form data
 first_name = request.form['first_name']
 last_name = request.form['last_name']
 username = request.form['username']
 password = request.form['password']
 role = request.form['role']
 phone = request.form['phone']
 country = request.form['country']
 state = request.form['state']
60
 address = request.form['address']
 pincode = request.form['pincode']
 # Validate admin password if role is admin
 if role == "admin" and password != ADMIN_PASSWORD:
 return 'Incorrect admin password'
 
 # Check if username already exists
 sql_check_username = "SELECT * FROM USER WHERE username = %s"
 cursor.execute(sql_check_username, (username,))
 existing_user = cursor.fetchone()
 if existing_user:
 return 'Username already exists'
 
 # SQL query to insert user into the database
 sql = "INSERT INTO USER (first_name, last_name, username, password, 
role, phone, country, state, address, pincode) VALUES (%s, %s, %s, %s, %s, %s, 
%s, %s, %s, %s)"
 values = (first_name, last_name, username, password, role, phone, country, 
state, address, pincode)
 
 # Execute the SQL query
 cursor.execute(sql, values)
 conn.commit()
 
 return 'User added successfully'
 elif request.method == 'GET':
 return 'GET request is not supported for this route'
61
@app.route('/shop')
def shop():
 return render_template('shop.html')
# Route to handle shop selection and display items
# Route to handle shop selection and display items
@app.route('/shop/<shop_name>')
def display_items(shop_name):
 try:
 # Find shop ID based on shop name
 sql_shop_id = "SELECT shop_id FROM Shops WHERE shop_name = %s"
 cursor.execute(sql_shop_id, (shop_name,))
 shop_id_tuple = cursor.fetchone()
 # Extract the shop ID from the tuple
 shop_id = shop_id_tuple[0] if shop_id_tuple else None
 print("Requested shop name:", shop_name)
 print("Retrieved shop ID:", shop_id)
 if shop_id:
 # Use shop ID to search for items in Items table
 sql_items = "SELECT * FROM Items WHERE shop_id = %s"
 cursor.execute(sql_items, (shop_id,))
 items = cursor.fetchall()
62
 return render_template('shop_items.html', items=items, 
shop_name=shop_name)
 else:
 return 'Shop not found'
 except Exception as e:
 return f"An error occurred: {str(e)}"
@app.route('/order', methods=['GET'])
def order_item():
 item_id = request.args.get('item_id')
 if item_id:
 # Process the item_id to fetch the price from the database
 cursor.execute("SELECT price FROM Items WHERE item_id = %s", 
(item_id,))
 price = cursor.fetchone()
 if price:
 amount = price[0] # Retrieve the price from the database result
 print("Item ID:", item_id)
 print("Amount:", amount)
 # You can also perform additional database operations or processing here
 return render_template('payment.html', amount=amount) # Pass the 
amount to the template
 else:
 return 'Price for item not found'
 else:
 return 'Item ID not provided'
@app.route('/payment')
63
def payment():
 return render_template('payment.html')
@app.route('/place_order', methods=['POST'])
def place_order():
 # Add your order placement logic here
 # For now, let's just return a success message
 return 'Order placed successfully'
 
@app.route('/bill')
def process_payment():
 
 return render_template('bill.html')
def get_current_time():
 return datetime.now()
if _name_ == '_main_':
 app.run(debug=True)
