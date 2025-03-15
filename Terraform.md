## Import pipeline from GitHub
1. On Harness "Continous Delivery & GitOps" section, click on the dropdown besides the "+ Create a Pipeline" button, and select "+ Import From Git"
2. Choose the "Third-party Git provider" option
3. On Git Connector, click on the selector and create a new connector.
4. Choose "GitHub Connector", and type in "Devops Harness ECS connector" as its name. Click on "Continue".
5. For "URL Type" choose "Repository", and "HTTP" for its "Connecetion Type"
6. Input the URL the your fork of this repository. Click on "Continue".
7. You may choose a different "Authentication" mode, but for here we will use "Username and Token"
8.  Type in your GitHub username
9. Click on "Create or Select a Secret" and a modal window will appear.
10. Click on "+ New Secret Text"
11. Type in a name for your secret.
12. For the secret value you will need to create a GitHub access token (classic) with the following permissions scope: "admin:repo_hoo", "repo", "user", see here for more information: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic
13. Click on "Save", and then on "Apply Selected"
14. Select "Enable API access", you may use the same access token. Click on "Continue".
15. Choose "Connect through Harness Platform". Click on "Save and Continue".
16. Click on "Finish" after the connection test is successful.
17. The "Repository" and "Git Branch" should autopopulate with the name of the repository and "main" respectively
18. For "YAML Path", the input should look like similar to this: `.harness/Pipeline_file.yaml`.