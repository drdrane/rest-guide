node("maven") {

  stage ("\u2460 Checkout SCM") {
    checkout([$class: 'GitSCM',
              branches: [[name: '*/master']],
              doGenerateSubmoduleConfigurations: false,
              extensions: [],
              submoduleCfg: [],
              userRemoteConfigs: [[
                url: 'git@git-cicd.gcloud.belgium.be:rest/styleguide.git',
                credentialsId: 'git_technical_user'
              ]]
    ]);
  }

  stage("\u2461 Generate Website") {
    sh "mvn site"
    stash name:"site", includes:"target/site/doc/**"
    sh '''
      APPLICATION_VERSION=$(mvn -Dexec.executable='echo' -Dexec.args='${project.version}' --non-recursive exec:exec -q)
      echo ${APPLICATION_VERSION} > APPLICATION_VERSION
    '''
    env.APPLICATION_VERSION = readFile('APPLICATION_VERSION').trim()
    echo "${APPLICATION_VERSION}"
  }

}

node () {

  stage("\u2462 Build Website Docker Image") {
    checkout([$class: 'GitSCM',
              branches: [[name: '*/build']],
              doGenerateSubmoduleConfigurations: false,
              extensions: [],
              submoduleCfg: [],
              userRemoteConfigs: [[
                url: 'git@git-cicd.gcloud.belgium.be:rest/styleguide.git',
                credentialsId: 'git_technical_user'
              ]]
    ]);
    unstash name:"site"
    sh "cp -rf target/site/doc/* docker/contrib/src"
    sh "oc start-build gcloud-rest-styleguide-website --follow=true --build-loglevel=9 --from-dir='./docker'"
    openshiftVerifyBuild(bldCfg: "gcloud-rest-styleguide-website", waitTime: '240', waitUnit: 'sec', verbose: 'false')
  }

  stage('\u2463 Deploy Website') {
    openshiftDeploy(depCfg: "gcloud-rest-styleguide-website", verbose: 'false', waitTime: '240', waitUnit: 'sec')
    openshiftScale(depCfg: "gcloud-rest-styleguide-website", replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: '240', waitUnit: 'sec')
    openshiftVerifyDeployment(depCfg: "gcloud-rest-styleguide-website", replicaCount: '1', verbose: 'false', verifyReplicaCount: 'true', waitTime: '240', waitUnit: 'sec')
    openshiftVerifyService(svcName: "gcloud-rest-styleguide-website", verbose: 'false')
  }

}

