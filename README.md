# 🥗 AI-Powered Diet Optimization App

A full-stack web application that generates personalized, high-adherence meal plans using a custom **Hierarchical Clustering Multilayer Perceptron (HCMLP)** machine learning model. Users receive calorie/macro targets and a data-driven meal plan based on their body metrics, fitness goals, and food preferences.

---

## 🌐 Deployment

- Frontend (Render): https://weightloss-app-frontend.onrender.com
- Backend API (Railway): `https://weightlossapp-production.up.railway.app` — currently offline, so plan generation on the hosted frontend needs the API redeployed.

---

## ✨ Features

- **AI Diet Plan Generation** — Enter age, gender, weight, height, goal, and activity level to receive a personalized meal plan with macro targets per meal.
- **HCMLP Adherence Score** — The model predicts the likelihood (%) you'll stick to the plan.
- **Dynamic Meal Splits** — Supports 3, 4, or 5 meals per day with automatically adjusted calorie and protein distribution.
- **Food Preference Filtering** — Keyword-based filtering (e.g., "chicken", "vegan", "rice") from a real food database.
- **Food Logging** — Log daily meals (food item, calories, protein, meal type, date) and view your full history.
- **User Authentication** — JWT-based register/login with bcrypt password hashing. Protected routes on both frontend and backend.
- **Persistent Plans** — Dashboard loads your most recent diet plan automatically on every visit.

---

## 🏗️ Project Structure

```
Personalized-Weight-Loss-and-Protein-Intake-Optimization/
├── backend/
│   ├── main.py                  # FastAPI app — all API routes
│   ├── ml_model.py              # HCMLP inference + food selection logic
│   ├── schemas.py               # Pydantic request/response models
│   ├── database.py              # MongoDB (Motor async) connection
│   ├── auth_utils.py            # JWT creation, bcrypt hashing/verification
│   ├── requirements.txt         # Python dependencies
│   └── deployment_artifacts/
│       ├── food_database.csv    # Real food nutrition database (~100MB)
│       ├── minmax_scaler.pkl    # Fitted MinMaxScaler
│       ├── cluster_encoder.pkl  # OneHotEncoder for cluster IDs
│       ├── hc_cluster_model.pkl # Agglomerative Clustering model
│       ├── proxy_discriminator.keras # Auxiliary model from training (not loaded at inference)
│       └── hcmlp_model.keras    # Trained HCMLP neural network (TensorFlow/Keras)
└── frontend/
    ├── src/
    │   ├── App.jsx              # Router + auth state
    │   ├── main.jsx             # React entry point
    │   ├── index.css            # Global styles (Tailwind directives)
    │   ├── components/
    │   │   └── Navbar.jsx       # Navigation bar with auth-aware links
    │   └── pages/
    │       ├── Home.jsx         # Landing page
    │       ├── Dashboard.jsx    # Diet plan form + results display
    │       ├── LogFood.jsx      # Meal logging + history
    │       ├── Login.jsx        # Login form
    │       ├── Register.jsx     # Registration form
    │       └── About.jsx        # Project info & tech stack
    ├── package.json
    ├── vite.config.js
    ├── tailwind.config.js
    └── postcss.config.js
```

---

## 🛠️ Tech Stack

### Machine Learning
| Library | Role |
|---|---|
| **TensorFlow / Keras** | HCMLP neural network model (`.keras` format) |
| **Scikit-learn** | Agglomerative Clustering, MinMaxScaler, OneHotEncoder |
| **NumPy** | Feature array construction and manipulation |
| **Pandas** | Food database loading and filtering |
| **Joblib** | Serialization of sklearn model artifacts (`.pkl`) |

### Backend
| Technology | Role |
|---|---|
| **Python 3** | Core backend language |
| **FastAPI** | REST API framework with automatic OpenAPI docs |
| **Uvicorn** | ASGI server |
| **Pydantic v2** | Request/response schema validation |
| **Motor** | Async MongoDB driver (built on PyMongo) |
| **MongoDB Atlas** | Cloud NoSQL database (3 collections: users, predictions, food_logs) |
| **PyJWT** | JWT access token encoding/decoding (HS256) |
| **bcrypt** (`passlib[bcrypt]`) | Password hashing and verification |
| **python-dotenv** | Environment variable loading from `.env` |

