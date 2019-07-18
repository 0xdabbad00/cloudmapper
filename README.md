CloudMapper
========
[![Build Status](https://travis-ci.org/duo-labs/cloudmapper.svg?branch=master)](https://travis-ci.org/duo-labs/cloudmapper)

CloudMapper helps you analyze your Amazon Web Services (AWS) environments.  The original purpose was to generate network diagrams and display them in your browser.  It now contains much more functionality, including auditing for security issues.

- [Network mapping demo](https://duo-labs.github.io/cloudmapper/)
- [Report demo](https://duo-labs.github.io/cloudmapper/account-data/report.html)
- [Intro post](https://duo.com/blog/introducing-cloudmapper-an-aws-visualization-tool)
- [Post to show usage in spotting misconfigurations](https://duo.com/blog/spotting-misconfigurations-with-cloudmapper)

# Commands

- `api_endpoints`: List the URLs that can be called via API Gateway.
- `audit`: Check for potential misconfigurations.
- `collect`: Collect metadata about an account. More details [here](https://summitroute.com/blog/2018/06/05/cloudmapper_collect/).
- `find_admins`: Look at IAM policies to identify admin users and roles and spot potential IAM issues. More details [here](https://summitroute.com/blog/2018/06/12/cloudmapper_find_admins/).
- `find_unused`: Look for unused resources in the account.  Makes a best effort. Currently finds unused Security Groups, Elastic IPs, network interfaces, and Volumes.
- `prepare`/`webserver`: See [Network Visualizations](docs/network_visualizations.md)
- `public`: Find public hosts and port ranges. More details [here](https://summitroute.com/blog/2018/06/13/cloudmapper_public/).
- `sg_ips`: Get geoip info on CIDRs trusted in Security Groups. More details [here](https://summitroute.com/blog/2018/06/12/cloudmapper_sg_ips/).
- `stats`: Show counts of resources for accounts. More details [here](https://summitroute.com/blog/2018/06/06/cloudmapper_stats/).
- `weboftrust`: Show Web Of Trust. More details [here](https://summitroute.com/blog/2018/06/13/cloudmapper_wot/).
- `report`: Generate HTML report. Includes summary of the accounts and audit findings. More details [here](https://summitroute.com/blog/2019/03/04/cloudmapper_report_generation/).
- `iam_report`: Generate HTML report for the IAM information of an account. More details [here](https://summitroute.com/blog/2019/03/11/cloudmapper_iam_report_command/).


If you want to add your own private commands, you can create a `private_commands` directory and add them there.

# Screenshots

<img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/ideal_layout.png" width=100% alt="Ideal layout">
<table border=0>
<tr><td>
<img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/report_resources.png" alt="Report screenshot">
<td><img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/report_findings_summary.png" alt="Findings summary">
<tr><td>
<img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/report_findings.png" alt="Findings">
<td><img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/iam_report-inactive_and_detail.png" alt="IAM report">
<tr><td>
<img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/command_line_audit.png" alt="Command-line audit">
<td><img src="https://raw.githubusercontent.com/duo-labs/cloudmapper/master/docs/images/command_line_public.png" alt="Command-line public command">
</table>


## Installation

Requirements:
- python 3 (3.7.0rc1 is known to work), `pip`, and `virtualenv`
- You will also need `jq` (https://stedolan.github.io/jq/) and the library `pyjq` (https://github.com/doloopwhile/pyjq), which require some additional tools installed that will be shown.

On macOS:

```
# clone the repo
git clone https://github.com/duo-labs/cloudmapper.git
# Install pre-reqs for pyjq
brew install autoconf automake libtool jq awscli python3 pipenv
cd cloudmapper/
pipenv install --skip-lock
pipenv shell
```

On Linux:
```
# clone the repo
git clone https://github.com/duo-labs/cloudmapper.git
# (AWS Linux, Centos, Fedora, RedHat etc.):
# sudo yum install autoconf automake libtool python3-devel.x86_64 python3-tkinter python-pip jq awscli
# (Debian, Ubuntu etc.):
# You may additionally need "build-essential"
sudo apt-get install autoconf automake libtool python3.7-dev python3-tk jq awscli
cd cloudmapper/
pipenv install --skip-lock
pipenv shell
```


## Run with demo data

A small set of demo data is provided.  This will display the same environment as the demo site https://duo-labs.github.io/cloudmapper/ 

```
python cloudmapper.py prepare --config config.json.demo --account demo
python cloudmapper.py webserver
```

This will run a local webserver at http://127.0.0.1:8000/


# Setup

1. Configure information about your account.
2. Collect information about an AWS account.

## 1. Configure your account

### Option 1: Edit config file manually
Copy the `config.json.demo` to `config.json` and edit it to include your account ID and name (ex. "prod"), along with any external CIDR names. A CIDR is an IP range such as `1.2.3.4/32` which means only the IP `1.2.3.4`.

### Option 2: Generate config file
CloudMapper has commands to configure your account:

```
python cloudmapper.py configure {add-account|remove-account} --config-file CONFIG_FILE --name NAME --id ID [--default DEFAULT]
python cloudmapper.py configure {add-cidr|remove-cidr} --config-file CONFIG_FILE --cidr CIDR --name NAME
```

This will allow you to define the different AWS accounts you use in your environment and the known CIDR IPs.


## 2. Collect data about the account

This step uses the CLI to make `describe` and `list` calls and records the json in the folder specified by the account name under `account-data`.

### AWS Privileges required
You must have AWS credentials configured that can be used by the CLI with read permissions for the different metadata to collect.  I recommend using [aws-vault](https://github.com/99designs/aws-vault).  CloudMapper will collect IAM information, which means you MUST use MFA.  Only the `collect` step requires AWS access.

You must have the following privileges (these grant various read access of metadata):

- `arn:aws:iam::aws:policy/SecurityAudit`
- `arn:aws:iam::aws:policy/job-function/ViewOnlyAccess`


### Collect the data

Collecting the data is done as follows:

```
python cloudmapper.py collect --account my_account
```

### Alternatives
For network diagrams, you may want to try https://github.com/lyft/cartography or https://github.com/anaynayak/aws-security-viz

For auditng and other AWS security tools see https://github.com/toniblyx/my-arsenal-of-aws-security-tools

Licenses
--------
- cytoscape.js: MIT
  https://github.com/cytoscape/cytoscape.js/blob/master/LICENSE
- cytoscape.js-qtip: MIT
  https://github.com/cytoscape/cytoscape.js-qtip/blob/master/LICENSE
- cytoscape.js-grid-guide: MIT
  https://github.com/iVis-at-Bilkent/cytoscape.js-grid-guide
- cytoscape.js-panzoom: MIT
  https://github.com/cytoscape/cytoscape.js-panzoom/blob/master/LICENSE
- jquery: JS Foundation
  https://github.com/jquery/jquery/blob/master/LICENSE.txt
- jquery.qtip: MIT
  https://github.com/qTip2/qTip2/blob/master/LICENSE
- cytoscape-navigator: MIT
  https://github.com/cytoscape/cytoscape.js-navigator/blob/c249bd1551c8948613573b470b30a471def401c5/bower.json#L24
- cytoscape.js-autopan-on-drag: MIT
  https://github.com/iVis-at-Bilkent/cytoscape.js-autopan-on-drag
- font-awesome: MIT
  http://fontawesome.io/
- FileSave.js: MIT
  https://github.com/eligrey/FileSaver.js/blob/master/LICENSE.md
- circular-json: MIT
  https://github.com/WebReflection/circular-json/blob/master/LICENSE.txt
- rstacruz/nprogress: MIT
  https://github.com/rstacruz/nprogress/blob/master/License.md
- mousetrap: Apache
  https://github.com/ccampbell/mousetrap/blob/master/LICENSE
- akkordion MIT
  https://github.com/TrySound/akkordion/blob/master/LICENSE
