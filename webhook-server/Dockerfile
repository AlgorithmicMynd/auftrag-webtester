FROM python:3.11-slim

WORKDIR /app

# Copy parent directory files (visual_qa.py, form_automation.py, etc.)
COPY ../. /app/
COPY ./requirements.txt /app/webhook-server/

# Install all dependencies
RUN pip install --no-cache-dir -r /app/requirements.txt
RUN pip install --no-cache-dir -r /app/webhook-server/requirements.txt

# Install Playwright browsers
RUN playwright install chromium

EXPOSE 5000

# Run the webhook server with gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "1", "--timeout", "300", "webhook-server.app:app"]
