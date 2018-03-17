pipeline {
	agent none 

	environment {
		MAJOR_VERSION = 1
	}

	options {
		buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
	}
	
	stages {
		stage('Unit Tests') {
			agent {label 'apache'}
			steps {
				sh 'ant -f test.xml -v'
				junit 'reports/result.xml'
			}
		}

		stage('build') {
			agent {label 'apache'}
			steps {
				sh 'ant -f build.xml -v'
			}
			post {
                		success {
                        		archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
                		}
        		}
		}

		stage('deploy') {
			agent {label 'apache'}
			steps {
				sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
				sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
			}
		}

		stage('Running on CentOS') {
			agent {label 'CentOS'}
			steps {
				sh "wget http://34.209.28.140/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
			}
		}

		stage('Running on Debian') {
			agent {
				docker 'openjdk:8u151-jre'
			}
			steps {
				sh "wget http://34.209.28.140/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
                                sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
			}
		}

		stage('Promote to Green') {
			agent {label 'apache'}
			when {
				branch 'master'
			}
			steps {
				sh "cp /var/www/html/rectangles/all/master/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
			}
		}

		stage('Promote Development Branch to Master') {
			agent {label 'apache'}
			when {branch 'development'}
			steps {
				echo "Stashing Any Local Changes"
				sh 'git stash'
				echo "Checking Out Development Branch"
				sh 'git checkout development'
				echo "Updating Local Development Branch"
				sh 'git pull'
				echo "Checking Out Master Branch"
				sh 'git checkout master'
				echo "Merging Development into Master"
				sh 'git merge development'
				echo "Pushing to Origin Master"
				sh 'git push origin master'
				echo "Tagging the Release"
				sh "git tag rectangle-${MAJOR_VERISION}.${env.BUILD_NUMBER}"
				sh "pit push origin rectangle-${MAJOR_VERISION}.${env.BUILD_NUMBER}"
			}
		}
	}
}
