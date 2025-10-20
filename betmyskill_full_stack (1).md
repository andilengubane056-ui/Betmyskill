// server.js (updated with admin authentication)

import express from "express";
import mongoose from "mongoose";
import cors from "cors";
import dotenv from "dotenv";
import bodyParser from "body-parser";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import authRoutes from "./routes/auth.js";
import tournamentRoutes from "./routes/tournament.js";
import depositRoutes from "./routes/deposit.js";
import screenshotRoutes from "./routes/screenshot.js";

dotenv.config();
const app = express();
app.use(cors());
app.use(bodyParser.json());

// MongoDB connection
mongoose.connect(process.env.MONGO\_URI, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log("✅ MongoDB connected"))
.catch(err => console.error(err));

// Authentication middleware for admin routes
function verifyAdmin(req, res, next) {
const token = req.headers.authorization?.split(" ")[1];
if (!token) return res.status(401).json({ message: "Access denied" });

try {
const decoded = jwt.verify(token, process.env.JWT\_SECRET);
if (decoded.role !== "admin") return res.status(403).json({ message: "Forbidden" });
req.user = decoded;
next();
} catch (err) {
res.status(400).json({ message: "Invalid token" });
}
}

// Routes
app.use("/api/auth", authRoutes);
app.use("/api/tournament", tournamentRoutes);
app.use("/api/deposit", depositRoutes);
app.use("/api/screenshots", screenshotRoutes);

// Example protected admin route
app.get("/api/admin/overview", verifyAdmin, async (req, res) => {
res.json({ message: "Welcome Admin, here’s your overview." });
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));

// models/Admin.js
import mongoose from "mongoose";
const AdminSchema = new mongoose.Schema({
username: { type: String, required: true, unique: true },
password: { type: String, required: true }
});
export default mongoose.model("Admin", AdminSchema);

// routes/auth.js
import express from "express";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import Admin from "../models/Admin.js";
const router = express.Router();

// Admin registration (run once to create admin)
router.post("/register-admin", async (req, res) => {
const { username, password } = req.body;
const existing = await Admin.findOne({ username });
if (existing) return res.status(400).json({ message: "Admin already exists" });

const hashed = await bcrypt.hash(password, 10);
const admin = new Admin({ username, password: hashed });
await admin.save();
res.json({ message: "Admin registered" });
});

// Admin login
router.post("/login-admin", async (req, res) => {
const { username, password } = req.body;
const admin = await Admin.findOne({ username });
if (!admin) return res.status(404).json({ message: "Admin not found" });

const valid = await bcrypt.compare(password, admin.password);
if (!valid) return res.status(401).json({ message: "Invalid password" });

const token = jwt.sign({ id: admin.\_id, role: "admin" }, process.env.JWT\_SECRET, { expiresIn: "2h" });
res.json({ token });
});

export default router;

git init

git add .

git commit -m "Initial commit for BetMySkill platform"

git branch -M main

git remote add origin https\://github.com/YOUR-USERNAME/betmyskill.git

git push -u origin main