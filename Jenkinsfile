#!groovy

// For guidance on Jenkinsfile Pipeline syntax, use the Snippet Generator feature
// accessed via a Pipeline build job.  For instance, the Snippet Generator for
// a Pipeline job named 'SamplePipeline' is accessed via:
// http://<JENKINS_MASTER_HOSTNAME>/job/SamplePipeline/pipeline-syntax/
node {
  properties([
    // Parameters for specifying Database connection information.
    // NOTE: The actual username / password are managed via Jenkins master
    //       credentials, see:
    //       http://<JENKINS_MASTER_HOSTNAME>/credentials/ The credentials are
    //       referenced in the build step using the withCredentials directive.

    parameters([string(defaultValue: 'dev', description: 'Default database to connect to', name: 'DEFAULTDB'),
    string(defaultValue: 'mazurcluster.cxco9mwgn8j6.us-west-1.redshift.amazonaws.com', description: 'Endpoint hostname of the redshift cluster', name: 'DBHOST'),
    string(defaultValue: '5439', description: 'Port number of the redshift cluster', name: 'DBPORT')]),
    // Keeping our Jenkins master instance nice and tidy, cleaning up after ourselves
    buildDiscarder(logRotator(
      artifactDaysToKeepStr: '5',
      artifactNumToKeepStr: '10',
      daysToKeepStr: '7',
      numToKeepStr: '10')),
    // Keep builds single threaded (for now)
    disableConcurrentBuilds(),
    [$class: 'RebuildSettings', autoRebuild: true, rebuildDisabled: false],
    // Run the job on a schedule, within the first five minutes of every hour
    pipelineTriggers([[$class: "TimerTrigger", spec: "H(0-5) * * * *"]])
  ])
  // Get 'code', in this case from a GitHub repository
  stage('Preparation') {
    git 'https://github.com/danm-slalom/pipeline.git'
  }
  // Check whether we are prepared to proceed to a subsequent step
  // Assumes the presence of DBUSER and DBPASSWORD credentials configured in
  // the Jenkins master.
  stage('Check Status') {
    node('build slave') { // Targeting a specific slave instance, 'build slave',
                          // which has the required software (psql) installed.
                          // See http://<JENKINS_MASTER_HOSTNAME>/computer/
                          // for the full list of nodes
      withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'redshift-creds',
          usernameVariable: 'DBUSER', passwordVariable: 'DBPASSWORD']]) {
        if (isUnix()) {
          // Execute the bourne shell script, defined inline below
          returnVal = sh returnStatus: true, script: '''export PGPASSWORD=$DBPASSWORD
              export RESULT=`psql -A -t --host $DBHOST --port $DBPORT --username $DBUSER -c "select count(iscompleted) from abac_file_list where file_name = 'file1.txt' and iscompleted = 'Y';" $DEFAULTDB`
              echo "Rowcount result is: $RESULT"
              if [ $RESULT = "1" ]; then exit 0; else exit 99; fi
          '''
          if (returnVal != 0) {
            msg = "Data is not is the desired state (returnStatus: ${returnVal}).  Stopping the build..."
            echo msg
            currentBuild.result = 'UNSTABLE'
            // TODO: Figure out how to halt the pipeline build at this step
            //       without indicating 'FAILURE'.
            error(msg)
          }
        } else {
          // TODO: Supply the windows-specific implementation of the psql call
          //       above, if desired/necessary.
          echo('sorry charlie.')
        }
      }
    }
  }
  // Run the artifact script:
  stage('Build') {
    if (isUnix()) {
      sh "./copy_red_shift/copy_red_shift_run.sh"
    } else {
      bat(/copy_red_shift\copy_red_shift_run.bat/)
    }
  }
  // Run another arbitrary script:
  stage('Script') {
    if (isUnix()) {
      sh "./scripts/simple-echo.sh"
    }
  }
  // In a more traditional CI build we would parse the test results in this step
  stage('Results') {
    echo "If we had run tests, this is the the stage in which we would process them."
  }
  // In a more traditional CI build we would publish the newly built artifact
  // to a binary repository, i.e., artifactory
  stage('Publish') {
    echo "This is the stage at which we would ship off the freshly built artifact to the binary repository."
  }
  // In a more traditional CI build we would deploy the newly built artifact
  // to an environment where further testing (integration, perhaps manual) is
  // preformed
  stage('Deploy') {
    echo "This is the stage at which we would deploy the artifact to an integration environment."
  }
  stage('Signing Off') {
    echo "The build is done, go in ☮"
  }
}
