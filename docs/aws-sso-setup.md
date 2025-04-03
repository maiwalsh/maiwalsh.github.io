---
tags:
- wiki
- recro
- work
- aws
- sso
---

# AWS SSO Setup (User/Client)

Set up your `aws` CLI to begin.

## *nix

```ad-warning
title: Assumes no valid credentials set, file or ENV!

	> aws s3 ls

	Unable to locate credentials. You can configure credentials by running "aws configure".

```

Open your command line and start the interactive configuration wizard with:

```bash
aws configure sso
```

There are four pieces of information you'll be prompted to provide:

- **SSO Session Name** is the reference string that will go into your `~/.aws/config` file. `recro` is fine if it's your only session to maintain. If not, a more clear meaninful name like "company-project" or "username-role" will help keep things organized.

- **SSO Start URL** is the URL that our endpoint serves our identity from, where your CLI will hit and authenticate.  Ours is `https://recrocog.awsapps.com/start`

- **SSO Region** is `us-east-1`

- **SSO Session Scope** keep default as `sso:account:access`

```ad-danger
title: Browser Window Pop-up!

To authenticate, the CLI will attempt to open your system's browser window. If this fails for whatever reason, it will ALSO spit out the URL for you to manually enter yourself. You must follow this URL to the webpage to proceed with the login.
```

Sign in with your known username, and then verify with the email address.

```ad-summary
title: Example for cwilson with adminAccess



	╰─❯ aws configure sso
	
	SSO session name (Recommended): recro
	SSO start URL [None]: https://recrocog.awsapps.com/start
	SSO region [None]: us-east-1
	SSO registration scopes [sso:account:access]: sso:account:access
	Attempting to automatically open the SSO authorization page in your default browser.
	If the browser does not open or you wish to use a different device to authorize this request, open the following URL:
	
		https://oidc.us-east-1.amazonaws.com/authorize?response_type=code&client_id=<----snipped-for-length---->
		
	The only AWS account available to you is: XXXXXXXXXXXX
	Using the account ID XXXXXXXXXXXXXXXX
	The only role available to you is: adminAccess
	Using the role name "adminAccess"
	Default client Region [None]: us-east-1
	CLI default output format (json if not specified) [None]: json
	Profile name [adminAccess-XXXXXXXXXXXXXXXX]: recrocog-admin
	To use this profile, specify the profile name using --profile, as shown:
	
	aws sts get-caller-identity --profile recrocog-admin

```


## Verify

Run the command output from above:

```bash

╰─❯ aws sts get-caller-identity --profile recrocog-admin

{
    "UserId": "XXXXXXXXXXXXXXXX:cwilson",
    "Account": "XXXXXXXXXXXX",
    "Arn": "arn:aws:sts::XXXXXXXXXXXX:assumed-role/AWSReservedSSO_adminAccess_xXxXxXxXxXxX/cwilson"
}
```

## Set Environment Variable for Convenience

The file `~/.aws/config` is where the profile and SSO session information are stored to do things like log you back in after token expiry with `aws sso login`. The name of the profile is how the CLI will select it from that file - setting the default profile name to look for is how you can change this default behavior.

```bash
╰─❯ cat config 
	[profile recrocog-admin]
	sso_session = recro
	sso_account_id = XXXXXXXXXXXXXXXX
	sso_role_name = adminAccess
	region = us-east-1
	output = json
	[sso-session recro]
	sso_start_url = https://recrocog.awsapps.com/start
	sso_region = us-east-1
	sso_registration_scopes = sso:account:access
```

The CLI will automatically use the profile you set in the environment variable `AWS_DEFAULT_PROFILE`:

```bash
╰─❯ aws s3 ls

	Unable to locate credentials. You can configure credentials by running "aws configure".

╰─❯ export AWS_DEFAULT_PROFILE=recrocog-admin

╰─❯ aws s3 ls

	2024-09-26 15:11:02 linux-airgap-oracle
	2024-12-02 07:39:39 backup-tarballs
	2025-02-03 11:15:59 another-bucket
	2025-04-01 12:19:46 terraform-state-bucket

```