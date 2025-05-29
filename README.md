
# Health and Fitness Recommendation System – Dockerized Flask + MongoDB App

This is a Flask web application that provides personalized health recommendations—specifically exercise and diet—based on user profiles. It uses trained machine learning models and is fully containerized using Docker and Docker Compose for consistent deployment across environments.

---

## Features

- User registration and login with secure password hashing
- Form-based input collection (age, weight, height, BMI, etc.)
- ML-based predictions for recommended exercise and diet
- MongoDB integration to store user data and submission history
- Dashboard to view past recommendations and trends

---

## Tech Stack

### Backend

- Flask for server-side logic
- Flask-Login for session handling
- PyMongo for MongoDB interactions
- Joblib to load pre-trained ML models
- Scikit-learn used for training the models

### Machine Learning Models

- `rf_exercises_model.pkl`: Recommends exercises
- `rf_diet_model.pkl`: Suggests appropriate diet
- `label_encoders.pkl`: Stores encoders for categorical fields

### Database

- MongoDB (running inside a Docker container)
  - Database: `fitness`
  - Collections:
    - `users`: for user accounts
    - `submissions`: for storing predictions and inputs

---

## Setup Instructions 

### Step 1: Install Docker Desktop

1. Download Docker Desktop from: https://www.docker.com/products/docker-desktop
2. Install it using default settings.
3. After installation, restart your system if prompted.

---

## Docker-Based App Setup

### Project Structure

```
mlproject/
├── app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── rf_exercises_model.pkl
├── rf_diet_model.pkl
├── label_encoders.pkl
├── templates/
└── static/
```

### Dockerfile

Defines how the Flask image is built:

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
ENV FLASK_RUN_HOST=0.0.0.0
CMD ["python", "app.py"]
```

### docker-compose.yml

Defines how services (Flask app and MongoDB) work together:

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - mongo
    environment:
      - MONGO_URI=mongodb://mongo:27017/
    volumes:
      - .:/app
    restart: always

  mongo:
    image: mongo:6
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

### How Docker Compose Works

- `web` service builds your Flask app from the Dockerfile and exposes it on port 5000.
- `mongo` service pulls the official MongoDB image and runs it on port 27017.
- Both services run in the same Docker network, allowing the Flask app to connect to MongoDB using `mongo:27017`.
- The volume `mongo_data` ensures MongoDB data is persistent.

---

### Run the App

From your project directory:

```bash
docker compose up --build
```

To restart containers:

```bash
docker compose up -d
```

To stop:

```bash
docker compose stop
```

To remove:

```bash
docker compose down
```

---

### Access the App

Open your browser and navigate to:

```
http://localhost:5000
```

---

## View MongoDB Data in MongoDB Compass

### Step 1: Connect Compass

URI for Docker MongoDB:

```
mongodb://127.0.0.1:27017/
```

### Step 2: Troubleshooting

- Make sure the Flask app has inserted data (MongoDB shows collections only after data insertion).
- Use this command to check data directly:

```bash
docker exec -it mongodb mongosh
```

Then run:

```js
use fitness
db.users.find().pretty()
db.submissions.find().pretty()
```

If Compass doesn't show data, it's likely connected to a non-Docker MongoDB.

---

## Useful Docker Commands

| Command | Description |
|--------|-------------|
| `docker ps -a` | List all containers |
| `docker start <name>` | Start container |
| `docker stop <name>` | Stop container |
| `docker rm <name>` | Remove container |
| `docker images` | List images |
| `docker rmi <image>` | Delete image |
| `docker logs <name>` | View logs |
| `docker exec -it <name> bash` | Open shell in container |
| `docker system prune` | Clean up unused containers/images |

---

## Final Notes

- Always use relative paths for model loading inside containers.
- MongoDB Compass may not show databases until collections are created via the app.
- Use Docker volumes for persistent data storage.
