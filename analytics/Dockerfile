# Use Python 3.9 slim image from ECR Public to avoid Docker Hub rate limits
FROM public.ecr.aws/docker/library/python:3.9-slim

# Set working directory in the container
WORKDIR /app

# Copy requirements file first to leverage Docker layer caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose the port the app runs on
EXPOSE 5153

# Set environment variables with default values
ENV APP_PORT=5153
ENV DB_HOST=127.0.0.1
ENV DB_PORT=5432
ENV DB_NAME=postgres

# Run the application
CMD ["python", "app.py"]