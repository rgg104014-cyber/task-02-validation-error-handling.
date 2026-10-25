# task-02-validation-error-handling.

const express = require("express");
const { z } = require("zod");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Schema for validating user input
const userSchema = z.object({
  name: z.string().trim().min(2, "Name must have at least 2 characters"),
  email: z.string().trim().email("Enter a valid email address"),
  age: z.number().int().min(18, "Age must be at least 18"),
});

// Custom error for invalid requests
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// Route 1: Health check
app.get("/", (req, res) => {
  res.status(200).json({
    success: true,
    message: "Validation API is running",
  });
});

// Route 2: Get API information
app.get("/api", (req, res) => {
  res.status(200).json({
    success: true,
    routes: ["GET /", "GET /api", "POST /users"],
  });
});

// Route 3: Create a user with validation
app.post("/users", (req, res, next) => {
  const result = userSchema.safeParse(req.body);

  if (!result.success) {
    return next(
      new AppError(
        "Validation failed",
        400
      )
    );
  }

  res.status(201).json({
    success: true,
    message: "User created successfully",
    data: result.data,
  });
});

// Handle unknown routes
app.use((req, res, next) => {
  next(new AppError("Route not found", 404));
});

// Central error-handling middleware
app.use((err, req, res, next) => {
  if (res.headersSent) {
    return next(err);
  }

  const statusCode = err.statusCode || 500;

  if (statusCode >= 500) {
    console.error(err.message);
  }

  res.status(statusCode).json({
    success: false,
    error:
      statusCode >= 500
        ? "Internal server error"
        : err.message,
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
