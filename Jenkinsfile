def onmyduffynode(script){
    ansiColor('xterm'){
        timestamps{
            sh 'ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -l root ${DUFFY_NODE}.ci.centos.org -t \"export REPO=${REPO}; export BRANCH=${BRANCH};\" "' + script + '"'
        }
    }
}

def synctoduffynode(source)
{
    sh 'scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -r ' + source + " " + "root@" + "${DUFFY_NODE}.ci.centos.org:~/"
}

node('userspace-containerization'){

    stage('Checkout'){
        checkout scm
    }

    stage('Build'){
        try{
            stage ("Allocate node"){
                env.CICO_API_KEY = readFile("${env.HOME}/duffy.key").trim()
                duffy_rtn=sh(
                            script: "cico --debug node get -a ${arch} -f value -c hostname -c comment",
                            returnStdout: true
                            ).trim().tokenize(' ')
                env.DUFFY_NODE=duffy_rtn[0]
                env.DUFFY_SSID=duffy_rtn[1]
            }

            stage ("setup"){
                onmyduffynode "yum -y install docker make"
                synctoduffynode "*" // copy all source files
            }

            stage("build test image"){
                onmyduffynode "make test-image"
            }

            stage("run test suite inside the container"){
                onmyduffynode "make test-in-container"
            }

        } catch (e) {
            currentBuild.result = "FAILURE"
            throw e
        } finally {
            stage("Cleanup"){
                sh 'cico node done ${env.SSID}'
            }
        }
    }
}
