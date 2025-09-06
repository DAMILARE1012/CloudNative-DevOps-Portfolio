
FROM python:3.10-slim

# Set workdir
WORKDIR /app

# Copy project
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose app port
EXPOSE 8000

# Start app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

