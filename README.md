**AWS EventBridge project** that demonstrates triggering a Lambda function when a file is uploaded to an S3 bucket. It includes setup steps, IAM permissions, code, and testing instructions.

---

### ✅ **Project Title**: Event-Driven File Processing Using EventBridge and Lambda

---

### 🔧 **Architecture Overview**:

1. **S3 Bucket**: Receives uploaded files.
2. **EventBridge Rule**: Detects the `PutObject` event in S3.
3. **Lambda Function**: Processes or logs the upload event.

---

### 🧰 **Tools & Services Used**:

* Amazon S3
* Amazon EventBridge
* AWS Lambda
* IAM Roles and Policies
* AWS CLI / Console

---

### 📁 **Project Structure**:

```
eventbridge-s3-lambda/
│
├── lambda_function/
│   └── index.js
├── deploy.sh
└── README.md
```

---

### 📝 **Step-by-Step Guide**

---

#### 🔹 Step 1: Create an S3 Bucket

```bash
aws s3api create-bucket --bucket my-eventbridge-demo-bucket --region us-east-1
```

---

#### 🔹 Step 2: Create IAM Role for Lambda

```bash
aws iam create-role --role-name lambda-eventbridge-role \
  --assume-role-policy-document file://trust-policy.json
```

**`trust-policy.json`**:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

Attach permissions:

```bash
aws iam attach-role-policy --role-name lambda-eventbridge-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

#### 🔹 Step 3: Create a Lambda Function

**`index.js`**:

```javascript
exports.handler = async (event) => {
    console.log("Event Received:", JSON.stringify(event, null, 2));
    const bucket = event.detail.bucket.name;
    const key = event.detail.object.key;
    console.log(`New file uploaded: ${bucket}/${key}`);
    return { statusCode: 200 };
};
```

**Deploy the Lambda:**

```bash
zip function.zip index.js

aws lambda create-function \
  --function-name s3FileHandler \
  --zip-file fileb://function.zip \
  --handler index.handler \
  --runtime nodejs18.x \
  --role arn:aws:iam::<YOUR_ACCOUNT_ID>:role/lambda-eventbridge-role
```

---

#### 🔹 Step 4: Create EventBridge Rule

```bash
aws events put-rule \
  --name S3PutObjectRule \
  --event-pattern '{
    "source": ["aws.s3"],
    "detail-type": ["Object Created"],
    "detail": {
      "bucket": {
        "name": ["my-eventbridge-demo-bucket"]
      }
    }
  }'
```

---

#### 🔹 Step 5: Add Lambda as Target

```bash
aws events put-targets \
  --rule S3PutObjectRule \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:<YOUR_ACCOUNT_ID>:function:s3FileHandler"
```

Grant permissions to EventBridge:

```bash
aws lambda add-permission \
  --function-name s3FileHandler \
  --statement-id s3eventbridge \
  --action 'lambda:InvokeFunction' \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:<YOUR_ACCOUNT_ID>:rule/S3PutObjectRule
```

---

#### 🔹 Step 6: Upload File to Trigger Event

```bash
aws s3 cp test.txt s3://my-eventbridge-demo-bucket/
```

Check CloudWatch logs to see Lambda was triggered.

---

### ✅ Output:

* S3 upload triggers EventBridge.
* EventBridge invokes Lambda.
* Lambda logs upload info to CloudWatch.

---
