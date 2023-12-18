pipeline {
    agent { 
        label "ubuntu-slave" 
    }

    tools {
        // Install the Maven.
        maven "M3"
    }

    stages {
        stage('Build') {
            steps {
                sh " rm -rf WildFly-Servlet-Example"
                
                // Get some code from a GitHub repository
                sh "git clone https://github.com/5073707/WildFly-Servlet-Example.git"

                // Run Maven on a Unix agent.
                sh "cd WildFly-Servlet-Example/ && mvn clean install"

            }
    
    post { 
        always { 
            echo 'Hello!'
        }
    }
        }
    }
}
