pipeline {
    agent {
      node {
        label "windows && linux"
      }
    }
    stages {
        stage("Build"){
            steps {
                echo("Hello Build")
            }
        }
        stage("Test"){
            steps {
                echo("Hello Test")
            }
        }
        stage("Deploy"){
            steps {
                echo("Hello Deploy")
            }
        }
        stage("Post-deploy"){
            steps {
                echo("Hello Post-Deploy")
            }
        }
    }
}
