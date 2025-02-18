[[_TOC_]]

# Overview 

This document describes three related but stand-alone procedures:
1. Updating the github version of SDLH (e.g. creating a new release) - aimed at ANS engineers who are adding new features
2. Performing an initial deploying the SDLH source code to a customer environment using DevOps pipelines (which uses the github version as its source).
3. Deploying future updates of the SDLH into customer environments when that customer already has the initial deployment (2).

 

# 0. Pre-requisites 

- You have access to the ANS Github repo (https://github.com/ans-coe)  
- Sigma deployed to customer environment. 
- ANS has standard access to customer Azure environment (e.g. Synapse admin, contributor role on data subscriptions). 
- ANS has DevOps access (‘contributor’ role on the project as minimum, but ideally ‘admin’ role).  
- You have a DevOps licence (Basic) 
- Metadata databases deployed. 
- A spark cluster that meets the current naming and version requirements in dev, test and prod Synapse. 
- The Synapse Managed Identity has built-in role ‘data queue storage contributor’ on data storage account (in dev, test, and prod). 
- Metadata databases have storage account blob contributor on log storage accounts. 
- A managed private endpoint to the resource ‘storage account queue’ has been created in Synapse and then approved in the networking options within the storage account. 
- The service principal used by DevOps should have the RBAC role ‘storage blob data contributor’ on the dev storage account. 

 
# 1. Releasing a new version of SDLH 
SDLH is made of several componets: Synapse Workspace, Synapse Serverless Database, Azure SQL Database (used for metadata). Each is it its own github repository and can be deployed and updated separately. 

## 1a. Synapse Workspace releases

1. Clone the github version of the synapse workspace to a local folder (from your command line, ‘git clone [repo url] 
2. Clone the current main branch of the ANS POC version of the synapse workspace from devops to a local folder (from your command line ‘git clone [repo url] 
3. Open your local copy of the github repo in VSCode 
4. In VSCode, create a new branch – name your branch following the release naming conventions in GitHub (In GitHub, check the next available release to use as your branch name e.g. if the previous release was named ‘release/2.3.12’, then the branch you create in VSCode should be named ‘release/2.3.13’. To create a branch in VSCode, click on ‘main’ in the bottom left corner and select ‘new branch’. 
5. Update the file named ‘VERSION’ with your version number and ‘Commit changes..' 
6. Using Windows file explorer (or mac finder!), navigate to your local copy of the ANS POC version of the synapse workspace. For any files that you know are ahead in the ANS POC version compared to the GitHub version, drag them into the VSCode Explorer. 
7. VSCode will show you the changes – visually double check these are what you expect. 
8. For notebooks specifically, some unnecessary changes will be automatically added – which need to be manually removed. These can be discovered by looking at the ‘git compare’ within VSCode. To watch out for are ‘tracking ids’, ‘spark pools’ being attached and ‘bigpool’ references. These parts of your new notebooks to their original state – leaving only the changes you have made. 
9. In VSCode, go to ‘Source control’. Type in a commit message 
10. Click ‘Commit’ (it may ask you to stage all changes – select ‘yes’) 
11. Click ‘Publish’ (it may ask you to authenticate in github – follow the prompts) 
12. Navigate to the GitHub repo. You should see the option to ‘Compare & pull request’. Click on this button. 
13. Add your comment into the box and click ‘Create pull request’. 
14. GitHub should refresh the screen – you may then need to force the merge by selecting ‘Merge without waiting for requirements to be met (bypass branch protections), and now click ‘Merge pull request’ and then ‘Confirm merge’. 
15. Confirm that the new release was successful (you should get an email from github actions bot – also your new release should be displayed here: 

## 1b. Synapse Serverless releases

In progress..

## 1c. Azure SQL (metadata database) releases

In progress..

## 1d. Documentation GitHub Repo

1. Edit the github repo (www.github.com/ans-coe/sdlh-docs) directly inside a branch you create using the naming convention 'release/x.x.x' where x.x.x is the next available version number. Documentation that is deployed to customer environments is inside the folder 'Documentation'. Edit or create files within this folder. Increment the version number in the file 'VERSION'.
2. Merge your branch back into main with a pull request.
3. This should trigger the github action job to create a new release.

# 2. Initial Deployment to customer environment (github -> customer DevOps)

The steps below are also shown in the video: ANS DCoE - SDLH - Initial Deployment Procedure.mp4 (accessible to ANS only)

## Initial Steps (in customer DevOps)

1. Generate PAT for usage in deployments 
2. Click on ‘User Settings’> Personal access tokens 
3. Then ‘New Token’ 
4. Name your token 
5. Select ‘Scopes: Full Access’ and then ‘Create’ 
6. Copy token into text editor 

## Deploy Synapse Workspace

1. Navigate to SDLH project within the ANS repo: https://github.com/ans-coe/sdlh-synapse   
2. Download sdlh_synapse.pipeline.yml from github repo. 
3. Navigate to customer DevOps project. 
4. Go to the ‘synapse_workspace’ repo. 
5. Change into branch ‘workspace_publish’ 
6. Upload downloaded yml file to: Repo: ‘synapse_workspace’; Branch: ‘workspace_publish’; Location: ‘pipelines\’. 
7. Navigate to ‘pipelines’ in left hand menu and click on ‘create new pipeline’ 
8. Select’ Azure DevOps’ as the source of your code 
9. Click ‘create pipeline from existing pipeline’ 
10. Change branch to ‘workspace_publish’ 
11. Point to file that was uploaded in the previous step. 
12. Click ‘continue’ 
13. Click ‘Save’ on the pipeline (drop-down under ‘Run’). 
14. Click ‘3 dots’ next to the newly created pipeline and select Rename/move 
15. Re-name the newly created pipeline to ‘sdlh_synapse_deployment'. 
16. Click ‘save’ 
17. Click ‘Edit’ inside the pipeline and click on the the ‘one elicpe’ (three dots) and select ‘triggers’. 
18. Navigate to the ‘YAML’ tab. 
19. Select ‘Get sources’ and then change the ‘Default branch for manual and scheduled builds’ to ‘workspace_publish’. 
20. Click ‘save’ (not ‘Save & queue’). 
21. Duplicate tabs and navigate to second tab. 
22. Navigate to Pipelines > Pipelines > All. 
23. Open sdlh_synapse_deployment 
24. Click ‘Edit’ 
25. Add customer shir name to ‘parameters: - name: synapse_shir_name, default: ‘[shir_name_goes_here..]’. 
26. Add customer spark pool name to ‘parameters: -name: spark_pool_name..,default; ‘[customer_spark_pool_name_goes_here]’. 
27. Click ‘Save’. 
28. Click ‘Run’ (enter the saved Personal Access Token in the modal window). 
29. Approve the pipeline run when prompted. 
30. Wait for pipeline run to complete. 
31. In customer Synapse dev (or directly in DevOps), preview the newly created branch. 
32. Create PR to merge to main (in Synapse or directly in DevOps). 
33. In Synapse, switch back to ‘main’ branch and then click ‘Publish’. 

 
## Deploy Azure SQL 

1. Navigate to SDLH project within the ANS repo: https://github.com/ans-coe/sdlh-azuresql 
2. Download sdlh_azuresql.pipeline.yml from github repo. 
3. Download sdlh_metadata_form/metadata_form.csv github repo. 
4. Download sdlh_curation_metadata_form/sdlh_curation_metadata_form.csv github repo. 
5. Navigate to customer DevOps project. 
6. Go to the sql repo customer devops. 
7. Create new branch (e.g. ‘sql_sdlh_deployment’ – name not important). 
8. Upload file sdlh_azuresql.pipeline.yml to ‘pipelines\’ within your new branch. 
9. Go to metadata folder, click 'new > file' and create file/folder called 'sdlh_metadata_form/metadata_form.csv'. 
10. Copy the metadata form contents from downloaded file metadata_form.csv  into the new file and press 'commit'. 
11. Go to metadata folder, click 'new > file' and create file/folder called sdlh_curation_metadata_form/sdlh_curation_metadata_form.csv'. 
12. Copy the curation metadata form contents from downloaded file curation _metadata_form.csv  into the new file and press 'commit'. 
13. Create and complete a pull request to pull your branch into ‘main’.  
14. Go to pipelines in the left hand menu and create 'new pipeline', selecting the ‘sql’ repository, ‘existing YAML’ and then selecting the sdlh_azuresql.pipeline.yml file. 
15. Click ‘Save’ (under the ‘Run’ dropdown). 
16. Click ‘3 dots’ next to the newly created pipeline and select Rename/move 
17. Rename pipeline to 'sdlh_azuresql_deployment' and click ‘Save’. 
18. Click ‘Run’ and enter the ‘Personal Access Token’ in the modal window. 
19. Click ‘Run’ again (in the modal window). 
20. Once the pipeline run has completed, navigate to the newly created preview branch in the sql repo. 
21. Review preview branch in customer devOps. 
22. Create ‘Pull Request’ to merge into main. 
23. Approve and Complete the ‘Pull Request’. 
24. Cancel pipeline run ‘azure_sql_db_build_and_deployment’. (If it has automatically started) 
25. Cancel pipeline run ‘metadata_files_deployment’. (If it has automatically started) 
26. Run pipeline ‘azure_sql_db_build_and_deployment’ but select 'deploy_dev_enabled', uncheck 'deploy_test_enabled', uncheck 'deploy_prod_enabled', check ‘deploy_metadata_enabled’ and finally set metadata_overwrite_files to true (options in the modal window). 
27. Once complete, check the metadata form has been successfully loaded into the storage account and the Azure SQL database has required updated config. 

## Deploy Synapse Serverless SQL 

1. Navigate to SDLH project within the ANS repo: https://github.com/ans-coe/sdlh-synapse-serverless-sql 
2. Download sdlh_synapse_serverless.pipeline.yml from github repo. 
3. Navigate to customer DevOps project. 
4. Go to the sql repo customer devops. 
5. Create new branch (e.g. ‘sql_sdlh_deployment’ – name not important). 
6. Upload file sdlh_synapse_serverless.pipeline.yml to ‘pipelines\’ within your new branch. 
7. Create and complete a pull request to pull your branch into ‘main’.  
8. Go to pipelines in the left hand menu and create 'new pipeline', selecting the ‘sql’ repository, ‘existing YAML’ and then selecting the sdlh_synapse_serverless.pipeline.yml file. 
9. Click ‘Save’ (under the ‘Run’ dropdown). 
10. Click ‘3 dots’ next to the newly created pipeline and select Rename/move 
11. Rename pipeline to 'sdlh_synapse_serverless_deployment ' and click ‘Save’. 
12. Click ‘Run’ and enter the ‘Personal Access Token’ in the modal window. 
13. Click ‘Run’ again (in the modal window). 
14. Once the pipeline run has completed, navigate to the newly created preview branch in the sql repo. 
15. Review preview branch in customer devOps. 
16. Create ‘Pull Request’ to merge into main. 
17. Approve and Complete the ‘Pull Request’. 
18. Cancel pipeline run ‘synapse_sql_serverless_deployment’. (If it has automatically started) 
19. Run pipeline ‘synapse_sql_serverless_deployment’ but select 'deploy_dev_enabled', uncheck 'deploy_test_enabled', uncheck 'deploy_prod_enabled' and finally uncheck ‘deploy_metadata_enabled’ if checked (options in the modal window). 
20. Once complete, check the metadata form has been successfully loaded into the storage account.

## Deploy Document Repo to Customer DevOps

1. Download the file 'sdlh_docs.pipeline.yml' from the sdlh-docs repo.
2. In customer devops, create a new repo named 'documentation'
4. In the customer devops 'documentation' repo, create a folder called 'pipelines' and upload the 'sdlh_docs.pipeline.yaml' file to it. Commit this change.
5. In the customer devops 'documentation' repo, create a folder called 'docs' (you will have to put in a dummy file, because you can't create empty folders in devops)
6. Go to pipelines in the left hand menu and create 'new pipeline', select 'Azure Repos Git' to the question 'Where is your code?'.
7. Select the ‘documentation’ repository, and then ‘Existing Azure Pipelines YAML file’, and then select the sdlh_docs.pipeline.yml file. 
8. Click ‘Save’ (under the ‘Run’ dropdown). 
9. Click ‘3 dots’ next to the newly created pipeline and select Rename/move 
10. Rename pipeline to 'sdlh_documentation_deployment ' and click ‘Save’. 
11. Click ‘Run’ and enter the ‘Personal Access Token’ in the modal window. 
12. Click ‘Run’ again (in the modal window). 
13. Once the pipeline run has completed, navigate to the newly created preview branch in the documentation repo. 
15. Review preview branch in customer devOps. 
16. Create ‘Pull Request’ to merge into main. 
17. Approve and Complete the ‘Pull Request’.
18. Go to 'Overview > Wiki' in Devops (customer)
19. Click 'Publish code as wiki'
20. Select the 'documentation' repository in the modal window that appears
21. Select the 'main' branch
22. Select the 'docs' folder
23. Name your wiki (e.g. 'Data Platform')
24. Click 'Publish'

# 3. Updating SDLH deployment

## Synapse Upgrade 

1. Run pipeline ‘sdlh_synapse_deployment’. 
2. When complete, navigate to dev Synapse workspace and go to the newly created branch, which will be named ‘sdlh_updates_YYYY_MM_DD’. 
3. Complete a pull request for this branch and then approve the merge in DevOps. 
4. Go back to Synapse and click ‘Publish’. 
5. Wait for the pipeline ‘synapse_workspace_publish’ to trigger, then approve the deployment to test and next approve the deployment to production. 


## Azure SQL Upgrade  

1. Run pipeline 'sdlh_azuresql_deployment’. 
2. Navigate to Reps then switch to ‘sql’ and find the newly created branch, with a name like ‘preview-sdlh-updates-build243’. 
3. Review preview branch in customer devOps. 
4. Create ‘Pull Request’ to merge into main. 
5. Approve and Complete the ‘Pull Request’ (you’ll need to wait for the test run of the dacpac build to complete before you can press ‘complete’ on the merge). 
6. Cancel the pipeline ‘azure_sql_db_build_and_deployment’. (If it has automatically started) 
7. Run pipeline ‘azure_sql_db_build_and_deployment’ but select 'deploy_dev_enabled', uncheck 'deploy_test_enabled', uncheck 'deploy_prod_enabled'. 
8. Now, run the pipeline ‘azure_sql_db_build_and_deployment’ but enable ‘deploy to test’ and ‘deploy to prod’ 
9. Approve the deployments to test and prod. 


## Synapse Serverless SQL Upgrade  

1. Run pipeline ‘sdlh_synapse_serverless_deployment’. 
2. Navigate to Reps then switch to ‘sql’ and find the newly created branch, with a name like ‘preview-sdlh-updates-build243’. 
3. Review preview branch in customer devOps. 
4. Create ‘Pull Request’ to merge into main. 
5. Approve and Complete the ‘Pull Request’ (you’ll need to wait for the test run of the dacpac build to complete before you can press ‘complete’ on the merge). 
6. Cancel the pipeline ‘sdlh_synapse_serverless_deployment’. (If it has automatically started) 
7. Run pipeline ‘sdlh_synapse_serverless_deployment’ but select 'deploy_dev_enabled', uncheck 'deploy_test_enabled', uncheck 'deploy_prod_enabled'. 
8. Now, run the pipeline ‘sdlh_synapse_serverless_deployment’ but enable ‘deploy to test’ and ‘deploy to prod’ 
9. Approve the deployments to test and prod. 

## Documentation Upgrade

1. Run the pipeline 'sdlh_documentation_deployment'
2. When complete, navigate to 'documentation' repo and go to the newly created branch 
3. Complete a pull request for this branch and then approve the merge 
