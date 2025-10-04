# create a new repository on the command line

echo "# project_10" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/tesfayande/project_10.git
git push -u origin main

## push an existing repository from the command line

git remote add origin https://github.com/tesfayande/project_10.git
git branch -M main
git push -u origin main

## pushing

git add .
git commit -m "first commit"
git branch -M main
git push -u origin main

## Backend Actions

## SonarCloud

name: SonarCloud Front (Quality)

on:
  workflow_call:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  Frontend:
    name: Frontend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: front
    strategy:
      matrix:
        node-version: [16.x]
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Use Node.js ${{matrix.node-version}}
        uses: actions/setup-node@v3
        with:
          node-version: ${{matrix.node-version}}
      
      - name: Run npm install
        run: npm install
      
      
      - name: Build and analyse
        uses: SonarSource/sonarqube-scan-action@v6
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

        with:
          projectBaseDir: front
         


## Complète

name: CI/CD Pipeline Complète

on:
  push:
    branches: [develop,main]
  pull_request:
    branches: [ main ]
    types: [opened, synchronize, reopened]

   
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
  DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

jobs:

  # Étape 1: Tests Backend
  
  backend-tests:
    
    name: Tests Backend & Coverage
    uses: ./.github/workflows/backend-tests-coverage.yml
    secrets: inherit

  # Étape 2: Tests Frontend
  frontend-tests:
   name: Tests Frontend & Coverage
   uses: ./.github/workflows/frontend-tests-coverage.yml
   secrets: inherit

  # Étape 3: Analyse Qualité Backend (déclenchée si tests OK)
  sonar-backend:
    name: SonarCloud Backend
    needs: [backend-tests, frontend-tests]
    if: success() && needs.backend-tests.result == 'success' && needs.frontend-tests.result == 'success'
    uses: ./.github/workflows/sonarcloud_back.yml
    secrets: inherit

  # Étape 4: Analyse Qualité Frontend (déclenchée si tests OK)
  sonar-frontend:
    name: SonarCloud Frontend
    needs: [backend-tests, frontend-tests]
    if: success() && needs.backend-tests.result == 'success' && needs.frontend-tests.result == 'success'
    #If: github.event_name == 'push' && github.ref == 'refs/heads/main' && needs.backend-tests.result == 'success' && needs.frontend-tests.result == 'success'
    uses: ./.github/workflows/sonarcloud_front.yml
    secrets: inherit
  # Étape 5: Build & Push Docker (déclenchée si qualité OK)
  docker-build:
    name: Build & Push Docker
    needs: [sonar-backend, sonar-frontend]
    #If: always() && needs.sonar-backend.result == 'success' && needs.sonar-frontend.result == 'success'
    if: github.event_name == 'push' && github.ref == 'refs/heads/main' && needs.sonar-backend.result == 'success' && needs.sonar-frontend.result == 'success'
    uses: ./.github/workflows/docker.yml
    secrets: inherit
  