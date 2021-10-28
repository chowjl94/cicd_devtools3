# cicd_devtools3

#### day3 dev tools 

# Setup node workflow
- Create a new Github repository
- In your repository create the `.github/workflows` folder
- In the newly created folder, create a file named `setup-node.yml` with this content
```yaml
name: setup-node
on: [push]
jobs:
  setup-node:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '14'
      - run: npm -v
      - run: node -v
```
- Commit these changes and push them to your Github repository
- Now visit `https://github.com/<your-github-username>/<your-github-repository-name>/actions` to view the newly created workflow and it's execution details
- There's also a detailed guide on how to view workflows in [Github Action's official documentation](https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions#viewing-the-jobs-activity)

**END**

# Event types
## Manual
- In the same repository, create a file named `manual.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: manual
on:
  workflow_dispatch:
    inputs:
      name:
        description: The name to echo
        default: World
        required: false
jobs:
  manual:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello ${{ github.event.inputs.name }}"
```
- Commit and push the changes
- Visit the Github Actions page and click on the newly created workflow
- Look for the "Run workflow" button to run the workflow, you should then be asked for an input
- View the workflow logs and see that `Hello world` was printed
- Run the workflow again, this time supply a different value to the input. You should see it being picked up in the job logs
- Feel free to checkout the [official documentation](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#manual-events) to see all available configuration options

## Scheduled
- In the same repository, create a file named `scheduled.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: scheduled
on:
  schedule:
    - cron: '*/5 * * * *'
jobs:
  scheduled:
    runs-on: ubuntu-latest
    steps:
      - run: echo This job runs every 5 minutes
```
- Commit and push the changes
- Visit the Github Actions page, click on the newly created workflow and view the logs
- Feel free to update the schedule, you can use [crontab.guru](https://crontab.guru/) to know more about the cron scheduling syntax and try them out
- If you're done playing around with the scheduled workflow. It is strongly recommended to [disable the workflow](https://docs.github.com/en/actions/managing-workflow-runs/disabling-and-enabling-a-workflow) so we don't unnecessary consume resources in Github's hosted runners.

## Webhooks
- In the same repository, create a file named `on-push.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: on-push
on:
  push:
    branches:
      - "main"
      - "foo"
    tags:        
      - v1
      - v1.*
jobs:
  on-push:
    runs-on: ubuntu-latest
    steps:
      - run: echo This job runs on push to main and foo branch and on tag v1, v1.0, v1.1 to v1.9
```
- Commit and push the changes
- You should see that the newly created workflow got immediately triggered
- Feel free to checkout the [official documentation](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#webhook-events) to see all available configuration options


**END**

# Variables and Secrets
- Follow the instructions [here](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository) to create a repository secret. Name the secret as `EXAMPLE_API_KEY`
- In the same repository, create a file named `variables-secrets.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: variables-secrets
on:
  push:
    branches:
      - main
env:
  baz_variable: BAZ

jobs:
  variables-secrets-1:
    runs-on: ubuntu-latest
    env:
      bar_variable: BAR
    steps:
      - name: Retrieve secret via an env var
        env:
          example_api_key: ${{ secrets.EXAMPLE_API_KEY }}
          foo_variable: FOO
        run: echo "$example_api_key $foo_variable $baz_variable"
      - name: Directly retrieve a secret
        run: echo "${{ secrets.EXAMPLE_API_KEY }} $bar_variable"
  variables-secrets-2:
    runs-on: ubuntu-latest
    steps:
      - name: Retrieve global env var
        run: echo "$baz_variable $bar_variable $foo_variable"
```
- Commit and push the changes
- View the workflow and the execution logs of `variable-secrets-1` job
- You should see that the secret is masked when printed
```
Run echo "$example_api_key $foo_variable $baz_variable"
*** FOO BAZ
```
- You should also see that it also works when accessing the secret directly on the other step
```
Run echo "*** $bar_variable"
*** BAR
```
- Now view the execution logs of `variable-secrets-2` job
- You should see that the only `BAZ` was printed

**END**

# Using Docker containers
- In the same repository, create a file named `docker-containers.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: docker-containers
on:
  push:
    branches:
      - "main"
jobs:
  setup-node:
    runs-on: ubuntu-latest
    container:
      image: node:16
      # credentials:
      #   username: ${{ github.actor }}
      #   password: ${{ secrets.ghcr_token }}
      env:
        NODE_ENV: development
      ports:
        - 80
      volumes:
        - my_docker_volume:/tmp
      options: --cpus 0.2 --network-alias maincontainer
    steps:
      - uses: actions/checkout@v2
      - run: npm -v
      - run: node -v
```
- Commit and push the changes
- View the workflow and the execution logs
- View the logs for `Initialize container`
- You should see the command used to create the container, along with the config options we specified
```
/docker create --name d56d88d798344bc0af269fcb002de9c7_node16_fe31d6 --label 8a33c1 --workdir /__w/github-actions-examples/github-actions-examples --network github_network_dacc60e44f474c36849c7d8a46b292b3 -p 80 --cpus 0.2 --network-alias maincontainer -e "NODE_ENV=development" ...
```
**END**
# Using Service containers
- In the same repository, create a file named `service-containers.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: service-containers
on:
  push:
    branches:
      - "main"
jobs:
  container-job:
    runs-on: ubuntu-latest
    container:
      image: ubuntu
    env:
      DATABASE_NAME: my_database
      DATABASE_USER: foo_user
      DATABASE_PASSWORD: password
    services:
      postgres:
        image: postgres
        env:
          POSTGRES_DB: ${{ env.DATABASE_NAME }}
          POSTGRES_USER: ${{ env.DATABASE_USER }}
          POSTGRES_PASSWORD: ${{ env.DATABASE_PASSWORD }}
    steps:
      - name: Install psql client
        run: apt-get -y update && apt-get -y install postgresql-client
      - name: Connect to PostgreSQL
        env: 
          PGPASSWORD: ${{ env.DATABASE_PASSWORD }}
        run: psql -U ${{ env.DATABASE_USER }} -d ${{ env.DATABASE_NAME }} -h postgres -c 'select * from information_schema.tables;'
```
- Commit and push the changes
- View the workflow and exection logs
- Observe the `Initilize containers` step
- There should be another command being run for the service container, `Starting postgres service container`
- View the `Connect to PostgreSQL` step
- You should see that it was able to connect to the database and run the query successfully

**END**
# Conditionals - Jobs
- In the same repository, create a file named `conditionals-jobs.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: conditionals
on: [push]
jobs:
 regular_job:
   if: ${{ github.ref != 'refs/heads/main' }}
   runs-on: ubuntu-latest
   steps:
     - run: echo "Regular job"   
 special_job:
   if: contains( github.ref, 'main')
   runs-on: ubuntu-latest
   steps:
     - run: echo "Special job"
```
- Commit and push the changes
- View the workflow and the execution logs
- You should see that the `special_job` got triggered and the `regular_job` got skipped
- Now create a new branch with any name and push the changes
- View the workflow and the execution logs
- You should see that the `regular_job` got triggered and the `special_job` got skipped

# Conditionals - Steps
- In the same repository, create a file named `conditionals-steps.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: conditionals
on: [push]
jobs:
  regular_job:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Regular step"
      # - name: Force fail
      #   run: exit 1
      - name: Sucess notification
        if: ${{ success() }}
        run: echo "Send a success notification to Slack"
      - name: Failure notfication
        if: ${{ failure() }}
        run: echo "Send a failure notification to Slack"
```
- Commit and push the changes
- View the workflow and the execution logs
- You should see that the job succeeded and that the `Failure notification` step got skipped
- Now, uncomment the step that forces the job to fail. Commit and push the changes
- View the workflow and the execution logs
- You should see that the job failed and that the `Success notification` step got skipped
**END**

# Dependent jobs
- In the same repository, create a file named `dependent-jobs.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: dependent-jobs
on:
  push:
    branches:
      - main
jobs:
  build_a:
    runs-on: ubuntu-latest
    steps:
      - run: echo "This job will be run first."

  build_b:
    runs-on: ubuntu-latest
    steps:
      - run: echo "This job will be run first, in parallel with build_a"
    # - run: exit 1

  test_ab:
    runs-on: ubuntu-latest
    needs: [build_a,build_b]
    steps:
      - run: echo "This job will run after build_a and build_b have finished"

  independent_test:
    if: always()
    runs-on: ubuntu-latest
    needs: [build_a,build_b]
    steps:
      - run: echo "This job will run regardless of the status of build_a and build_b"

  deploy_ab:
    runs-on: ubuntu-latest
    needs: [test_ab]
    steps:
      - run: echo "This job will run after test_ab is complete"
```
- Commit and push the changes
- View the workflow run, you should see that there is a dependency graph for the jobs
- Now, edit the code and uncomment the `# - run: exit 1` line
- Commit and push the changes
- View the workflow run
- On the dependency graph you should see that since the one of the first jobs failed, the succeeding jobs got skipped 
**END**
# Caching
- In the same repository, create a folder named `cache-files`
- In the newly created folder, create two files named `foo.txt` and `bar.txt`
- Add any text or words you want in each newly created files
- create a file named `caching.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: caching
on:
  push:
    branches:
      - main
jobs:
  caching:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Cache files
      id: cache-step
      uses: actions/cache@v2
      env:
        cache-name: cache-files
      with:
        path: ./cache-files
        key: cache-files-${{ hashFiles('./cache-files/*') }}
        restore-keys: cache-files-
    - run: echo "Cache was not hit "
      if: ${{ steps.cache-step.outputs.cache-hit != 'true' }}
    - run: echo "Cache was hit"
      if: ${{ steps.cache-step.outputs.cache-hit == 'true' }}
```
- Commit and push the changes
- View the workflow and the execution logs
- On the `Cache files` step logs you should see
```
Cache not found for input keys
```
- You should also see that 
  - `Run echo "Cache was not hit "` was ran
  - `Run echo "Cache was hit"` was skipped
- Now, on the upper right portion of the page. There should be a "Re-run jobs" button/dropdown
- Click the button and choose "Re-run all jobs"
- Observe the logs on the new execution
- You should also see that this time
  - `Run echo "Cache was not hit "` was skipped
  - `Run echo "Cache was hit"` was ran
  - `Cache files` step logs
```
Cache restored successfully
Cache restored from key: cache-files-xxxxxxx
```
- Re-running it again and again should still use the cache
- Now, go back to your code and update the contents of `foo.txt` and `bar.txt`
- You can update it to anything of your choosing, as long as you made changes
- Commit and push the changes
- View the workflow and the new execution logs
- You should again see that
  - `Run echo "Cache was not hit "` was ran
  - `Run echo "Cache was hit"` was skipped
- But the `Cache files` step still logs
- See more info about the cache action [here](https://github.com/actions/cache)
```
Cache restored successfully
Cache restored from key: cache-files-xxxxxxx
```

**END**
# Artifacts
- In the same repository, create a file named `artifacts.yml` in the `.github/workflows` folder
- Add the code below to the newly created file
```yaml
name: artifacts
on: [push]
jobs:
  addition:
    name: Add 3 and 7
    runs-on: ubuntu-latest
    steps:
      - shell: bash
        run: expr 3 + 7 > math-homework.txt
      - name: Upload math result for addition job
        uses: actions/upload-artifact@v2
        with:
          name: homework
          path: math-homework.txt
  multiplication:
    name: Multiply by 9
    needs: addition
    runs-on: windows-latest
    steps:
      - name: Download math result from addition job
        uses: actions/download-artifact@v2
        with:
          name: homework
      - shell: bash
        run: |
          value=`cat math-homework.txt`
          expr $value \* 9 > math-homework.txt
      - name: Upload math result for multiplication job
        uses: actions/upload-artifact@v2
        with:
          name: homework
          path: math-homework.txt
  display:
    name: Display results
    needs: multiplication
    runs-on: macos-latest
    steps:
      - name: Download math result from multiplication job
        uses: actions/download-artifact@v2
        with:
          name: homework
      - name: Print the final result
        shell: bash
        run: |
          value=`cat math-homework.txt`
          echo The result is $value
```
- Commit and push the changes
- View the workflow and the execution logs
- You should see how the file is being passed across jobs
- Now, go back to the Summary page of the workflow run
- You should see an Artifacts section, this is how you can download or delete the artifacts

**END**
