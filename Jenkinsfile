pipeline {
    agent any

    stages{
        stage('Star Build'){
            steps{
                script{
                    //def isDeploy="${params.isDeploy}"
                    //def pkgTag="${params.pkgTag}"
                    def now = new Date()
                    def pack_version = now.format("yyyyMMddHHmm")
                    def smd="${params.ModList}"
                    def smod_list=smd.split(',')
                    def docker_build=[:]
                    def account_web_build=[:]
                    def helm_build=[:]

                    //sh "sh /data/pack_branches.sh ${pkgTag}"

                    def version_tag
                    def remote = [:]
                    remote.name = '10.10.0.56'
                    remote.host = '10.10.0.56'
                    remote.port = 22
                    remote.allowAnyHosts = true
                    //通过withCredentials调用Jenkins凭据中已保存的凭据，credentialsId需要填写，其他保持默认即可
                    withCredentials([sshUserPrivateKey(credentialsId: 'star-jenkins', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = "${userName}"
                        remote.identityFile = identity
                        sshCommand remote: remote, command: "cd /data && sh pack_branches0.2.7.0_jenkins.sh ${pack_version} ${smd}"
                        version_tag = sshCommand remote: remote, command: "cat /tmp/starold-version.tag"
                        if(smod_list.contains("gateway") || smod_list.contains("game")){
                            sshGet remote: remote, from: "/data/package/php.star.thedream.cc_${version_tag}.tar.gz", into: "${WORKSPACE}/build/dockerfiles/src/php.dragonica.thedream.cc_${version_tag}.tar.gz", override: true
                        }
                        if(smod_list.contains("arena")){
                            sshGet remote: remote, from: "/data/package/arena.star.thedream.cc_${version_tag}.tar.gz", into: "${WORKSPACE}/build/dockerfiles/src/arena.dragonica.thedream.cc_${version_tag}.tar.gz", override: true
                        }
                    }

                    for(int i=1;i<=smod_list.size();i++){
                        def index=i-1
                        def dkfile=''

                        pkgdir="/data/package"
                        //sh "cp -v ${pkgdir}/*_${pkgTag}.tar.gz ${WORKSPACE}/build/dockerfiles/src/"


                        //指定相应的Dockerfile
                        if("${smod_list[index]}" != "account-web"){
                            dkfile="Dockerfile-${smod_list[index]}"
                            // docker_build["Build ${smod_list[index]}"]={
                            //     stage("BUILD ${smod_list[index]}"){
                            //         sh "echo ========= docker ${smod_list[index]} =========="
                            //         slackSend channel: "#devops",color:"good",message:"Build start ${smod_list[index]} - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                            //     }
                            // }
                            
                            docker_build["Build ${smod_list[index]}"]={
                                stage("BUILD ${smod_list[index]}"){
                                    slackSend channel: "#devops",color:"good",message:"Build start ${smod_list[index]} - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                                    //开始docker build及push debug:'afd322fb-3814-4060-b199-0bb42c9aff3f'
                                        // docker.withRegistry('https://dream-platform-registry.cn-shanghai.cr.aliyuncs.com', 'startw-aliyun-registry') {
                                        //     def dragonImage = docker.build("starchs/${smod_list[index].toLowerCase()}:${version_tag}", "-f ${WORKSPACE}/build/dockerfiles/${dkfile} ${WORKSPACE}/build/dockerfiles --build-arg version_num=${version_tag}")
                                        //     dragonImage.push()
                                        // }
                                        docker.withRegistry('https://swr.cn-east-3.myhuaweicloud.com', 'star-gank-huawei') {
                                            def starImage = docker.build("gankgames/dragon-huawei-gank/${smod_list[index].toLowerCase()}:${version_tag}", "-f ${WORKSPACE}/build/dockerfiles/${dkfile} ${WORKSPACE}/build/dockerfiles --build-arg version_num=${version_tag}")
                                            starImage.push()
                                        }
                                }
                            }
                        }else{
                            /*
                            account_web_build["Build ${smod_list[index]}"]={
                                stage("Build ${smod_list[index]}"){
                                    sh "echo =========== rsync ${smod_list[index]} ============"
                                }
                            }
                            */
                            /*
                            account_web_build["Build ${smod_list[index]}"]={
                                stage("Build ${smod_list[index]}"){
                                    sh "sh ${WORKSPACE}/build/account-web.sh ${pkgTag} ${isDeploy} ${pkgdir}"
                                }
                            }*/
                        }
                    }

                    // if(smod_list.contains("gateway") || smod_list.contains("game") || smod_list.contains("arena")){
                    //     sml=smod_list.join(' ')
                    //     helm_build["Build helm pack"]={
                    //         stage("Build ${sml} helm pack"){
                    //             //sh "sh ${WORKSPACE}/build/pack-helm.sh ${pkgTag} ${isDeploy} '${sml}'"
                    //             //sh "cd ${WORKSPACE}/build/dockerfiles && sh tmp-build.sh ${pkgTag} '${sml}' ${isDeploy}"
                    //             sh "echo tmp-build.sh '${sml}'"
                    //             sh "docker login -u cn-east-3@V3JNTQAPJVCMIZ2VFAIY -p a8c93188649259d5ef99459eb93654a6587acbddda842c2d1edb388153ea6c04 swr.cn-east-3.myhuaweicloud.com"
                    //             sh "docker push swr.cn-east-3.myhuaweicloud.com/gankgames/dragon-huawei-gank/nginx:1.23.0-alpine"
                    //             slackSend channel: "#devops",color:"good",message:"Build start ${sml} - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                    //         }
                    //     }
                    // }

                    parallel(docker_build)
                    wrap([$class: 'BuildUser']) {
                        BUILD_USER="${BUILD_USER}"
                    }
                    sh """\
                    curl 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=5465c076-514f-445c-9852-0bea4e89a327' \
                    -H 'Content-Type: application/json' \
                    -d '
                    {
                            "msgtype": "markdown",
                            "markdown": {
                                "content": "'"星觉b0.2.7.0版-${smod_list}镜像已制作 \n>版本tag：${version_tag} \n>发布者：${BUILD_USER}"'"
                            }
                    }'
                    """

                    env.PKG_TAG = version_tag
                }
            }
        }
    }

    post{
        failure {
            slackSend failOnError:true,channel: "#devops",color:"danger",message:"Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        success{
            slackSend channel: "#devops",color: "good",message:"Build successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            script{
                //清理
                sh "rm -vf ${WORKSPACE}/build/dockerfiles/src/*.tar.gz"
                wrap([$class: 'BuildUser']) {
                    BUILD_USER="${BUILD_USER}"
                }
                addShortText background: "yellow",borderColor: "grey",border: 1,text: "${params.ModList} ${env.PKG_TAG} ${BUILD_USER}"
                //manager.addShortText(manager.getEnvVariable("BUILD_USER"))
            }
        }
    }


}