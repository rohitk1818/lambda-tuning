---
schemaVersion: '0.3'
description: 'Run AWS Lambda Power Tuner Step Function for a given Lambda function ARN'
assumeRole: '{{ AutomationAssumeRole }}'

parameters:
  LambdaFunctionArn:
    type: String
    description: 'ARN of the Lambda function to tune'

  PowerTunerStateMachineArn:
    type: String
    default: 'arn:aws:states:REGION:ACCOUNT_ID:stateMachine:LambdaPowerTunerStateMachine'
    description: 'ARN of the Lambda Power Tuner Step Function'

  AutomationAssumeRole:
    type: String
    description: '(Optional) IAM role that allows automation to run this document'

mainSteps:
  - name: startPowerTuner
    action: 'aws:executeAwsApi'
    inputs:
      Service: 'stepfunctions'
      Api: 'StartExecution'
      Parameters:
        stateMachineArn: '{{ PowerTunerStateMachineArn }}'
        input: |
          {
            "lambdaARN": "{{ LambdaFunctionArn }}",
            "powerValues": [128,256,512,1024,1536,3008],
            "num": 10,
            "payload": {},
            "parallelInvocation": false,
            "strategy": "cost"
          }
    outputs:
      - Name: ExecutionArn
        Selector: $.executionArn
        Type: String

