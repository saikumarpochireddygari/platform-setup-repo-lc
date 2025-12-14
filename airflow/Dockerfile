FROM apache/airflow:2.9.0-python3.11

# Run pip as 'airflow' user, not root (Airflow images enforce this)
USER airflow

RUN pip install --no-cache-dir \
    "mlflow[boto3]>=2.0.0,<3.0.0" \
    boto3 \
    python-json-logger \
    pandas \
    scikit-learn