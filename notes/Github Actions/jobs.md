## Jobs:

- A **job** is a set of **steps** in a workflow that is executed on the same **runner**. Each step is either a shell script that will be executed, or an **action** that will be run.
    - **For example**, you can have a step that builds your application followed by a step that tests the application that was built.
        - BUILD——>TEST——→DEPLOY
- You can configure a **job's dependencies** with other jobs; by default, jobs have no dependencies and **run in parallel**. When a job takes a dependency on another job, it waits for the dependent job to complete before running.
    - **For example**, you might configure multiple build jobs for different architectures without any job dependencies and a packaging job that depends on those builds. The build jobs run in parallel, and once they complete successfully, the packaging job runs.
- Parallel and Sequential Jobs
    - **Parallel** Execution
    
    ```yaml
    on: workflow_dispatch
    jobs:
    	Build:
    		name: Build
    		runs-on: ubuntu-latest
    		steps:
    			- name: Build
    				run: |
    					echo "Build Step"
    					exit 1
    	Test:
    		name: Test
    		runs-on: ubuntu-latest
    		steps:
    			- name: Test
    				run: |
    					echo "Test Step"
    					exit 1
    	
    ```
    
    - **Sequential** Execution
    
    ```yaml
    on: workflow_dispatch
    jobs:
    	Build:
    		name: Build
    		runs-on: ubuntu-latest
    		steps:
    			- name: Build
    				run: |
    					echo "Build Step"
    					exit 1
    	Test:
    		name: Test
    		runs-on: ubuntu-latest
    		needs:
    				- Build
    		steps:
    			- name: Test
    				run: |
    					echo "Test Step"
    					exit 1
    	
    ```
    
    - **Always** execute the job
    
    ```yaml
    on: workflow_dispatch
    jobs:
    	Build:
    		name: Build
    		runs-on: ubuntu-latest
    		steps:
    			- name: Build
    				run: |
    					echo "Build Step"
    					exit 1
    	Test:
    		name: Test
    		runs-on: ubuntu-latest
    		if: ${{ always() }}
    		needs:
    				- Build
    		steps:
    			- name: Test
    				run: |
    					echo "Test Step"
    					exit 1
    	
    ```
    
    - Using the **Strategy**
    
    ```yaml
    name: Parallel Tests Workflow
    
    on:
      push:
        branches:
          - main
    
    jobs:
      parallel-tests:
        runs-on: ubuntu-latest
        **strategy:
    	    # Number of parallel jobs to be run
    	    max-parallel: 1
          matrix:
            test-type: [unit, integration, e2e]**
    
        steps:
          - name: Checkout code
            uses: actions/checkout@v3
    
          - name: Set up Node.js
            uses: actions/setup-node@v3
            with:
              node-version: 16
    
          - name: Install dependencies
            run: npm install
    
    				# This will execute all 'unit', 'integration' and 'e2e' test parallely
          **- name: Run ${{ matrix.test-type }} tests
            run: npm run test:${{ matrix.test-type }}**
    
    ```