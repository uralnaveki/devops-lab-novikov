FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    pip install --no-cache-dir Werkzeug==2.0.3

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]