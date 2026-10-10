# HROne - Employee Attendance & Analytics API

This repository contains the completed backend engineering assignment for the Employee Attendance & Analytics API.

## Project Status
* **Part A (Review):** Completed. All intentional defects in `app/main.py` were identified, fixed, and documented in `REVIEW.md`.
* **Part B (Build):** Completed. All endpoints, including the MongoDB aggregation pipelines for analytics, are implemented exactly as per the OpenAPI contract. 
* **Part C (Prove & Explain):** Completed. The `/admin/explain/` endpoint is active, and architectural choices are detailed in `DECISIONS.md`.

## How to Run

1. **Set up the virtual environment:**
   ```bash
   python -m venv .venv
   source .venv/Scripts/activate  
   requre mago db 
   Configure Environment Variables
   Start the Server
   uvicorn app.main:app --port 8000
    