@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def dummy
deployRemoteClusterNode(configSecretName: 'tiger-config'){
    properties(
        [
            pipelineTriggers([cron('0 */3 * * *')]),
        ]
    )

    def repoNames = 'fabric8io/openshift-jenkins-s2i-config'

    def utils = new io.fabric8.Utils()

    // check the repos for open PRs
    def ghProjects = splitRepoNames(repoNames)

    // get a list of openshft project names that are pull requests
    def namespaces
    container('clients'){
        def namespacelist = sh(script: 'kubectl get namespaces | grep Active | grep jenkins-pr-', returnStdout: true).toString().trim()
        namespaces = splitNamespaceNames(namespacelist)
    }

    // remove any names that still have open PRs in github
    for (ghProject in ghProjects) {
        ghProject = ghProject.toString().trim()

        def openPRs = utils.getOpenPRs(ghProject)
        def items = ghProject.split('/')
        openPRs = covertToProjectNames(openPRs, items[1])

        if (openPRs){
            namespaces.removeAll(openPRs as Object[])
        }

        openPRs = null
    }
    if (namespaces){
        def command = 'kubectl delete namespace'
        for (n in namespaces) {
            command = command + ' ' + n

        }
        echo "running: ${command}"
        container('clients'){
            echo "command:$command"
            //sh command
        }
    } else {
        echo 'no namespace to delete'
    }
}

@NonCPS
def splitRepoNames(repoNames) {
    def repos = repoNames.split(',')
    def list = []
    for (name in repos) {
        echo "project to process ${name}"
        list << name
    }
    repos = null
    return list
}

@NonCPS
def splitNamespaceNames(namespaceNames) {
    def namespaces = namespaceNames.split('\\n')
    def list = []
    for (ns in namespaces) {
        list << ns.replace("Active", "").trim()
    }
    namespaces = null
    return list
}

@NonCPS
def covertToProjectNames(openPRs, repo) {
    def list = []
    for (pr in openPRs) {
        def name = 'jenkins-pr-' + pr + '-' + repo
        echo "look for openshift project ${name}"
        list << name
    }
    return list
}
