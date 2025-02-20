def newFunction (arg1, arg2, output) {
    echo "${env.JOB_NAME}"
    echo "${env.BUILD_ID}"
    echo "${env.output}"
}

def output = ""

pipeline {
    agent any

    parameters {
        string defaultValue: 'pipeline.zip', description: 'Input build parameter  - final zipfile name', name: 'ARTIFACT_NAME'
        booleanParam defaultValue: true, description: 'Controls if pipeline needs to fail', name: 'FAIL_PIPELINE'
        string defaultValue: 'pipeline', description: 'Default branch', name: 'GIT_BRANCH'
        booleanParam defaultValue: true, description: 'Run test', name: 'RUN_TEST'
        booleanParam defaultValue: true, description: 'Send email', name: 'SEND_EMAIL'
    }
    
    stages {
        stage("Download"){
            steps {
                cleanWs()
                
                // withCredentials (
                //   [usernamePassword(creadentialsID: 'VK', passwordVariable: 'password', usernameVariable: 'userName')]
                // ){
                //     echo "password: ${password}"
                //     echo "userName: ${userName}"
                // }
                
                echo "Download stage"
                dir('jenkins-course')
                {
                    echo "To!"
                    git branch: 'pipeline', url: 'https://github.com/KLevon/jenkins-course.git'
                }
                rtDownload(
                    serverId: 'Artifactory',
                    spec: '''{
                        "files": [
                        {
                            "pattern": "generic-local/libraries/printer.zip",
                            "target" : "printer.zip",
                            "flat": true
                        }
                        ]
                    }'''
                )
                unzip(
                    zipFile: "printer.zip",
                    dir: "jenkins-course"
                    )
            }
        }
        stage("Build"){
            steps {
                echo "Build stage"
                dir('jenkins-course')
                {
                    bat 'Makefile.bat'
                }
            }
        }
        stage("Test"){
            when {
                    equals expected: true,
                    actual: params.RUN_TEST
                }
                
            steps {
                echo "Test stage"
                
                script {
                env.output = ""   
                def array = ['printer', 'scanner', 'main']
                for (elem in array) {
                    env.output += bat(
                        script: """
                            jenkins-course/Tests.bat $elem
                        """, returnStdout: true
                        ).trim()
                    }
                }
            }
        }
        stage("Publish"){
            environment{
                GLOBAL = 'Global!'
            }
            steps {
                echo "Publish stage"
                script {
                    zip (
                        zipFile: "${ARTIFACT_NAME}",
                        archive: true,
                        dir: "jenkins-course")
                }
                 rtUpload (
                     serverId: 'Artifactory',
                     spec: """{
                        "files": [
                        {
                            "pattern": "${ARTIFACT_NAME}",
                            "target" : "generic-local/vanja/${BUILD_ID}/${params.ARTIFACT_NAME}/"
                        }
                        ]
                    }"""
                    )
                script {
                    if (FAIL_PIPELINE == "true")
                    {
                        bat "exit 1"
                    }
                }
                // script {
                //     newFunction (env.JOB_NAME, env.BUILD_ID)
                // }
                script{
                    sh 'printenv'
                }
            }
        }
    }
    
    post {
        failure{
            script {
                if (SEND_EMAIL == "true"){
                    newFunction (env.JOB_NAME, env.BUILD_ID, env.output)
                }
            }
        }
        success{
            echo "Build success!"
        }
    }
}



