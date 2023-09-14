node {
    stage('Build') {
        def pythonImage = 'python:3.11.5-alpine3.18'
        checkout scm
        docker.image(pythonImage).inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    stage('Test') {
        def pytestImage = 'qnib/pytest'
        docker.image(pytestImage).inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }
}
