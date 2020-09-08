.PHONY: az_create_storage az_upload_data toy_dataset project_dir clean lint git_init git_repo_test

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
RESOURCE_GROUP = {{ cookiecutter.azure_resource_group}}
STORAGE_ACCOUNT = {{ cookiecutter.azure_storage_account}}
STORAGE_CONTAINER = {{ cookiecutter.azure_storage_container}}
ASSIGNEE = {{ cookiecutter.author_azure_account_active_directory}}
SSH_OR_HTTPS = {{ cookiecutter.ssh_or_https }}


ifeq ("ssh", "$(SSH_OR_HTTPS)")
GIT_PATH=git@ssh.dev.azure.com:v3/{{ cookiecutter.azure_org }}/{{ cookiecutter.azure_project}}/{{ cookiecutter.repo_name}}
else
GIT_PATH=https@fix-me-later
endif

ifeq (,$(shell which conda))
HAS_CONDA=False
else
HAS_CONDA=True
endif

#################################################################################
# COMMANDS                                                                      #
#################################################################################

## Test Git
git_repo_test:
	echo $(SSH_OR_HTTPS)
	echo $(GIT_PATH)


## Simplify creating a Git Repo from the template
git_init:
	cd $(PROJECT_DIR)
	git init
	git add .
	git add .gitignore
	git commit -m "Initial commit of cookiecutter folder structure"
	git remote add origin $(GIT_PATH)
	git push -u origin master

## Create an Azure Blob Storage Account and Container and Add Role
az_create_storage:
	az storage account create --name $(STORAGE_ACCOUNT) --resource-group $(RESOURCE_GROUP) --sku Standard_LRS
	az storage container create --name $(STORAGE_CONTAINER) --account-name $(STORAGE_ACCOUNT) --fail-on-exist --public-access blob
	az role assignment create --role "Storage Blob Data Contributor" --assignee $(ASSIGNEE) --resource-group DLI

## Sync data from local repo to Azure storage
az_upload_data:
	azcopy sync "$(PROJECT_DIR)/data" "https://$(STORAGE_ACCOUNT).blob.core.windows.net/$(STORAGE_CONTAINER)/data" --put-md5

## Download some toy data to start with
toy_dataset:
	curl https://code.datasciencedojo.com/datasciencedojo/datasets/repository/master/archive.tar.bz2 --output $(PROJECT_DIR)/data/raw/data.tar.bz2

## Echo Project Directory
project_dir:
	echo $(PROJECT_DIR)


## Delete all compiled Python files
clean:
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

## Lint using flake8
lint:
	pylint src

