Creating a full-stack registration application using React for the frontend, Express.js for the backend, and MongoDB for the database involves several steps. Below is a detailed guide to help you set up the application.

### Step-by-Step Guide

#### 1. **Setup Your Environment**

- **Install Node.js and npm**: Ensure you have Node.js and npm installed on your machine.
- **Install MongoDB**: Set up MongoDB locally or use MongoDB Atlas for a cloud solution.

#### 2. **Create the Backend**

1. **Initialize the Project**:
   ```bash
   mkdir registration-app
   cd registration-app
   mkdir backend
   cd backend
   npm init -y
   ```

2. **Install Required Packages**:
   ```bash
   npm install express mongoose bcryptjs jsonwebtoken cors dotenv
   ```

3. **Create the Basic Server**:
   - Create a file named `server.js`:
   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');
   const cors = require('cors');
   require('dotenv').config();

   const app = express();
   const PORT = process.env.PORT || 5000;

   // Middleware
   app.use(cors());
   app.use(express.json());

   // MongoDB Connection
   mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
     .then(() => console.log('MongoDB connected'))
     .catch(err => console.error(err));

   // User Model
   const UserSchema = new mongoose.Schema({
     username: { type: String, required: true, unique: true },
     password: { type: String, required: true },
   });
   const User = mongoose.model('User', UserSchema);

   // Register Route
   app.post('/register', async (req, res) => {
     const { username, password } = req.body;
     const hashedPassword = await bcrypt.hash(password, 10);
     const newUser = new User({ username, password: hashedPassword });

     try {
       await newUser.save();
       res.status(201).json({ message: 'User registered successfully' });
     } catch (error) {
       res.status(400).json({ error: 'User registration failed' });
     }
   });

   // Start the server
   app.listen(PORT, () => {
     console.log(`Server running on http://localhost:${PORT}`);
   });
   ```

4. **Create a `.env` File**:
   ```plaintext
   MONGODB_URI=mongodb://localhost:27017/registrationdb
   JWT_SECRET=your_jwt_secret
   ```

5. **Run the Backend**:
   ```bash
   node server.js
   ```

#### 3. **Create the Frontend**

1. **Setup React Application**:
   - Open a new terminal and navigate to your project folder:
   ```bash
   npx create-react-app frontend
   cd frontend
   ```

2. **Install Axios**:
   ```bash
   npm install axios
   ```

3. **Create Registration Component**:
   - In `src`, create a file named `Register.js`:
   ```javascript
   import React, { useState } from 'react';
   import axios from 'axios';

   const Register = () => {
     const [username, setUsername] = useState('');
     const [password, setPassword] = useState('');
     const [message, setMessage] = useState('');

     const handleSubmit = async (e) => {
       e.preventDefault();
       try {
         const response = await axios.post('http://localhost:5000/register', { username, password });
         setMessage(response.data.message);
       } catch (error) {
         setMessage('Registration failed');
       }
     };

     return (
       <div>
         <h1>Register</h1>
         <form onSubmit={handleSubmit}>
           <input 
             type="text" 
             placeholder="Username" 
             value={username} 
             onChange={(e) => setUsername(e.target.value)} 
           />
           <input 
             type="password" 
             placeholder="Password" 
             value={password} 
             onChange={(e) => setPassword(e.target.value)} 
           />
           <button type="submit">Register</button>
         </form>
         {message && <p>{message}</p>}
       </div>
     );
   };

   export default Register;
   ```

4. **Modify `App.js`**:
   ```javascript
   import React from 'react';
   import Register from './Register';

   const App = () => {
     return (
       <div>
         <Register />
       </div>
     );
   };

   export default App;
   ```

5. **Run the Frontend**:
   ```bash
   npm start
   ```

### 4. **Test the Application**

- Open your browser and go to `http://localhost:3000`.
- You should see a registration form. Enter a username and password, and submit it.
- If registration is successful, a message will display confirming the registration.

### 5. **Deployment (Optional)**

- Deploy the backend on platforms like Heroku or Render.
- Deploy the frontend using services like Vercel or Netlify.
- Ensure to update the API endpoint in your frontend code to match the deployed backend URL.

### Conclusion

This guide provides a basic implementation of a registration app using React, Express, and MongoDB. You can extend this application by adding features such as user login, validation, and better error handling.
