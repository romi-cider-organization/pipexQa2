pipeline {
    agent any
    environment {
            DISABLE_AUTH = 'true'
            DB_ENGINE    = 'sqlite'
    }
    stages {
        stage('cucumber tests') {
            steps {
                dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                        cd IntegrationTests/
                        npm install
                        ./node_modules/.bin/wdio --suite login
                        ./node_modules/junit-viewer/bin/junit-viewer --results=reports --save=reports/index.html
                    """
                     publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'IntegrationTests/reports',
                        reportFiles: 'index.html',
                        reportName: "Cucumber Tests Report"
                    ])
                }
            }
        }
        stage ('Maven functional test'){
            steps {
                dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                        mvn test
                        mvn test -P SmokeTests
                    """
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/surefire-reports',
                        reportFiles: 'index.html',
                        reportName: "Smoke Tests Report"
                    ])
                }
            }
        }
        stage ('Maven selenium functional test'){
            steps {
                dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                    cd IntegrationTests/
                    npm install wdio-phantomjs-service
                    cd ../
                    mvn test -P SeleniumTests
                    """
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/surefire-reports',
                        reportFiles: 'index.html',
                        reportName: "Selenium Tests Report"
                    ])
                }
            }
        }
        stage('Performance testing') {
            steps {
                dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                    /opt/gatling/bin/gatling.sh -nr -rf \${PWD}/performanceTests/gatling/results -sf \${PWD}/performanceTests/gatling -s computerdatabase.advanced.AdvancedSimulationStep06
                    mkdir -p performanceTests/gatling/results/reports
                    ls -la performanceTests/gatling/results/
                    mv performanceTests/gatling/results/advancedsimulationstep06*/simulation.log performanceTests/gatling/results/reports/06.log
                    /opt/gatling/bin/gatling.sh -ro \${PWD}/performanceTests/gatling/results/reports
                    """
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: 'performanceTests/gatling/results/reports',
                    reportFiles: 'index.html',
                    reportName: "Performance Tests Report"
                    ])
                }
            }
        }
        stage('bruteForce testing') {
             steps {
                 dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                      cd penetration_testing
                      python bruteForceOpenzip.py -f locked.zip -d dictionary.txt
                    """
                }
             }
        }
        stage('bruteForce Site testing') {
            steps {
                dockerNode(image: "jawedm/automation-jenkins") {
                    git "https://github.com/jawedmokhtar/testing-artifact"
                    sh """
                        cd penetration_testing
                        python3 bruteForceSite.py -H http://automationpractice.com/index.php?controller=authentication -u jmores047@gmail.com -F dictionary.txt
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'This will always run'
            gatlingArchive()
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}