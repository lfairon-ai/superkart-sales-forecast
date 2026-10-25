# SuperKart deployment: GitHub Codespaces runbook

These folders are ready to push as the repository root:

```
backend/   app.py, requirements.txt, Dockerfile, superkart_model.joblib
frontend/  app.py, requirements.txt, Dockerfile
```

The model was trained with the exact versions pinned in `backend/requirements.txt`
(Python 3.9, scikit-learn 1.6.1, xgboost 2.1.4, pandas 2.2.2, numpy 2.0.2), so it loads in the
`python:3.9-slim` image without retraining. The same files and versions were run and tested
outside Docker (Gunicorn and Streamlit in clean Python 3.9 environments): single and batch
predictions return the values shown in the presentation. The Docker images themselves were not
built in that test; the first build is the one in step 2 below.

## 1. Push to GitHub

Either run the notebook's "Pushing Deployment Files to GitHub" cell (Colab secret named
`Gittoken`, repository `superkart-sales-forecast`), or create the repository by hand and
upload the two folders.

## 2. Open a Codespace on the repository, then in its terminal

```
docker network create superkart-net

cd backend
docker build -t superkart-backend .
docker run -d --name backend --network superkart-net -p 7860:7860 superkart-backend

cd ../frontend
docker build -t superkart-frontend .
docker run -d --name frontend --network superkart-net -p 8501:8501 superkart-frontend

docker ps
```

The frontend reaches the API at `http://backend:7860` because the backend container is named
`backend` on the shared network. Keep that name.

## 3. Ports

In the Codespace **Ports** tab:

- set port **7860** to **Public** (the notebook's inference cells call it from Colab);
- open port **8501** in the browser for the Streamlit app.

Copy the forwarded URL for 7860 into the notebook cell `model_root_url = "..."`, run the
online and batch inference cells, and paste the outputs and the URL into the presentation.

## 4. Screenshots for the presentation

- Slide "Deployment - Single Inference": the Streamlit page after **Predict**, with the
  browser address bar showing the `-8501.app.github.dev` URL; update the two URLs on the slide.
- Slide "Deployment - Batch Inference": upload `Batch_Data_SuperKart.csv`, click
  **Predict for Batch**, capture the result table.

## Troubleshooting

- `ModuleNotFoundError: pkg_resources` when the backend starts: gunicorn 20.1.0 needs
  setuptools; `setuptools<81` is already pinned in `backend/requirements.txt`.
- Frontend shows "Unable to connect to the prediction API": check `docker ps`, and that the
  backend container is named `backend` and joined to `superkart-net`.
- Logs: `docker logs backend` and `docker logs frontend`.
