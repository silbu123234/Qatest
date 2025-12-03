pipeline {
  agent any
  tools { nodejs 'nodejs' }   // Manage Jenkins > Global Tool Configuration에 등록한 이름

  environment {
    REPORT_DIR = 'newman'
    ENV_FILE   = 'GameQA_ENV.postman_environment.json'
  }

  stages {
    stage('Prepare') {
      steps {
        bat '''
          chcp 65001 > nul
          if not exist "%REPORT_DIR%" mkdir "%REPORT_DIR%"
        '''
      }
    }

    stage('Install Newman (local)') {
      steps {
        // 글로벌(-g) 설치 대신 workspace에 로컬 설치 -> PATH 문제 제거
        bat '''
          chcp 65001 > nul
          npm config set fund false
          npm config set audit false
          npm install --no-progress --save-dev newman newman-reporter-htmlextra
        '''
      }
    }

    stage('Run Postman Collections') {
      steps {
        // npx로 실행하면 로컬 설치된 newman을 확실히 찾습니다.
        bat """
          chcp 65001 > nul

          npx --no newman run Login_API_Test.postman_collection.json ^
            -e "%ENV_FILE%" ^
            -r cli,htmlextra ^
            --reporter-htmlextra-export "%REPORT_DIR%\\Login_Report.html" ^
            --export-environment "%ENV_FILE%"

          npx --no newman run Ranking_API_Test.postman_collection.json ^
            -e "%ENV_FILE%" ^
            -r cli,htmlextra ^
            --reporter-htmlextra-export "%REPORT_DIR%\\Ranking_Report.html"

          npx --no newman run Item_Grant_API_Test.postman_collection.json ^
            -e "%ENV_FILE%" ^
            -r cli,htmlextra ^
            --reporter-htmlextra-export "%REPORT_DIR%\\ItemGrant_Report.html"
        """
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'newman/*.html', fingerprint: true

      publishHTML(target: [
        reportDir: 'newman',
        reportFiles: 'Login_Report.html,Ranking_Report.html,ItemGrant_Report.html',
        reportName: 'Newman Reports',
        keepAll: true,
        alwaysLinkToLastBuild: true
      ])
    }
  }
}