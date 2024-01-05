### Experiment tracking

 [MLflow](https://mlflow.org/) to track our experiments and store our models and the [MLflow Tracking UI](https://www.mlflow.org/docs/latest/tracking.html#tracking-ui) to view our experiments. We use DVC central location to store and track all of our experiments. Then we spin up MLflow server in [Comet](https://www.comet.ml/), etc.

```bash
export MODEL_REGISTRY=$(python -c "from  mlops import config; print(config.MODEL_REGISTRY)")
mlflow server -h 0.0.0.0 -p 8080 --backend-store-uri $MODEL_REGISTRY
```
### Anyscale Services

Now we launch our serve our model to production.

```yaml
ray_serve_config:
  import_path: deploy.services.serve_model:entrypoint
  runtime_env:
    working_dir: .
    upload_path: s3://mlops/$GITHUB_USERNAME/services  
    env_vars:
      GITHUB_USERNAME: $GITHUB_USERNAME  
```

Now we're ready to launch our service:

```bash
# Rollout service
anyscale service rollout -f deploy/services/serve_model.yaml

# Query
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer $SECRET_TOKEN" -d '{
  "title": "recommendation system",
  "description": ""
}' $SERVICE_ENDPOINT/predict/

# Rollback (to previous version of the Service)
anyscale service rollback -f $SERVICE_CONFIG --name $SERVICE_NAME

# Terminate
anyscale service terminate --name $SERVICE_NAME
```



Run the following command on your Anyscale Workspace terminal to generate the public URL to your MLflow server.

  ```bash
  APP_PORT=8080
  echo https://$APP_PORT-port-$ANYSCALE_SESSION_DOMAIN
  ```

### Continual learning

 It becomes really easy to extend on this foundation to connect to scheduled runs (cron), drift detection and online evaluation etc. with continual learning.

 <div align="center">
  <img src="https://github.com/schopra6/ML-Ops/blob/main/images/workflow.png">
</div>


### Serving

  ```bash
  # Start
  ray start --head
  ```

  ```bash
  # Set up
  export EXPERIMENT_NAME="llm"
  export RUN_ID=$(python madewithml/predict.py get-best-run-id --experiment-name $EXPERIMENT_NAME --metric val_loss --mode ASC)
  python madewithml/serve.py --run_id $RUN_ID
  ```

  Once the application is running, we can use it via cURL, Python, etc.:

  ```python
  # via Python
  import json
  import requests
  json_data = json.dumps({ "data": data})
  requests.post("http://127.0.0.1:8000/predict", data=json_data).json()
  ```

  ```bash
  ray stop  # shutdown
  ```

