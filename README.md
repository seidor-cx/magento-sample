# Magento local environments

## Installing DDEV for creating environments
- Prerequisite: having WSL installed and assigning a default Ubuntu distribution.
1.  Open PowerShell as administrator and run the following command:
   
	```	
	Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
	iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/ddev/ddev/main/scripts/install_ddev_wsl2_docker_inside.ps1'))
	```
    This command is used to:
    - Installing Chocolatey (package manager for Windows).
    - Installing Docker and configuring WSL2.
    - Installing DDEV.
    - Configuring Ubuntu on WSL2 to integrate with Docker.

3. Open Settings and go to Windows Update -> Advanced Options -> Receive updates from other Microsoft products

# Manually creating a new environment using ddev

1. Create a new folder (for organizational purposes, it is recommended to create a folder to store all your projects, such as Sites) 
  - ``` mkdir Sites && cd Sites ```
  - ``` mkdir magentotest && cd magentotest```
    
2. The creation of a project with DDEV in the directory begins.
  ``` ddev config --project-type=magento2 --docroot=pub --create-docroot --project-name=magentotest --mutagen-enabled --disable-settings-management --host-db-port=50000```
  - --host-db-port It must be set to the database port and must be unique for each project.
  - --project-name Make sure the project name is the one you want to use for your project, or leave it empty. The project will be created with the folder name.

3. Check the versions of the requirements in the .ddev/config.yaml file and make sure they are the correct ones according to the Magento version.
[System requirements](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements).
Verify that the host_db_port is uncommented and with the port assigned to it.

4. Creating containers with config.yaml versions
   - Installing elasticsearch  ```ddev get drud/ddev-elasticsearch``` or opensearch ```ddev get ddev/ddev-opensearch```
   - Redis installation ```ddev get drud/ddev-redis```
   - Initialize containers ```ddev start```
     
5. Add credentials and download Magento source files. To obtain these credentials you must access the Magento marketplace, go to the profile and in access key you will find them.
   - ```ddev composer global config http-basic.repo.magento.com <public_key> <private_key>```
   - ```ddev composer create --no-install --repository=https://repo.magento.com/project-community-edition:2.4.x```
   - ```ddev composer install```
     
6. Creating the env.php file with essential settings for Magento. This is just a base, change the values ​​according to your environment configuration.
   ```
   ddev magento setup:install --base-url="https://magentotest.ddev.site/" \
    --cleanup-database --db-host=db --db-name=db --db-user=db --db-password=db \
    --elasticsearch-host=elasticsearch --search-engine=elasticsearch7 --elasticsearch-port=9200 \
    --admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com \
    --admin-user=admin --admin-password=Password123 --language=en_US
   ```
7. Commands to finish configuring the local environment and delete static files
   - ```ddev magento deploy:mode:set developer```
   - ```ddev config --disable-settings-management=false```
   - ```
       ddev magento s:up && ddev magento deploy:mode:set developer && ddev magento c:f && ddev exec rm -rf pub/static var/cache var/view_preprocessed generated
     ```
8. (optional) installing sample data
   ```ddev magento sampledata:deploy && ddev magento setup:upgrade```

Now with these settings you can access from the url
   
   

# Manual creation of an existing environment using ddev

1. Clone the repository to your computer (for organizational purposes, it is recommended to create a folder to hold all your projects, such as Sites)   
  - ``` mkdir Sites && cd Sites ```
  - ```git clone git@github.com:seidor-cx/ ```
  - ```cd (project folder) ```
2. The creation of a project with DDEV in the directory begins.
  ```ddev config --project-type=magento2 --docroot=pub --create-docroot --project-name=name --host-db-port=50000```
  - --host-db-port It must be set to the database port and must be unique for each project.
  - --project-name Make sure the project name is the one you want to use for your project, or leave it empty. The project will be created with the folder name.

3. Check the versions of the requirements in the .ddev/config.yaml file and make sure they are the correct ones according to the Magento version.
[System requirements](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements).
Verify that the host_db_port is uncommented and with the port assigned to it.

4. Creating containers with config.yaml versions
   - Installing elasticsearch  ```ddev get drud/ddev-elasticsearch``` or opensearch ```ddev get ddev/ddev-opensearch```
   - Redis installation ```ddev get drud/ddev-redis```
   - Initialize containers ```ddev start```

5. Magento configuration and installation
 - Installing packages ```ddev composer install```
 - Import database ```ddev import-db --file=/home/user/Backups/file.sql```. To connect to the database manager, you can run ```ddev describe``` and all the necessary values ​​are in the DB         information.
   - Validate that the *catalog/search/engine*, *catalog/search/elasticsearch7_server_hostname*, *catalog/search/elasticsearch7_server_port* settings have the corresponding value with the search engine you chose.
   - In the database, filter the configuration table by URLs (```select * from core_config_data where path like '%url%';```) and modify it with the URL that corresponds to the project. Do this also with the cookies. *Run ```ddev describe``` to display the environment host*
 - Apply changes and delete static.
   - Make sure that the *app/etc/config.php* file exists, if it doesn't, run the command ```ddev magento module:enable --all```
   - Make sure the file app/etc/env.php exists if it does not exist run the command
     ```
      ddev magento setup:install --base-url="https://${MAGENTO_HOSTNAME}.ddev.site/" \
      --cleanup-database --db-host=db --db-name=db --db-user=db --db-password=db \
      --elasticsearch-host=elasticsearch --search-engine=elasticsearch7 --elasticsearch-port=9200 \
      --admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com \
      --admin-user=admin --admin-password=Password123 --language=en_US
     ```
     This is just a base, change the values ​​according to your environment configuration.
   - Once you verify that these two files are there, run the following commands:
     ```
       ddev magento s:up && ddev magento deploy:mode:set developer && ddev magento c:f && ddev exec rm -rf pub/static var/cache var/view_preprocessed generated
     ```

Now with these settings you can access from the url





