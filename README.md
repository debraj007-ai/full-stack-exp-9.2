// Import dependencies
const express = require("express");
const jwt = require("jsonwebtoken");
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());

// Secret key for signing JWTs (in production, use environment variable)
const JWT_SECRET = "mybanksecretkey";

// Hardcoded user credentials (for demo)
const USER = {
  username: "customer1",
  password: "securepass123"
};

// Simple in-memory bank account (for demo)
let account = {
  balance: 1000
};

function authenticateToken(req, res, next) {
  const authHeader = req.headers["authorization"];
  const token = authHeader && authHeader.split(" ")[1]; // Get token part of "Bearer <token>"

  if (!token) {
    return res.status(401).json({ error: "Missing Authorization token" });
  }

  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: "Invalid or expired token" });
    }
    req.user = user; // attach user data to request
    next();
  });
}

app.post("/login", (req, res) => {
  const { username, password } = req.body;

  if (username === USER.username && password === USER.password) {
    // Generate JWT token
    const token = jwt.sign({ username }, JWT_SECRET, { expiresIn: "1h" });
    return res.json({ message: "Login successful", token });
  } else {
    return res.status(401).json({ error: "Invalid username or password" });
  }
});

// View account balance
app.get("/balance", authenticateToken, (req, res) => {
  res.json({ balance: account.balance });
});

// Deposit money
app.post("/deposit", authenticateToken, (req, res) => {
  const { amount } = req.body;
  if (!amount || amount <= 0) {
    return res.status(400).json({ error: "Invalid deposit amount" });
  }
  account.balance += amount;
  res.json({ message: `Deposited $${amount} successfully`, newBalance: account.balance });
});

// Withdraw money
app.post("/withdraw", authenticateToken, (req, res) => {
  const { amount } = req.body;
  if (!amount || amount <= 0) {
    return res.status(400).json({ error: "Invalid withdrawal amount" });
  }
  if (amount > account.balance) {
    return res.status(400).json({ error: "Insufficient balance" });
  }
  account.balance -= amount;
  res.json({ message: `Withdrew $${amount} successfully`, newBalance: account.balance });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(` Banking API running at http://localhost:${PORT}`);
});
