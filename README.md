✅ Spring Boot Microservices: Complete Step-by-Step Project Guide

This document will walk you through creating a Spring Boot Microservices project with:

🔹 ItemService

🔹 OrderService

🔹 Eureka Server

🗃️ MySQL integration

✅ REST APIs with validation

📮 Postman testing support



---

✅ 1. Project Setup

A. Create 3 Spring Boot Applications from Spring Initializr:

Microservice	Artifact Name	Packaging	Dependencies

Item Service	item-service	WAR	Spring Web, Spring Data JPA, MySQL Driver, Eureka Discovery Client
Order Service	order-service	WAR	Spring Web, Spring Data JPA, MySQL Driver, Eureka Discovery Client
Eureka Server	eureka-server	WAR	Eureka Server


B. Import all projects into your IDE (IntelliJ/STS/Eclipse).


---

✅ 2. application.properties (Initial Setup)

📁 eureka-server/src/main/resources/application.properties

server.port=8761
spring.application.name=eureka-server
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

📁 item-service/src/main/resources/application.properties

server.port=8081
spring.application.name=item-service

📁 order-service/src/main/resources/application.properties

server.port=8082
spring.application.name=order-service

Now you can build your basic entities and controllers.


---

✅ 3. Enable Eureka Support

📁 eureka-server

📄 EurekaServerApplication.java

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

📁 item-service and order-service

📄 Main Class

@SpringBootApplication
@EnableEurekaClient
public class ItemServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }
}

(Repeat similarly for OrderService.)

🔄 Update application.properties for Eureka

eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/


---

✅ 4. Add MySQL Configuration

Create databases itemdb and orderdb in MySQL.

📁 item-service/src/main/resources/application.properties

spring.datasource.url=jdbc:mysql://localhost:3306/itemdb
spring.datasource.username=root
spring.datasource.password=your_password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

📁 order-service/src/main/resources/application.properties

spring.datasource.url=jdbc:mysql://localhost:3306/orderdb
spring.datasource.username=root
spring.datasource.password=your_password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect


---

✅ 5. Code for Item Service

📁 com.synechron.itemservice.entity.Item.java

@Entity
public class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is required")
    @Size(min = 2, message = "Name must be at least 2 characters")
    private String name;

    @DecimalMin(value = "0.1", message = "Price must be greater than 0")
    private double price;

    // Getters and Setters
}

📁 com.synechron.itemservice.repository.ItemRepository.java

@Repository
public interface ItemRepository extends JpaRepository<Item, Long> {}

📁 com.synechron.itemservice.controller.ItemController.java

@RestController
@RequestMapping("/item")
public class ItemController {

    public static final String ITEM_SERVICE_URL = "http://localhost:8081/item";

    @Autowired
    private ItemRepository itemRepository;

    // POST - Add item
    @PostMapping
    public ResponseEntity<Item> addItem(@Valid @RequestBody Item item) {
        return new ResponseEntity<>(itemRepository.save(item), HttpStatus.CREATED);
    }

    // GET - List all items
    @GetMapping
    public ResponseEntity<List<Item>> getItems() {
        return ResponseEntity.ok(itemRepository.findAll());
    }
}


---

✅ 6. Code for Order Service

📁 com.synechron.orderservice.entity.Order.java

@Entity
@Table(name = "orders")
public class Order {

    @Id
    private java.sql.Date id; // Order ID as SQL Date

    @NotBlank(message = "Customer name is required")
    private String customerName;

    @NotBlank(message = "Item name is required")
    private String itemName;

    @Min(value = 1, message = "Quantity must be at least 1")
    private int quantity;

    // Getters and Setters
}

📁 com.synechron.orderservice.repository.OrderRepository.java

@Repository
public interface OrderRepository extends JpaRepository<Order, java.sql.Date> {}

📁 com.synechron.orderservice.controller.OrderController.java

@RestController
@RequestMapping("/order")
public class OrderController {

    public static final String ORDER_SERVICE_URL = "http://localhost:8082/order";

    @Autowired
    private OrderRepository orderRepository;

    // POST - Create order
    @PostMapping
    public ResponseEntity<Order> createOrder(@Valid @RequestBody Order order) {
        return new ResponseEntity<>(orderRepository.save(order), HttpStatus.CREATED);
    }

