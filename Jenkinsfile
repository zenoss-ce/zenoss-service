#!/usr/bin/env groovy

node {

  stage('Checkout') {
    checkout scm
  }
  def pkg_ver = '5.3.3'
  def pkg_iter = '1'

  stage('Build') {
    docker.image('zenossce/service-build').inside() { 
      withEnv(["PKG_VERSION=${pkg_ver}","ITERATION=${pkg_iter}"]) {
        sh '''
          FULL_NAME=zenoss-core-service
          EDITION=ce-5.3_0
          PKGROOT=$FULL_NAME
          MAINTAINER=serge@ocslab.com
          DESCRIPTION='Zenoss services definition'

          cd services
          rm -rf $PKGROOT
          mkdir -p $PKGROOT/opt/serviced/templates

          serviced template compile \
            -map zenoss/zenoss5x,zenossce/core_5.3:${PKG_VERSION}_$EDITION \
            -map zenoss/hbase:xx,zenoss/hbase:24.0.8 \
            -map zenoss/opentsdb:xx,zenoss/opentsdb:24.0.8 \
            Zenoss.core > $PKGROOT/opt/serviced/templates/zenoss-core-${PKG_VERSION}_$ITERATION.json

          for p in deb rpm
          do
            fpm \
              -n ${FULL_NAME} \
              -v ${PKG_VERSION} \
              --iteration ${ITERATION} \
              -s dir \
              -d serviced \
              -t $p \
              -a noarch \
              -C ${PKGROOT} \
              -m ${MAINTAINER} \
              --description "${DESCRIPTION}" \
              --deb-user root \
              --deb-group root 
          done

        '''
        withCredentials( [file(credentialsId: 'sign.gpg', variable: 'gpgkey')] ) {
          sh """
            gpg --import ${gpgkey} ; \
            debsigs --sign=origin -k 24AD250F4836F7F1 services/zenoss-core-service_${pkg_ver}-${pkg_iter}_all.deb ; \
            rpmsign --resign -D '_gpg_name serge@ocslab.com' services/zenoss-core-service-${pkg_ver}-${pkg_iter}.noarch.rpm
          """
        }
      }
    }
  }

  stage('Publish') {
    withCredentials( [usernamePassword(credentialsId: 'Bintray', usernameVariable: 'username', passwordVariable: 'password')] ) {
      sh """
          curl -s -T services/zenoss-core-service_${pkg_ver}-${pkg_iter}_all.deb -u${username}:${password} 'https://api.bintray.com/content/${username}/zenoss-ce-deb/zenoss-core-service/${pkg_ver}-${pkg_iter}_all/zenoss-core-service_${pkg_ver}-${pkg_iter}_all.deb;deb_distribution=stretch,xenial,bionic;deb_component=main;deb_architecture=all' ; \
          curl -s -X POST -u${username}:${password} 'https://api.bintray.com/content/${username}/zenoss-ce-deb/zenoss-core-service/${pkg_ver}-${pkg_iter}_all/publish' ; \

          curl -s -T services/zenoss-core-service-${pkg_ver}-${pkg_iter}.noarch.rpm -u${username}:${password} 'https://api.bintray.com/content/${username}/zenoss-ce-rpm/zenoss-core-service/${pkg_ver}-${pkg_iter}/7/x86_64/zenoss-core-service-${pkg_ver}-${pkg_iter}.noarch.rpm' ; \
          curl -s -X POST -u${username}:${password} 'https://api.bintray.com/content/${username}/zenoss-ce-rpm/zenoss-core-service/${pkg_ver}-${pkg_iter}/publish' 
         """
    }
  }
}
