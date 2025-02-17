```bash
#!/bin/bash

cd /home/ec2-user/SageMaker

#BACKUP SCRIPT
# Get the first matching bucket name (if it exists)
BUCKET_NAME=$(aws s3 ls | grep "sagemaker-ebs-backup-" | awk '{print $3}' | head -n 1)

# Get the current notebook name
NOTEBOOK_NAME=$(jq -r '.ResourceName' /opt/ml/metadata/resource-metadata.json)

# Generate a timestamp
TIMESTAMP=$(date +%F-%H-%M-%S)

# Define the snapshot name
SNAPSHOT=${NOTEBOOK_NAME}_${TIMESTAMP}

# Start sync process
echo "Syncing files to S3 bucket..."
aws s3 sync /home/ec2-user/SageMaker/ s3://${BUCKET_NAME}/${SNAPSHOT}/ --exclude "lost+found/*"
```