    // GET - List all orders
    @GetMapping
    public ResponseEntity<List<Order>> getAllOrders() {
        return ResponseEntity.ok(orderRepository.findAll());
    }
}


---

✅ 7. Testing with Postman

🔹 Get all items:

GET http://localhost:8081/item

🔹 Create item:

POST http://localhost:8081/item
Content-Type: application/json
{
  "name": "Pen",
  "price": 12.99
}

🔹 Get all orders:

GET http://localhost:8082/order

🔹 Create order:

POST http://localhost:8082/order
Content-Type: application/json
{
  "id": "2025-07-29",
  "customerName": "Ravi",
  "itemName": "Pen",
  "quantity": 3
}


---

Let me know when you're ready for:

📘 Swagger Integration

🧩 Connecting Order to Item service via REST

🐳 Docker + Docker Compose setup


All set to build and test this now!





























____________________________________________________
jwt-demo/
├── server.js
├── jwtUtils.js
├── authMiddleware.js
├── routes/
│   ├── auth.js
│   └── profile.js
├── package.json   <-- after npm init

jwtUtils.js

const jwt = require("jsonwebtoken");

const SECRET_KEY = "mySuperSecretKey"; // use .env in real apps

function createToken(payload, expiry = "1h") {
  return jwt.sign(payload, SECRET_KEY, { expiresIn: expiry });
}

function verifyToken(token) {
  try {
    return jwt.verify(token, SECRET_KEY);
  } catch (error) {
    return null;
  }
}

module.exports = { createToken, verifyToken };

authMiddleware.js 
const { verifyToken } = require("./jwtUtils");

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({ message: "Token missing" });
  }

  const token = authHeader.split(" ")[1];
  const user = verifyToken(token);

  if (!user) return res.status(403).json({ message: "Invalid or expired token" });

  req.user = user;
  next();
}

module.exports = authenticate;

routes/auth.js
const express = require("express");
const router = express.Router();
const { createToken } = require("../jwtUtils");

router.post("/login", (req, res) => {
  const { username, password } = req.body;

  // Dummy check
  if (username === "admin" && password === "1234") {
    const token = createToken({ username, role: "admin" });
    res.json({ message: "Login successful", token });
  } else {
    res.status(401).json({ message: "Invalid credentials" });
  }
});

module.exports = router;


routes/profile.js
const express = require("express");
const router = express.Router();
const authenticate = require("../authMiddleware");

router.get("/", authenticate, (req, res) => {
  res.json({ message: `Welcome ${req.user.username}`, role: req.user.role });
});

module.exports = router;


server.js
const express = require("express");
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());

const authRoutes = require("./routes/auth");
const profileRoutes = require("./routes/profile");

app.use("/auth", authRoutes);
app.use("/profile", profileRoutes);

app.listen(3000, () => {
  console.log(" Server running on http://localhost:3000");
});

___________________________________________



// PROJECT STRUCTURE:
// bookstore-customer-app/
// ├── backend/
// │   ├── models/
// │   │   ├── Customer.js
// │   │   └── Counter.js
// │   ├── routes/
// │   │   └── customers.js
// │   ├── server.js
// │   └── .env
// └── frontend/
//     └── src/
//         └── components/
//             ├── CustomerForm.js
//             └── CustomerList.js
//         App.js

// === BACKEND CODE ===

// backend/.env
MONGO_URI=mongodb://127.0.0.1:27017/bookstore
PORT=5000

// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const customerRoutes = require('./routes/customers');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => console.log("MongoDB connected"));

