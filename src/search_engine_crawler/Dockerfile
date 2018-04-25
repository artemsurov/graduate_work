FROM python:3.6.4-jessie

WORKDIR /app
COPY . /app

RUN pip install -r /app/requirements.txt

#ENV MONGO 
#ENV RMQ_HOST
#ENV RMQ_QUEUE
#ENV RMQ_USERNAME
#ENV RMQ_PASSWORD
#ENV CHECK_INTERVAL
ENV EXCLUDE_URLS '.*github.com'

ENTRYPOINT ["python", "-u", "/app/crawler/crawler.py"]
CMD ["https://vitkhab.github.io/search_engine_test_site/"]