# But IAM is a global AWS service, and IAM API events are delivered to EventBridge in US East (N. Virginia) — us-east-1. AWS specifically documents that global-service API events such as IAM are available in us-east-1 for EventBridge.
# Use N.Virgina for IAM Services.


# 1. Introduction to EventBridge

EventBridge is a serverless event bus service used to build event-driven architecture.

Instead of writing scripts to monitor AWS services, EventBridge automatically captures events from AWS services and routes them to targets like:

Lambda
SNS
SQS
Step Functions
EC2
ECS

Basic Flow:

AWS Service → EventBridge → Rule → Target

In this lab:
EC2 state change → SNS email
IAM user creation/deletion → SNS email

---

# 2. Create CloudTrail (Mandatory for IAM Events)

Important:

EC2 state-change events are native events.
IAM API events (CreateUser/DeleteUser) are captured through CloudTrail.

Step-by-Step:

1. Go to CloudTrail
2. Click Create trail
3. Trail name: kk-eventbridge-trail
4. Storage location:

   * Create new S3 bucket
   * Example: kk-eventbridge-cloudtrail-logs
5. Enable:

   * Management events → Read/Write
   * Multi-region trail → Recommended
6. Click Create trail

Now IAM API calls will be recorded.

---

# 3. Create SNS Topic and Subscription

Step 1: Create SNS Topic

1. Go to SNS
2. Click Create topic
3. Type: Standard
4. Name: kk-eventbridge-topic
5. Create topic

Step 2: Create Subscription

1. Open the topic
2. Click Create subscription
3. Protocol: Email
4. Endpoint: your email address
5. Create subscription
6. Confirm email from inbox

Now SNS is ready.

---

# 4. EventBridge Rule for EC2 Stopping and Stopped State

Objective:
Send email when EC2 instance goes into stopping or stopped state.

---

Step 1: Create Rule

1. Go to EventBridge
2. Click Rules
3. Create rule

Name:
kk-ec2-state-rule

Rule type:
Event pattern

Event source:
AWS services

Service:
EC2

Event type:
EC2 Instance State-change Notification

---

Step 2: Use Custom Event Pattern (Advanced)

Instead of selecting manually, use this JSON:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopping", "stopped"]
  }
}
```

This filters only stopping and stopped.

---

Step 3: Select Target

Target type:
SNS topic

Select:
kk-eventbridge-topic

---

Step 4: Configure Input Transformer

Click Configure input transformer

Input Paths:

```json
{
  "instance": "$.detail.instance-id",
  "state": "$.detail.state",
  "region": "$.region",
  "time": "$.time"
}
```

Input Template:

```json
{
  "Message": "EC2 Instance <instance> is now <state> in region <region> at <time>"
}
```

Save rule.

---

Testing EC2 Rule

1. Launch EC2 instance
2. Stop the instance
3. You will receive email like:

EC2 Instance i-xxxx is now stopping in region us-east-1 at time

---

# 5. EventBridge Rule for IAM User Creation and Deletion

Important:

IAM events are captured through CloudTrail.
So Event source = CloudTrail API calls.

---

Step 1: Create New Rule

Name:
kk-iam-user-events-rule

Rule type:
Event pattern

Event source:
AWS services

Service:
CloudTrail

Event type:
AWS API Call via CloudTrail

---

Step 2: Use Custom Event Pattern

```json
{
  "source": ["aws.iam"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["iam.amazonaws.com"],
    "eventName": ["CreateUser", "DeleteUser"]
  }
}
```

This filters:
IAM user creation
IAM user deletion

---

Step 3: Select Target

Target:
SNS topic

Select:
kk-eventbridge-topic

---

Step 4: Configure Input Transformer

Input Paths:

```json
{
  "username": "$.detail.requestParameters.userName",
  "event": "$.detail.eventName",
  "time": "$.detail.eventTime",
  "caller": "$.detail.userIdentity.arn"
}
```

Input Template:

```json
{
  "Message": "IAM Event: <event> performed on user <username> at <time> by <caller>"
}
```

Save rule.

---

Testing IAM Rule

1. Go to IAM
2. Create new user
3. Delete that user
4. Check email

You will receive:

IAM Event: CreateUser performed on user test-user at time by arn:aws:iam::xxxx:user/admin



---

Real-Time Project Explanation (Interview Style)

"In our production environment, we implemented event-driven monitoring using EventBridge. We configured rules to capture EC2 state changes and IAM user management activities via CloudTrail. Using Input Transformers, we formatted events before sending them to SNS, ensuring readable notifications for operations and security teams. This helped us implement automated monitoring without custom scripts."

---

Common Troubleshooting

1. IAM rule not triggering?
   → Check CloudTrail is enabled
   → Ensure Management events are enabled

2. Not receiving email?
   → Confirm SNS subscription
   → Check spam folder

3. Rule not matching?
   → Use EventBridge → Test pattern feature


