```bash
#!/bin/bash

cd /home/ec2-user/SageMaker

#SYNC SCRIPT
# Define the new notebook name
NOTEBOOK_NAME=$(jq -r '.ResourceName' /opt/ml/metadata/resource-metadata.json)

# Get the bucket name
BUCKET_NAME=$(aws s3 ls | grep "sagemaker-ebs-backup-" | awk '{print $3}' | head -n 1)

# Get the latest snapshot for the old notebook
LATEST_SNAPSHOT=$(aws s3 ls s3://${BUCKET_NAME}/ | grep "${NOTEBOOK_NAME}_" | sort | tail -n 1 | awk '{print $2}' | sed 's/\/$//')

# Sync the snapshot data to the new notebook directory
echo " Syncing snapshot from $LATEST_SNAPSHOT to new notebook directory: /home/ec2-user/SageMaker/${NOTEBOOK_NAME}/"
aws s3 sync s3://${BUCKET_NAME}/${LATEST_SNAPSHOT}/ /home/ec2-user/SageMaker/
```
