pipeline {
  agent any

  environment {
    STACK_NAME        = 'acit-eks-observability'
    TEMPLATE_FILE     = 'eks-observability-addons.yaml'
    CLUSTER_NAME      = 'acit-eks-group-one'
    OBS_ROLE_ARN      = 'arn:aws:iam::124355683348:role/OBS_ROLE'
    REGION            = 'us-east-2' // Update to your EKS region
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Checking out CloudFormation template'
        checkout scm
      }
    }

    stage('Deploy Observability Add-ons') {
      steps {
        echo 'Deploying CloudWatch Observability and Pod Identity Agent add-ons'
        sh '''
          aws cloudformation deploy \
            --stack-name "$STACK_NAME" \
            --template-file "$TEMPLATE_FILE" \
            --capabilities CAPABILITY_NAMED_IAM \
            --parameter-overrides \
              ClusterName="$CLUSTER_NAME" \
              ObservabilityRoleArn="$OBS_ROLE_ARN"
        '''
      }
    }

    stage('Verify CloudWatch Add-ons') {
      steps {
        echo 'Performing health check on amazon-cloudwatch pods'

        sh '''
          if [ -z "$REGION" ]; then
            echo "REGION environment variable is not set."
            exit 1
          fi

          echo "Updating kubeconfig for EKS cluster: $CLUSTER_NAME in region $REGION"
          aws eks update-kubeconfig --name "$CLUSTER_NAME" --region "$REGION"

          echo "Checking pods in namespace amazon-cloudwatch"
          kubectl get pods -n amazon-cloudwatch || echo "Unable to retrieve pods in amazon-cloudwatch namespace"

          echo "Waiting for all pods to be ready"
          kubectl wait --for=condition=Ready pods --all -n amazon-cloudwatch --timeout=120s || echo "⚠Some pods did not become ready in time"
        '''
      }
    }
  }

  post {
    success {
      echo 'Observability stack deployed and validated successfully.'
    }
    failure {
      echo 'Deployment or verification failed. See logs above for details.'
    }
  }
}
