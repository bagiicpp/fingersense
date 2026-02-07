# Hand Detector App

This is a simple web application that detects the aspect of a human hand (dorsal/palmar, left/right) from webcam images. The application demonstrates the full end-to-end workflow from machine learning model development to deployment in a web interface using Svelte and FastAPI.

## Project Overview

The machine learning model used in this project was developed in a separate repository, [`fingersense-notebook`](https://github.com/bagiicpp/fingersense-notebook), where I trained and evaluated a logistic regression classifier on the 11,000+ hands dataset. The model achieved high accuracy (~95%) in classifying hand aspects and serves as a strong baseline before moving to deep learning models.

This project showcases:

* Data exploration and preprocessing of hand images.
* Training a baseline machine learning model.
* Exporting the trained model (`hand_model.pkl`) for use in a web application.
* Building a Python backend using FastAPI to serve predictions.
* Creating a Svelte frontend that captures webcam images and communicates with the backend.

## Features

* Live webcam capture of hand images.
* Real-time prediction of hand aspect:

  * `dorsal left`
  * `dorsal right`
  * `palmar left`
  * `palmar right`
* Easy-to-understand, minimal codebase suitable for demonstration purposes.

## Project Structure

```
hand-detector-app/
├─ backend/
│  ├─ app.py               # FastAPI backend server
│  ├─ hand_model.pkl       # Exported trained ML model
│  └─ requirements.txt     # Python dependencies
├─ frontend/
│  ├─ src/
│  │  ├─ App.svelte       # Main Svelte app
│  │  ├─ components/
│  │  │  └─ WebcamCapture.svelte
│  │  └─ assets/
│  ├─ package.json
│  └─ svelte.config.js
└─ README.md
```

## Usage

### Backend

1. Navigate to the `backend` directory.
2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```
3. Run the FastAPI server:

   ```bash
   uvicorn app:app --reload
   ```
4. The backend will be available at `http://127.0.0.1:8000`.

### Frontend

1. Navigate to the `frontend` directory.
2. Install dependencies:

   ```bash
   npm install
   ```
3. Run the Svelte development server:

   ```bash
   npm run dev
   ```
4. Open the app in your browser (usually `http://localhost:5173`).
5. Allow webcam access and click "Detect Hand" to see predictions.

## Notes

* The model currently uses a logistic regression baseline. Future improvements could include convolutional neural networks (CNNs) for higher accuracy and robustness.
* This app is meant as a **portfolio demonstration** of end-to-end ML deployment and not for production use.

## References

* Model and experiments: [`fingersense-notebook`](https://github.com/bagiicpp/fingersense-notebook)
* Hands dataset: 11,000+ hand images with labels for aspect, age, gender, and other features.

---

This project highlights practical skills in data preprocessing, model training, API development, and frontend integration — demonstrating the ability to take a machine learning model from research to a working web application.
