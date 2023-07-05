FROM python:3.8-slim
RUN mkdir -p /app
ADD . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["uvicorn", "api:app", "--host", "0.0.0.0"]