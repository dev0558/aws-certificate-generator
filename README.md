# AWS Serverless Certificate Generation System

> From days to seconds. An end-to-end serverless pipeline that automates certificate generation and delivery using AWS managed services.

![Architecture Diagram](./assets/architecture-diagram.png)

## Overview

A fully serverless certificate generation system built on AWS that reduced certificate processing time from **days to under 30 seconds**. The system handles certificate requests at scale with zero server management overhead, processing everything from individual requests to batch operations of 1,000+ certificates seamlessly.

Designed for organizations that need to generate and distribute certificates for training completions, webinar attendees, course graduates, and event participants.

## Architecture

The system follows an event-driven, decoupled architecture using AWS managed services:

```
Request ──▶ API Gateway ──▶ Lambda (Validator) ──▶ SQS Queue ──▶ Lambda (Generator) ──┬──▶ S3 (Storage & Delivery)
                                                                                       │
                                                                                       └──▶ DynamoDB (Metadata & Tracking)
```

**Request Flow:**

1. A certificate request hits the REST API via **API Gateway**
2. The first **Lambda** function validates the payload and enqueues the request
3. **SQS** queues the message for reliable, decoupled processing
4. The second **Lambda** function picks up the message, generates the certificate from a template, and stores it
5. The generated certificate is uploaded to **S3** for secure storage and delivery
6. Certificate metadata and tracking information are persisted to **DynamoDB**

## Tech Stack

| Service | Role |
|---|---|
| **AWS API Gateway** | REST API endpoint for incoming certificate requests |
| **AWS Lambda** | Serverless compute for validation and certificate generation |
| **Amazon SQS** | Message queue for decoupled, reliable processing |
| **Amazon S3** | Certificate storage and delivery |
| **Amazon DynamoDB** | Certificate metadata, status tracking, and audit logs |
| **Python** | Runtime language for Lambda functions |
| **Pillow / ReportLab** | Certificate template rendering and PDF generation |

## Features

- **Sub-30-Second Processing**: Certificates are generated and delivered in under 30 seconds, down from a manual process that took days
- **Auto-Scaling**: Handles 1 or 1,000+ certificates without any manual intervention or provisioning
- **Event-Driven Design**: SQS decouples request intake from generation, ensuring fault tolerance and retry capability
- **Template-Based Generation**: Dynamic certificate generation from configurable templates with personalized fields (name, course, date, ID)
- **Batch Processing**: Supports bulk certificate generation for large cohorts
- **Metadata Tracking**: Every certificate is tracked in DynamoDB with status, timestamps, and unique identifiers
- **Cost Efficient**: Pay-per-use serverless model with no idle compute costs
- **Secure Storage**: Certificates stored in S3 with configurable access policies

## Project Structure

```
.
├── assets/
│   └── architecture-diagram.png
├── lambda/
│   ├── validator/
│   │   ├── handler.py              # API request validation and SQS enqueue
│   │   └── requirements.txt
│   └── generator/
│       ├── handler.py              # Certificate generation and storage
│       ├── requirements.txt
│       └── templates/              # Certificate templates
├── infrastructure/
│   └── template.yaml               # AWS SAM / CloudFormation template
├── tests/
│   ├── test_validator.py
│   └── test_generator.py
├── docs/
│   └── api-reference.md
├── .gitignore
├── LICENSE
└── README.md
```

## API Reference

### Generate Certificate

```
POST /certificates
```

**Request Body:**

```json
{
  "recipient_name": "John Doe",
  "course_title": "Advanced Cloud Security",
  "completion_date": "2026-03-25",
  "template_id": "standard_v1"
}
```

**Response:**

```json
{
  "status": "queued",
  "certificate_id": "cert-a1b2c3d4",
  "message": "Certificate request queued for processing."
}
```

### Get Certificate Status

```
GET /certificates/{certificate_id}
```

**Response:**

```json
{
  "certificate_id": "cert-a1b2c3d4",
  "status": "completed",
  "recipient_name": "John Doe",
  "download_url": "https://s3.amazonaws.com/...",
  "generated_at": "2026-03-25T14:30:00Z"
}
```

### Batch Generate

```
POST /certificates/batch
```

**Request Body:**

```json
{
  "template_id": "standard_v1",
  "recipients": [
    {
      "recipient_name": "John Doe",
      "course_title": "Advanced Cloud Security",
      "completion_date": "2026-03-25"
    },
    {
      "recipient_name": "Jane Smith",
      "course_title": "Advanced Cloud Security",
      "completion_date": "2026-03-25"
    }
  ]
}
```

## Deployment

### Prerequisites

- AWS CLI configured with appropriate IAM permissions
- Python 3.9+
- AWS SAM CLI (for infrastructure deployment)

### Steps

```bash
# Clone the repository
git clone https://github.com/dev0558/aws-certificate-generator.git
cd aws-certificate-generator

# Install dependencies
pip install -r lambda/validator/requirements.txt
pip install -r lambda/generator/requirements.txt

# Deploy infrastructure
cd infrastructure
sam build
sam deploy --guided
```

## Environment Variables

| Variable | Description |
|---|---|
| `SQS_QUEUE_URL` | URL of the SQS queue for certificate requests |
| `S3_BUCKET_NAME` | S3 bucket for storing generated certificates |
| `DYNAMODB_TABLE` | DynamoDB table name for certificate metadata |
| `TEMPLATE_BUCKET` | S3 bucket containing certificate templates |

## Key Design Decisions

**Why SQS between the two Lambda functions?**
Decoupling the validator from the generator means a spike in requests does not overwhelm the generation pipeline. SQS provides built-in retry with dead letter queues, so failed generations are not lost.

**Why DynamoDB over RDS?**
Certificate metadata is a classic key-value lookup pattern. DynamoDB provides single-digit millisecond reads at any scale with zero operational overhead, which fits the serverless ethos of the entire system.

**Why not Step Functions?**
The workflow is linear and simple enough that SQS-triggered Lambda handles it cleanly. Step Functions would add orchestration overhead and cost without meaningful benefit for this use case.

## Lessons Learned

- Serverless is a genuinely powerful paradigm when the use case fits. Not every workload needs containers or EC2.
- Event-driven design with proper queuing (SQS) makes systems inherently resilient to load spikes and transient failures.
- Sometimes the biggest impact you can make is automating the mundane so your team can focus on what actually matters.

## Future Enhancements

- [ ] Webhook notifications on certificate completion
- [ ] Multiple template format support (PDF, PNG, SVG)
- [ ] Certificate verification portal with QR code validation
- [ ] CloudWatch dashboards for monitoring and alerting
- [ ] Presigned URL expiration policies for certificate downloads

## License

This project is licensed under the MIT License.

## Author

**Bhargav Raj Dutta**
Offensive Lab Engineer | CTF Specialist

- GitHub: [@dev0558](https://github.com/dev0558)
- LinkedIn: [Bhargav Raj Dutta](https://www.linkedin.com/in/bhargav-raj-dutta-80251a1b4/)
