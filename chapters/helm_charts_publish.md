# Publishing Charts


To publish your Helm charts using GitHub Pages, you'll need a GitHub account and a new or existing repository where you can store and serve your charts. Here's a detailed step-by-step process to get your Helm charts published using GitHub Pages:

### Step 1: Prepare Your GitHub Repository

**Create a New Repository**: If you don't already have one, create a new GitHub repository. It can be named anything, but something like `helm-charts` is descriptive and common.
![](Screenshot%202024-05-13%20at%209.04.15%20PM.png)
### Step 2: Clone Your Repository Locally

```bash

cd~
git clone https://github.com/xxxxxx/helm-charts.git
cd helm-charts
```

Replace `<username>` with your GitHub username and `<repository-name>` with the name of your repository.

### Step 3: Package Your Helm Chart

Assuming you have your chart directory for instavote ready, move that inside the helm-charts

e.g.

```
mv instavote helm-charts/
cd helm-charts
git add instavote/
git commit -am "added instavote chart"
git push origin main
git log
```

```bash
helm package instavote
ls
```

This command will create a `.tgz` file of your chart.

### Step 4: Create or Update the Helm Chart Repository Index

After packaging your chart, create an index file that Helm can use to fetch chart metadata:

```bash
helm repo index . --url https://xxxxxx.github.io/helm-charts/
```

This command generates or updates an `index.yaml` file with information about charts stored in the directory.

### Step 5: Switch to the `gh-pages` Branch

If the `gh-pages` branch does not exist, create it and switch to it:

```bash
git checkout --orphan gh-pages
git status
git rm -rf instavote README.md
```


### Step 6: Commit and Push Your Changes

```bash
git add *.tgz index.yaml
git commit -m "Publish Helm charts"
git push origin gh-pages
```


### Step 7: Ensure GitHub Pages is Set Up Correctly

- Go to your repository's settings.![](Screenshot%202024-05-13%20at%209.04.53%20PM.png)
- Find the "Pages" section.
- Set the source to the `gh-pages` branch (you may need to create this branch in your repository if it doesn't exist yet).
![](Screenshot%202024-05-13%20at%209.09.09%20PM.png)


### Step 8: Add Your Repository to Helm

Now that your chart is published, you can add your repository to Helm:

```bash
helm repo add instavote https://xxxxxx.github.io/helm-charts/
```


### Step 9: Update Helm Repositories

```bash
helm repo list

helm repo update
```

This will update your local Helm client with the latest charts from your newly published repository.

### Step 10: Installing Charts

You and others can now install charts from your repository:

```bash
helm search repo  instavote
helm install instavote instavote/instavote
```

Replace `<release-name>` with a name for your Helm deployment and `<chart-name>` with the name of the chart in your repository.

This process sets up a straightforward, cost-effective, and robust method for hosting and managing your Helm charts using GitHub and GitHub Pages.
