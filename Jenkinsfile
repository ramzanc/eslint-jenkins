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
                bat 'git status'
                def changed = bat(
                    script: "git diff --name-only ${env.BASE_COMMIT}..HEAD -- \"*.js\" \"*.ts\"",
                    returnStdout: true
                ).trim()

                if (!changed) {
                    echo "✅ No JS/TS changes found since ${env.BASE_COMMIT}"
                    currentBuild.result = 'SUCCESS'
                    skipRemainingStages()
                } else {
                    env.ESLINT_FILES = changed.replaceAll('\r\n', ' ').replaceAll('\n', ' ')
                    echo "🎯 Linting changed files:\n${env.ESLINT_FILES}"
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
