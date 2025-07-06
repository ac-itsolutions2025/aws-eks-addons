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
        echo 'Cloning repo containing observability template'
        checkout scm
      }
    }

    stage('Deploy Observability Add-ons') {
      steps {
        echo 'Deploying CloudWatch Observability and Pod Identity Agent to EKS cluster'
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

    stage('Confirm Stack Status') {
      steps {
        echo 'Verifying EKS Observability stack outputs'
        sh '''
          aws cloudformation describe-stacks \
            --stack-name "$STACK_NAME" \
            --query "Stacks[0].Outputs[*].[OutputKey,OutputValue]" \
            --output table || echo 'No outputs available (this is expected for this stack)'
        '''
      }
    }
  }

  post {
    success {
      echo 'Observability add-ons deployed successfully!'
    }
    failure {
      echo 'Deployment failed. Please review CloudFormation events for more context.'
    }
  }
}
