node {
    stage('Build') {
        docker.image('python:2-alpine').inside { 
            sh 'python -m py_compile ./sources/add2vals.py ./sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml ./test-reports/results.xml ./sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }
    stage('Manual Approval') {
        try {
            input message: 'Apakah Anda ingin melanjutkan ke tahap deploy?',
                    ok: 'Proceed',
                    parameters: []
        } catch (err) {
            echo "Pipeline dihentikan oleh pengguna pada tahap Manual Approval."
            currentBuild.result = 'ABORTED'
            error("Pipeline dihentikan oleh pengguna.")
        }
    }
    stage('Deploy') {
        docker.image('cdrx/pyinstaller-linux:python2').inside('--user root') {
            sh '''
            echo "Checking if Git is installed..."
            if ! command -v git &> /dev/null; then
                echo "Git is not installed. Installing Git..."
                apt-get update && apt-get install -y git
            else
                echo "Git is already installed."
            fi
            '''

            sh 'pyinstaller --onefile sources/add2vals.py'
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
            sh '''
            echo 'Configuring Git in Docker environment...'
            git config --global user.name "Your Name"
            git config --global user.email "your.email@example.com"
            git config --global --add safe.directory /var/jenkins_home/workspace/submission-cicd-pipeline-Clown69

            # Pastikan Git sudah diinisialisasi
            
            echo "Initializing Git repository..."
            git init
            git remote set-url origin https://${GITHUB_TOKEN}@github.com/RestuAlamBagaskara/simple-python-installer-app.git
            

            # Pastikan branch target ada
            git fetch origin master
            git checkout master
            git pull origin master

            git add .
            git status
            git commit -m "Jenkins: Deployed build files" || echo "No changes to commit"
            git push origin master --verbose || echo "Failed to push changes"
            '''
            }
            sleep 60
        }
    }
}