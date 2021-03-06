export AWS_ACCESS_KEY_ID ?= test
export AWS_SECRET_ACCESS_KEY ?= test
export SERVICES = s3,codecommit
export AWS_DEFAULT_REGION=us-east-1
export REPO_FOLDER = /tmp/test.codecommit.repo1

usage:       ## Show this help
	@fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$//' | sed -e 's/##//'

install:     ## Install dependencies
	@npm install
	@which localstack || pip install localstack
	@which awslocal || pip install awscli-local
	@test -e .venv || (virtualenv .venv; . .venv/bin/activate; pip install -r requirements.txt)

run:         ## Deploy and run the sample locally
	@echo "-----\nStep 1: Creating new CodeCommit git repository"; \
		awslocal codecommit create-repository --repository-name repo1; \
		gitUrl=$$(awslocal codecommit get-repository --repository-name repo1 | \
			jq -r .repositoryMetadata.cloneUrlSsh); \
		echo "-----\nStep 2: Cloning repo to temporary folder"; \
		test -e $(REPO_FOLDER) || git clone $$gitUrl $(REPO_FOLDER); \
		echo 'test file content 123' >> $(REPO_FOLDER)/test.txt; \
		echo "-----\nStep 3: Committing and pushing new file to the repository"; \
		(cd $(REPO_FOLDER); git add test.txt; git commit -am test_commit; git push); \
		echo "-----\nStep 4: Cloning repo to second temporary folder"; \
		test -e $(REPO_FOLDER).copy || git clone $$gitUrl $(REPO_FOLDER).copy; \
		echo "-----\nStep 5: Printing file content from second clone of repo"; \
		cat $(REPO_FOLDER).copy/test.txt

test-ci:
	make install && make run || true

.PHONY: usage install run
