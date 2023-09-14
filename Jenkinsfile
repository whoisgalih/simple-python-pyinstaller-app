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

    stage('Deliver') {
        def volume = "${pwd()}/sources/src"
        def image = 'cdrx/pyinstaller-linux:python2'
        node {
            try {
                dir(buildId) {
                    unstash name: 'compiled-results'
                    sh "docker run --rm -v ${volume} ${image} 'pyinstaller -F add2vals.py'"
                }
            } finally {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v ${volume} ${image} 'rm -rf build dist'"
            }
        }
    }
}
