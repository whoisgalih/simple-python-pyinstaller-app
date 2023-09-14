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
        def volume = "${pwd()}/sources:/src"
        def image = 'cdrx/pyinstaller-linux:python2'
        def buildId = env.BUILD_ID
        node {
            try {
                dir(buildId) {
                    unstash name: 'compiled-results'
                    sh "docker run --rm -v ${volume} ${image} 'pyinstaller -F add2vals.py'"
                }
            } finally {
                archiveArtifacts artifacts: "${buildId}/compiled-results/sources/dist/add2vals", allowEmptyArchive: true
                sh "docker run --rm -v ${volume} ${image} 'rm -rf build dist'"
            }
        }
    }
}
