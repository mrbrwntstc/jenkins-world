Jenkins Pipeline - Intermediate
-----------------------------

# Software at the speed of ideas

### Focus
The focus of this class is Declarative Pipelines with shared libraries

-- -- -- -- -- -- -- -- -- --

### Don't forget
Jenkins is an orchestrator. It's not meant to build code.

-- -- -- -- -- -- -- -- -- --

### Jenkins Pipeline - Beginner
Main focus: Blue Ocean

Blue Ocean allows you to write pipelines without writing any Jenkinsfiles

-- -- -- -- -- -- -- -- -- --

### New Declarative features
 - equals
 ```
 	stage('Test') {
 		when { equals expected: 2, actual: currentBuild.number }
 	}
 ```
 - changerequest: check and see if the pipeline is building a change request, e.g. pull request
 ``` when { changeRequest() } ```
 - buildingTag: check if running a tag rather than a branch or a specific commit
 ``` when { buildingTag() } ```
 - tag: check tag name itself
 ``` when { tag "release-*" } ```
 - beforeAgent: you will not have access to the agent’s workspace, but you can avoid unnecessary SCM checkouts and waiting for a valid agent to be available
 ``` stage('Test') {
	 	when {
	 		beforeAgent true
	 		branch 'production'
	 	}
	 }
 ```

-- -- -- -- -- -- -- -- -- --

### New post commands
- fixed: Checks to see if the current run is successful and if the previous run was either failed or unstable
- regression:
    Checks to see if the current run’s status is worse than the previous run’s status

    If the previous run was successful and the current run is unstable, this fires and its block of steps executes

    It also runs if the previous run was unstable and the current run is a failure, etc

-- -- -- -- -- -- -- -- -- --

### New options
- checkoutToSubdirectory: 
	override location that automatic SCM checkout uses

	checkoutToSubdirectory("foo") checks out to $WORKSPACE/foo

- newContainerPerStage
	ensure every stage has a fresh container used if using a top level docker image or Dockerfile

-- -- -- -- -- -- -- -- -- --

### Building with Docker
```
pipeline{
	agent{
		docker { image 'node:7-alpine' }
	}
	stage('build') {}
}
```

-- -- -- -- -- -- -- -- -- --

### Caching data for containers
```
pipeline{
	agent{
		docker {
			image 'maven:3-alpine'
			args '-v $HOME/.m2:/root/.m2'
		}
	}

	stages{
		stage('build'){
			steps{
				sh 'mvn -B'
			}
		}
	}
}
```

-- -- -- -- -- -- -- -- -- --

### Using multiple containers
```
pipeline{
	agent none
	stages{
		stage('java back-end'){
			agent{
				docker { image 'maven:3-alpine' }
			}
			steps{
				sh 'mvn --version'
			} // end steps
		} // end stage

		stage('javascript front-end'){
			agent{
				docker{ image 'node:7-alpine' }
			} // end agent
			steps{
				sh 'node --version'
			} // end steps
		} // end stage('javascript front-end')
	} // end stages
} // end pipeline
```

-- -- -- -- -- -- -- -- -- --

### Using a Dockerfile
- dockerfile
	```
		FROM node:7-alpine
		RUN apk add -U subversion
	```
- jenkinsfile
	```
	pipeline{
		agent { dockerfile true }
		stages{
			stage('test'){
				steps{
					sh 'node --version'
					sh 'svn --version'
				}
			}
		}
	}
	```

-- -- -- -- -- -- -- -- -- --

### Specifying a Docker label
By default, Pipeline assumes any configured agent is capable of running Docker-based pipelines, which is problematic if not every agent is a Docker images

Labels can be configured globally under "Manage Jenkins"

-- -- -- -- -- -- -- -- -- --

### Scripted pipeline
Declarative vs Sctipted
- both Jenkins DSL
- durable implementations as "pipeline as code"
- both able to use steps built into Pipeline or associated plugins
- both can utilize shared libraries

Limitations of Declarative
- more strict and pre-defined structure

Limitations of Scripted
- fewer safeguards against errors that Declarative would normally catch

_**always start with Declarative Pipeline syntax**_

