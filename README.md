[comment]: # (You may find the following markdown cheat sheet useful: https://www.markdownguide.org/cheat-sheet/. You may also consider using an online Markdown editor such as StackEdit or makeareadme.) 

## Project title: *Stock Control System*

### Student name: Krishna Harish Nellaiappan

### Student email: khn8@student.le.ac.uk

### Project description: 
*In many warehouses, stock tracking is still handled manually, leading to issues like over-ordering, stockouts, and delays. These challenges grow in B2B environments where efficiency is key and large quantities are involved. This project proposes a full-stack stock control system for managing computer hardware inventory in such settings. Admin and Staff users can manage items, track warehouse locations, and place orders that trigger automatic invoice generation. The system’s standout feature is its use of machine learning (via scikit-learn in Python) to predict restocking needs based on historical usage trends. Admins can review these suggestions, adjust quantities if needed, and add justifications. The platform is developed using React (frontend), Spring Boot (backend), and PostgreSQL (database), all containerized using Docker for consistent local deployment.*

### List of requirements (objectives): 

[comment]: # (You can add as many additional bullet points as necessary by adding an additional hyphon symbol '-' at the end of each list) 

Essential:
- Secure login with JWT-based role authentication (Admin/Staff)
- CRUD operations on stock items
- Track exact warehouse location of each item during stock assembly
- Real-time stock monitoring (quantity per item)
- Auto-generate invoice when an order is placed
- Reporting dashboard (low-stock alerts, usage trends)
- RESTful backend using Spring Boot and PostgreSQL
- React frontend with forms, tables, and UI validation
- GitLab repository with commit history
- Full Docker + Docker Compose setup for local deployment

Desirable:
- ML-based restock quantity prediction based on usage data
- Admin can review ML restock suggestions, adjust the predicted quantity, and provide a mandatory justification comment before approval
- Dashboard displays ML-estimated “days until stockout” per item to support restocking decisions
- Seeded dataset: 1 admin, 1 staff, 10+ items

Optional:
- Cloud deployment (e.g., AWS EC2/S3, Railway, or Vercel) based on feasibility
- AI chatbot for admin queries (e.g., low stock, most used items)
- QR scanner feature for optional item lookup or quick search
- CSV/PDF report exports
- Notification system for low stock or delayed restocks
- Real-time stock updates using WebSockets
- Visual floorplan interface for item location display
- Version history for stock modifications to support audit trails
- GitLab CI/CD integration


## Information about this repository
This is the repository that you are going to use **individually** for developing your project. Please use the resources provided in the module to learn about **plagiarism** and how plagiarism awareness can foster your learning.

Regarding the use of this repository, once a feature (or part of it) is developed and **working** or parts of your system are integrated and **working**, define a commit and push it to the remote repository. You may find yourself making a commit after a productive hour of work (or even after 20 minutes!), for example. Choose commit message wisely and be concise.

Please choose the structure of the contents of this repository that suits the needs of your project but do indicate in this file where the main software artefacts are located.


# Stock Control System (SCS): A full-stack inventory & restock prediction system

# Frontend: React + Vite (JWT auth, role-based UI, dashboards, reports)
# Backend: Spring Boot (REST API, PostgreSQL, security, PDF invoices)
# ML Service: Flask (predict restock quantity & days-left; trained via Python)


# Repo Layout
Frontend/
Backend/stock-control/
ML/

# 1) Prerequisites

Node.js 18+ (Node 20 recommended)
Java 21 (OpenJDK)
Maven 3.9+
Python 3.11+
PostgreSQL 14+
GitLab Runner if you want to use the provided CI pipeline locally (Optional)

# 2) Database Setup (PostgreSQL)
Create DB and user 
If you want initial stock movement data for testing/demos:
load stock_movements.sql into DB
psql -h localhost -U postgres -d postgres -f stock_movements.sql


# 3) Environment Configuration
Backend (Spring Boot)

