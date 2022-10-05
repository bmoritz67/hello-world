pipeline {
    agent any

    tools {
        maven "maven 3.633"
    }

    stages {
        stage('Hello_World Maven') {
            steps {
                git 'https://github.com/bmoritz67/hello-world.git'
                sh "mvn clean install package"
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }
        stage('Hello_World Sonar') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn clean package sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL"
                }
            }
        }
        stage('Hello_World Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'webapp/target/webapp.war']], mavenCoordinate: [artifactId: 'maven-project', groupId: 'com.example.maven-project', packaging: 'war', version: '1.2']]]
            }
        }
        stage('Hello_World Docker Build') {
            steps {
                sh "rm -Rf webapp.war && \
                wget http://nexus:8081/repository/maven-releases/com/example/maven-project/maven-project/1.2/maven-project-1.2.war -O ${WORKSPACE}/webapp.war && \
                docker build -t hello-world-afip:latest ."
             }
        }
        stage('Hello_World Docker Run') {
            steps {
                script {
                    def set_container = sh(script: '''CONTAINER_NAME="hello-world-run"
                                                    OLD="$(docker ps --all --quiet --filter=name="$CONTAINER_NAME")"
                                                    if [ -n "$OLD" ]; then
                                                        docker rm -f $OLD
                                                    fi
                                                    docker run -d --name hello-world-run -p 18090:8080 hello-world-afip
                                                ''')
                    }
                }
            }
        stage('Hello_World JMeter') {
            steps {
                sh "jmeter -JUSER=100 -Jjmeter.save.saveservice.output_format=xml -Jjmeter.save.saveservice.response_data.on_error=true -n -t jmeter_test_plan.jmx  -l testresult.jlt"
                logParser failBuildOnError: true, parsingRulesPath:'', useProjectRule: true, projectRulePath: 'parserules'
                }
            }


    }
}
