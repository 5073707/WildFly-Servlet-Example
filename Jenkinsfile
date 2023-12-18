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
    stage('Deploy') { 
            steps {
                sudo apt update && sudo apt upgrade -y
                sudo apt install openjdk-11-jdk -y
                sudo apt install apache2 -y

                wget https://github.com/wildfly/wildfly/releases/download/30.0.0.Final/wildfly-30.0.0.Final.tar.gz
                tar xf wildfly-30.0.0.Final.tar.gz
                sudo mv wildfly-30.0.0.Final /opt/wildfly

                sudo useradd --system --no-create-home --user-group wildfly
                sudo chown -R wildfly:wildfly /opt/wildfly
                sudo bash -c 'cat > /etc/systemd/system/wildfly.service << EOF
                [Unit]
                Description=The WildFly Application Server
                After=syslog.target network.target
                
                [Service]
                User=wildfly
                Group=wildfly
                ExecStart=/opt/wildfly/bin/standalone.sh -b=0.0.0.0
                ExecReload=/bin/kill -HUP $MAINPID
                Restart=always
                RestartSec=10
                
                [Install]
                WantedBy=multi-user.target
                EOF'

                sudo systemctl daemon-reload
                sudo systemctl start wildfly
                sudo mv your-app.war /opt/wildfly/standalone/deployments/
                sudo systemctl enable wildfly
                sudo mkdir /etc/apache2/ssl
                sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt

                sudo bash -c 'cat > /etc/apache2/sites-available/000-default.conf << EOF
                <VirtualHost *:80>
                   ServerName localhost
                   Redirect / https://localhost/
                </VirtualHost>
                
                <VirtualHost *:443>
                   SSLEngine on
                   SSLCertificateFile /etc/apache2/ssl/apache.crt
                   SSLCertificateKeyFile /etc/apache2/ssl/apache.key
                
                   ProxyRequests Off
                   ProxyPreserveHost On
                   ProxyPass / http://localhost:8080/
                   ProxyPassReverse / http://localhost:8080/
                </VirtualHost>
                EOF'

                sudo a2enmod ssl
                sudo a2enmod proxy
                sudo a2enmod proxy_http

                sudo systemctl restart apache2
                
                
                // 
            }
        }
            
    post { 
        always { 
            echo 'Hello!'
        }
    }
        }
    }
}
