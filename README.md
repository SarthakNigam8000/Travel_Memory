# 🌍 Travel Memory

A full-stack web application designed to let users store, organize, and revisit their travel memories, photos, and trip logs.

---

## 📌 Key Features

* 📝 **Trip Journaling:** Add posts with titles, descriptions, dates, and locations.
* 📸 **Photo Gallery:** Attach and view photos associated with specific travel logs.
* 📍 **Location Tagging:** Keep track of places visited across various destinations.
* 🔒 **User Authentication:** Secure user registration and login functionality to keep personal journals private.
* 🔍 **Search & Filter:** Easily filter through past trips by location or title.

---

## 🛠️ Tech Stack

* **Frontend:** React.js / HTML5, CSS3, JavaScript
* **Backend:** Node.js, Express.js
* **Database:** MongoDB (via Mongoose)
* **Authentication:** JSON Web Tokens (JWT) / Bcrypt

---

## 🚀 Getting Started

Follow the steps below to set up and run the project locally.

### Prerequisites

Ensure you have installed:
* [Node.js](https://nodejs.org/) (v14+ recommended)
* [Git](https://git-scm.com/)
* [MongoDB](https://www.mongodb.com/) (Local instance or MongoDB Atlas account)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/SarthakNigam8000/Travel_Memory.git
   cd Travel_Memory
   ```

2. **Backend Setup:**
   ```bash
   # Navigate to the backend directory (or root if single folder)
   cd backend
   npm install
   ```

3. **Frontend Setup:**
   ```bash
   # Navigate to the frontend directory
   cd ../frontend
   npm install
   ```

4. **Environment Variables:**
   Create a `.env` file in the backend directory with the following variables:
   ```env
   PORT=5000
   MONGO_URI=your_mongodb_connection_string
   JWT_SECRET=your_secret_key
   ```

5. **Run the Application:**

   * Start the backend server:
     ```bash
     cd backend
     npm start
     ```
   * Start the frontend development server:
     ```bash
     cd frontend
     npm start
     ```

---

## 📂 Project Structure

```text
Travel_Memory/
├── backend/
│   ├── config/        # Database and app configurations
│   ├── controllers/   # Request handling logic
│   ├── models/        # Database schemas
│   ├── routes/        # API routes
│   └── server.js      # Entry point
├── frontend/
│   ├── public/        # Static assets
│   └── src/           # Components, pages, and UI logic
└── README.md
```

---

## 🤝 Contributing

Contributions are always welcome!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/NewFeature`)
3. Commit your Changes (`git commit -m 'Add NewFeature'`)
4. Push to the Branch (`git push origin feature/NewFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the `LICENSE` file for details.