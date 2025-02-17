# **AWS SageMaker Studio Migration Guide**

🚨 **Important AWS Update: SageMaker Studio Classic is no longer maintained as of January 1st, 2025!** 🚨

As part of the transition to the new **SageMaker Studio with Jupyter 4**, AWS is urging users to migrate their notebooks for enhanced features. For those facing issues with the official AWS tutorial, fear not! Today, I'm sharing **two easy-to-use scripts** to help you with **backup and sync** processes for your notebooks in the new environment.

---

## Prerequisites

Before starting, ensure you have the following:

1. An **S3 bucket** named `sagemaker-ebs-backup-$(YOUR_BUCKET_NAME)`
2. A **new notebook** with the same name as your old one, but with `-new` appended (e.g., `old-notebook-name-new`).

Once these two things are set, we can get started!

---

## Step 1: Backup your old notebook to S3

1. Start your **old notebook** and open a terminal.
2. Copy and paste the following script:

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
Once the sync completes, check your S3 bucket to confirm the snapshot is uploaded.

## Step 2: Sync data to the new notebook

1. Start your new notebook and open a terminal.
2. Copy and paste the following script:

```bash
#!/bin/bash

cd /home/ec2-user/SageMaker

#SYNC SCRIPT
# Define the new notebook name
NEW_NOTEBOOK_NAME=$(jq -r '.ResourceName' /opt/ml/metadata/resource-metadata.json)

# Derive the old notebook name by removing '-new' suffix
OLD_NOTEBOOK_NAME=${NEW_NOTEBOOK_NAME%-new}

# Get the bucket name
BUCKET_NAME=$(aws s3 ls | grep "sagemaker-ebs-backup-" | awk '{print $3}' | head -n 1)

# Get the latest snapshot for the old notebook
LATEST_SNAPSHOT=$(aws s3 ls s3://${BUCKET_NAME}/ | grep "${OLD_NOTEBOOK_NAME}_" | sort | tail -n 1 | awk '{print $2}' | sed 's/\/$//')

# Sync the snapshot data to the new notebook directory
echo " Syncing snapshot from $LATEST_SNAPSHOT to new notebook directory: /home/ec2-user/SageMaker/${NEW_NOTEBOOK_NAME}/"
aws s3 sync s3://${BUCKET_NAME}/${LATEST_SNAPSHOT}/ /home/ec2-user/SageMaker/

```

## Step 3: (Optional) Rename the new notebook to match the old one

Once you've synced the snapshot data to your new notebook, you can rename it to match the original name:

1. Delete the old notebook.
2. Create a new notebook with the same name as the original.
3. Open the terminal in the new notebook and run this script:

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

Finally, delete the notebook with -new in its name.

🎉 And that's it! You now have a fresh notebook with all the data from your old one, running on the latest version of SageMaker Studio! 💻✨

Feel free to reach out if you have any questions. Happy coding! 🚀

Additional Resources
AWS Blog: Migrate Your Work to Amazon SageMaker Notebook Instance with Amazon Linux 2
