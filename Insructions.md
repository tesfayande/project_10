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
         

