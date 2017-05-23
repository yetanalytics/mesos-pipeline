def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t tenkster/mesos-pipeline:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'henk-docker',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e hnk.reder@gmail.com"
        sh "docker push tenkster/mesos-pipeline:${gitCommit()}"
    }
}

 // Deploy
    stage 'Deploy'

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: false,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appid: 'nginx-pipeline-test',
        docker: "tenkster/mesos-pipeline:${gitCommit()}".toString()
    )
