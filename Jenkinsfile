pipeline {
	agent any

	stages {
		stage('Setup Environment') {
			steps {
				echo 'Downloading GraalVM...'
				sh '[ -d "$WORKSPACE/graalvm-25.2.4+7.1" ] || wget https://gds.oracle.com/download/graal/25i2/archive/graalvm-jdk-25i2-25.0.4_linux-x64_bin.tar.gz && tar -xf graalvm-jdk-25i2-25.0.4_linux-x64_bin.tar.gz'
				sh 'export GRAALVM_HOME=$WORKSPACE/graalvm-25.2.4+7.1'
			}
		}
		stage('Setup Code Environment') {
			when {
				changeset 'folia-server/**'
				changeset 'folia-api/**'
			}
			steps {
				echo 'Patching...'
				sh 'gradlew applyAllPatches'
			}
		}
		stage('Build Base') {
			steps {
				echo 'Building base layer...'
				sh 'gradlew nativeCompile --no-configuration-cache'
				archiveArtifacts artifacts: '${env.WORKSPACE}/folia-server/build/native/nativeCompile/*', fingerprint: true
			}
		}
		stage('Plugins') {
			when {
				expression {
					currentBuild.result == null || currentBuild.result == 'SUCCESS'
				}
			}
			steps {
				echo 'Building plugin image...'
				sh 'gradlew nativePluginCompile --no-configuration-cache'
				archiveArtifacts artifacts: '${env.WORKSPACE}/folia-server/build/native/nativePluginCompile/*', fingerprint: true
			}
		}
	}
}