node ('docker') {

  stage('\u2464 Crawling') {
    echo "Running crawler here ..."
    echo "Do some unit test on deployed website here ..."
    sh '''
      set +e
      RETURN_CRAWLER=$(docker run -i -e TARGET=http://gcloud-rest-styleguide-website.test.paas.services.gcloud.belgium.be/rest/index.html container-release.gcloud.belgium.be/crawler:latest)
      echo "--"
      echo "${RETURN_CRAWLER}"
      echo "--"
      echo "OK!"
    '''
  }

  stage('\u2465 Undeploy Website') {
    openshiftScale(depCfg: "gcloud-rest-styleguide-website", replicaCount: '0', verbose: 'false', verifyReplicaCount: 'false', waitTime: '240', waitUnit: 'sec')
  }

  stage ('\u2466 Auto Release Tag.') {
    echo "We use an Imagestream for Rest Styleguide deployment in Test environment (Openshift Project : ssb-test-community-tools)"

    sh '''
      curl -sSfLo ${WORKSPACE}/getNextDockerGCloudReleaseTag http://git-cicd.gcloud.belgium.be/openshift/scripts/raw/master/artifactory-helper/getNextDockerGCloudReleaseTag.sh
      curl -sSfLo ${WORKSPACE}/promoteDockerImage http://git-cicd.gcloud.belgium.be/openshift/scripts/raw/master/artifactory-helper/promoteDockerImage.sh
      chmod +x ${WORKSPACE}/promoteDockerImage            \
               ${WORKSPACE}/getNextDockerGCloudReleaseTag

      export ARTIFACTORY_USERNAME="apidockerpromoting"
      export ARTIFACTORY_TOKEN="eyJ2ZXIiOiIyIiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYiLCJraWQiOiJ4Z0pPNEVDLWhFZGdMVjdTVFAxcE9ENzc4bnRlVzJlVFpIRVFfb3JwOUZBIn0.eyJzdWIiOiJqZnJ0QDAxYnZoanRtbjdmNmZ5MGNhemp6OXMwYnQyXC91c2Vyc1wvYXBpZG9ja2VycHJvbW90aW5nIiwic2NwIjoibWVtYmVyLW9mLWdyb3VwczpEb2NrZXJQcm9tb3RpbmcgYXBpOioiLCJhdWQiOiJqZnJ0QDAxYnZoanRtbjdmNmZ5MGNhemp6OXMwYnQyIiwiaXNzIjoiamZydEAwMWJ2aGp0bW43ZjZmeTBjYXpqejlzMGJ0MiIsImlhdCI6MTUwODkyMTIyMSwianRpIjoiMTg5NjE0NzctYTE1MC00YzZlLWI0YmQtMDgyMWVhMDM0YzNkIn0.J_bIptVx8snAeZDATqyffOH6NxbnJabRqa4M4lyawVYvVPjGq2akGOThRap6TMUDluSydARNKFDZIjwS2L4NiSI2oVAYpsdzO9BfCX7OqBGG1H-V5HhyhpUXafryUU9E3IgnSVUGLwKLor5g21xaJ3JWJM2qIXcgpqjeJbZ2kG6EbkFL2eppu_tgDQl9yI8VbGviaWD5vMs8UvwFzmP-Cdp9jwHrzHTfzkXqjuPaDDN4-nDP6rsuQGl9pmG08R3UpZH50VBGUs6OCoz5uByLC8SWpSvQW8CLS8peelSstVa6cHAYXBaUeGWw7fqZahMTJjm4UMqQfRRJSfIK0fMWqw"

      GCLOUD_DOCKER_TAG=$(${WORKSPACE}/getNextDockerGCloudReleaseTag --artifactory_url="https://repo.gcloud.belgium.be/artifactory" \
                                      --artifactory_username=${ARTIFACTORY_USERNAME} \
                                      --artifactory_token=${ARTIFACTORY_TOKEN} \
                                      --docker_registry="docker.release" \
                                      --docker_image="gcloud-rest-styleguide-website" \
                                      --docker_version_in_tag="${APPLICATION_VERSION}" )

      echo ${GCLOUD_DOCKER_TAG} > GCLOUD_DOCKER_TAG

      ${WORKSPACE}/promoteDockerImage --artifactory_url="https://repo.gcloud.belgium.be/artifactory" \
                                      --artifactory_username=${ARTIFACTORY_USERNAME} \
                                      --artifactory_token=${ARTIFACTORY_TOKEN} \
                                      --repoKey="docker.snapshot" \
                                      --targetRepo="docker.release" \
                                      --dockerRepository="gcloud-rest-styleguide-website" \
                                      --tag="latest" \
                                      --targetTag="${GCLOUD_DOCKER_TAG}" \
                                      --copy="true"

      oc tag --scheduled=true --alias=false container-release.gcloud.belgium.be/gcloud-rest-styleguide-website:${GCLOUD_DOCKER_TAG} gcloud-rest-styleguide-release:${GCLOUD_DOCKER_TAG}

    '''

    GCLOUD_DOCKER_TAG = readFile('GCLOUD_DOCKER_TAG').trim()

  }

  stage ('\u2467 Deployment in Test Env.') {
    echo "Refresh [tst] ImagestreamTag."
    echo "Sleep while Openshift masters refresh all imagestreams ..."
    sh 'sleep 120'
    openshiftTag(srcStream: "gcloud-rest-styleguide-release", srcTag: "${GCLOUD_DOCKER_TAG}",
                 destStream: "gcloud-rest-styleguide-release", destTag: "tst",
                 alias: 'true', verbose: 'false')

  }

  stage ('\u2468 Deployment in Production Env.') {
    def userInput = input (id: 'INPUT_APPROVE_ID',
                           message: 'G-Cloud Rest Styleguide Website Pipeline Promotion parameters : ',
                           ok: 'OK',
                           parameters: [booleanParam(name: 'INPUT_APPROVE',
                                                     defaultValue: false,
                                                     description: 'Check this box to promote this image for deployment in Production environment.')],
                           submitter: 'smals-wisa-*,smals-sem-*', submitterParameter: 'INPUT_APPROVE_SUBMITTER')

    if ( userInput.INPUT_APPROVE ) {
      echo "The deployment of this image is approved in test.  Refresh [prd] ImagestreamTag."

      openshiftTag(srcStream: "gcloud-rest-styleguide-release", srcTag: "${GCLOUD_DOCKER_TAG}",
                   destStream: "gcloud-rest-styleguide-release", destTag: "prd",
                   alias: 'true', verbose: 'false')

    } else {
      echo "The deployment of this image is not approved in test.  Do not tag this image for Production deployment."
    }
  }

}
