pipeline {
    
    agent any
    
    tools{
        maven 'Maven'
        
    }

    stages {
        
        stage ('Initialise'){
            
            steps{
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
            }
        }

         stage('Build and analyze code') {

            steps {
                 sh '''
                    # Set up CodeQL paths
                    export CODEQL_HOME=/home/kali/codeql
                    export CODEQL_PATH=$CODEQL_HOME/codeql
                    export PATH=$CODEQL_HOME:$PATH

                    # Initialize CodeQL database
                    $CODEQL_PATH database init --language=java --source-root=app --build-mode=manual --overwrite --begin-tracing ./codeql-dbs/webapp2-db

                    # Start CodeQL tracing environment
                    . ./codeql-dbs/webapp2-db/temp/tracingEnvironment/start-tracing.sh

                    # Clean Maven build to ensure javac is traced
                    mvn clean compile -f app/pom.xml

                    # Finalize CodeQL database
                    $CODEQL_PATH database finalize ./codeql-dbs/webapp2-db

                    # End tracing environment (unset vars)
                    . ./codeql-dbs/webapp2-db/temp/tracingEnvironment/end-tracing.sh

                    # Run CodeQL analysis using built-in Java query suite
                    mkdir -p ./temp
                    $CODEQL_PATH database analyze ./codeql-dbs/webapp2-db java-security-extended.qls --format=sarif-latest --output=./temp/results-java.sarif
                '''
            }        
    }
              stage('Upload results to GitHub') {
                   steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_PAT')]) {
                    sh '''
                        echo "$GITHUB_PAT" | $CODEQL_PATH github upload-results \
                          --sarif=./temp/results-java.sarif \
                          --github-auth-stdin \
                          --github-url=https://github.com/ \
                          --repository=p4rz1v4l-VB/verademo \
                          --ref=refs/heads/main
                    '''
                }
            }
            
        }
    
}
}
