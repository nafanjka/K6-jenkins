pipeline {
  agent any

  parameters {
    choice(name: 'RUN_MODE', choices: ['local', 'cloud'], description: 'Local run vs k6 Cloud')
    choice(name: 'OUTPUT_MODE', choices: ['none', 'cloud', 'json'], description: 'Extra output (local only)')
    booleanParam(name: 'QUIET_MODE', defaultValue: true, description: 'Use --quiet for local runs')
    string(name: 'HOST_URL', defaultValue: 'http://localhost:8000', description: 'Target API base URL')
    string(name: 'VUS', defaultValue: '5', description: 'Virtual users')                  // updated default
    string(name: 'DURATION', defaultValue: '2m', description: 'Test duration (e.g. 30s, 2m)')
    string(name: 'JSON_FILENAME', defaultValue: 'detailed.json', description: 'Only used when OUTPUT_MODE=json')
    string(name: 'CLOUD_PROJECT_ID', defaultValue: '6019612', description: 'Only used when RUN_MODE=cloud')
    booleanParam(name: 'ENABLE_DASHBOARD', defaultValue: false, description: 'Enable K6_WEB_DASHBOARD (local run only)')
  }
// previous emulated step
  stages {
    stage('Build & Deploy') {
      steps {
        echo 'Building app...'

        sh 'echo build ok'
        echo 'Deploying app...'

        sh 'echo deploy ok'
      }
      post {
        success {
          echo 'Build & Deploy succeeded.'
        }
        unsuccessful {
          echo 'Build & Deploy failed. Stopping pipeline.'
        }
        failure {
          error 'Build & Deploy failed. Pipeline aborted.'
        }
      }
    }

    stage('Checkout Test Repo') {
      steps {
        git branch: 'main', url: 'https://github.com/nafanjka/K6-lessons.git'
      }
    }

    stage('Performance Test (k6)') {
      steps {
        script {
          def cmd = []
          if (params.RUN_MODE == 'local' && params.ENABLE_DASHBOARD) {
            cmd << 'K6_WEB_DASHBOARD=true'
          }
          if (params.RUN_MODE == 'cloud') {
            cmd << "K6_CLOUD_PROJECT_ID=${params.CLOUD_PROJECT_ID}"
          }

          cmd << 'k6'
          cmd << (params.RUN_MODE == 'cloud' ? 'cloud' : 'run')

          if (params.RUN_MODE == 'local') {
            if (params.QUIET_MODE) {
              cmd << '--quiet'
            }

            if (params.OUTPUT_MODE == 'cloud') {
              cmd << '--out cloud'
            } else if (params.OUTPUT_MODE == 'json' && params.JSON_FILENAME?.trim()) {
              cmd << "--out json=${params.JSON_FILENAME.trim()}"
            }
          }

          cmd << "-e BASE_URL=${params.HOST_URL}"
          cmd << "-e VUS=${params.VUS}"
          cmd << "-e DURATION=${params.DURATION}"
          cmd << "-e CLOUD_PROJECT_ID=${params.CLOUD_PROJECT_ID}"
          cmd << 'fulle2etest.js'

          def status = sh(script: cmd.join(' '), returnStatus: true)
          echo "k6 exit code: ${status}"
          if (status != 0) {
            error 'Performance test failed. Pipeline aborted.'
          }
        }
      }
      post {
        success {
          echo 'k6 performance test succeeded.'
        }
        failure {
          echo 'k6 performance test failed.'
        }
      }
    }
    
// emulating next ci step
    stage('Promote') {
      steps {
        echo 'Performance test passed. Promoting to next environment...'

      }
    }
  }

  post {
    failure {
      echo 'Pipeline failed during performance test; skipping promotion.'
    }
  }
}