app.use('/api/customers', customerRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

// backend/models/Customer.js
const mongoose = require('mongoose');

const customerSchema = new mongoose.Schema({
  customerId: Number,
  fullName: String,
  email: String,
  mobileNumber: String,
  password: String,
  gender: String,
  dob: String,
  address: String,
  city: String,
  pincode: String,
  registerOn: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Customer', customerSchema);

// backend/models/Counter.js
const mongoose = require('mongoose');

const counterSchema = new mongoose.Schema({
  id: { type: String, required: true },
  seq: { type: Number, default: 0 }
});

module.exports = mongoose.model('Counter', counterSchema);

// backend/routes/customers.js
const express = require('express');
const router = express.Router();
const Customer = require('../models/Customer');
const Counter = require('../models/Counter');

router.post('/', async (req, res) => {
  try {
    let counter = await Counter.findOne({ id: 'customerId' });
    if (!counter) counter = new Counter({ id: 'customerId', seq: 0 });
    counter.seq += 1;
    await counter.save();

    const newCustomer = new Customer({
      customerId: counter.seq,
      ...req.body
    });

    await newCustomer.save();
    res.json({ message: 'Customer added!', customerId: counter.seq });
  } catch (err) {
    res.status(500).json({ error: 'Error adding customer' });
  }
});

router.get('/', async (req, res) => {
  try {
    const customers = await Customer.find().sort({ customerId: 1 });
    res.json(customers);
  } catch (err) {
    res.status(500).json({ error: 'Error fetching customers' });
  }
});

router.delete('/:id', async (req, res) => {
  try {
    await Customer.findOneAndDelete({ customerId: req.params.id });
    res.json({ message: 'Customer deleted!' });
  } catch (err) {
    res.status(500).json({ error: 'Error deleting customer' });
  }
});

module.exports = router;

// === FRONTEND CODE ===

// frontend/src/components/CustomerForm.js
import { useState } from 'react';
import axios from 'axios';

export default function CustomerForm({ refresh }) {
  const [form, setForm] = useState({
    fullName: '', email: '', mobileNumber: '', password: '',
    gender: '', dob: '', address: '', city: '', pincode: ''
  });

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async () => {
    await axios.post('http://localhost:5000/api/customers', form);
    alert('Customer Added');
    setForm({ fullName: '', email: '', mobileNumber: '', password: '', gender: '', dob: '', address: '', city: '', pincode: '' });
    refresh();
  };

  return (
    <div>
      <h2>Register Customer</h2>
      <input name="fullName" value={form.fullName} onChange={handleChange} placeholder="Full Name" /><br />
      <input name="email" value={form.email} onChange={handleChange} placeholder="Email" /><br />
      <input name="mobileNumber" value={form.mobileNumber} onChange={handleChange} placeholder="Mobile Number" /><br />
      <input name="password" value={form.password} onChange={handleChange} placeholder="Password" type="password" /><br />
      <select name="gender" value={form.gender} onChange={handleChange}>
        <option value="">Select Gender</option>
        <option value="Male">Male</option>
        <option value="Female">Female</option>
        <option value="Other">Other</option>
      </select><br />
      <input name="dob" type="date" value={form.dob} onChange={handleChange} /><br />
      <textarea name="address" value={form.address} onChange={handleChange} placeholder="Address"></textarea><br />
      <input name="city" value={form.city} onChange={handleChange} placeholder="City" /><br />
      <input name="pincode" value={form.pincode} onChange={handleChange} placeholder="Pincode" /><br />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}

// frontend/src/components/CustomerList.js
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function CustomerList({ refreshSignal }) {
  const [customers, setCustomers] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/api/customers')
      .then(res => setCustomers(res.data));
  }, [refreshSignal]);

  const deleteCustomer = async (id) => {
    await axios.delete(`http://localhost:5000/api/customers/${id}`);
    alert('Deleted');
  };

  return (
    <div>
      <h2>All Customers</h2>
      {customers.map(c => (
        <div key={c.customerId}>
          <p><strong>ID:</strong> {c.customerId}</p>
          <p><strong>Name:</strong> {c.fullName}</p>
          <p><strong>Email:</strong> {c.email}</p>
          <p><strong>Mobile:</strong> {c.mobileNumber}</p>
          <p><strong>Gender:</strong> {c.gender}</p>
          <p><strong>DOB:</strong> {c.dob}</p>
          <p><strong>Address:</strong> {c.address}, {c.city} - {c.pincode}</p>
          <p><strong>Registered On:</strong> {new Date(c.registerOn).toLocaleDateString()}</p>
          <button onClick={() => deleteCustomer(c.customerId)}>Delete</button>
          <hr />
        </div>
      ))}
    </div>
  );
}

// frontend/src/App.js
import { useState } from 'react';
import CustomerForm from './components/CustomerForm';
import CustomerList from './components/CustomerList';

function App() {
  const [refresh, setRefresh] = useState(false);
  const triggerRefresh = () => setRefresh(!refresh);

  return (
    <div className="App">
      <CustomerForm refresh={triggerRefresh} />
      <hr />
      <CustomerList refreshSignal={refresh} />
    </div>
  );
}

export default App;








2nd one:
// === BACKEND ===

// backend/.env
MONGO_URI=mongodb://127.0.0.1:27017/bookstore
PORT=5000

// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const customerRoutes = require('./routes/customers');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => console.log("MongoDB connected"));

app.use('/api/customers', customerRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

// backend/models/Customer.js
const mongoose = require('mongoose');

const customerSchema = new mongoose.Schema({
  customerId: Number,
  fullName: String,
  email: String,
  mobileNumber: String,
  password: String,
  gender: String,
  dob: String,
  address: String,
  city: String,
  pincode: String,
  registerOn: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Customer', customerSchema);

// backend/models/Counter.js
const mongoose = require('mongoose');

const counterSchema = new mongoose.Schema({
  id: { type: String, required: true },
  seq: { type: Number, default: 0 }
});

module.exports = mongoose.model('Counter', counterSchema);

// backend/routes/customers.js
const express = require('express');
const router = express.Router();
const Customer = require('../models/Customer');
const Counter = require('../models/Counter');

router.post('/', async (req, res) => {
  try {
    let counter = await Counter.findOne({ id: 'customerId' });
    if (!counter) counter = new Counter({ id: 'customerId', seq: 0 });
    counter.seq += 1;
    await counter.save();

    const newCustomer = new Customer({
      customerId: counter.seq,
      ...req.body
    });

    await newCustomer.save();
    res.json({ message: 'Customer added!', customerId: counter.seq });
  } catch (err) {
    res.status(500).json({ error: 'Error adding customer' });
  }
});

router.get('/', async (req, res) => {
  try {
    const customers = await Customer.find().sort({ customerId: 1 });
    res.json(customers);
  } catch (err) {
    res.status(500).json({ error: 'Error fetching customers' });
  }
});

router.delete('/:id', async (req, res) => {
  try {
    await Customer.findOneAndDelete({ customerId: req.params.id });
    res.json({ message: 'Customer deleted!' });
  } catch (err) {
    res.status(500).json({ error: 'Error deleting customer' });
  }
});

router.put('/:id', async (req, res) => {
  try {
    const updated = await Customer.findOneAndUpdate(
      { customerId: req.params.id },
      req.body,
      { new: true }
    );
    res.json({ message: 'Customer updated!', updated });
  } catch (err) {
    res.status(500).json({ error: 'Error updating customer' });
  }
});

module.exports = router;


// === FRONTEND ===

// frontend/src/index.css
body {
  font-family: 'Segoe UI', sans-serif;
  background: url('/bg.jpg') no-repeat center center fixed;
  background-size: cover;
  margin: 0;
  padding: 0;
  color: #333;
}

.App {
  background-color: rgba(255, 255, 255, 0.9);
  margin: 40px auto;
  max-width: 800px;
  padding: 20px 40px;
  border-radius: 15px;
  box-shadow: 0 0 15px rgba(0,0,0,0.2);
}

input, select, textarea {
  width: 100%;
  padding: 8px;
  margin: 6px 0 12px;
  border: 1px solid #aaa;
  border-radius: 5px;
  font-size: 16px;
}

button {
  padding: 8px 16px;
  margin: 6px 5px;
  background-color: #1d72b8;
  color: white;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  background-color: #155d8a;
}

.customer-card {
  background: #f2f2f2;
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 15px;
}

// frontend/src/components/CustomerForm.js
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function CustomerForm({ refresh, editData, clearEdit }) {
  const [form, setForm] = useState({
    fullName: '', email: '', mobileNumber: '', password: '',
    gender: '', dob: '', address: '', city: '', pincode: ''
  });

  useEffect(() => {
    if (editData) setForm(editData);
  }, [editData]);

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async () => {
    if (editData && editData.customerId) {
      await axios.put(`http://localhost:5000/api/customers/${editData.customerId}`, form);
      alert('Customer Updated');
      clearEdit();
    } else {
      await axios.post('http://localhost:5000/api/customers', form);
      alert('Customer Added');
    }
    setForm({ fullName: '', email: '', mobileNumber: '', password: '', gender: '', dob: '', address: '', city: '', pincode: '' });
    refresh();
  };

  return (
    <div>
      <h2>{editData ? 'Edit Customer' : 'Register Customer'}</h2>
      <input name="fullName" value={form.fullName} onChange={handleChange} placeholder="Full Name" /><br />
      <input name="email" value={form.email} onChange={handleChange} placeholder="Email" /><br />
      <input name="mobileNumber" value={form.mobileNumber} onChange={handleChange} placeholder="Mobile Number" /><br />
      <input name="password" value={form.password} onChange={handleChange} placeholder="Password" type="password" /><br />
      <select name="gender" value={form.gender} onChange={handleChange}>
        <option value="">Select Gender</option>
        <option value="Male">Male</option>
        <option value="Female">Female</option>
        <option value="Other">Other</option>
      </select><br />
      <input name="dob" type="date" value={form.dob} onChange={handleChange} /><br />
      <textarea name="address" value={form.address} onChange={handleChange} placeholder="Address"></textarea><br />
      <input name="city" value={form.city} onChange={handleChange} placeholder="City" /><br />
      <input name="pincode" value={form.pincode} onChange={handleChange} placeholder="Pincode" /><br />
      <button onClick={handleSubmit}>{editData ? 'Update' : 'Submit'}</button>
      {editData && <button onClick={() => { clearEdit(); setForm({ fullName: '', email: '', mobileNumber: '', password: '', gender: '', dob: '', address: '', city: '', pincode: '' }); }}>Cancel</button>}
    </div>
  );
}

// frontend/src/components/CustomerList.js
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function CustomerList({ refreshSignal, onEdit }) {
  const [customers, setCustomers] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/api/customers')
      .then(res => setCustomers(res.data));
  }, [refreshSignal]);

  const deleteCustomer = async (id) => {
    await axios.delete(`http://localhost:5000/api/customers/${id}`);
    alert('Deleted');
    onEdit(null);
  };

  return (
    <div>
      <h2>All Customers</h2>
      {customers.map(c => (
        <div key={c.customerId} className="customer-card">
          <p><strong>ID:</strong> {c.customerId}</p>
          <p><strong>Name:</strong> {c.fullName}</p>
          <p><strong>Email:</strong> {c.email}</p>
          <p><strong>Mobile:</strong> {c.mobileNumber}</p>
          <p><strong>Gender:</strong> {c.gender}</p>
          <p><strong>DOB:</strong> {c.dob}</p>
          <p><strong>Address:</strong> {c.address}, {c.city} - {c.pincode}</p>
          <p><strong>Registered On:</strong> {new Date(c.registerOn).toLocaleDateString()}</p>
          <button onClick={() => onEdit(c)}>Edit</button>
          <button onClick={() => deleteCustomer(c.customerId)}>Delete</button>
        </div>
      ))}
    </div>
  );
}

// frontend/src/App.js
import { useState } from 'react';
import CustomerForm from './components/CustomerForm';
import CustomerList from './components/CustomerList';
import './index.css';

function App() {
  const [refresh, setRefresh] = useState(false);
  const [editCustomer, setEditCustomer] = useState(null);

  const triggerRefresh = () => setRefresh(!refresh);

  return (
    <div className="App">
      <CustomerForm refresh={triggerRefresh} editData={editCustomer} clearEdit={() => setEditCustomer(null)} />
      <hr />
      <CustomerList refreshSignal={refresh} onEdit={setEditCustomer} />
    </div>
  );
}

export default App;





app.js
import { useState } from 'react';
import CustomerForm from './components/CustomerForm';
import CustomerList from './components/CustomerList';
import './App.css';

function App() {
  const [refresh, setRefresh] = useState(false);
  const triggerRefresh = () => setRefresh(!refresh);

  return (
    <div className="app-container">
      <div className="content-wrapper">
        <h1 className="main-title">Bookstore Customer Management</h1>
        <CustomerForm refresh={triggerRefresh} />
        <hr className="divider" />
        <CustomerList refreshSignal={refresh} />
      </div>
    </div>
  );
}

export default App;


app.css
body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background: url('/background.jpg') no-repeat center center fixed;
  background-size: cover;
}

.app-container {
  background-color: rgba(255, 255, 255, 0.9);
  min-height: 100vh;
  padding: 20px;
}

.content-wrapper {
  max-width: 900px;
  margin: auto;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
  background: #ffffff;
}

.main-title {
  text-align: center;
  color: #2c3e50;
  font-size: 32px;
  margin-bottom: 30px;
}

.divider {
  margin: 40px 0;
  border: none;
  border-top: 2px solid #ccc;
}


custormerform.js
import { useState } from 'react';
import axios from 'axios';
import './CustomerForm.css';

export default function CustomerForm({ refresh }) {
  const [form, setForm] = useState({
    fullName: '', email: '', mobileNumber: '', password: '',
    gender: '', dob: '', address: '', city: '', pincode: ''
  });

  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async () => {
    await axios.post('http://localhost:5000/api/customers', form);
    alert('Customer Added');
    setForm({ fullName: '', email: '', mobileNumber: '', password: '', gender: '', dob: '', address: '', city: '', pincode: '' });
    refresh();
  };

  return (
    <div className="form-card">
      <h2>Register Customer</h2>
      <div className="form-grid">
        <input name="fullName" value={form.fullName} onChange={handleChange} placeholder="Full Name" />
        <input name="email" value={form.email} onChange={handleChange} placeholder="Email" />
        <input name="mobileNumber" value={form.mobileNumber} onChange={handleChange} placeholder="Mobile Number" />
        <input name="password" type="password" value={form.password} onChange={handleChange} placeholder="Password" />
        <select name="gender" value={form.gender} onChange={handleChange}>
          <option value="">Select Gender</option>
          <option value="Male">Male</option>
          <option value="Female">Female</option>
          <option value="Other">Other</option>
        </select>
        <input name="dob" type="date" value={form.dob} onChange={handleChange} />
        <textarea name="address" value={form.address} onChange={handleChange} placeholder="Address"></textarea>
        <input name="city" value={form.city} onChange={handleChange} placeholder="City" />
        <input name="pincode" value={form.pincode} onChange={handleChange} placeholder="Pincode" />
      </div>
      <button className="submit-btn" onClick={handleSubmit}>Submit</button>
    </div>
  );
}


Customer form.css

.form-card {
  padding: 20px;
  border-radius: 12px;
  background: #f9f9f9;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

input, select, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
}

textarea {
  resize: vertical;
  min-height: 60px;
}

.submit-btn {
  margin-top: 20px;
  padding: 12px 20px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
}

.submit-btn:hover {
  background-color: #2980b9;
}


customerlist.css
.customer-list {
  background: #f0f4f8;
  padding: 20px;
  border-radius: 10px;
}

.customer-card {
  background: #fff;
  margin-bottom: 20px;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.customer-card p {
  margin: 4px 0;
}

.delete-btn {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  margin-top: 10px;
}

.delete-btn:hover {
  background: #c0392b;
}


customerlist.js
import { useEffect, useState } from 'react';
import axios from 'axios';
import './CustomerList.css';

export default function CustomerList({ refreshSignal }) {
  const [customers, setCustomers] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/api/customers').then(res => setCustomers(res.data));
  }, [refreshSignal]);

  const deleteCustomer = async (id) => {
    await axios.delete(`http://localhost:5000/api/customers/${id}`);
    alert('Deleted');
  };

  return (
    <div className="customer-list">
      <h2>All Customers</h2>
      {customers.map(c => (
        <div key={c.customerId} className="customer-card">
          <p><strong>ID:</strong> {c.customerId}</p>
          <p><strong>Name:</strong> {c.fullName}</p>
          <p><strong>Email:</strong> {c.email}</p>
          <p><strong>Mobile:</strong> {c.mobileNumber}</p>
          <p><strong>Gender:</strong> {c.gender}</p>
          <p><strong>DOB:</strong> {c.dob}</p>
          <p><strong>Address:</strong> {c.address}, {c.city} - {c.pincode}</p>
          <p><strong>Registered On:</strong> {new Date(c.registerOn).toLocaleDateString()}</p>
          <button className="delete-btn" onClick={() => deleteCustomer(c.customerId)}>Delete</button>
        </div>
      ))}
    </div>
  );
}



updated models/Customer.js 
const mongoose = require('mongoose');

const addressSchema = new mongoose.Schema({
  address: String,
  city: String,
  pincode: String,
  country: String
}, { _id: false });

const customerSchema = new mongoose.Schema({
  customerId: Number,
  fullName: String,
  email: String,
  mobileNumber: String,
  password: String,
  gender: String,
  dob: String,
  address: addressSchema, // now embedded object
  registerOn: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Customer', customerSchema);


updated customerform.js
import { useState } from 'react';
import axios from 'axios';
import './CustomerForm.css';

export default function CustomerForm({ refresh }) {
  const [form, setForm] = useState({
    fullName: '', email: '', mobileNumber: '', password: '',
    gender: '', dob: '',
    address: {
      address: '', city: '', pincode: '', country: ''
    }
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    if (name.startsWith('address.')) {
      const key = name.split('.')[1];
      setForm(prev => ({
        ...prev,
        address: {
          ...prev.address,
          [key]: value
        }
      }));
    } else {
      setForm({ ...form, [name]: value });
    }
  };

  const handleSubmit = async () => {
    await axios.post('http://localhost:5000/api/customers', form);
    alert('Customer Added');
    setForm({
      fullName: '', email: '', mobileNumber: '', password: '',
      gender: '', dob: '',
      address: { address: '', city: '', pincode: '', country: '' }
    });
    refresh();
  };

  return (
    <div className="form-card">
      <h2>Register Customer</h2>
      <div className="form-grid">
        <input name="fullName" value={form.fullName} onChange={handleChange} placeholder="Full Name" />
        <input name="email" value={form.email} onChange={handleChange} placeholder="Email" />
        <input name="mobileNumber" value={form.mobileNumber} onChange={handleChange} placeholder="Mobile Number" />
        <input name="password" type="password" value={form.password} onChange={handleChange} placeholder="Password" />
        <select name="gender" value={form.gender} onChange={handleChange}>
          <option value="">Select Gender</option>
          <option value="Male">Male</option>
          <option value="Female">Female</option>
          <option value="Other">Other</option>
        </select>
        <input name="dob" type="date" value={form.dob} onChange={handleChange} />

        <textarea name="address.address" value={form.address.address} onChange={handleChange} placeholder="Address" />
        <input name="address.city" value={form.address.city} onChange={handleChange} placeholder="City" />
        <input name="address.pincode" value={form.address.pincode} onChange={handleChange} placeholder="Pincode" />
        <input name="address.country" value={form.address.country} onChange={handleChange} placeholder="Country" />
      </div>
      <button className="submit-btn" onClick={handleSubmit}>Submit</button>
    </div>
  );
}


updated customerlist.js
import { useEffect, useState } from 'react';
import axios from 'axios';
import './CustomerList.css';

export default function CustomerList({ refreshSignal }) {
  const [customers, setCustomers] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/api/customers').then(res => setCustomers(res.data));
  }, [refreshSignal]);

  const deleteCustomer = async (id) => {
    await axios.delete(`http://localhost:5000/api/customers/${id}`);
    alert('Deleted');
  };

  return (
    <div className="customer-list">
      <h2>All Customers</h2>
      {customers.map(c => (
        <div key={c.customerId} className="customer-card">
          <p><strong>ID:</strong> {c.customerId}</p>
          <p><strong>Name:</strong> {c.fullName}</p>
          <p><strong>Email:</strong> {c.email}</p>
          <p><strong>Mobile:</strong> {c.mobileNumber}</p>
          <p><strong>Gender:</strong> {c.gender}</p>
          <p><strong>DOB:</strong> {c.dob}</p>
          <p><strong>Address:</strong> {c.address?.address}, {c.address?.city}, {c.address?.country} - {c.address?.pincode}</p>
          <p><strong>Registered On:</strong> {new Date(c.registerOn).toLocaleDateString()}</p>
          <button className="delete-btn" onClick={() => deleteCustomer(c.customerId)}>Delete</button>
        </div>
      ))}
    </div>
  );
}