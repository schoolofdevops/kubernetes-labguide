# GitHub Actions Workflow File Explanation

Here is the GHA workflow code 

```
#.github/workflows/ci.yml
name: CI

on:
  push:
    branches:
      - main
    paths-ignore:
      - 'Dockerfile'
      - 'Jenkinsfile'
      - 'chart/**'
  pull_request:
    branches:
      - main
    paths-ignore:
      - 'Dockerfile'
      - 'Jenkinsfile'
      - 'chart/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: pip install -r requirements.txt

  unit-test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: nose2

  image-bp:
    runs-on: ubuntu-latest
    needs: unit-test
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        run: |
          COMMIT_HASH=$(echo ${{ github.sha }} | cut -c1-7)
          docker build -t ${{ secrets.DOCKER_USERNAME }}/vote:$COMMIT_HASH .
          docker push ${{ secrets.DOCKER_USERNAME }}/vote:$COMMIT_HASH
```



This GitHub Actions workflow file defines a Continuous Integration (CI) pipeline that runs on pushes and pull requests targeting the main branch, with three main jobs: building the environment, running unit tests, and building & pushing a Docker image. Here’s a detailed, step-by-step explanation:

---

### 1. Workflow Trigger and Naming

- **Name of the Workflow:**  

  ```yaml
  name: CI
  ```  

  This gives the workflow a clear identifier ("CI") that shows up in the GitHub Actions UI.

- **Triggers (`on`):**  

  ```yaml
  on:
    push:
      branches:
        - main
      paths-ignore:
        - 'Dockerfile'
        - 'Jenkinsfile'
        - 'chart/**'
    pull_request:
      branches:
        - main
      paths-ignore:
        - 'Dockerfile'
        - 'Jenkinsfile'
        - 'chart/**'
  ```  

  - **Push and Pull Request:** The workflow will run when:
    - A push is made to the `main` branch.  
    - A pull request is opened or updated that targets the `main` branch.  
  
  - **Paths-ignore:** It ignores triggers if changes are made only to specific files (like `Dockerfile`,   `Jenkinsfile`) or any files under the `chart` directory. This helps avoid unnecessary runs when only documentation or non-code files are modified.
  
---

### 2. Jobs Overview

The workflow defines **three jobs**:  
- **build:** Sets up the environment and installs dependencies.  
- **unit-test:** Runs tests and depends on the successful completion of the `build` job.  
- **image-bp:** Logs in to DockerHub, builds a Docker image, and pushes it. This job runs only after the   `unit-test` job passes.  

Each job is executed on an `ubuntu-latest` runner, meaning they run in a fresh Ubuntu environment provided by GitHub Actions.  

---

### 3. Job: Build


```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
```

- **Environment Setup:**    
  `runs-on: ubuntu-latest` ensures the job runs on the latest Ubuntu runner.  
  
- **Steps:**

  1. **Checkout code:**  
     - **Action Used:** `actions/checkout@v2`  
     - **Purpose:** Retrieves your repository’s code so that subsequent steps can work on it.  
  
  2. **Set up Python:**  

     ```yaml
      - name: Set up Python  
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'
     ```  

     - **Action Used:** `actions/setup-python@v2`  
     - **Purpose:** Installs and configures Python (any version in the 3.x series) for the job.  
  
  3. **Install dependencies:**  

     ```yaml
      - name: Install dependencies
        run: pip install -r requirements.txt
     ```  
     - **Command:** Runs `pip install -r requirements.txt`  
     - **Purpose:** Installs all Python dependencies listed in the `requirements.txt` file.  

---

### 4. Job: Unit Test 


```yaml
  unit-test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
```

- **Dependency:**   
  - `needs: build` indicates that this job will run only if the `build` job completes successfully.  
  
- **Steps (Similar Setup to Build Job):**
  1. **Checkout code:** Again, uses `actions/checkout@v2` to ensure that the code is available.  
  2. **Set up Python:** Uses `actions/setup-python@v2` to configure Python.  
  3. **Install dependencies:** Installs Python packages using `pip install -r requirements.txt`.  
  4. **Run tests:**  
     ```yaml
      - name: Run tests
        run: nose2
     ```  
     - **Command:** Runs `nose2`  
     - **Purpose:** Executes unit tests using the nose2 test runner.  

---

### 5. Job: Image Build and Push

```yaml
  image-bp:
    runs-on: ubuntu-latest
    needs: unit-test
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
```

- **Dependency:**    
  - `needs: unit-test` means this job will only start if the unit tests have passed.  
  
- **Steps:**
  1. **Checkout code:** Once more, checks out the repository using `actions/checkout@v2`.  
  
  2. **Log in to DockerHub:**  

     ```yaml
      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
     ```  

     - **Action Used:** `docker/login-action@v2`  
     - **Purpose:** Authenticates to DockerHub using credentials stored in GitHub Secrets (`DOCKER_USERNAME`   and `DOCKER_PASSWORD`), ensuring that the Docker commands can push images to the DockerHub repository  securely.  
  
  3. **Build and push Docker image:**  
     ```yaml
      - name: Build and push Docker image
        run: |
          COMMIT_HASH=$(echo ${{ github.sha }} | cut -c1-7)
          docker build -t ${{ secrets.DOCKER_USERNAME }}/initcron:$COMMIT_HASH .
          docker push ${{ secrets.DOCKER_USERNAME }}/initcron:$COMMIT_HASH
     ```    
     - **Commands Explained:**  
       - **Generate a Commit Hash:**    
         ```bash
         COMMIT_HASH=$(echo ${{ github.sha }} | cut -c1-7)
         ```    
         This extracts the first 7 characters of the commit SHA (a unique identifier for the commit) to use as a tag.  
       - **Build the Docker Image:**  
         ```bash
         docker build -t ${{ secrets.DOCKER_USERNAME }}/initcron:$COMMIT_HASH .
         ```    
         Builds a Docker image using the Dockerfile in the repository’s root directory, tagging it with a combination of the DockerHub username and the commit hash.  
       - **Push the Docker Image:**  

         ```bash
         docker push ${{ secrets.DOCKER_USERNAME }}/initcron:$COMMIT_HASH
         ```  

         Pushes the built image to DockerHub so that it can be used elsewhere (e.g., in production or further testing).
  
---

### Summary

1. **Workflow Triggers:**  
   The pipeline triggers on pushes and pull requests to the main branch, excluding specific file changes.  

2. **Build Job:**    

   - Checks out the code.  
   - Sets up Python.  
   - Installs dependencies from `requirements.txt`.  

3. **Unit Test Job:**  

   - Depends on the build job.  
   - Repeats the setup steps.  
   - Runs tests using `nose2`.  

4. **Image Build and Push Job:**  

   - Depends on the unit-test job.  
   - Checks out the code.  
   - Logs into DockerHub with secure credentials.  
   - Builds a Docker image tagged with a short commit hash.  
   - Pushes the image to DockerHub.  


This setup ensures that code changes are automatically tested and, if successful, packaged as a Docker image ready for deployment or further use.