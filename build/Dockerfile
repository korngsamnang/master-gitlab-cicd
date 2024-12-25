FROM python:3.9-slim-buster

LABEL Name="Python Flask Demo App" Version=1.4.2
LABEL org.opencontainers.image.source = "https://github.com/benc-uk/python-demoapp"

ARG srcDir=src
WORKDIR /app

# Copy requirements.txt from the src directory (relative to the build context)
COPY $srcDir/requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy run.py and app folder from the src directory to the container
COPY $srcDir/run.py .
COPY $srcDir/app ./app

# Expose port for the app
EXPOSE 5000

# Start the application using Gunicorn
CMD ["gunicorn", "-b", "0.0.0.0:5000", "run:app"]