[More stuff about scripted pipelines](https://jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline)

-- -- -- -- -- -- -- -- -- --

# If a command doesn't have sh '', it's running on master. All Groovy code, by default, gets pushed up to master

-- -- -- -- -- -- -- -- -- --

### Using a Jenkinsfile
- String interpolation
	```
	def username='Jenkins'
	echo 'hello Mr. ${username}' // hello Mr. ${username}
	echo "hello Mr. ${username}" // hello Mr. Jenkins
	```
- environment variables
	```
	pipeline{
		agent any
		environment{
			CC = 'clang'
		}
		stages{
			stage('example'){
				env{
					DEBUG_FLAGS='g'
				}
				steps{
					sh "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
					sh "printenv"
				}
			}
		}
	}
	```
- credentials
	* secret text
	```
	pipeline{
		agent {}
		environment{
			AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
      AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
		}
	}
	```
	* username/password
	* secret files
- handling parameters
	```
	pipeline{
		agent any
		parameters{
			string(name: 'Greeting', defaultValue= 'Hello', description: 'how should i greet the world?')
		}
		stages{
			stage('example'){
				steps{
					echo "${params.Greeting} World!"
				}
			}
		}
	}
	```
-handling failure
	```
	pipeline{
		agent any
		stages{
			stage('test'){
				echo "running tests"
			}
		}
		post{
			always{
				echo "we did it!"
			}
			failure{
				mail to: team@example, subject: "The pipeline failed :("
			}
		}
	}
	```
- optional steps
	``` git url: 'git://example.com/amazing-project.git', branch: 'master'
			git (url: 'git://example.com/amazing-project.git', branch: 'master')
	 ```

-- -- -- -- -- -- -- -- -- --

### Multibranch pipelines
All new pipelines should be created as multibranch pipelines

Marker files
	A marker file is a file that Jenkins will search for that will be the reason to create a new jenkins job. For example, for the Martin Framework, Jenkins would specifically look for "pom.xml" files

[Getting started with Blue Ocean multibranch](https://jenkins.io/doc/book/blueocean/getting-started)

[Multibranch pipelines](https://jenkins.io/doc/book/pipeline/multibranch)

[Some more examples](https://jenkins.io/blog/2015/12/03/pipeline-as-code-with-multibranch-workflows-in-jenkins)

-- -- -- -- -- -- -- -- -- --

### Pipelines without Blue Ocean
Multibranch Pipeline jobs

-- -- -- -- -- -- -- -- -- --

### Shared Libraries
- Share and reuse Pipeline code
- Scale Jenkins Pipeline usage
- Help admins manage code sprawls
- Use tooling to avoid silos (breaking silos)

What is it?
- separate SCM repo containing reusable custom steps that can be called from Pipelines 
- Configured once per Jenkins instance
- Cloned at build time
- Loaded and used as code libraries for Jenkins pipelines
- Modifications made to shared library custom steps are applied to all Pipelines that call that custom step

Notes
- powerful
- slight learning curve
	* requires deeper understanding of Pipelines
- adds some overhead
	* testing
	* maintenance
- many uses

[More info about shared libraries](https://jenkins.io/doc/book/pipeline/shared-libraries)

-- -- -- -- -- -- -- -- -- --

### Implement shared library
- create new SCM
	```
	+- src                     # Groovy source files
	|  +- org
	|     +- foo
	|        +- Bar.groovy  # for org.foo.Bar class
	+- vars
	|  +- foo.groovy          # for global 'foo' variable
	|  +- foo.txt             # help for 'foo' variable
	+- resources               # resource files (external libraries only)
	|  +- org
	|     +- foo
	|        +- bar.json    # static helper data for org.foo.Bar
	```
	a. *src* dir should look like standard Java source directory
	b. *vars* dir contains scripts that define custom steps accessible from a Pipeline. Each file should define one step and its name should be the name of the step, *camelCased*, with the .groovy suffix
	c. *resources* directory allows libraryResources step to be sued form an external library to load associated non-Groovy files
- **restrict access to SCM**
- configure Global Pipeline Library in Jenkins
- code custom step and check it into the shared libarary SCM repository
- call custom step from Pipeline

-- -- -- -- -- -- -- -- -- --

### Configure shared library
- all global libraries configured by a Jenkins admin are considered trusted
- libraries configured at a branch/folder level are _**not**_ considered trusted (runs in groovy sandbox)
- set "Default Version" to master to have Pipelines call custom steps from the master branch

-- -- -- -- -- -- -- -- -- --

### Customization of shared libraries
- Parameters can be passed using a Map
```
def call(String name, String dayOfWeek) {
	sh "echo hello ${name}. It is ${dayOfWeek}"
}
```
```
def call(Map config) {
	sh "hello ${config.name}. It is ${config.dayOfWeek}"
}
```
-- -- -- -- -- -- -- -- -- --

### Custom load files
For files that make Jenkinsfiles overly complex and unwiedly, you can put those in a file and store in the *resources* directory in the shared library

This makes the Jenkinsfile more concise and easier to read for everybody

libraryResource goes directly to the 'resources' directory in the shared library project

vars/sendNotifications.groovy:
```
def renderTemplate(input, binding) {
  def engine = new groovy.text.GStringTemplateEngine()
  def template = engine.createTemplate(input).make(binding)
  return template.toString()
}

def call(Map config) {
  <... removed Slack ...>
  def rawBody = libraryResource 'emailtemplates/build-results.html'
  def binding = [
    applicationName: env.JOB_NAME,
    buildNumber    : env.BUILD_NUMBER,
    buildUrl       : env.BUILD_URL,
    message        : config.message
  ]
  def emailBody = renderTemplate(rawBody,binding)

  // send to email
  emailext (
    subject: "${config.message}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
    body: emailBody,
    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
  )
}
```

resources/emailtemplates/build-results.html
```
<p>$message: Job '$applicationName [$buildNumber]':</p>
<p>Check console output at <a href="$buildUrl">$applicationName [$buildNumber]</a></p>
```

-- -- -- -- -- -- -- -- -- --

### Abstracting away the pipeline
Set parameters and take away the actual pipeline backend away from the developers

Jenkinsfile
```
@Library('shared-library') _
corporatePipeline {
    deployToDefault = "dev"
}
```

shared-library/resources/corporatePipeline.groovy
```
    def call(body) {
        def pipelineParams= [:]
        body.resolveStrategy = Closure.DELEGATE_FIRST
        body.delegate = pipelineParams
        body()

      	pipeline {
          agent none  
          stages{
            stage('Fluffy Build') {
              parallel {
                stage('Build Java 8') {
                  agent {
                    node {
                      label 'java8'
                    }
                  }
                  steps {
                    sh "./jenkins/build.sh"
                  }
                  post {
                    success {
                      stash(name: 'Java 8', includes: 'target/**')
                    }
                  }
                }
                stage('Build Java 7') {
                  agent {
                    node {
                      label 'java7'
                    } // end node 
                  } // end agent
                  steps {
                    sh './jenkins/build.sh'
                  } // end steps
                  post {
                    success {
                      postBuildSuccess(stashName: "Java 7")
                    } // end success
                  } // end post
                } // end stage
              } // end parallel
            } // end stage
            stage('Fluffy Test') {
              parallel {
                stage('Backend Java 8') {
                  agent {
                    node {
                      label 'java8'
                    }
                  }
                  steps {
                    unstash 'Java 8'
                    sh './jenkins/test-backend.sh'
                  }
                  post {
                    always {
                      junit 'target/surefire-reports/**/TEST*.xml'
                    }
                  }
                }
                stage('Frontend') {
                  agent {
                    node {
                      label 'java8'
                    }
                  }
                  steps {
                    unstash 'Java 8'
                    sh './jenkins/test-frontend.sh'
                  }
                  post {
                    always {
                      junit 'target/test-results/**/TEST*.xml'
                    }
                  }
                }
                stage('Performance Java 8') {
                  agent {
                    node {
                      label 'java8'
                    }
                  }
                  steps {
                    unstash 'Java 8'
                    sh './jenkins/test-performance.sh'
                  }
                }
                stage('Static Java 8') {
                  agent {
                    node {
                      label 'java8'
                    }
                  }
                  steps {
                    unstash 'Java 8'
                    sh './jenkins/test-static.sh'
                  }
                }
                stage('Backend Java 7') {
                  agent {
                    node {
                      label 'java7'
                    }
                  }
                  steps {
                    unstash 'Java 7'
                    sh './jenkins/test-backend.sh'
                  }
                  post {
                    always {
                      junit 'target/surefire-reports/**/TEST*.xml'
                    }
                  }
                }
                stage('Frontend Java 7') {
                  agent {
                    node {
                      label 'java7'
                    }
                  }
                  steps {
                    unstash 'Java 7'
                    sh './jenkins/test-frontend.sh'
                  }
                  post {
                    always {
                      junit 'target/test-results/**/TEST*.xml'
                    }
                  }
                }
                stage('Performance Java 7') {
                  agent {
                    node {
                      label 'java7'
                    }
                  }
                  steps {
                    unstash 'Java 7'
                    sh './jenkins/test-performance.sh'
                  }
                }
                stage('Static Java 7') {
                  agent {
                    node {
                      label 'java7'
                    }
                  }
                  steps {
                    unstash 'Java 7'
                    sh './jenkins/test-static.sh'
                  }
                }
              }
            } // end stage
            stage('Confirm Deploy') {
              when {
                branch 'master'
              }
              steps {
                timeout(time: 3, unit: 'MINUTES' ) {
                  input(message: "Okay to Deploy to Staging?", ok: "Let's Do it!")
                }
              }
            } // end stage
            stage('Fluffy Deploy') {
              when {
                branch 'master'
              }
              agent {
                node {
                  label 'java7'
                }
              }
              steps {
                unstash 'Java 7'
                sh "./jenkins/deploy.sh ${pipelineParams.deployToDefault}"
              }
            } // end stage              
          } // end stages
        } // end pipeline
    }
```

-- -- -- -- -- -- -- -- -- --

### Pipeline durability
when to use high-performance durability settings
- basic pipelines that simply build and test code

when to not use high-performance durability settings
- pipelines wait for bash/shell scripts
- not running pipelines
- modify state of critical infrastructure
- production deployment

durability settings
- max survivability - little to no loss if Jenkins has a dirty shutdown
- survival_nonatomic - avoids atomic writes
- performance_optimized - probably will lose data if Jenkins has a dirty shutdown

"Manage Jenkins" -> "Configure System" -> "Pipeline Speed and Durability"

in Jenkinsfile
```
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    options {
        durabilityHint('PERFORMANCE_OPTIMIZED')
    }
}
```

Best practices
- general build/test: PERFORMANCE_OPTIMIZED
	* MAX_SURVIVABILITY for *critical infrastructure*
- auditing: *MAX_SURVIVABILITY* or *SURVIVAL_NONATOMIC*
- force pipelines to persist data by pausing them

[further reading for scaling pipelines](https://jenkins.io/doc/book/pipeline/scaling-pipeline/)

-- -- -- -- -- -- -- -- -- --

### Restart from a Stage
Any completed Declarative Pipeline can be restarted from any top-level stage that ran in that pipeline
- transient/environmental considerations
- AWS crashed
- and many more

This is available for *Declarative Pipeline* jobs only

Checkpoint is a more sophisticated stage restart 

- Stages that are skipped (when conditions) will not be available for restart
- Parent stages for parallel stages are also not available

But how does it know what to do?

Stashing! Run 'stash' and 'unstash' at every stage so it can run from any accessible stage

Enable stash preservation
- preserveStashes property (1-50) (default value is 0)

best practices: retain as many stashes as there are number of builds for the job

```
options{
	preserveStashes()
	preserveStashes(buildCount: 4)
}
```

-- -- -- -- -- -- -- -- -- --

### Groovy Sandbox
Groovy sandbox is enabled by default because Groovy can become very dangerous. The sandbox will run a test script without risking damage to the Jenkins environment

- Not necessary to wait for an admin
- when running, each method call, object construction, and field access is checked against a *whitelist* of approved operations
- if unapproved operation is attempted, the script is killed

-- -- -- -- -- -- -- -- -- --

### Best Practices
- Keep it simple. The more Groovy, the harder it is to read
- Use command line tools for XML and JSON parsing (xmllint/XMLStarlet, jq)
- Use external scripts and tools. Avoid complex logic in the Pipeline itself
- Reduce the number of steps in a Pipeline ( < 300 steps )