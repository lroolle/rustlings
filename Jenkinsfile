import groovy.json.JsonSlurperClassic

parameters {
    string(name: 'BRANCH_NAME')
    string(name: 'CHANGE_BRANCH')
    string(name: 'CHANGE_TARGET')
}

def BRANCH_NAME = params.BRANCH_NAME ? params.BRANCH_NAME : env.BRANCH_NAME
def CHANGE_BRANCH = params.CHANGE_BRANCH ? params.CHANGE_BRANCH : env.CHANGE_BRANCH
def CHANGE_TARGET = params.CHANGE_TARGET ? params.CHANGE_TARGET : env.CHANGE_TARGET
def settings = [
    label: "jn-rustlings-${UUID.randomUUID().toString()}",
    git: [
        credentialsId: 'test',
        branch: CHANGE_BRANCH ? CHANGE_BRANCH : BRANCH_NAME,
        repo: 'https://git.netisdev.com/scm/~eric.wang/rustlings.git',
    ],
    docker: [
        credentialsId: 'jn.robo',
        secrets: ['jn.robo'],
        registry: 'https://k8stest-harbor.netisdev.com',
        prefix: 'k8s-harbor.netisdev.com/',
        unittest: [
            image: 'jn/rust-base',
            tag: 'latest'
        ]
    ]
]

@NonCPS
def get_version(text) {
    def matcher = text =~ /(\d+)(\.\d+)(\.\d+)(-?\w+)?(\.\d+)?/
    matcher ? matcher[0].tail().findAll({it->it}) : null
}

// properties([
//     [$class: 'HudsonNotificationProperty',
//         endpoints: [
//             [
//                 event: "all",
//                 urlInfo: [urlType: "PUBLIC", urlOrId: 'https://ci-mon.netisdev.com/projects/37db7876-9328-4863-93aa-1b8cd4a795d4/status']
//             ]
//         ]
//     ]
// ])

podTemplate(
    label: settings.label,
    cloud: 'kubernetes',
    containers: [
        containerTemplate(
            name: 'rust-base',
            image: "${settings.docker.prefix}${settings.docker.unittest.image}:${settings.docker.unittest.tag}",
            ttyEnabled: true,
            command: 'bash',
            alwaysPullImage: true),
        containerTemplate(
            name: 'jnlp',
            image: "jenkins/inbound-agent:4.9-1",
            ttyEnabled: true,
            alwaysPullImage: true)
    ],
    imagePullSecrets: settings.docker.secrets,
    volumes: [
        persistentVolumeClaim(
            mountPath: "/var/jenkins/",
            claimName: "jenkins-nfs-pvc",
        )
    ],
) {
    node(settings.label) {
        try {
            stage('SCM Checkout') {
                checkout scm: [
                    $class: 'GitSCM', userRemoteConfigs: [
                        [url: settings.git.repo, credentialsId: settings.git.credentialsId]
                    ], branches: [
                        [name: settings.git.branch]
                    ]
                ], poll: false
            }
            container('rust-base') {
                stage('Cargo test') {
                    sh "cargo test"
                }
            }
        } catch(err) {
            throw err
        } finally {
            stage('Report Result') {
                // junit testResults: 'junit.xml', allowEmptyResults: true
                // cobertura coberturaReportFile: 'coverage.xml'
                // publishHTML([
                //     allowMissing: true,
                //     reportFiles: 'coverage.html',
                //     reportDir: 'coverage',
                //     keepAll: true,
                //     reportName: 'Code Coverage'
                // ])
                echo "Finished..."
            }
        }
    }
}
