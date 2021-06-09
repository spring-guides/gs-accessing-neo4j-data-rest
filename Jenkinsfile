pipeline {
	agent none

	triggers {
		pollSCM 'H/10 * * * *'
	}

	options {
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '14'))
	}

	stages {
		stage('Publish OpenJDK 8 + Neo4j') {
			when {
				changeset "ci/Dockerfile"
			}
			agent any

			steps {
				script {
					def image = docker.build("springci/gs-accessing-neo4j-data-rest-neo4j-with-tools", "ci/")
					docker.withRegistry('', 'hub.docker.com-springbuildmaster') {
						image.push()
					}
				}
			}
		}

		stage("test: baseline (jdk8)") {
			agent {
				label 'data'
			}
			options { timeout(time: 30, unit: 'MINUTES') }

			environment {
				DOCKER_HUB = credentials('hub.docker.com-springbuildmaster')
			}

			steps {
				script {
					docker.withRegistry('', 'hub.docker.com-springbuildmaster') {
						docker.image('springci/gs-accessing-neo4j-data-rest-neo4j-with-tools:latest').inside('-u root -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -v $HOME:/tmp/jenkins-home') {
							sh "docker login --username ${DOCKER_HUB_USR} --password ${DOCKER_HUB_PSW}"
							sh '/neo4j-community-installed/bin/neo4j start &'
							sh 'sleep 1'
							sh './wait-for-neo4j.bash'
							sh 'curl -v -u neo4j:neo4j -X POST localhost:7474/user/neo4j/password -H "Content-type:application/json" -d \'{\"password\":\"secret\"}\''
							sh 'test/run.sh'
						}
					}
				}
			}
		}

	}

	post {
		changed {
			script {
				slackSend(
						color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
						channel: '#sagan-content',
						message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")
				emailext(
						subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
						mimeType: 'text/html',
						recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
						body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
			}
		}
	}
}
