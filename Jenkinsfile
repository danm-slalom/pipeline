#!groovy

node {
   properties([
     buildDiscarder(logRotator(
       artifactDaysToKeepStr: '5',
       artifactNumToKeepStr: '10',
       daysToKeepStr: '7',
       numToKeepStr: '10')),
     disableConcurrentBuilds(),
     [$class: 'RebuildSettings', autoRebuild: true, rebuildDisabled: false],
     pipelineTriggers([
       [$class: "TimerTrigger", spec: "35 * * * *"]
     ])
   ])
   stage('Preparation') {
      // Get 'code', in this case from a GitHub repository
      git 'https://github.com/danm-slalom/pipeline.git'
   }
   stage('Check Status') {
     // Check whether we are prepared to proceed to a subsequent step
     // Assumes the presence of DBUSER and DBPASSWORD credentials configured in
     // the Jenkins master.
     node('build slave') {
       withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'redshift-creds',
usernameVariable: 'DBUSER', passwordVariable: 'DBPASSWORD']]) {
         if (isUnix()) {
            sh '''
            export PGPASSWORD=$DBPASSWORD
            psql --host  mazurcluster.cxco9mwgn8j6.us-west-1.redshift.amazonaws.com --port 5439 \
     --username ${DBUSER} -c 'select * from abac_file_list where file_name = ''file1.txt'';' dev
            '''
         } else {
            echo('sorry charlie.')
         }
       }
     }
   }
   stage('Build') {
      // Run the artifact script
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
