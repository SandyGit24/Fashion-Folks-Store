name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout the code
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Log in to AWS ECR
      - name: Log in to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
        env:
          AWS_REGION: eu-west-1
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      # Step 3: Build the Docker image
      - name: Build Docker image
        run: |
          docker build -t dublin-blog .
          docker tag dublin-blog:latest 677442829651.dkr.ecr.eu-west-1.amazonaws.com/dublin-blog:latest

      # Step 4: Push Docker image to ECR
      - name: Push to Amazon ECR
        run: |
          docker push 677442829651.dkr.ecr.eu-west-1.amazonaws.com/dublin-blog:latest

      # Step 5: SSH into EC2 and deploy the container
      - name: Deploy on EC2
        uses: appleboy/ssh-action@v0.1.8
        with:
          host: ${{ secrets.EC2_PUBLIC_IP }}
          username: ec2-user
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            # Configure AWS CLI with IAM User credentials
            aws configure set aws_access_key_id ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws configure set aws_secret_access_key ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws configure set region eu-west-1

            # Authenticate Docker with ECR
            aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 677442829651.dkr.ecr.eu-west-1.amazonaws.com

            # Pull the latest Docker image from ECR
            docker pull 677442829651.dkr.ecr.eu-west-1.amazonaws.com/dublin-blog:latest

            # Stop and remove the existing container if it exists
            docker stop dublin-blog || true
            docker rm dublin-blog || true

            # Run the updated container
            docker run -d -p 8000:8000 --name dublin-blog 677442829651.dkr.ecr.eu-west-1.amazonaws.com/dublin-blog:latest

            
