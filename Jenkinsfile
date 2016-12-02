#!groovy

node {
   properties([
     buildDiscarder(logRotator(
       artifactDaysToKeepStr: '3',
       artifactNumToKeepStr: ''5,
       daysToKeepStr: '',
       numToKeepStr: '')),
     disableConcurrentBuilds(),
     [$class: 'RebuildSettings', autoRebuild: true, rebuildDisabled: false],
     pipelineTriggers([
       [$class: "TimerTrigger", spec: "5 * * * *"]
     ])
   ])
   stage('Preparation') { // for display purposes
      // Get 'code' from a GitHub repository
      git 'https://github.com/danm-slalom/pipeline.git'
   }
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "./copy_red_shift/copy_red_shift_run.sh"
      } else {
         bat(/copy_red_shift\copy_red_shift_run.bat/)
      }
   }
   stage('Results') {
     echo "If we had run tests, this is the the satge in which we would process them."
   }
   stage('Publish') {
      echo "This is the stage at which we would ship off the freshly built artifact to the binary repository."
   }
   stage('Sign Off') {
      echo "And thus concludes the pipeline build.  May you go in peace."
   }
}
