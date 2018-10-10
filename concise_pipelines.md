Concise Pipelines with Shared Libraries
---------------------------------------

*Pipelines need to be like miniskirts: not too long, but short enough to keep it interesting*

### What is just right?
- Concise
- Clear
- DevOps!

### Shared Libraries
- Layer of abstraction
- Create custom steps
- Share common logic across multiple projects
- **Separate repository**

Most importantly, give developers a layer of abstraction

-- -- -- -- -- -- -- -- -- -- -- --

### Examples

```
pipeline{
  libraries{}

  agent{}

  options{}

  stages{
    stage('1') {}

    stage('2') {}
  }

  post{
    always{
      result = currentBuild.result ?: 'SUCCESS'
      echo ${result}
    }
  }
}
```

-- -- -- -- -- -- -- -- -- -- -- --

### What about ...

- DRY (Don't Repeat Yourself)
    + make a function out of it and put it in a shared library
- More complex scenarios
    + go to the Jenkins community to see if your scenario is being worked on
- Maintainability
    + always go towards Declarative if possible
- Validation
    + pipelines need to be validated
    + shared libraries are groovy, and groovy can be tested

-- -- -- -- -- -- -- -- -- -- -- --

### Recommendations
- use declarative pipeline
- use shared libraries
- democratize the continuous delivery process
    + make it understandable to everyone on the team (Ops, Managers, other Devs)

-- -- -- -- -- -- -- -- -- -- -- --

### Testing shared libraries
- use brances?
- Jenkinsfile runner
- Pipeline Unit Testing?
  + mock responses
  + test the pipeline
- Spock
  + Unit framework for testing Groovy
    * test shared libraries