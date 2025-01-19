node {
    stage('Build') {
        docker.image('python:2-alpine').inside { 
            sh 'python -m py_compile $WORKSPACE/sources/add2vals.py $WORKSPACE/sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml $WORKSPACE/test-reports/results.xml $WORKSPACE/sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile $WORKSPACE/sources/add2vals.py'
        }
        archiveArtifacts 'dist/add2vals'
    }
}