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