### Frontend
| Technology | Role |
|---|---|
| **React 18** | UI component library |
| **Vite 5** | Build tool and dev server |
| **React Router DOM v6** | Client-side routing with protected/reverse-guarded routes |
| **Axios** | HTTP client (installed; fetch API used in components) |
| **Tailwind CSS v3** | Utility-first CSS styling |
| **PostCSS + Autoprefixer** | CSS processing pipeline |

### Deployment
| Service | Role |
|---|---|
| **Render** | Frontend hosting |
| **Railway** | Backend hosting |
| **MongoDB Atlas** | Database hosting |

---

## 🤖 ML Model — How It Works

The `predict_diet()` function in `ml_model.py` runs the following pipeline:

1. **BMR & TDEE Calculation** — Uses the Mifflin-St Jeor equation with activity multipliers (1.2 – 1.725).
2. **Goal-Based Macro Targets** — Applies a ±500 kcal surplus/deficit for gain/loss goals; protein targets scaled by body weight.
3. **Feature Engineering** — Constructs a 15-feature vector (protein target, calorie target, macro splits, activity level, gender proxy, BMI proxy, meal frequency proxy, etc.).
4. **Scaling** — MinMaxScaler normalizes the feature vector to the range used during training.
5. **Clustering** — The Agglomerative Clustering model assigns the user to a dietary cluster; the cluster ID is one-hot encoded and concatenated to the scaled features.
6. **HCMLP Inference** — The Keras model outputs an adherence probability (0–1), multiplied by 100 for the displayed score.
7. **Data-Driven Food Selection** — `data_driven_food_selector()` filters the real food database by calorie range and junk-food exclusion list, prioritizes high protein-density items to meet the protein target, then fills remaining calories with carb/fat sources.

---

## 🚀 Local Setup

### Prerequisites
- Python 3.10+
- Node.js 18+
- A MongoDB Atlas cluster (free tier works)

### Backend

```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env   # set MONGO_URI and SECRET_KEY

# Start the server
uvicorn main:app --reload --port 8000
```

The API will be available at `http://localhost:8000`.
Interactive docs: `http://localhost:8000/docs`

> **Note:** The `deployment_artifacts/` folder must exist with all 4 model files and `food_database.csv` before starting.

### Frontend

```bash
cd frontend

# Install dependencies
npm install

# Start the dev server
npm run dev
```

The app will be available at `http://localhost:5173`.

> **Note:** API calls in `Dashboard.jsx` and `LogFood.jsx` are hardcoded to the Railway production URL. Change these to `http://localhost:8000` for local development.

---

## 📡 API Reference

All protected routes require the header: `Authorization: Bearer <token>`

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/register` | No | Create a new user account |
| `POST` | `/login` | No | Login and receive a JWT token |
| `POST` | `/predict` | Yes | Generate a diet plan from user metrics |
| `GET` | `/latest_plan` | Yes | Fetch the user's most recent diet plan |
| `POST` | `/log_food` | Yes | Log a meal entry |
| `GET` | `/my_logs` | Yes | Retrieve all food logs for the current user |

### Example: `/predict` Request Body
```json
{
  "age": 22,
  "gender": "male",
  "weight": 75.0,
  "height": 175.0,
  "goal": "lose",
  "activity_level": "moderate",
  "meals_per_day": 4,
  "food_preference": "chicken"
}
```

### Example: `/predict` Response
```json
{
  "adherence_score": 78.42,
  "recommended_calories": 2108.0,
  "recommended_protein": 150.0,
  "diet_plan": {
    "breakfast": { "macros": "527 kcal, 38g protein", "foods": ["..."] },
    "lunch":     { "macros": "738 kcal, 60g protein", "foods": ["..."] },
    "dinner":    { "macros": "633 kcal, 53g protein", "foods": ["..."] },
    "snacks":    { "macros": "211 kcal, 0g protein",  "foods": ["..."] }
  }
}
```

---

## 🔒 Authentication Flow

1. User registers → password hashed with **bcrypt** and stored in MongoDB.
2. User logs in → server verifies hash and returns a signed **JWT** (HS256, 24h expiry).
3. Token stored in **localStorage** on the frontend.
4. Every protected API call sends `Authorization: Bearer <token>` in the header.
5. FastAPI dependency `get_current_user` decodes and validates the token on each request.
