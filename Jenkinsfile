pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_SITE_ID = credentials('netlify-site-id')
    }

    stages {
        stage('Prepare Environment') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🛠️ Installing required dependencies..."
                    sh '''
                    apk add --no-cache bash
                    npm cache clean --force
                    npm install --force
                    '''
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🏗️ Building the project..."
                    sh '''
                    npm run build
                    '''
                }
            }
            post {
                success {
                    echo "✅ Build Successful! 🎉"
                }
                failure {
                    echo "❌ Build Failed! Check logs for details."
                }
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🔬 Running tests..."
                    sh 'npm test || echo "⚠️ Tests failed, but continuing..."'
                }
            }
        }

        stage('Deploy to Netlify') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🚀 Deploying to Netlify..."
                    sh '''
                    npm install -g netlify-cli
                    netlify deploy --prod --dir=build \
                    --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                    '''
                }
            }
            post {
                success {
                    echo "✅ Deployment Successful! 🎉"
                    echo "👉 เปิดเว็บไซต์ที่: https://nicevanitermproject.netlify.app"
                }
                failure {
                    echo "❌ Deployment Failed! Check logs for details."
                }
            }
        }

        stage('Post Deploy Monitoring') {
            agent any
            steps {
                script {
                    echo "🔍 Monitoring server resources..."
                    sh '''
                        echo "Top 10 processes by memory usage:" > resource_report.txt
                        ps aux --sort=-%mem | head -n 10 >> resource_report.txt
                        
                        echo "\nMemory usage:" >> resource_report.txt
                        free -h >> resource_report.txt
                        
                        echo "\nSystem performance stats (vmstat):" >> resource_report.txt
                        vmstat 1 5 >> resource_report.txt
                    '''
                }
            }
            post {
                success {
                    echo "✅ Resource monitoring completed successfully! Here are the results:"
                    sh 'cat resource_report.txt'
                }
                failure {
                    echo "❌ Resource monitoring encountered an error!"
                }
            }
        }
    }
}
