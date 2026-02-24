# AWS Bedrock Lambda CloudWatch — Serverless AI Monitoring Platform

A serverless AI inference pipeline integrating AWS Bedrock foundation models with AWS Lambda for compute and CloudWatch for observability. Enables scalable, event-driven LLM invocations with full logging, metrics, and alerting.

## Description

This project demonstrates how to build a production-ready serverless AI platform on AWS. AWS Lambda functions invoke foundation models via Amazon Bedrock, while CloudWatch captures invocation logs, latency metrics, error rates, and custom business metrics. The architecture is fully event-driven and scales automatically with demand.

The solution implements best practices for AI workload observability: structured logging, custom CloudWatch dashboards, metric alarms for latency spikes, and cost tracking per model invocation.

## Features

- **Amazon Bedrock Integration** — Invoke Claude, Titan, and Llama 2 models via boto3 Bedrock Runtime client
- - **AWS Lambda Handlers** — Lightweight Python Lambda functions for model invocation with error handling
  - - **CloudWatch Logging** — Structured JSON logs for every invocation with model name, prompt tokens, and latency
    - - **Custom Metrics** — CloudWatch custom namespaces tracking inference latency, token usage, and error rates
      - - **CloudWatch Alarms** — Automated alerts when p99 latency exceeds thresholds or error rate spikes
        - - **CloudWatch Dashboards** — Visual dashboards for real-time AI workload monitoring
          - - **IAM Role Management** — Least-privilege IAM roles for Lambda-to-Bedrock access
            - - **Cost Tracking** — Per-invocation cost estimation logged as CloudWatch metric
             
              - ## Architecture
             
              - ```
                Client Request
                      |
                      v
                API Gateway --> AWS Lambda (Python)
                                     |
                                     +--> Amazon Bedrock (Claude / Titan / Llama)
                                     |         |
                                     |         v
                                     |    LLM Response
                                     |
                                     v
                              CloudWatch Logs (structured JSON)
                              CloudWatch Metrics (latency, tokens, errors)
                              CloudWatch Alarms --> SNS --> Email/Slack
                              CloudWatch Dashboard
                ```

                ## Tech Stack

                | Component | Technology |
                |-----------|-----------|
                | Compute | AWS Lambda (Python 3.11) |
                | AI Models | Amazon Bedrock (Claude, Titan, Llama 2) |
                | Monitoring | Amazon CloudWatch |
                | API | Amazon API Gateway |
                | Notifications | Amazon SNS |
                | IaC | AWS SAM / CloudFormation |
                | SDK | boto3 |

                ## Setup Instructions

                ### Prerequisites

                - AWS CLI configured with appropriate permissions
                - - Python 3.11+
                  - - AWS SAM CLI
                    - - Bedrock model access enabled in your AWS account
                     
                      - ### Installation
                     
                      - ```bash
                        git clone https://github.com/sanikacentric/AWS_Bedrock_Lambda_CloudWatch.git
                        cd AWS_Bedrock_Lambda_CloudWatch
                        pip install -r requirements.txt
                        ```

                        ### Deploy with SAM

                        ```bash
                        sam build
                        sam deploy --guided
                        ```

                        ### Test the Lambda Function

                        ```bash
                        aws lambda invoke \
                          --function-name bedrock-inference-fn \
                          --payload '{"prompt": "Explain quantum computing in simple terms"}' \
                          output.json
                        cat output.json
                        ```

                        ### View CloudWatch Logs

                        ```bash
                        aws logs tail /aws/lambda/bedrock-inference-fn --follow
                        ```

                        ### Example Lambda Handler

                        ```python
                        import boto3
                        import json
                        import time

                        bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")
                        cloudwatch = boto3.client("cloudwatch")

                        def handler(event, context):
                            prompt = event.get("prompt", "")
                            start = time.time()

                            response = bedrock.invoke_model(
                                modelId="anthropic.claude-v2",
                                body=json.dumps({"prompt": prompt, "max_tokens_to_sample": 512})
                            )

                            latency_ms = (time.time() - start) * 1000

                            cloudwatch.put_metric_data(
                                Namespace="BedrockInference",
                                MetricData=[{"MetricName": "InvocationLatency", "Value": latency_ms, "Unit": "Milliseconds"}]
                            )

                            return json.loads(response["body"].read())
                        ```

                        ## License

                        MIT License
