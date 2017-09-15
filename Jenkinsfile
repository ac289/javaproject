pipeline {
  agent none

  options {
	buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr: '1'))
  }

  stages {
    stage ('Unit Tests') {
	  agent {
		label 'apache'
	  }

	  steps {
		sh 'ant -f test.xml -v'
		junit 'reports/result.xml'
	  }
	}
	stage('build') {
      agent {
		label 'apache'
	  }
	  steps {
		sh 'ant -f build.xml -v'
	  }
	  post {
		success {
			archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
		}
	  }
    }
	stage('Deploy') {
	  agent {
		label 'apache'
	  }
	  steps {
        sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
	  }
	}
	stage("Running on CentOS") {
	  agent {
		label 'CentOS'
	  }
	  steps {
			sh "wget http://alancamp6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
			sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
	  }
	}
	stage("Test on Debian") {
		agent {
			docker 'openjdk:8u121-jre'
		}
		steps {
			sh "wget http://alancamp6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
			sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
		}
	}
  stage('Promote to Green') {
    agent {
      label 'apache'
   }
    when {
	  branch 'master'
   }
    steps {
        sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"
      }
    }
	stage('Promote Development Branch to Master') {
      agent {
        label 'apache'
      }
      when {
        branch 'development'
      }
      steps {
        echo "Stashing Any Local Changes"
        sh 'git stash'
        echo "Checking Out Development Branch"
        sh 'git pull origin development'
        sh 'git checkout development'
        echo 'Checking Out Master Branch'
        sh 'git pull origin'
        sh 'git checkout master'
        echo 'Merging Development into Master Branch'
        sh 'git merge development'
        echo 'Pushing to Origin Master'
        sh 'git push origin master'
        echo 'Tagging the Release'
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
		}
	}
  }
}

