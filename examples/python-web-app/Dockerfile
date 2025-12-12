FROM ubuntu

WORKDIR /app

# Copy project files
COPY requirements.txt /app
COPY devops /app

RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip install -r requirements.txt && \
    cd devops

# Run Django server
ENTRYPOINT ["manage.py", "runserver", "0.0.0.0:8000"]
