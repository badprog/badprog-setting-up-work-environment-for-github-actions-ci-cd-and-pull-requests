Best practises are unlimited so we have to set limits.  
In this tutorial we are going to see how to set up a CI/CD environment in order to manage a project (here in Python but any langage will be fine).  

# What we need

- **Github**: https://github.com/
- **Github Actions**: https://github.com/features/actions  
We don't need to install anything for using Actions because they are already integrated in your Github environment, this latter link is for information purpose.
- **Docker Desktop** (if needed): https://www.docker.com/products/docker-desktop/  
For more information how to install it (and also VSCode Dev Containers extension): https://github.com/badprog/badprog-connecting-docker-desktop-with-vscode-dev-containers-extension

## The directory tree

```
.
├── .devcontainer
│   └── devcontainer.json
├── Dockerfile
├── .git
├── .gitattributes
├── .github
│   └── workflows
│       └── ci.yml
├── .gitignore
├── README.md
├── requirements.txt
├── src
│   └── main.py
└── tests
    └── test_main.py
```

# How it works

Our project will be separate in several parts:

- Source files
- Unit tests
- Github PR
- Github Rule
- Github Actions
- Docker image

## Github Actions

The main part is in the ci.yml file.
This is a GitHub Actions workflow file written in YAML, named "CI Pipeline."  
It automates the process of building, testing, and deploying a Docker image for a Python application.  

### 1. Workflow triggers
The workflow is triggered in two scenarios:

- **push**: Runs on any push to any branch (branches: ["**"] matches all branches).
- **pull_equest**: Runs when a pull request targets the main branch.

### 2. Permissions

- **contents: read**: Grants read-only access to the repository's contents.
- **packages: write**: Allows writing (pushing) to the GitHub Container Registry (GHCR).

### 3. Jobs
The workflow defines two jobs: **build-and-test** and **push-docker**.

#### a. build-and-test
This job builds the Docker image and runs tests.

Runs on: ubuntu-latest (the latest Ubuntu virtual machine provided by GitHub Actions).

The steps are easy to understand because there is a name explaining what they are suppose to do.

#### b. push-docker
This job pushes the Docker image to the GHCR (GitHub Container Registry), but only if the code is pushed to the main branch.

It depends on the **build-and-test** job (this job only runs if build-and-test succeeds).
The condition is that it only runs when the push is to the main branch (if: **github.ref == 'refs/heads/main'**).

As the previsous job, the step are easy to get with their respective name.

## Dockerfile
As our goal is to create a Docker image, we need to create a Dockerfile.  
Ours is quite simple, it derives from the **mcr.microsoft.com/devcontainers/python:3.11** base image.  

## Unit tests
Our unit tests are basic but they can prove that our system is working correctly.

## Github rule
Before pushing on the main branch, we have to be sure that a PR has been created and validated.  
For that we need to set a Github rule.  
To create one, let's go on your Github repository > Settings > Branches > Add classic branch protection rule.  
In the new form, write **main** in the **Branch name pattern area**.  
Then below select:
- **Require a pull Request before merging**  
  If you are working alone on this project, unselect the **Require approval option** (otherwise someone else than you will have to review your PR) 
- **Require status checks to pass before merging**
    - **Require branches to be up to date before merging**  
      In the below area, write: **build-and-test**
- **Do not allow bypassing the above settings**

## Github PR
Once our project is ready to be pushed we have to create another branch in order to push our commits on this temporary branch.  
It's here that we are going to validate these commits by creating a PR.  
If our tests pass, then we'll be authorized to validate this PR.  
And once the PR is validated, the Docker image will be pushed on the GHCR and the commits on the main branch.  


### Where to see the Actions in action
To check if Actions are running you can check that directly from your Github repository by clicking the Actions tab (on the right of the **Pull request** one).  
You can indeed to see if Actions have failed or succeeded (if they are already finished to work).  
      
If everything succeeded then well done, our Github repository is ready to be used.  
😎
