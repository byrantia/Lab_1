# SPMart Supermarket Self-Checkout System
______


## Project Details:

This project is a self-checkout system with two parts:
1) **In-store self-checkout (Raspberry Pi)**  
   Customers scan items using a camera, view totals on an LCD, and pay using RFID (PayWave) or a 6-digit PIN.

2) **Staff Mode**
   Staff must authenticate using the staff card (RFID) before restocking an item. By using the camera to scan the specific item barcode that needs to be restocked 
   

3) **Online self-checkout (Website)**  
   Customers browse items from a website, add to cart, choose self-collection or home delivery (+$4), pay, and receive a QR code for self-collection.


   


## Key Features

### In-store (Raspberry Pi system)
   
![Instore](https://github.com/user-attachments/assets/46685b83-5639-4f59-a321-86c8f2886d2d)

1. LCDs options for customers to choose between payment and scanning items
   - LCD Shows the main options:
     - `*` = scan item
     - `#` = payment

   - Scanning items flow:
      - LCD prompts user to scan items
      - Customers scan the barcode using a camera
      - System reads item from central database (Firestore)
      - LCD shows item name, price and total price
      - System would show if item out of stock
        
   - Payment modes flow:
      - LCDs payment option for customers:
           - `1` = PayWave
           - `2` = Pin
      - Paywave Selected: Customers use PayWave(RFID) to pay
      - Pin Selected: Customers enter 6 digits on keypad (masked), `#` to confirm, `*` to cancel
      - On success: System deducts stock from Firestore and clears cart/total
      - On fail: system returns to main options and still stores price
      - 
 ## Staff mode:
  ![STAFF MODE](https://github.com/user-attachments/assets/67fde9a4-a8fb-469f-987c-323d03928a48)
  - Staff mode is controlled by the slide switch
  - Staff must press `0` to start staff access
  - Staff authorised using staff card (RFID)
  - Staff scans the item barcode using the camera
  - Staff enters restock quantity using keypad:
  - digits = input quantity
  - `*` = backspace
  - `#` = confirm
   - System updates Firestore stock

## Online (Website)
- Website reads items from Central Database(FireStore)
- Only items with `quantity > 0` are displayed
### Cart functions
- Add item to cart
- Increase quantity (+)
- Decrease quantity (-)
- Remove item (X)
- Totals update live:
  - subtotal
  - delivery fee (+$4 if delivery selected)
  - total
### Checkout:
- Customer selects fulfilment:
  - self-collection
  - home delivery (+$4 and address required)
- Customer enters name and email (address required only for delivery)
- System creates an order record in Firestore
- System updates stock using a Firestore transaction:
  - checks stock is enough
  - reduces stock safely
  - prevents overselling
- System shows order summary
- For self-collection: the system generates a QR code using the order number

## System Flow Summary
### In-store mode switching
- Slide switch controls mode:
  - `1` = Customer checkout mode
  - `0` = Staff mode

### Customer checkout controls
- `*` = Scan item
- `#` = Payment
- Payment selection:
  - `1` = RFID (PayWave)
  - `2` = PIN

### Staff mode controls
- Press `0` to start staff card scan
- After authorised:
  - Scan item barcode
  - Enter restock quantity using keypad
    - Digits = quantity input
    - `*` = backspace
    - `#` = confirm

## Folder and Files (Application Layer)

Python (In-store):
- `main.py`  
  Entry point for the in-store system.
- `instore_checkout.py`  
  Runs the in-store checkout system. Switches between customer checkout and staff refill mode using the slide switch.
- `product_lookup.py`  
  Scans a barcode (expected 10 digits) using the camera and queries Firestore to return item name, price, and stock quantity.
- `transaction_manager.py`  
  Stores and updates the current transaction total (add, get total, reset).
- `payment_manager.py`  
  Handles payment selection (RFID or 6-digit PIN), validates payment, returns result (success, fail, cancelled).
- `display_manager.py`  
  Displays LCD screens for checkout, payment, staff flows; manages cart and receipt list; deducts stock from Firestore after successful payment.
- `rfid.py`  
  Checks a staff RFID card to decide whether staff access is allowed.

Website (Online):
- `index.html`  
  Web page layout: product list, cart, totals, checkout form, QR section.
- `firebaseauth.js`  
  Initialises Firebase for the website and exports Firestore functions.
- `script.js`  
  Web logic: load products, cart management, totals, checkout, order creation, transaction stock update, QR generation.
- `styles.css`  
  Page styling (layout, cards, cart UI, modal, order summary).

HAL (Hardware Abstraction Layer):
- `hal_camera.py`  
  Captures camera frames using PiCam2, decodes barcodes, returns barcode string.
- `hal_lcd.py`, `hal_keypad.py`, `hal_rfid_reader.py`, `hal_buzzer.py`, `hal_led.py`, `hal_input_switch.py`  
  Hardware interfaces for the Raspberry Pi.

## Firestore Data Structure

### Collection: `ItemCollection`
Each item document should include:
- `item` (string)  
- `price` (number or string convertible to float)  
- `barcode` (string, expected 10 digits)  
- `quantity` (integer)

### Orders
Orders are stored under:
- `Orders / PickUp / Orders / {orderNo}` for self-collection
- `Orders / OnlineOrder / Orders / {orderNo}` for delivery

Each order includes:
- `orderNo`
- `fulfilment` (`self` or `delivery`)
- `customer` `{ name, email, address }`
- `items` list (productId, name, price, qty, lineTotal)
- `subtotal`, `deliveryFee`, `total`
- `status` (example: `"paid"`)
- `createdAt` (serverTimestamp)

## Setup and Run (Raspberry Pi In-store)

### Requirements
- Raspberry Pi with PiCam2
- RFID reader, keypad, LCD, slide switch, buzzer, LED
- Python 3

### Install dependencies (example)
Depending on your Pi OS, you may need:
- `picamera2`
- `pyzbar`
- `firebase-admin`
- `google-cloud-firestore`
- `libzbar0` (system library for barcode decode)

### Firebase service account
Your Python code uses a service account JSON file, for example:
- `/home/pi/project/.../firebase-adminsdk-....json`

Make sure:
- The file path is correct on your Pi
- Do not upload the service account JSON to public repos

### Run
From the project folder:
```bash
python3 main.py
