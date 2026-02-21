FROM python:3.12-slim
#creates directory app and make it as workdir
WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER nobody
CMD ["python", "app.py"] 


