import json
import boto3
import base64
import requests
import os

def lambda_handler(event, context):
    # Inputs from runbook
    developer_email = event['developer_email']
    dev_app_name = event['dev_app_name']
    
    # Fetch Basic Auth credentials from Secrets Manager
    secret_name = os.environ['AUTH_SECRET_NAME']
    secret_client = boto3.client('secretsmanager')
    secret_response = secret_client.get_secret_value(SecretId=secret_name)
    secret_data = json.loads(secret_response['SecretString'])
    
    username = secret_data['username']
    password = secret_data['password']
    
    # Encode credentials
    basic_auth = base64.b64encode(f"{username}:{password}".encode()).decode()
    
    # Make the POST call
    url = "https://apigee.example.com/api/createKey"
    headers = {
        "Authorization": f"Basic {basic_auth}",
        "Content-Type": "application/json"
    }
    body = {
        "developer_email": developer_email,
        "dev_app_name": dev_app_name
    }
    response = requests.post(url, headers=headers, json=body)
    
    if response.status_code != 200:
        raise Exception(f"Apigee API failed: {response.status_code} - {response.text}")
    
    data = response.json()
    client_id = data['client_id']
    client_secret = data['client_secret']
    
    # Store in Secrets Manager
    target_secret_name = os.environ['TARGET_SECRET_NAME']
    secret_client.put_secret_value(
        SecretId=target_secret_name,
        SecretString=json.dumps({
            "client_id": client_id,
            "client_secret": client_secret
        })
    )
    
    return {"status": "success"}
