// Script to trigger ibm initiator
// Global variables section
def nodeName = "centos-7"
def cephVersion
def rhcephVersion
def repository
def platforms

// Pipeline script entry point
node(nodeName) {

    timeout(unit: "MINUTES", time: 30) {
        stage('Preparing') {
            if (env.WORKSPACE) {
                sh script: "sudo rm -rf *"
            }
            checkout([
                $class: 'GitSCM',
                branches: [[ name: 'refs/remotes/origin/init_gvy' ]],
                doGenerateSubmoduleConfigurations: false,
                extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'recipe']],
                submoduleCfg: [],
                userRemoteConfigs: [[
                    url: 'https://github.com/tintumathew10/rhceph-qe-recipe.git'
                ]]
            ])
            println "workspace : " + env.WORKSPACE
        }
    }

    stage('Get Recipe') {
        def commitMsg = sh(
            script: "cd recipe; git show -s --format=%s",
            returnStdout: true
        ).trim()
        println "commit msg : ${commitMsg}"
        def commitFileName= commitMsg.split(' ').find({x -> x.contains("RHCEPH-")})
        println "file name : " + commitFileName
        def fileContent = readYaml file: "${env.WORKSPACE}/recipe/${commitFileName}"
        println "rhceph file content : " + fileContent
        cephVersion = fileContent["ceph-version"]
        rhcephVersion = fileContent["RHCephVersion"]
        repository = fileContent["repository"]
        platforms = fileContent['platforms']
        println "cephVersion : ${cephVersion}, rhcephVersion : ${rhcephVersion}, repository : ${repository}, platforms : ${platforms}"
    }

    stage('Create Repo') {
        def loginCmd = "ssh -l root ceph-qe-infra-01.syd.qe.rhceph.local "
        sh(script: "${loginCmd} 'rclone-cron'", returnStdout: true)
        def rhcephVer = rhcephVersion.substring(7,10)

        for (platform in platforms){
            println "platform : " + platform
            def m = sh(script: "${loginCmd} 'cd /data/site/${platform}/ceph-${rhcephVer}-${platform}; pwd; for x in \$(ls | cat | head -n -1) ; do echo \$x ; done'", returnStdout: true).trim()
            println "m : " + m
//             sh(script: "${loginCmd} 'if [ ! -d /data/site/${platform}1/ceph-${rhcephVer}-${platform} ]; then mkdir -p /data/site/${platform}1/ceph-${rhcephVer}-${platform}; fi'", returnStdout: true)
//             sh(script: "${loginCmd} 'cp -a /data/cos/${platform}/ceph-${rhcephVer}-${platform}/${cephVersion} /data/site/${platform}1/ceph-${rhcephVer}-${platform}/'", returnStdout: true)
//             sh(script: "${loginCmd} 'createrepo -v /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/Tools/ -g /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/Tools/comps.xml'", returnStdout: true)
//             sh(script: "${loginCmd} 'createrepo -v /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/OSD/ -g /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/OSD/comps.xml'", returnStdout: true)
//             sh(script: "${loginCmd} 'createrepo -v /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/MON/ -g /data/site/${platform}1/ceph-${rhcephVer}-${platform}/${cephVersion}/MON/comps.xml'", returnStdout: true)
//             sh(script: "${loginCmd} 'cd /data/site; chown -R apache:apache .'", returnStdout: true)
        }

//         if (repository) {
//
//             sh(script: "cd /home/cephuser; podman login brew.registry.redhat.io -u \$(jq -r .credentials.username .brew.json) -p \$(jq -r .credentials.password .brew.json)", returnStdout: true)
//             sh(script: "cd /home/cephuser; skopeo copy docker://brew.registry.redhat.io/rh-osbs/rhceph:${repository} docker://ceph-qe-registry.syd.qe.rhceph.local/rh-osbs/rhceph:${repository}", returnStdout: true)
//         }


    }
}
