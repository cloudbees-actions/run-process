# Run an Application Process on a remote CloudBees CD/RO instance
## Description
This is one of several GitHub Actions provided on an "as-is" basis by CloudBees that enable users to write GitHub Action Workflows that send work to an external CloudBees CD/RO instance. This Action enables workflows to run an existing Application Process on a remote CloudBees CD/RO instance.
## Intended audience
For teams utilizing GitHub Actions for build and continuous integration, the CloudBees CD/RO Actions provide a mechanism for releasing software in a secure, governed, and auditable manner with deep visibility, as is required in regulated industries, for example. Platform or shared services teams can build reusable content in the CloudBees CD/RO platform that conforms to company standards and removes the burden of release automation from the application teams.
## Prerequisites
CloudBees CD/RO is an enterprise "on-premise" product that automates software delivery processes, including production deployments and releases. To use utilize this GitHub Action, it is necessary to have access to a CloudBees CD/RO instance, in particular, 
- A CloudBees CD/RO instance that GitHub Actions can access through REST calls (TCP port 443)
- A valid API token for the CloudBees CD/RO instance. A token can be generated from the _Access Token_ link on the user profile page of the CloudBees CD/RO user interface; see [Manage access tokens via the UI](https://docs.beescloud.com/docs/cloudbees-cd/latest/intro/sign-in-cd#_manage_access_tokens_via_the_ui) documentation for details.
These values should be stored as GitHub Action secrets to be referenced securely in a GitHub Actions workflow.
## Usage
The CloudBees CD/RO GitHub Actions are called from _steps_ in a GitHub Actions _workflow_. The following workflow extract illustrates how to run an existing Application Process on a remote CloudBees CD/RO instance, including _actual parameters_.
```yaml
steps:
  - name: Run Process
    uses: cloudbees-actions/run-process@main
    env:
      CDRO_URL: ${{ secrets.CDRO_URL }}
      CDRO_TOKEN: ${{ secrets.CDRO_TOKEN }}
    with:
      projectName: GHA Project
      applicationName: GHA Application
      processName: GHA Process
      environmentName: QA
      actualParameter: |
        Input1: xyz
        Input2: abc
        Input3: "line1\nline2"
```
### Inputs
| Name                   | Description                                                            | Required |
|------------------------|------------------------------------------------------------------------|----------|
| projectName            | Project name of the application                                        | yes      |
| applicationName        | Application name                                                       | yes      |
| processName            | Process name                                                           | yes      |
| environmentName        | Environment name                                                       | yes      |
| environmentProjectName | Project name of the environment                                        | yes      |
| ignore-unverified-cert | Ignore unverified SSL certificate                                      | no       |
### Secrets and Variables
The following GitHub secrets are needed to run the Action. These can be set in the _Secrets and variable_ section of the workflow repository _Settings_ tab.
| Name                   | Description                                                            | Required |
|------------------------|------------------------------------------------------------------------|----------|
| CDRO_URL               | CloudBees CD/RO server URL, e.g., `https://my-cdro.net` or `https://74.125.134.147` | yes |
| CDRO_TOKEN             | CloudBees CD/RO API Access token                                       | yes      |
## Examples
### Create and run an Application Process
1. Set up secrets in the repository settings for Actions. In the GitHub repository, select the _Settings_ tab, _Secrets_, _Variables_, and _Actions_. Use the _New Repository_ button to create the CDRO_URL and CDRO_TOKEN secrets.
2. Create a DSL file in the root directory of your repository, for example, `simple-process-dsl.groovy`:
```groovy
project "Default",{
	environment "QA",{
		environmentTier "AppServer",{
		  resource "AppServer_QA",
			hostName: getResource(resourceName: "local").hostName,
			hostType: "CONCURRENT"
		}
	}
	application "GHA Application",{
		applicationTier "App"
		process "GHA Process",{
			formalParameter "Input1"
			formalParameter "Input2"
			formalParameter "Input3", type: "textarea"
			processStep "echo",
			applicationTierName: "App",
			actualParameter: [ commandToRun: 'echo'],
			subprocedure: 'RunCommand',
			subproject: '/plugins/EC-Core/project'
		}
		tierMap 'QA', {
			applicationName = applicationName
			environmentName = tierMapName
			environmentProjectName = 'GHA Test'
			tierMapping 'App-AppServer', {
				applicationTierName = 'App'
				environmentTierName = 'AppServer'
			}
		}
	}
}
```
3. Create a new workflow file in the `.github/workflows` directory, for example, `simple-process.yml`:
```yaml
name: Create and run process

on:
  workflow_dispatch:

jobs:
  create-and-run-process:
    runs-on: ubuntu-latest
    env:
      CDRO_URL: ${{ secrets.CDRO_URL }}
      CDRO_TOKEN: ${{ secrets.CDRO_TOKEN }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Create process
        uses: cloudbees-actions/eval-dsl@main
        with:
          dsl-file: simple-process-dsl.groovy

      - name: Run process
        uses: cloudbees-actions/run-process@main
        with:
          projectName: Default
          applicationName: GHA Application
          processName: GHA Process
          environmentName: QA
          actualParameter: |
            Input1: xyz
            Input2: abc
            Input3: "line1\nline2"
```
4. Go to the GitHub `Actions` tab and run the workflow `Create and run process`
## License
The scripts and documentation in this project are released under the MIT License.
## Documentation
For more details about the CloudBees CD/RO product, view the [online documentation](https://docs.cloudbees.com/docs/cloudbees-cd/latest/).

