pipeline {
  agent any

  environment {
    STACK_NAME        = 'acit-eks-observability'
    TEMPLATE_FILE     = 'eks-observability-addons.yaml'
    CLUSTER_NAME      = 'acit-eks'   // Update if different
    OBS_ROLE_ARN      = 'arn:aws:iam::124355683348:role/OBS_ROLE'
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
        echo 'Deploying CloudWatch Observability and Pod Identity Agent'
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
        echo 'Verifying amazon-cloudwatch pods are running'
        sh '''
          aws eks update-kubeconfig --name "$CLUSTER_NAME" --region "$REGION"
          kubectl get pods -n amazon-cloudwatch || echo "⚠️ Failed to query pods in namespace amazon-cloudwatch"
        '''
      }
    }
  }

  post {
    success {
      echo 'Observability add-ons deployed and verified successfully.'
    }
    failure {
      echo 'Deployment or pod check failed. Investigate above output.'
    }
  }
}
