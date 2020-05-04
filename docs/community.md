# Community
Welcome to the Secure Device Onboard community!

## Join the Discussion

Report issues using each component's respective github repository.

## Contribute

Secure Device Onboard is made available under the Apache License 2.0.

The project accepts contributions via GitHub pull requests.

If you are contemplating a large change or major contribution to the Secure Device Onboard project, please first submit an issue outlining your proposed changes or contribution. This will allow the community to provide feedback on the proposal and ensure alignment with project goals and processes.

### The Commit Process

 When contributing code, please follow these guidelines:

- Your contribution must be submitted under the Apache License 2.0. Include the Apache License header or SPDX header in every source file. See the following for specific guidance: 
    - <https://www.apache.org/licenses/LICENSE-2.0#apply>
    - <https://spdx.org/licenses/Apache-2.0.html>

- Any 3rd-party dependencies required by your code to execute must be available under the Apache 2.0 license. Exceptions should be discussed in advance by submitting an issue for review.

- Fork the repository and make your changes in a feature branch

- Include unit and integration tests for any new features, as well as updates to existing tests

- Ensure that existing unit and integration tests run successfully. Instructions on how to run unit tests can be found in the readme in each component's respective repository.

- Ensure that coding style checks pass. Instructions on how to run coding style checks can be found in the readme in each components respective repository.

### Pull Request Guidelines

A pull request can contain a single commit or multiple commits. The most important guideline is that a single commit should map to a single fix or enhancement. Here are some example scenarios:

- If a pull request adds a feature but also fixes two bugs, the pull request should have three commits: one commit for the feature change and two commits for the bug fixes.

- If a pull request is opened with five commits that contain changes to fix a single issue, the pull request should be rebased to a single commit.

- If a pull request is opened with several commits, where the first commit fixes one issue and the rest fix a separate issue, the pull request should be rebased to two commits (one for each issue).


!!! Note
    Your pull request should be rebased against the current master branch. Do not merge the current master branch in with your topic branch. Do not use the Update Branch button provided by GitHub on the pull request page.

### Commit Messages

Commit messages should follow common Git conventions, such as using the imperative mood, separate subject lines, and a line length of 72 characters. These guidelines and more are documented here: <https://chris.beams.io/posts/git-commit/#seven-rules>

### Signed-off-by

Each commit must include a “Signed-off-by” line in the commit message (git commit -s). This sign-off indicates that you agree the commit satisfies the Developer Certificate of Origin ([DCO](https://developercertificate.org/)).

### Commit Email Address

Your commit email address must match your GitHub email address. For more information, see <https://help.github.com/articles/setting-your-commit-email-address-in-git/>

### Important GitHub Requirements

A pull request cannot merged until it has passed these status checks:

- The build must pass on Jenkins
- The pull request must be approved by at least two reviewers without any outstanding requests for changes

## Reporting Issues

Report issues using each component's respective github repository.

## Code of Conduct

When participating, please be respectful and courteous.

The Secure Device Onboard project follows the LF Edge Code of Conduct: <https://lfprojects.org/policies/code-of-conduct/>

## Acknowledgements
This project uses software developed by the OpenSSL* Project for use in the OpenSSL\* Toolkit (<http://www.openssl.org/>).

This project relies on other third-party components. For details, see the NOTICE file and/or folder in each components respective repository.

