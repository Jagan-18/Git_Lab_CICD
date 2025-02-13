---

# **GitLab CI/CD Notes**

### **1. What is GitLab CI/CD?**
GitLab CI/CD is a continuous integration/continuous deployment service that is built into GitLab. It automates the process of building, testing, and deploying applications, ensuring software changes are automatically integrated, tested, and deployed with minimal manual intervention.

### **2. Key Concepts**

- **Pipeline**: A collection of jobs that run sequentially or in parallel to automate your build, test, and deploy processes.
- **Job**: A single task in the pipeline, such as running tests, building the project, or deploying code.
- **Stage**: A logical grouping of jobs. Jobs within a stage run sequentially.
- **Runner**: A lightweight agent that runs the jobs defined in the pipeline.
- **Artifacts**: Files that are generated by a job, which can be used by other jobs or downloaded after the pipeline is finished.
- **Cache**: Temporary storage for files that don’t change often (e.g., dependencies), speeding up pipeline execution by reusing them.
- **Environment Variables**: Variables that hold values like API keys, project secrets, or configuration settings.

---

### **3. Basic `.gitlab-ci.yml` Structure**

The `.gitlab-ci.yml` file defines the CI/CD pipeline. It's located at the root of your project repository.

#### **Structure:**

```yaml
stages:
  - build
  - test
  - deploy

job_name:
  stage: build
  script:
    - echo "Job script goes here"
```

**Sections of `.gitlab-ci.yml`:**
1. **stages**: Defines the order of the pipeline stages (e.g., `build`, `test`, `deploy`).
2. **jobs**: Define the steps to be run in each stage. A job has:
   - **stage**: Which stage the job belongs to.
   - **script**: Commands to run for that job.

---

### **4. Key Configuration Options in `.gitlab-ci.yml`**

- **`stages`**: A list of stages that will be executed in order.
- **`job_name`**: The name of a job that runs within a stage.
- **`script`**: The commands to execute during a job.
- **`before_script` / `after_script`**: Commands that run before or after a job's script.
- **`only` / `except`**: Conditions for when a job should run.
- **`rules`**: A more advanced way to define job conditions, replacing `only` and `except`.
- **`artifacts`**: Specifies files or directories to keep after a job finishes.
- **`cache`**: Defines which files to cache across jobs and pipeline runs.

---

### **5. Stages and Jobs**

#### **Stages**
Stages represent the flow of your pipeline. By default, GitLab runs jobs in parallel within a stage, and jobs in different stages run sequentially. For example:
```yaml
stages:
  - build
  - test
  - deploy
```

#### **Jobs**
Each job corresponds to a task in a particular stage.

```yaml
build_job:
  stage: build
  script:
    - echo "Building project"
    - make

test_job:
  stage: test
  script:
    - echo "Running tests"
    - npm test
```

---

### **6. Example of a Simple Pipeline**

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - echo "Build commands go here"
    - make build

test:
  stage: test
  script:
    - echo "Test commands go here"
    - npm test

deploy:
  stage: deploy
  script:
    - echo "Deploy commands go here"
    - ./deploy.sh
  only:
    - main  # Only deploy when changes are pushed to the main branch
```

---

### **7. Environment Variables**

Environment variables can be defined in your GitLab project settings or in the `.gitlab-ci.yml` file.

#### **Defining Variables in `.gitlab-ci.yml`**:

```yaml
variables:
  NODE_ENV: "production"
  DB_PASSWORD: "mysecretpassword"
```

#### **Setting Variables in GitLab UI**:
- Navigate to **Settings > CI / CD > Variables** and add your key-value pairs for secret keys, tokens, etc.

---

### **8. Caching and Artifacts**

#### **Cache**: Used to store files that do not change often, such as dependencies, to speed up pipelines.

```yaml
cache:
  paths:
    - node_modules/
    - .npm/
```

#### **Artifacts**: Store files or directories generated by a job to pass them to subsequent jobs or to download later.

```yaml
artifacts:
  paths:
    - dist/
  expire_in: 1 hour
```

---

### **9. Conditional Execution with `rules`, `only`, and `except`**

#### **`only` and `except`**:
- `only` runs a job under specific conditions (e.g., only on a certain branch).
- `except` excludes a job from running under specific conditions.

```yaml
deploy:
  script: ./deploy.sh
  only:
    - main
```

#### **`rules`**: A more flexible way to control job execution, replacing `only` and `except` in modern GitLab CI configurations.

```yaml
deploy:
  script: ./deploy.sh
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
```

---

### **10. Runners**

A **GitLab Runner** is an agent that runs jobs in your pipeline. GitLab provides shared runners, but you can also configure your own custom runners.

#### **To register a runner**:
1. Install GitLab Runner on your machine.
2. Register it with your GitLab instance by running the command:
   ```bash
   gitlab-runner register
   ```

---

### **11. Advanced Features**

#### **Parallel Jobs**
Run multiple jobs at the same time to speed up pipeline execution.

```yaml
stages:
  - test

test_job_1:
  stage: test
  script:
    - npm test --env=chrome

test_job_2:
  stage: test
  script:
    - npm test --env=firefox
```

#### **Matrix Jobs**
Test the same code on different versions of a language, platform, or environment.

```yaml
stages:
  - test

test_matrix:
  stage: test
  script:
    - npm install
    - npm test
  matrix:
    - NODE_VERSION: ["14", "16", "18"]
```

---

### **12. Docker and GitLab CI**

GitLab CI allows integration with Docker for containerized workflows.

#### **Example of Docker Build and Push**:

```yaml
stages:
  - build
  - deploy

docker_build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t my-app:latest .

docker_push:
  stage: deploy
  script:
    - docker push my-app:latest
  only:
    - main
```

---

### **13. CI/CD Pipeline Visualization**

- **Pipeline Status**: View the status of your pipeline in the GitLab UI: Green for success, Red for failure.
- **Logs**: Click on each job to view detailed logs to understand failures.
- **Pipeline Graph**: GitLab shows a graphical representation of your pipeline execution order, including which jobs run in parallel.

---

### **14. Best Practices**

- **Keep it simple**: Start with a simple pipeline and gradually add complexity as needed.
- **Use stages logically**: Organize stages (e.g., `build`, `test`, `deploy`) to reflect the logical flow of your workflow.
- **Secure secrets**: Use GitLab's CI/CD variables for secure management of API keys and other sensitive information.
- **Cache effectively**: Cache dependencies and other unchanged files to optimize pipeline execution time.
- **Monitor and refine**: Continuously monitor pipeline execution and optimize for speed and efficiency.

---

### **15. Resources for Further Learning**

- **GitLab Documentation**: [Official GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
- **CI/CD Examples**: [GitLab CI/CD Examples](https://docs.gitlab.com/ee/ci/examples/)
- **GitLab CI Syntax Reference**: [GitLab CI YAML Syntax](https://docs.gitlab.com/ee/ci/yaml/)

---
This should provide you with a well-rounded set of GitLab CI notes that you can refer to for setting up, configuring, and optimizing your CI/CD pipelines in GitLab.



