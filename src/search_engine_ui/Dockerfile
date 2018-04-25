FROM python:3.6.4-jessie

WORKDIR /app
COPY . /app

RUN pip install -r /app/requirements.txt

ENV MONGO mongo
ENV FLASK_APP ui.py

WORKDIR /app/ui
ENTRYPOINT ["gunicorn", "ui:app", "-b","0.0.0.0"]