Set these (via application.properties or env vars):

# src/main/resources/application.properties
spring.application.name=stock-control
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=kh
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
server.port=8081

# JWT
app.jwt.secret=<your-strong-secret>
app.jwt.expiration=86400000

# ML Service (Flask)

Place trained models at the project root (created by the train_and_save script):
restock_model_predicted_quantity.pkl
restock_model_predicted_days_left.pkl

ML reads Postgres with:
host: localhost
db: postgres
user: postgres
password: kh
port: 5432
inside ml_predict_service.py or env vars set accordingly.

# Frontend (React)

The UI calls the backend at http://localhost:8081.
VITE_API_BASE=http://localhost:8081

# 4) Running the App

# ML Service
cd ML
# create venv
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt

# Train and save models
python train_and_save_models.py

# Run Flask service (default port 5001)
python ml_predict_service.py

Health/ping: GET http://localhost:5001/predict?days=7

Supported days: 7, 14, 30

# Backend API
cd Backend/stock-control
# Package
mvn -DskipTests package
# Run
mvn spring-boot:run
# or
java -jar target/stock-control-*.jar

Backend runs at: http://localhost:8081

Key PI endpoints:
Auth: /api/auth/login, /api/auth/register 
Items: /api/items, /api/items/{id}
Invoices: /api/invoice/generate, /api/invoice/history, /api/invoice/download/{id}
Reports: /api/reports/low-stock?threshold=20, /api/reports/usage-summary, /api/reports/usage-total, /api/reports/usage-trend

Make sure you have at least one admin user to log in and seed data (via a registration endpoint, a seed SQL, or manual insert).

# Frontend UI
cd Frontend
npm run dev

Frontend runs at: http://localhost:5173 and App will call the backend at http://localhost:8081
Use your login page to authenticate, JWT is stored in localStorage and attached to requests.

# 5) Running Tests
# ML Tests (pytest)
cd ML
pytest -q --maxfail=1 --disable-warnings \
  --cov=. --cov-report=html:ml-reports/html --cov-report=xml:ml-reports/coverage.xml \
  --junitxml=ml-reports/junit.xml

Artifacts:

Coverage HTML: ML/ml-reports/html/index.html
Coverage XML: ML/ml-reports/coverage.xml
JUnit XML: ML/ml-reports/junit.xml

# Backend Tests (JUnit + JaCoCo)
cd Backend/stock-control
mvn clean test jacoco:report surefire-report:report-only
mvn -DskipTests=false clean verify

Artifacts:

JUnit XML: target/surefire-reports/*.xml
JaCoCo HTML: target/site/jacoco/index.html
Surefire HTML summary: target/site/surefire-report.html

# Frontend Tests (Jest + ReactTestingLibrary)
cd Frontend
npm run test:ci

Artifacts:

Coverage (Istanbul): Frontend/coverage/
JUnit XML: Frontend/reports/jest/junit.xml

# 6) Run all components Dev Run (3 terminals)

# Terminal 1 – ML

cd ML
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python train_and_save_models.py
python ml_predict_service.py

# Terminal 2 – Backend

cd Backend/stock-control
mvn spring-boot:run

# Terminal 3 – Frontend

cd Frontend
npm run dev

Open the UI at http://localhost:5173 and log in.

# 7) GitLab CI/CD

A ready-to-use .gitlab-ci.yml is included with build and test stages for:

Frontend (Jest)

Backend (JUnit + JaCoCo)

ML (pytest + coverage)

Pipeline artifacts (coverage & JUnit) are uploaded after each run.

# 8) Possible Errors you may get

CORS issues: ensure backend has CORS enabled for http://localhost:5173.
JWT “Unauthorized”: log in again; token expiry is 24h by default (86400000 ms).
DB connection errors: confirm Postgres is running and credentials match both the Backend and ML configs.
Ensure *.pkl files exist. try re-training via train_and_save_models.py.
