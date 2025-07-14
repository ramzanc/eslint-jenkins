pipeline {
  agent any

  environment {
    LINT_REPORT = 'eslint-report.txt'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        bat 'git fetch --all'
      }
    }

    stage('Determine Base Commit') {
        steps {
            script {
                def fallback = 'origin/main'
                def hasPrev = bat(
                    script: 'git rev-parse --verify %GIT_PREVIOUS_SUCCESSFUL_COMMIT%',
                    returnStatus: true
                ) == 0

                if (hasPrev) {
                    def prev = bat(
                    script: 'git rev-parse %GIT_PREVIOUS_SUCCESSFUL_COMMIT%',
                    returnStdout: true
                    ).trim()
                    env.BASE_COMMIT = prev
                    echo "✅ Using previous successful commit: ${env.BASE_COMMIT}"
                } else {
                    def fallbackCommit = bat(
                    script: "git rev-parse ${fallback}",
                    returnStdout: true
                    ).trim()
                    env.BASE_COMMIT = fallbackCommit
                    echo "🔁 Fallback to ${fallback}: ${env.BASE_COMMIT}"
                }
            }
        }
    }

    stage('Find Changed JS/TS Files') {
    steps {
        script {
            echo "📌 BASE_COMMIT: ${env.BASE_COMMIT}"

            // Write the diff output to a file safely
            bat """
                git diff --name-only ${env.BASE_COMMIT} HEAD -- *.js *.ts > changed.txt
            """

            def changed = readFile('changed.txt').trim()
            if (!changed) {
                echo "✅ No changed JS/TS files"
                currentBuild.result = 'SUCCESS'
                skipRemainingStages()
            } else {
                env.ESLINT_FILES = changed.replaceAll('[\\r\\n]+', ' ')
                echo "🎯 Changed files: ${env.ESLINT_FILES}"
            }
        }
    }
}




    stage('Run ESLint') {
      steps {
        script {
          try {
            bat "npx eslint %ESLINT_FILES% --format stylish > %LINT_REPORT%"
          } catch (e) {
            bat "type %LINT_REPORT%"
            error("❌ ESLint failed")
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: env.LINT_REPORT, onlyIfSuccessful: false
        }
      }
    }
  }

  post {
    success { echo "✅ Build Success" }
    failure { echo "❌ Build Failed on ESLint errors" }
  }
}
