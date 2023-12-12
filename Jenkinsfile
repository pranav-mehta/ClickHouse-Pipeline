pipeline {
    agent { label 'ec2-fleet' }

    stages {
        stage('Setup parameters') {
            steps {
                script { 
                    properties([
                        parameters([
                            string(
                                defaultValue: '', 
                                name: 'GIT_URL', 
                                trim: true
                            ),
                            string(
                                defaultValue: '', 
                                name: 'GIT_BRANCH', 
                                trim: true
                            ),
                            string(
                                defaultValue: '', 
                                name: 'BUILD_VERSION', 
                                trim: true
                            ),
                            
                        ])
                    ])
                }
            }
        }
        stage('checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "${params.GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'SubmoduleOption',
                        disableSubmodules: false,
                        parentCredentials: false,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[url: "${params.GIT_URL}"]]])
            }
        }
        stage('clickhouse-build') {
            steps {
                sh '''output_dir="build_results"
                    ./docker/packager/packager --version $BUILD_VERSION --package-type=deb --output-dir "$output_dir"'''
               
                archiveArtifacts artifacts: 'build_results/clickhouse', followSymlinks: false, onlyIfSuccessful: true
            }
            
        }
        stage('clickhouse-image-build') {
            steps {
                sh '''cd build_results
                    python3 -m http.server &'''
                
                sh '''cd docker/server
                    docker build --network=host --build-arg="VERSION=$BUILD_VERSION" --build-arg="deb_location_url=http://localhost:8000" -t clickhouse/clickhouse-server:$BUILD_VERSION .
                    docker save --output clickhouse-server_$BUILD_VERSION.tar clickhouse/clickhouse-server:$BUILD_VERSION'''
                
                archiveArtifacts artifacts: 'docker/server/*.tar', followSymlinks: false, onlyIfSuccessful: true
            }
        }
    
    }
}
