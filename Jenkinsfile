node('java-agent') {
    stage ('Checkout') {
        checkout scm
    }

    stage ('Git mining') {
        discoverGitReferenceBuild()
        mineRepository()
        gitDiffStat()
    }

    stage ('Build, Test, and Static Analysis') {
        withMaven(mavenLocalRepo: '/var/data/m2repository', mavenOpts: '-Xmx768m -Xms512m') {
            sh 'mvn -V -e clean verify -Dmaven.test.failure.ignore -Dgpg.skip'
        }

        recordIssues tools: [java(), javaDoc()], aggregatingResults: 'true', id: 'java', name: 'Java'
        recordIssues tool: errorProne(), healthy: 1, unhealthy: 20

        junit testResults: '**/target/*-reports/TEST-*.xml'
        publishCoverage adapters: [jacocoAdapter('**/*/jacoco.xml')], sourceFileResolver: sourceFiles('STORE_ALL_BUILD')

        recordIssues tools: [checkStyle(pattern: 'target/checkstyle-result.xml'),
            spotBugs(pattern: 'target/spotbugsXml.xml'),
            pmdParser(pattern: 'target/pmd.xml'),
            cpd(pattern: 'target/cpd.xml'),
            taskScanner(highTags:'FIXME', normalTags:'TODO', includePattern: '**/*.java', excludePattern: 'target/**/*')],
            qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]]
    }

    stage ('Mutation Coverage') {
        withMaven(mavenLocalRepo: '/var/data/m2repository', mavenOpts: '-Xmx768m -Xms512m') {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        step([$class: 'PitPublisher', mutationStatsFile: 'target/pit-reports/**/mutations.xml'])
    }

    stage ('Collect Maven Warnings') {
        recordIssues tool: mavenConsole()
    }
}
